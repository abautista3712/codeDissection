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

2. Configure 'LocalStrategy' on 'passport-local'. 'Passport-local' is a module of 'passport' used to authenticate users via a username and password.

```
var LocalStrategy = require("passport-local").Strategy;
```

3. Import 'models' folder:

```
var db = require("../models");
```

4. Tell 'passport' we want to use a 'LocalStrategy' to login with a username/email and password:

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

Directive in ECMAScript 5 to indicate that code should be executed in "strict mode":

```
'use strict';
```

Import node package 'fs'. File System (fs) allows for the user to work with the file system of the computer. Common uses of the fs module include reading, creating, updating, deleting, and renaming files.

```
var fs        = require('fs');
```

Import node package 'path'. Per the nodejs.org documentation, the 'path' module provides utilities for working with file and directory paths.

```
var path      = require('path');
```

Import 'Sequelize'. 'Sequelize' at its core is an Object-Relational Mapper (ORM) meaning that it maps object syntax onto database schemas. 'Sequelize' (uppercase) references the standard library.

```
var Sequelize = require('sequelize');
```

Set the variable 'basename' as the last portion of the file path for 'module.filename':

```
var basename  = path.basename(module.filename);
```

Define the variable 'env' to be 'process.env.NODE_ENV' otherwise it is set to 'development'.

More specifically, 'process.env' is a global variable that checks the system environment during runtime.

Additionally, 'NODE_ENV' will check whether the particular environment is a 'production' or 'development' environment at start up otherwise the environment will be set to 'development' by default.

```
var env       = process.env.NODE_ENV || 'development';
```

Import the 'config.json' file and configure environment.

Particularly, '\_\_dirname' returns the absolute path of the directory where the currently executing file is (i.e., index.js). The absolute path is then used to reference the path to the 'config.json' file which is then configured to the enviornment variable.

```
var config    = require(__dirname + '/../config/config.json')[env];
```

An empty object is created under the variable 'db':

```
var db        = {};
```

The configuration environment is established using Sequelize.

'config.use_env_variable' will check to see if an environment variable is set. If it is, the settings for that environment variable will be used. Alternatively, the settings will be used as provided by the environment used.

```
if (config.use_env_variable) {
  var sequelize = new Sequelize(process.env[config.use_env_variable]);
} else {
  var sequelize = new Sequelize(config.database, config.username, config.password, config);
}
```

The models are read and imported.

'fs.readdirSync(\_\_dirname)' reads the absolute path of directory that 'index.js'

```
fs.readdirSync(__dirname)
```

The models are then filtered to make sure that there are files to read, the file is not the 'index.js' file, and that the file ends in '.js':

```
.filter(function(file) {
    return (file.indexOf('.') !== 0) && (file !== basename) && (file.slice(-3) === '.js');
  })
```

The filtered models are then imported into the database:

```
.forEach(function(file) {
    var model = sequelize['import'](path.join(__dirname, file));
    db[model.name] = model;
  });
```

Call up each filtered model in the database (i.e., user.js) and call its 'associate' function if it exists.

```
Object.keys(db).forEach(function(modelName) {
  if (db[modelName].associate) {
    db[modelName].associate(db);
  }
});
```

Two sequelize objects are created and brought into the db variable.

One 'sequelize' (lowercase) references our connection to the db whereas the other 'Sequelize' (uppercase) references the standard library:

```
db.sequelize = sequelize;
db.Sequelize = Sequelize;
```

The 'db' is then exported:

```
module.exports = db;
```

---

## user.js

'user.js' is already notated in the provided folder. In summary, the following actions occur:

1. The npm package bcrypt is imported. 'bcrypt' is a library that helps with password hashing:

```
var bcrypt = require("bcryptjs");
```

2. 'User' model is created. The specifications of the 'User' model are described in 3 and 4 below:

```
module.exports = function(sequelize, DataTypes) {
  var User = sequelize.define("User", {
```

3. The e-mail is a string, cannot be null, must be unique, and must be a proper e-mail before creation:

```
email: {
      type: DataTypes.STRING,
      allowNull: false,
      unique: true,
      validate: {
        isEmail: true
      }
    },
```

4. The password is a string and cannot be null:

```
password: {
      type: DataTypes.STRING,
      allowNull: false
    }
  });
```

5. A custom method is created for our 'User' model. The model will compare the unhashed password inputted by the user and check it against what is registered in the db

```
User.prototype.validPassword = function(password) {
    return bcrypt.compareSync(password, this.password);
  };
```

6. A hook is added that will automatically hash the password before a User is created:

```
User.addHook("beforeCreate", function(user) {
    user.password = bcrypt.hashSync(user.password, bcrypt.genSaltSync(10), null);
  });
```

7. The 'User' model is then returned:

```
  return User;
};
```

---

# node_modules

'node_modules' is a folder that contains all of the npm libraries downloaded. It is best practice to exclude it from version control (see .gitignore section below) since it can be easily recreated using the information listed in the package.json.

---

# public

The 'public' folder contains all the static files (e.g., HTML, CSS, JS, images) that are referenced in the program.

## login.html

'login.html' is the HTML page that the user encounters when first trying to log into the page.

'login.html' primarily consists of a form that has two fields: one field for e-mail address and one field for password. Once both the e-mail and password fields have been completed, the user has the option to press a button to login. The user can also alternatively click a different button to sign up for an account without filling in the form.

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

'form.signup' and 'form.login' are styled and given a margin on top of 50px:

```
form.signup,
form.login {
  margin-top: 50px;
}
```

---

# routes

## api-routes.js

Description about api-routes.js goes here

---

## html-routes.js

Description about html-routes.js goes here

---

# .gitignore

'.gitignore' is a file that can exclude certain files from being tracked by version control (i.e., GitHub). File names written in '.gitignore' will not be tracked for changes and will ultimately not be pushed up to GitHub.

In this particular case, 'node_modules' is excluded from version control since it can be easily recreated using the information provided in package.json.

```
node_modules
```
