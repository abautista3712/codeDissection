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

The page is styled with both Bootstrap and an external stylesheet. Additionally, jQuery is used and references the use of 'js/login.js':

```
<!DOCTYPE html>
<html lang="en">

<head>
  <title>Passport Authentication</title>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootswatch/3.3.7/lumen/bootstrap.min.css">
  <link href="stylesheets/style.css" rel="stylesheet">
</head>

<body>
  <nav class="navbar navbar-default">
    <div class="container-fluid">
      <div class="navbar-header">
      </div>
    </div>
  </nav>
  <div class="container">
    <div class="row">
      <div class="col-md-6 col-md-offset-3">
        <h2>Login Form</h2>
        <form class="login">
          <div class="form-group">
            <label for="exampleInputEmail1">Email address</label>
            <input type="email" class="form-control" id="email-input" placeholder="Email">
          </div>
          <div class="form-group">
            <label for="exampleInputPassword1">Password</label>
            <input type="password" class="form-control" id="password-input" placeholder="Password">
          </div>
          <button type="submit" class="btn btn-default">Login</button>
        </form>
        <br />
        <p>Or sign up <a href="/">here</a></p>
      </div>
    </div>
  </div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.2.1/jquery.min.js"></script>
  <script type="text/javascript" src="js/login.js"></script>

</body>

</html>

```

---

## members.html

'members.html' is the main page the user encounters once the user has been authenticated. Information can be populated onto this page dynamically. There is also a button on the top left with which the user can logout.

The page is styled with both Bootstrap and an external stylesheet. Additionally, jQuery is used and references the use of 'js/members.js':

```
<!DOCTYPE html>
<html lang="en">

<head>
  <title>Passport Authentication</title>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootswatch/3.3.7/lumen/bootstrap.min.css">
  <link href="stylesheets/style.css" rel="stylesheet">
</head>

<body>
  <nav class="navbar navbar-default">
  <div class="container-fluid">
    <div class="navbar-header">
      <a class="navbar-brand" href="/logout">
        Logout
      </a>
    </div>
  </div>
</nav>
  <div class="container">
    <div class="row">
      <div class="col-md-6 col-md-offset-3">
        <h2>Welcome <span class="member-name"></span></h2>
      </div>
    </div>
  </div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.2.1/jquery.min.js"></script>
  <script type="text/javascript" src="js/members.js"></script>

</body>

</html>

```

---

## signup.html

'signup.html' is the page the user encounters when trying to signup for an account. Similar to 'login.html', this file contains a form with two fields: one field for e-mail and one field for password. Once completed, the user can create a new account by pressing the 'submit' button. Alternatively, the user can press the 'login' button if they already have an account registered.

The page is styled with both Bootstrap and an external stylesheet. Additionally, jQuery is used and references the use of 'js/signup.js':

```
<!DOCTYPE html>
<html lang="en">

<head>
  <title>Passport Authentication</title>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootswatch/3.3.7/lumen/bootstrap.min.css">
  <link href="stylesheets/style.css" rel="stylesheet">
</head>

<body>
  <nav class="navbar navbar-default">
    <div class="container-fluid">
      <div class="navbar-header">
      </div>
    </div>
  </nav>
  <div class="container">
    <div class="row">
      <div class="col-md-6 col-md-offset-3">
        <h2>Sign Up Form</h2>
        <form class="signup">
          <div class="form-group">
            <label for="exampleInputEmail1">Email address</label>
            <input type="email" class="form-control" id="email-input" placeholder="Email">
          </div>
          <div class="form-group">
            <label for="exampleInputPassword1">Password</label>
            <input type="password" class="form-control" id="password-input" placeholder="Password">
          </div>
          <div style="display: none" id="alert" class="alert alert-danger" role="alert">
            <span class="glyphicon glyphicon-exclamation-sign" aria-hidden="true"></span>
            <span class="sr-only">Error:</span> <span class="msg"></span>
          </div>
          <button type="submit" class="btn btn-default">Sign Up</button>
        </form>
        <br />
        <p>Or log in <a href="/login">here</a></p>
      </div>
    </div>
  </div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.2.1/jquery.min.js"></script>
  <script type="text/javascript" src="js/signup.js"></script>

</body>

</html>

```

---

## js

### login.js

'login.js' is already notated but, in summary, the following actions occur:

1. jQuery waits until the DOM is loaded before continuing:

```
$(document).ready(function() {
```

2. Variables are assigned to easily target and refer to our form and inputs:

