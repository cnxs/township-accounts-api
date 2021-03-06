# township-accounts-api

route handlers to use in your REST API to handle user management

## quick start

```js
var Township = require('township-accounts-api')
var db = require('memdb') // can be any levelup e.g. level-party or level
var config = require('./your-config')
var ship = new Township(db, config)

// now you can use `ship` to handle (req, res) route handlers
```

`township` just provides route handler functions so you can integrate auth routes into your web server of choice.

here's an example using the `require('appa')` REST server module

```js
var memdb = require('memdb')
var createAppa = require('appa')
var Township = require('township-accounts-api')
var config = require('./your-config')

var app = createAppa()
var db = memdb()
var ship = new Township(db, config)

app.on('/register', function (req, res, ctx) {
  // appa provides `ctx` for us in the way we want out of the box
  ship.register(req, res, ctx, function (err, respCode, data) {
    if (err) return app.error(res, respCode, err.message)
    app.send(res, respCode, data)
  })
})
```

see also `test-server.js`

## API

### var Township = require('township-accounts-api')

returns a constructor you can use to make multiple instances

### var ship = new Township(db, config)

creates a new instance

`db` should be a levelup instance

`config` properties:

  - `secret` (String) - used with township-token
  - `email` (Object) - used to send emails with postmark
  - `email.fromEmail` (String) - from address
  - `email.postmarkAPIKey` (String)
  
### ship.verify(req, cb)

given a `request`, decodes and verifies the token in the authorization header and calls `cb` with the result

pass `req` from your http server. the request is expected to have an `Authorization: Bearer <token>` header.

`cb` will be called with `(error, decodedToken, rawToken)`.

`error` will be called if the token is missing from the request or had a problem being verified

`decodedToken` is a JS object with the result of `jwt.verify`.

`rawToken` is a string containing the encoded token value received from the request header

### ship.register(req, res, ctx, cb)

registers a new user. pass `req`, `res` from your http server.

`ctx` should be an object with:

- `body` (Object) - the POST JSON body as a parsed Object
- `body.email` (String)
- `body.password` (String)

`cb` is called with `(err, newToken)`

### ship.login(req, res, ctx, cb)

returns a token for an existing user

`ctx` should be an object with:

- `body` (Object) - the POST JSON body as a parsed Object
- `body.email` (String)
- `body.password` (String)

`cb` is called with `(err, token)`

### ship.updatePassword(req, res, ctx, cb)

changes a users password, invalidates old token and issues new token

`ctx` should be an object with:

- `body` (Object) - the POST JSON body as a parsed Object
- `body.email` (String)
- `body.password` (String)
- `body.newPassword` (String)

`cb` is called with `(err, newToken)`
