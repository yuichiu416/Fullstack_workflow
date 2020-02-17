Go back to [mongoDB.md](../mongoDB.md)

```js
// keys_prod.js
module.exports = {
  mongoURI: process.env.MONGO_URI,
  secretOrKey: process.env.SECRET_OR_KEY
}
```