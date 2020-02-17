It's based on [SWE Online -  GraphQL Curriculum Day 3 - Backend Authentication with GraphQL](https://open.appacademy.io/learn/swe-online/swe-online---graphql-curriculum/backend-authentication-with-graphql)

### Setting Up The User

Set up schema
```js
const UserSchema = new Schema({
  email: {
    type: String,
    required: true
  },
  password: {
    type: String,
    required: true
  }
});
```
```js
// user_type.js
const UserType = new GraphQLObjectType({
  name: "UserType",
  fields: () => ({
    // Mongoose automatically generates an ID field for our models so we'll want to be able to return it
    _id: { type: GraphQLID },
    email: { type: GraphQLString },
    // we'll be able to return a boolean to indicate if the user is logged In
    loggedIn: { type: GraphQLBoolean },
    // Adding token will allow us to generate and return an authentication token
    token: { type: GraphQLString }
  })
});
```

## Authentication Mutations
We'll require three additional libraries for authentication in our backend:

1. `jsonwebtoken` - to generate tokens to put into our incoming and outgoing requests.
2. `bcryptjs` - to salt and hash incoming passwords.
3. `validator` - for writing our database validations

Use [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) or other long random string generator to encode or decode the token.

```js
// config/keys.js
module.exports = {
  MONGO_URI: "blahblah",
  secretOrkey: "1234567-jsdakhjds-ajsjkdjkdsa"
};
```

## Server Validation
```js
// validations/valid_text.js

const validText = str => {
  return typeof str === "string" && str.trim().length > 0;
};
```

An example of validator:
```js
module.exports = function validateInputs(data) {
  data.email = validText(data.email) ? data.email : "";
  data.name = validText(data.name) ? data.name : "";
  data.password = validText(data.password) ? data.password : "";

  if (!Validator.isEmail(data.email)) {
    return { message: "Email is invalid", isValid: false };
  }

  if (Validator.isEmpty(data.email)) {
    return { message: "Email field is required", isValid: false };
  }

  if (Validator.isEmpty(data.name)) {
    return { message: "Name field is required", isValid: false };
  }

  if (Validator.isEmpty(data.password)) {
    return { message: "Password field is required", isValid: false };
  }

  return {
    message: "",
    isValid: true
  };
};
```

## Adding Authentication Services
For registering a new User we need to:

1. Validate our inputs using the validateInputs function.
2. Hash the incoming password
3. Construct and save a new user with everything that was passed in
4. Create our authentication token using jsonwebtoken
5. Set loggedIn to true
6. Return all the information to our resolver except the hashed password
   
For logging in an existing User we need to:

1. Validate our inputs using the validateInputs function.
2. Find a user based on the passed in email.
3. Validate the password upon the User we found using BCrypt
4. Create our authentication token using jsonwebtoken
5. Set loggedIn to true
6. Return all the information to our resolver except the hashed password


For example for registering a new User our `register` function would look something like this:
```js
// services/auth.js

// we use async/await in order to ensure this function will fully complete every step before returned
const register = async data => {
  try {
    // make sure our inputs our valid
    const { message, isValid } = validateRegisterInput(data);

    if (!isValid) {
      throw new Error(message);
    }

    const { name, email, password } = data;
    // make sure this sure doesn't already exist
    const existingUser = await User.findOne({ email });

    if (existingUser) {
      throw new Error("This user already exists");
    }
    // hash our password
    const hashedPassword = await bcrypt.hash(password, 10);
    // create a new User
    const user = new User(
      {
        name,
        email,
        password: hashedPassword
      },
      err => {
        if (err) throw err;
      }
    );

    user.save();
    // create our authentication token using the secretOrkey we created before
    const token = jwt.sign({ id: user._id }, key);

    // return to our mutation the token, verifying this User is logged in
    // as well as the rest of the user's info, and blanking out the password
    return { token, loggedIn: true, ...user._doc, password: null };
  } catch (err) {
    throw err;
  }
};
```

So the above function handles all the logic involved in registering a User. Since we need this function to complete each promise before returning we make sure to use the [async/await](https://javascript.info/async-await) syntax. With that done the `register` mutation can be completed by using the above function in the resolver:
```js
// schema/mutations.js
const mutation = new GraphQLObjectType({
  name: "Mutation",
  fields: {
    register: {
      // we make sure to return the User type. This type will also now
      // have fields for returning the token and loggedIn value.
      type: UserType,
      args: {
        name: { type: new GraphQLNonNull(GraphQLString) },
        email: { type: new GraphQLNonNull(GraphQLString) },
        password: { type: new GraphQLNonNull(GraphQLString) }
      },
      resolve(parentValue, data) {
        return AuthService.register(data);
      }
    }
  }
});
```

The following is a function that will take in a User's token and will then verify that token belongs to a valid User:
```js
const verifyUser = async data => {
  try {
    // it take in the authentication token
    const { token } = data;
    // we verify that token against our secretKey in order to get the User's original ID
    const decoded = jwt.verify(token, key);
    const { id } = decoded;

    // then we use that ID in order to see if that User does in fact exist
    // in our database
    const loggedIn = await User.findById(id).then(user => {
      return user ? true : false;
    });

    return { loggedIn };
  } catch (err) {
    return { loggedIn: false };
  }
};
```

## Conclusion
In summary for backend User Authentication using `GraphQL` - you need to:

1. Extend the User Type to add a `token` and `loggedIn` value. We'll use these values a great deal more on the frontend.
2. Create a Mutation allowing a User to register or login. This break down into a couple of steps:

   * Validate the incoming information from a User using `validator`
   * Create separate functions to handle the logic to `register`, `login`, and verifying a User's authentication token.
   * Add those functions to the `resolvers` for the `register` and `login` mutations in your schema.


# Unions and Conditional Fragments in GraphQL
## Union Types
```js
const UsernameAuth = new GraphQLObjectType({
  name: "UsernameAuth",
  fields: {
    name: { type: new GraphQLNonNull(GraphQLString) },
    username: { type: new GraphQLNonNull(GraphQLString) },
    password: { type: new GraphQLNonNull(GraphQLString) }
  }
});

const EmailAuth = new GraphQLObjectType({
  name: "EmailAuth",
  fields: {
    name: { type: new GraphQLNonNull(GraphQLString) },
    email: { type: new GraphQLNonNull(GraphQLString) },
    password: { type: new GraphQLNonNull(GraphQLString) }
  }
});
```
Now, we could define an unified `AuthType` to be the union of `UsernameAuth` and `EmailAuth`:
```js
const AuthType = new GraphQLUnionType({
  name: 'Auth',
  types: [UsernameAuth, EmailAuth]
  resolveType: resolveAuthType
});

function resolveAuthType(value) {
  return value.username ? UsernameAuth : EmailAuth;
}
```

## Conditional Fragments
This brings up a different problem - what if we don't know which `type` we are querying when our UnionType returns? We can't count on there being a `username` or an `email` if we don't know the incoming `type`.

The answer to this is using something we previously learned - conditional fragments:
```js
{
    allAuth {
        name // works for both `UsernameAuth` and `EmailAuth`
        ... on EmailAuth {
            email
        }
        ... on UsernameAuth {
            username
        }
    }
}
```