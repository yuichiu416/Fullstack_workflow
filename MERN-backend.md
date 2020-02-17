More code example refers to [mongoDB](./mongoDB.md)

## Set up routes

`app.use('/', files);`
This means `http://localhost:5000/` will be the root routes

Inside `routes/files.js`, if there is a code like this:
```js
router.get('/fetchAll/:email', (req, res) => {
    const email = req.params.email;
    File.find({ email }).then(files => (res.json(files)));
});
```
This means it will take all GET requests sent to `http://localhost:5000/fetchAll/:email`.
Same for `router.post`, `router.delete`, etc.

The `$.ajax()` is gone, but we use `axios` instead, the syntax is pretty much the same.

The server side's response is determined by the `res.json()`

Everything else should be really similar to fullstack project 1


Put ththe scripts into `package.json`:
```json
  "scripts": {
    "start": "node app.js",
    "server": "nodemon app.js",
    "server:debug": "nodemon --inspect app.js",
    "frontend-install": "npm install --prefix frontend",
    "frontend": "npm start --prefix frontend",
    "dev": "concurrently \"npm run server\" \"npm run frontend\"",
    "dev:debug": "concurrently \"npm run server:debug\" \"npm run frontend\""
  },
```
**NB:**

`npm run frontend` only runs the 3000 frontned server

`npm run server` only runs the 5000 backend server

`npm run dev:debug` runs both server and enables Chrome debugging

`npm run dev` can be used to run the server as normal

# Upload files to mongoDB
[Original article from here](https://medium.com/ecmastack/uploading-files-with-react-js-and-node-js-e7e6b707f4ef)
[Github repo](https://github.com/abachuk/uploading-files-react-node)

## Client
Here is a simple input field with an event handler:
```js
// Redux action
export function uploadSuccess({ data }) {
  return {
    type: 'UPLOAD_DOCUMENT_SUCCESS',
    data,
  };
}

export function uploadFail(error) {
  return {
    type: 'UPLOAD_DOCUMENT_FAIL',
    error,
  };
}

export function uploadDocumentRequest({ file, name }) {  
  let data = new FormData();
  data.append('file', document);
  data.append('name', name);

  return (dispatch) => {
    axios.post('/files', data)
      .then(response => dispatch(uploadSuccess(response))
      .catch(error => dispatch(uploadFail(error));
  };
}

/*
 ... A lot of Redux / React boilerplate happens here 
 like mapDispatchToProps and mapStateToProps and @connect ...
*/

// Component method
handleFileUpload({ file }) {
  const file = files[0];
  this.props.actions.uploadRequest({
     file,
     name: 'Awesome Cat Pic'
  })
}
  
// Component render
<input type="file" onChange={this.handleFileUpload} />
```

When the input changes (file is added), the event handler will fire the Redux action creator and pass the file and name as arguments. The action creator will build FormData object, which is required for handling `multipart/form-data`. After we append file and name to the form’s data object, it’s good to go to the server. In this case, we're POSTing data to Node.js route. Let’s look at what’s happening on the server.

## Server
```js
import express from 'express';
import axios from 'axios';
import multer from 'multer';

const app = express();

/**
 ... express.js boilerplate
 routes, middlewares, helpers, loggers, etc
**/

// configuring Multer to use files directory for storing files
// this is important because later we'll need to access file path
const storage = multer.diskStorage({
  destination: './files',
  filename(req, file, cb) {
    cb(null, `${new Date()}-${file.originalname}`);
  },
});

const upload = multer({ storage });

// express route where we receive files from the client
// passing multer middleware
app.post('/files', upload.single('file'), (req, res) => {
 const file = req.file; // file passed from client
 const meta = req.body; // all other values passed from the client, like name, etc..
 
 // send the data to our REST API
 axios({
    url: `https://api.myrest.com/uploads`,
    method: 'post',
    data: {
      file,
      name: meta.name,      
    },
  })
   .then(response => res.status(200).json(response.data.data))
   .catch((error) => res.status(500).json(error.response.data));
});
```

 First, we’re configuring multer to use local /files/ directory to store uploaded files from the client. It’s important to hold the file somewhere before we send it to REST API, because in most cases we’ll have to provide full path to the files. Another option is to hold it in memory, but that may cause the server to crash. Next, we want to make sure we generate unique file names with file extensions (not provided by default).
The next step is creating the actual route where the client sends FormData and where we pass multer middleware. When the route receives a file, it goes through the middleware first and is stored in our `/files` directory with a newly generated file name. So, when we get to the callback (which can be refactored to custom middleware), the file is available as part of req object. From here, we can do whatever is needed, in this case, calling our external API (could be S3 or any API that handles files) and passing file and meta info.

Another way to upload files is to use Node.js streams instead of storing temporary files before sending them to CDN.
```js
import axios from 'axios';
import cloudinary from 'cloudinary';

export default function fileUploadMiddleware(req, res) {
  cloudinary.uploader.upload_stream((result) => {
    axios({
      url: '/api/upload', //API endpoint that needs file URL from CDN
      method: 'post',
      data: {
        url: result.secure_url,
        name: req.body.name,
        description: req.body.description,
      },
    }).then((response) => {
      res.status(200).json(response.data.data);
    }).catch((error) => {
      res.status(500).json(error.response.data);
    });
  }).end(req.file.buffer);
}
```

Once you have this middleware to handle streaming files to CDN, you can add new route which will trigger the middleware

```js
import multer from 'multer';
import cloudinary from 'cloudinary';
import fileUploadMiddleware from './fileUploadMiddleware';

/* your servrer init and express code here */

cloudinary.config({
  cloud_name: 'xxx',
  api_key: 'xxxx',
  api_secret: 'xxxxx',
});

/**
  * Multer config for file upload
*/

const storage = multer.memoryStorage();
const upload = multer({ storage });
app.post('/files', upload.single('file'), fileUploadMiddleware);

/* the rest of your routes app.get('*', () => {}) */
/* the rest of your server code */
```

Now you can upload files from React (or any other client side framework). Here is an example:
```js
import React, { Component } from 'react';
import axios from 'axios';

class uploadMyFile extends Component {
  handleUploadFile = (event) => {
    const data = new FormData();
    data.append('file', event.target.files[0]);
    data.append('name', 'some value user types');
    data.append('description', 'some value user types');
    // '/files' is your node.js route that triggers our middleware
    axios.post('/files', data).then((response) => {
      console.log(response); // do something with the response
    });
    
    render() {
      <div>
        <input type="file" onChange={this.handleUploadFile} />
      </div>
    }
}

export default uploadMyFile;
```

## How to hide api key?
1. Create a file called `.env` in the root of your project's directory.  
- your_frontend_project_folder
  - node_modules
  - public
  - src
  - .env         <-- create it here
  - .gitignore
  - package-lock.json
  - package.json
2. Inside the `.env` file, prepend `REACT_APP_` to your API key name of choice and assign it.
    ```js
    // .env
    REACT_APP_API_KEY=your_api_key  
    ```
3. Add the `.env` file to your `.gitignore` file.
   ```
    // .gitignore
    # api keys
    .env       <-- add this line
    # dependencies
    /node_modules
    ```
4. Access the API key via the `process.env` object.
    ```js
    // src/App.js
    import React, { Component } from 'react';
    import './App.css';
    console.log(process.env.REACT_APP_WEATHER_API_KEY)
    ```