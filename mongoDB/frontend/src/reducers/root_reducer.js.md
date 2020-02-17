Go back to [mongoDB.md](../../../../mongoDB.md)


```js
// src/reducers/root_reducer.js

import { combineReducers } from 'redux';
import session from './session_reducer';

const RootReducer = combineReducers({
  session
});

export default RootReducer;
```