```
var loginForm = $("form.login");
var emailInput = $("input#email-input");
var passwordInput = $("input#password-input");
```

3. When the form is completed, the input is validated to check if there is an e-mail and password entered. If 'userData' is neither an email nor a password, nothing is returned:

```
loginForm.on("submit", function(event) {
    event.preventDefault();
    var userData = {
      email: emailInput.val().trim(),
      password: passwordInput.val().trim()
    };

    if (!userData.email || !userData.password) {
      return;
    }
```

4. If an e-mail and password are properly registered, the form is cleared out by the 'loginUser' function:

```
 loginUser(userData.email, userData.password);
    emailInput.val("");
    passwordInput.val("");
  });
```

5. 'loginUser' does a POST request to the '/api/login' route:

```
function loginUser(email, password) {
    $.post("/api/login", {
      email: email,
      password: password
    })
```

6. When the POST request is successful, the user is redirected to the '/members' page:

```
.then(function() {
        window.location.replace("/members");
```

7. If the POST request incurs an error, the error is logged:

```
 })
      .catch(function(err) {
        console.log(err);
      });
  }
});

```

---

### members.js

This short piece of code has two primary functions:

1. jQuery waits until the DOM has been loaded before continuing:

```
$(document).ready(function() {
```

2. As already notated in the file provided, the file will run a GET request to obtain which user is logged in and update that information onto the HTML:

```
$.get("/api/user_data").then(function(data) {
    $(".member-name").text(data.email);
  });
});
```

---

### signup.js

This section is also already notated however, in summary, the following actions occur:

1. jQuery waits until the DOM is loaded before continuing:

```
$(document).ready(function() {
```

2. Variables are assigned and give reference to our form and inputs:

```
var signUpForm = $("form.signup");
  var emailInput = $("input#email-input");
  var passwordInput = $("input#password-input");
```

3. When the form is submitted, the input is validated for an e-mail and password. If the input is not an e-mail nor a password, nothing is returned:

```
signUpForm.on("submit", function(event) {
    event.preventDefault();
    var userData = {
      email: emailInput.val().trim(),
      password: passwordInput.val().trim()
    };

    if (!userData.email || !userData.password) {
      return;
    }
```

4. Given a valid e-mail and password, 'signUpUser' is run and the form is cleared out:

```
 signUpUser(userData.email, userData.password);
    emailInput.val("");
    passwordInput.val("");
  });
```

5. 'signUpUser' function runs a POST request and POSTs to the '/api/signup' route:

```
function signUpUser(email, password) {
    $.post("/api/signup", {
      email: email,
      password: password
    })
```

6. When the POST request is successful, the user is redirected to the '/members' page:

```
 .then(function(data) {
        window.location.replace("/members");
```

7. When the POST request gives an error, a bootstrap alert is thrown:

```
     })
      .catch(handleLoginErr);
  }

  function handleLoginErr(err) {
    $("#alert .msg").text(err.responseJSON);
    $("#alert").fadeIn(500);
  }
});
```

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

This section is already notated however, in summary, the following actions occur:

1. The 'models' and 'passport' are required as configured:

```
var db = require("../models");
var passport = require("../config/passport");
```

2. Using the local strategy on 'passport.authenticate', the login credentials are checked for validity. If valid, the user will be redirected to the members page. If invalid, the user will be sent an error:

```
app.post("/api/login", passport.authenticate("local"), function(req, res) {
    res.json(req.user);
  });
```

3. The user's password is hashed and stored via the Sequelize User Model. If the 'User' is created successfully, the user is redirected to the 'login' page.

```
app.post("/api/signup", function(req, res) {
    db.User.create({
      email: req.body.email,
      password: req.body.password
    })
      .then(function() {
        res.redirect(307, "/api/login");
      })
```

4. If the 'User' is not created successfully, an error is returned:

```
  .catch(function(err) {
        res.status(401).json(err);
      });
  });
```

5. Route for logging the user out. Once singed out, the user is redirected to the home route:

```
app.get("/logout", function(req, res) {
    req.logout();
    res.redirect("/");
  });
```

6. Route for getting data about our user to be used client side. If the user is not logged in, an empty object is returned:

```
app.get("/api/user_data", function(req, res) {
    if (!req.user) {
      res.json({});
```

7. If the user is logged in, the user's e-mail and id are returned:

```
 } else {
   res.json({
        email: req.user.email,
        id: req.user.id
      });
    }
  });
};
```

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
