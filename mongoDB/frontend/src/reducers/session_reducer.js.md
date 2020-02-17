Go back to [mongoDB.md](../../../../mongoDB.md)


```js
import { RECEIVE_USER_LOGOUT } from '../actions/session_actions';

export default function(state = initialState, action) {
  switch (action.type) {
    case RECEIVE_USER_LOGOUT:
      return {
        isAuthenticated: false,
        user: undefined
      };
    default:
      return state;
  }
}
```