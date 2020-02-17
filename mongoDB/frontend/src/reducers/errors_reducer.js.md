Go back to [mongoDB.md](../../../../mongoDB.md)


```js
// src/reducers/errors_reducer.js

import { combineReducers } from 'redux';

import SessionErrorsReducer from './session_errors_reducer';

export default combineReducers({
  session: SessionErrorsReducer
});
```