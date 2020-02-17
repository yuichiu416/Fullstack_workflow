## An example flow
1. Implement entry file
2. Implement api-util file
3. Implement actions file
4. Implement reducers file
5. Implement store file
6. Implement index_container file
7. Implement index file
8. Implement index_item file
   
### Implement the project layout
* frontend/reducer/root_reducer.js
   ```js
    import { combineReducers } from 'redux';
    import entitiesReducer from './entities_reducer';
    import sessionReducer from './session_reducer';
    import errorsReducer from './errors_reducer';

    const rootReducer = combineReducers({
      entities: entitiesReducer,
      session: sessionReducer,
      errors: errorsReducer
    });

    export default rootReducer;
    ```
* frontend/store/store.js
    ```javascript
      //frontend/components/store/store.js
      import { createStore, applyMiddleware } from 'redux';
      import thunk from 'redux-thunk';
      import logger from 'redux-logger';
      import rootReducer from '../reducers/root_reducer';

      const configureStore = (preloadedState = {}) => {
        let middleware = [thunk];
        if (process.env.NODE_ENV !== 'production') {
            middleware = [...middleware, logger];
        }
        return createStore(
            rootReducer,
            preloadedState,
            applyMiddleware(...middleware)
        );
      };
      export default configureStore;
      ```
* frontend/components/app.jsx
   ```jsx
    import React from 'react';
    import SignupFormContainer from './session/signup_form_container';
    import LoginFormContainer from './session/login_form_container';
    import HomeContainer from './Home/Home_container';
    import StockShowContainer from './stocks/stock_show_container';
    import UserProfileContainer from './users/user_profile_container';
    import { Route, Redirect, Switch, Link } from 'react-router-dom';
    import { AuthRoute, ProtectedRoute } from '../util/route_util';

    const App = () => (
        <div>
            <Switch>
                <AuthRoute exact path='/signup' component={SignupFormContainer} />
                <AuthRoute exact path='/login' component={LoginFormContainer} />
                <ProtectedRoute exact path="/stocks/:ticker" component={StockShowContainer} />
                <ProtectedRoute exact path="/users/:id" component={UserProfileContainer} />
                <Route exact path='/' component={HomeContainer} />
                <Redirect to="/" />
            </Switch>
        </div>
    );

    export default App;
    ```
* frontend/components/root.jsx
    ```jsx
    import React from 'react';
    import { Provider } from 'react-redux';
    import { HashRouter } from 'react-router-dom';
    import App from './App';

    const Root = ({ store }) => (
      <Provider store={store}>
        <HashRouter>
          <App />
        </HashRouter>
      </Provider>
    );

    export default Root;
    ```
## Session forms and containers
```jsx
//frontend/components/session/login_container.jsx
import { connect } from 'react-redux';
import { login, clearErrors } from '../../actions/session_actions';
import SessionForm from './session_form';


const masStateToProps = state => ({
    errors: state.errors,
    formType: "Log In",
});
const mapDispatchToProps = dispatch => ({
    login: formUser => dispatch(login(formUser)),
    clearErrors: () => dispatch(clearErrors()),
});

export default connect(masStateToProps, mapDispatchToProps)(SessionForm);
```

```jsx
//frontend/components/session/session_form.jsx
import React from 'react';

class SessionForm extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            username: '',
            email: '',
            password: '',
            errors: props.errors,
        };
        props.clearErrors();
        this.handleLogIn = this.handleLogIn.bind(this);
        this.handleSignUp = this.handleSignUp.bind(this);
        this.handleDemo = this.handleDemo.bind(this);
    }

    handleInput(type) {
        return (e) => {
            this.setState({ [type]: e.target.value });
        };
    }
    handleLogIn(e) {
        e.preventDefault();
        this.setState(this.state);
        this.props.login(this.state)
            .then(() => this.props.history.push('/'));
    }
    handleSignUp(e) {
        e.preventDefault();
        this.props.createNewUser(this.state)
            .then(() => this.props.history.push('/'));
    }
    handleDemo(e) {
        e.preventDefault();
        this.state = {
            username: 'Demo',
            password: '123456',
        };
        this.props.login(this.state)
            .then(() => this.props.history.push('/'));
    }


    render() {
        const { errors, formType } = this.props;
        let errorList, errorUl, email, button;
        if (errors.length > 0) {
            errorList = this.props.errors.map(error => (
                <li key={error}>{error}</li>
            ));
            errorUl = <ul>{errorList}</ul>;
        }

        if (formType == "Sign Up") {
            email = <label>
                Email:
                        <input type="text" value={this.state.email} onChange={this.handleInput("email")} /><br/>
                    </label>;
            button = <button lassName="button" onClick={this.handleSignUp}>Sign Up!</button>;
        }
        else {
            button = <button lassName="button" onClick={this.handleLogIn}>Log In!</button>
        }
        return (
            <div className="sessionform">
                <div className="sessionform-image"></div>
                <div className="sessionform-text">
                    <h2>Welcome to Rock On</h2>
                    <form>
                        <label>
                            Username:
                            <input required type="text" value={this.state.username} onChange={this.handleInput('username')} />
                        </label>
                        <br />
                        {email}
                        <label>Password:
                            <input required type="password" value={this.state.password} onChange={this.handleInput('password')} />
                        </label>
                        <br />
                        {button}
                        <button className="button" onClick={this.handleDemo}>Demo Login</button>
                    </form>
                    <div className="sessionform-errors">{errorUl}</div>
                </div>
            </div>
        );
    }
}

export default SessionForm;

```

## How to get next state
Set up a container, map it from state to props

## How to click a link with javascript
Find the element then call `.click()`
ex.
```js
 document.getElementById(`match-${index}`).click();
```

## Add an image
Using an image tag seems to be easier, Rails magic doesn't help much
Putting everything in `./public/images/` saved a lot of time
ex.
`<img className="banner2" src="/images/banner2.png" alt="banner2" />`

## Set/clear interval properly
1. Make an array for interval ids
2. In `componentWillUnmount()`, clear all of them