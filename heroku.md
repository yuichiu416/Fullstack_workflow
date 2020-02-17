## Add heroku as a remote
`heroku git:remote -a appname`

or 

`git remote add heroku https://git.heroku.com/appname.git`

## Delete a branch on heroku
`git push --force origin <branch_name>` or `git push origin :branch_name`

 Some useful fixes by Carlos Catly
 
 1. no method error for component
    fix: `install case-sensitive-paths-webpack-plugin`
 2. some jquery error
     fix: remove all jquery files/requires from the project.
     
     OR
     
     You might have some jquery calls somewhere so if you want to keep it make sure:
     `//= require jquery`
     `//= require jquery_ujs`
     is in app/assets/javascripts/application.js
     and gem 'jquery-rails' in your Gemfile and IS NOT under any group.
     
 3. uglifer error about harmony
     - go to production.rb
     - modify this entry to be this config.assets.js_compressor = Uglifier.new(harmony: true)
 4. no method error in the logs and the website isn't not displaying anything.
     - If you set AWS before this you have to give heroku your masterkey
     - read more [here](https://open.appacademy.io/learn/swe-online/full-stack-project/active-storage-and-aws-s3-hosting-demo)
     - Also make sure the scripts in your package.json only has postinstall webpack on it.

## Push to heroku with docker
1. Set up config folder and keys
   ```js
   // config/keys_dev.js
   module.exports = {
    MONGO_URI: 'some uri here',
    secretOrKey: "some secret key here"
    }
    ```

    ```js
    // config/keys_prod.js
    module.exports = {
        mongoURI: process.env.MONGO_URI,
        secretOrKey: process.env.SECRET_OR_KEY
    }
    ```
    
    ```js
    // config/keys.js
    if (process.env.NODE_ENV === 'production') {
        module.exports = require('./keys_prod');
    } else {
        module.exports = require('./keys_dev');
    }
    ```
2. `heroku container:login` (if received errors, try `heroku plugins:install heroku-container-registry` or `heroku plugins:install @heroku-cli/plugin-container-registry`)
3. `heroku git:remote -a <app_name>`
4.  If you have one Dockerfile you can use `heroku container:push <process-type>` (web, frontend, etc.).
   If you have multiple services (Rails and React for example) you'll need two Dockerfiles. The naming format would as follows:
    ```Docker
    ls -R

    # Main Directory for the Rails Application (web is the default name for the service receiving HTTP requests)
    Dockerfile.web

    # /frontend - give your Dockerfile endings meaningful names
    Dockerfile.frontend
    ```
    Then to build all of your images for Heroku you would run `heroku container:push --recursive`
6. `heroku container:push --recursive -a <app_name>`