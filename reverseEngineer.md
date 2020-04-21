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

'config.json' is used to specify values used to connect to three different server connections.

---

## middleware

### isAuthenticated

'isAuthenticated.js' is a custom middleware used to restrict the routes can visit if they are not logged in.

If logged in, the user can continue to the restricted route:

```
module.exports = function(req, res, next) {
    if (req.user) {
    return next();
  }
```

If the user is not logged in, the middleware will redirect the user back to the login page:

```
return res.redirect("/");
};
```

---

## passport.js

'passport.js' is an authentication middleware for node.js. This file is already notated on the file provided:

```
var passport = require("passport");
var LocalStrategy = require("passport-local").Strategy;

var db = require("../models");

// Telling passport we want to use a Local Strategy. In other words, we want login with a username/email and password
passport.use(new LocalStrategy(
  // Our user will sign in using an email, rather than a "username"
  {
    usernameField: "email"
  },
  function(email, password, done) {
    // When a user tries to sign in this code runs
    db.User.findOne({
      where: {
        email: email
      }
    }).then(function(dbUser) {
      // If there's no user with the given email
      if (!dbUser) {
        return done(null, false, {
          message: "Incorrect email."
        });
      }
      // If there is a user with the given email, but the password the user gives us is incorrect
      else if (!dbUser.validPassword(password)) {
        return done(null, false, {
          message: "Incorrect password."
        });
      }
      // If none of the above, return the user
      return done(null, dbUser);
    });
  }
));

// In order to help keep authentication state across HTTP requests,
// Sequelize needs to serialize and deserialize the user
// Just consider this part boilerplate needed to make it all work
passport.serializeUser(function(user, cb) {
  cb(null, user);
});

passport.deserializeUser(function(obj, cb) {
  cb(null, obj);
});

// Exporting our configured passport
module.exports = passport;

```

In summary, the following actions occur:

1. The npm package 'passport' is imported. Passport is an Express-compatible authentication middleware.

```
var passport = require("passport");
```

2. Configure LocalStrategy on 'passport-local'. 'Passport-local' is a module of 'passport' used to authenticate users via a username and password.

```
var LocalStrategy = require("passport-local").Strategy;
```

3. Import 'models' folder:

```
var db = require("../models");
```

4. Tell passport we want to use a Local Strategy to login with a username/email and password:

```
passport.use(new LocalStrategy(
```

5. Username is assigned to be an e-mail:

```
  {
    usernameField: "email"
  },
    function(email, password, done) {
```

6. Code that starts when a user tries to sign in after which a promise is called. The promise has three cases (see 7-10 below).

```
db.User.findOne({
      where: {
        email: email
      }
    }).then(function(dbUser) {
```

7. The first case will run if there is no user with the given e-mail. The message "Incorrect email." will be returned:

```
if (!dbUser) {
        return done(null, false, {
          message: "Incorrect email."
        });
      }
```

8. The second case will run if there is a user with a given e-mail but the password is incorrect:

```
 else if (!dbUser.validPassword(password)) {
        return done(null, false, {
          message: "Incorrect password."
        });
      }
```

9. The third case will run if both the user e-mail exists and the password is correct. User is returned.

```
     return done(null, dbUser);
    });
  }
));
```

10. Sequelize is used to serialize and deserialize the user to help keep authentication state across HTTP requests:

```
passport.serializeUser(function(user, cb) {
  cb(null, user);
});

passport.deserializeUser(function(obj, cb) {
  cb(null, obj);
});
```

11. 'Passport' is exported

```
module.exports = passport;
```

---

# models

## index.js

Description about index.js goes here

---

## user.js

Description about user.js goes here

---

# node_modules

Description about node_modules goes here

---

# public

Description about public goes here

## login.html

Description about login.html goes here

---

## members.html

Description about members.html goes here

---

## signup.html

Description about signup.html goes here

---

## js

### login.js

Description about login.js goes here

---

### members.js

Description about members.js goes here

---

### signup.js

Description about signup.js goes here

---

## stylesheets

### style.css

Description about style.css goes here

---

# routes

## api-routes.js

Description about api-routes.js goes here

---

## html-routes.js

Description about html-routes.js goes here

---

# .gitignore

Description about .gitignore goes here
