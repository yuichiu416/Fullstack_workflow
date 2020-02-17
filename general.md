## Going to a random route should redirect or display a 404 page
There are a few ways to implement this:
1. Use `Switch` from `react-router`
    ``` Javascript
    import { Switch } from 'react-router';
    import BadRoute from './home/404';
    
    export default () => {
        return (
            <div>
                <Switch>
                    <AuthRoute path="/" component={GNavContainer} />
                    <AuthRoute exact path="/" component={Chunk1} />
                    <AuthRoute path="/login" component={LoginContainer} />
                    <BadRoute />  //This way all other bad routes will be redirected to the 404 page.
                </Switch>
            </div>
        );
    };
    ```
2. Use the controller to handle all bad routes, note that it doesn't work for react routes,
   ex. doesn't work on `http://localhost:3000/#/some_route`, but it can catch `http://localhost:3000/some_route`
   ```Ruby
   # config/routes.rb
    Rails.application.routes.draw do
        match '*path' => 'root#bad_route', via: :all
    end
    ```

## Errors should display for both /signup and /login.
1. Add an error action and reducer
    ```Javascript
    // frontend/actions/session.js
    export const CLEAR_SESSION_ERRORS = 'CLEAR_SESSION_ERRORS';
    export const receiveErrors = errors => ({
        type: RECEIVE_SESSION_ERRORS,
        errors
    });
            // add to our regular reducer
    export const createNewUser = formUser => dispatch => (
        postUser(formUser).then(user => (
            dispatch(receiveCurrentUser(user))
        ), err => (
            dispatch(receiveErrors(err.responseJSON))
        ))
    );

    // frontend/reducers/session_error_reducer.js
    import { RECEIVE_SESSION_ERRORS, CLEAR_SESSION_ERRORS } from '../actions/session';
    export default (state = [], action) => {
        Object.freeze(state);
        switch (action.type) {
            case RECEIVE_SESSION_ERRORS:
                return action.errors;
            case CLEAR_SESSION_ERRORS:
                return Object.assign({}, { errors: [] });
            default:
                return state;
        }
    };
    ```
2. Add the action to the container accordingly
    ```Javascript
    // frontend/components/session/login_container.jsx
    import { connect } from 'react-redux';
    import { login, clearErrors } from '../../actions/session';
    import Login from './login';

    const masStateToProps = state => ({
        errors: state.errors,
    });
    const mapDispatchToProps = dispatch => ({
        login: formUser => dispatch(login(formUser)),
        clearErrors: () => dispatch(clearErrors()),
    });

    export default connect(masStateToProps, mapDispatchToProps)(Login);
    ```
3. Render the error message in the component

##  Errors should clear when moving between /signup and /login
1. Use state to log the errors.
2. Create a clearError action, check code above.
3. Fire the action at the right place, such as in a constructor

## Stay logged in
**Bootstrap the user** (W14D3 Project, part 8)
1. Make a script in the general template
    ```erb
    //app/views/layouts/application.html.erb
        <% if current_user %>
        <script>
          window.currentUser = {
            "id": <%= current_user.id %>,
            "username": "<%= current_user.username %>",
            "email": "<%= current_user.email %>",
          };
        </script>
    <% end %>
    ```
2. Go to the entry file and set up the preloaded state 

```
//frontend/entry.jsx
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
})
```

## How to add an search icon
```css
 .search {
  background-color: white;
  background-image: url('/images/searchicon.png');
  background-size: 25px 25px;
  width: 30%;
  margin: 0 auto;
  box-sizing: border-box;
  border-radius: 4px;
  font-size: 16px;
  background-position: 10px 10px; 
  background-repeat: no-repeat;
  padding: 12px 20px 12px 40px;
}
```

## How to add a logo
```jsx
    //navbar.jsx
    <div className="header-nav-logo">
        <Link to="/"><img src="/images/logo.png" alt="logo" /></Link>
    </div>
```
```css
  .header-nav-logo {
    margin: 0;
    display:flex;
    align-items: center;
    img{
        width: 50px;
    } 
  }
```

## InvalidTokenError: Invalid token specified: Cannot read property 'replace' of undefined
Try `localStorage.removeItem('jwtToken')`

## Server running on port already (Windows)
`netstat -ano | findstr 3000`

`taskkill /pid <pid_here> /f`