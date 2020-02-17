
### Think about the project layout, include:
1. Frontend routes
   ex. 
   * Root
     * App
       * NavBar
       * (main component goes here)
       * Footer

   The following routes, defined in `App`, will render components between `NavBar` and `Footer`
   * `/`
     * RootNavBar - *Home icon*, *Search bar*, *Free Stock*, *Home*, *Notifications*, *Account*
     * PageContainer
       * ProfileOverviewContainer
         * ProfileOverviewComponent - shows *total value*, *change*, display in *1D*, *1W*, *1M*, *3M*, *1Y*, *ALL*
       * WatchlistContainer
         * WatchlistComponent - watchlist list
           * WatchlistItem - shows *Name*, *Price*, *Shares* (if applicable) and *Today*

   * `/login`
     * PageContainer
       * SessionForm

   * `/signup`
     * PageContainer
       * SessionForm

   * `/stocks/:stockName`
     * PageContainer
       * OwnedStockSummaryComponent if owns any shares
       * AboutComponent - *Your Equity* (total value), *Cost*, *Today's Return*, *Total Return*. *Your Average Cost*, *Shares*, *Portfolio Diversity*, *Cash Held for Exercise*.
       
       * TransactionContainer
         * TransactionComponent - floating on the right side of the screen, *buy/sell stock*, *Shares number*, *Market Price*, *Estimated Cost*, *Available Buying Power*


2. Backend routes
    ex.
    ### HTML
    * `Get /` - redirect to `StaticPagesController#root`

    ### API Endpoints
    #### `session`
    * `POST api/session` - log in
    * `DELETE api/session` - log out

    #### `users`
    * `POST users` - sign up

    #### `deposits`
    * `POST /deposits` - Make a deposit or withdraw
    * `GET /deposits/:user_id` - display all deposit infomation of a certain user

    #### `transactions`
    * `POST /transactions` - Make a transaction
    * `GET /transactions/:user_id?ticker=:ticker` - display all transaction information of a certain user buying/selling a certain stock

3. Sample state
    ex. 
    ```
    {
        entities: {
            stocks: {
                1: {
                    id: 1,
                    name: "AMD",
                    owned_shares: 33,
                    current_price: 30.46,
                    change: 2.51
                },
            },
            user: {
                id: 11,
                username: "roger",    
                imgUrl: "https://cdn.pixabay.com/photo/2015/10/01/16/43/toucan-967334_960_720.jpg"
            },
            watchlist: {
                id: 1,
                user_id: 1,
                stock_ids: [2, 3, 42, 89],
            },
        },
        ui: {
            loading: true/false
        },
        errors: {
            login: ["Incorrect username/password combination"],
            buying: ["Amount can't be 0"],
        },
        session: { currentUserId: 25 }
    }
    ```
4. Schema
  ex.
    ## `users`
    column name | data type | details
    --- | --- | ---
    `id` | integer | not null, primary key
    username | string | not null, indexed, unique
    email | string | not null, indexed, unique
    password_digest | string | not null
    session_token | string | not null, indexed, unique
    created_at | datetime | not null
    updated_at | datetime | not null

    * index on `username`, unique: true
    * index on `email`, unique: true
    * index on `session_token`, unique: true

### Start a new project
`rails new <project> --database=postgresql --skip-turbolinks`

### Update your Gemfile.

`gem 'better_errors'`

`gem 'binding_of_caller'`

`gem 'pry-rails'`

`gem 'annotate'`

`gem 'bcrypt'`

`gem 'jquery-rails'`

~~*`gem 'wdm', '>= 0.1.0'`*~~  **this line is for Windows user only**, make sure you removed it before pushing to heroku

Then, as usual, `bundle install`

### When using Rails 5.1+, update your `app/assets/javascripts/application.js` manifest to include:
`//= require rails-ujs`

`//= require jquery`

### Initialize the folder and connect to github
1. `git init`

2. Add these three babies your .gitignore
    ```
    node_modules/
    bundle.js
    bundle.js.map
    package-lock.json
    ```

3. Push
`git remote add origin`

`git pull origin master --rebase`

`git branch --set-upstream-to=origin/master master`

### Create a package.json file with the default setup.
`npm init --yes`

### Create a frontend folder at the root of your project with an entry file inside of it.
```
actions
  * actions.js
components
  * app.jsx
  * root.jsx
reducers
  * root_reducer.js
  * another_reducer.js //depends on the needs
store
  * store.js
util
  * util.js

//potentially
thunk
   * thunk.js
```

### Install the package needed
`npm install --save webpack webpack-cli react react-dom react-router-dom redux react-redux @babel/core @babel/preset-react @babel/preset-env babel-loader`

###  Create a webpack.config.js file.

ex:
```
const path = require('path');

module.exports = {
  context: __dirname,
  entry: "./frontend/entry.jsx",
  output: {
    path: "app/assets/javascripts",
    filename: "bundle.js"
  },
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /(node_modules)/,
        use: {
          loader: 'babel-loader',
          query: {
            presets: ['@babel/env', '@babel/react']
          }
        },
      }
    ]
  },
  devtool: 'source-map',
  resolve: {
    extensions: [".js", ".jsx", "*"]
  }
};
```

### Create the entry.jsx file
```jsx
// frontend/entry.jsx
import React from 'react';
import ReactDOM from 'react-dom';
import createStore from './store/store';
import Root from './components/root';

document.addEventListener('DOMContentLoaded', () => {
    let store;
    if (window.currentUser) {
        const preloadedState = {
            session: { currentUser: window.currentUser },
        };
        store = createStore(preloadedState);
        delete window.currentUser;
    } else {
        store = createStore();
    }
    const root = document.getElementById('root');
    ReactDOM.render(<Root store={store} />, root);
});
```

**NB:The entry point should be in frontend, e.g. entry: 'frontend/entry.jsx'.
The output path should be 'app/assets/javascripts'.
Configure your module.rules to use Babel transpilation for:
JSX or ES6**

### Set up `config/routes.rb`
ex. 
```rb
Rails.application.routes.draw do
  root to: 'root#index'
  namespace :api, defaults: {format: :json} do
    resources :users
    resource :session, only: [:create, :destroy]
  end

end
```
### According to the schema
1.  Set up the controller according to the routes
  ex. `rails g controller api/users create destroy`
  (*Don't forget the root controller*)

2. Set up the tables and the models
  ex. `rails g model users username:string:index`

Don't forget to run `rails db:setup` and `rails db:migrate`
Create a root html file under `app/views/` if you don't have one yet. Make sure it has a `<div>` with `id="root"`