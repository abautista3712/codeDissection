# Reverse Engineering

Upon opening up the 'Develop' folder, I see a package.json. Run the following command to install dependencies:

```
npm i
```

---

## package.json

Looking more closely at the 'package.json' file:

```
{
  "name": "1-Passport-Example",
  "version": "1.0.0",
  "description": "",
  "main": "server.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node server.js",
    "watch": "nodemon server.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "bcryptjs": "2.4.3",
    "express": "^4.17.0",
    "express-session": "^1.16.1",
    "mysql2": "^1.6.5",
    "passport": "^0.4.0",
    "passport-local": "^1.0.0",
    "sequelize": "^5.8.6"
  }
}
```

The object keys for 'name', 'version', and 'description' are self-explanatory.

The object key 'main' refers to the main entry point on how to start the program. In this case:

```
main: server.js
```

Suggests that the server can be started with the following code:

```
node ./server.js
```

This is further reiterated with the following code:

```
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node server.js",
    "watch": "nodemon server.js"
  },
```

These are relevant scripts with which the user can use in relation to the program.

```
"test": "echo \"Error: no test specified\" && exit 1"
```

Identifies that there are no code explicitly written to test the program.

```
"start": "node server.js"
```

Reiterates how to start the program.

```
"watch": "nodemon server.js"
```

Can be used to start nodemon, which restarts the server when changes have been made to the code.

The object keys for 'keywords' and 'author' are self-explanatory.

The license used is notated by the object key "license":

```
  "license": "ISC"
```

In particular, this line of code describes that the program uses the permissive free software license published by the Internet Systems Consortium.

Lastly, the object key 'dependencies' describes npm packages and version numbers used/needed in order to use the program:

```
"dependencies": {
    "bcryptjs": "2.4.3",
    "express": "^4.17.0",
    "express-session": "^1.16.1",
    "mysql2": "^1.6.5",
    "passport": "^0.4.0",
    "passport-local": "^1.0.0",
    "sequelize": "^5.8.6"
  }
```

---

## package-lock.json

This file lists information about each of the dependencies that are installed including the specific versions, the location of the modules, a hash verifying the integrity of each module, the list of packages that requires it, and a list of dependencies.

---

## server.js

The 'server.js' file, as described in the 'package.json' section, is where the program can be started. It is already notated in the code given. See below:

```
// Requiring necessary npm packages
var express = require("express");
var session = require("express-session");
// Requiring passport as we've configured it
var passport = require("./config/passport");

// Setting up port and requiring models for syncing
var PORT = process.env.PORT || 8080;
var db = require("./models");

// Creating express app and configuring middleware needed for authentication
var app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(express.static("public"));
// We need to use sessions to keep track of our user's login status
app.use(session({ secret: "keyboard cat", resave: true, saveUninitialized: true }));
app.use(passport.initialize());
app.use(passport.session());

// Requiring our routes
require("./routes/html-routes.js")(app);
require("./routes/api-routes.js")(app);

// Syncing our database and logging a message to the user upon success
db.sequelize.sync().then(function() {
  app.listen(PORT, function() {
    console.log("==> 🌎  Listening on port %s. Visit http://localhost:%s/ in your browser.", PORT, PORT);
  });
});
```

Summarizing 'server.js', as was already notated in the code given, the following actions occur:

1. npm packages (express and express-session) are required:

```
var express = require("express");
var session = require("express-session");
```

2. './config/passport' is required as configured:

```
var passport = require("./config/passport");
```

3. PORT is set up to be either process.env.PORT (assigned by Heroku) or 8080:

```
var PORT = process.env.PORT || 8080;
```

4. './models' is required:

```
var db = require("./models");
```

5. Express app is created:

```
var app = express();
```

6. Middleware is configured: Encodes the incoming requests with urlencoded payloads.

```
app.use(express.urlencoded({ extended: true }));
```

7. Middleware is configured: Encodes the incoming requests with JSON payloads.

```
app.use(express.json());
```

8. Middleware is configured: Allows the usage of static files in the 'public' folder

```
app.use(express.static("public"));
```

9. Express sessions is used to keep track of user's login status:

```
app.use(session({ secret: "keyboard cat", resave: true, saveUninitialized: true }));
app.use(passport.initialize());
app.use(passport.session());
```

10. html-routes.js and api-routes.js are required

```
require("./routes/html-routes.js")(app);
require("./routes/api-routes.js")(app);
```

11. Database is synced and server is made to listen to PORT. Message is sent to the console upon success:

```
db.sequelize.sync().then(function() {
  app.listen(PORT, function() {
    console.log("==> 🌎  Listening on port %s. Visit http://localhost:%s/ in your browser.", PORT, PORT);
  });
});
```

---

# config

## config.json

This file is used to

---

## middleware

### isAuthenticated

This file

---

## passport.js

This file

---
