# User Log in

video 23 to 29

create a new file in controller `userController`and being in `mongoose`

create login route

```javascript
router.get('/login', userController.loginForm);

exports.loginForm = (req, res) => {
  res.render('login', { title: 'Login' });
};

```

Template 

```jade
mixin loginForm()
  form.form(action="/login" method="POST")
    h2 Login
    label(for="email") Email Address
    input(type="email" name="email")
    label(for="password") Password
    input(type="password" name="password")
    input.button(type="submit" value="Log In →")


//after creating mixin bring it in login.pug

extends layout

include mixins/_loginForm
include mixins/_forgot

block content
  .inner
    +loginForm()
    +forgotForm()

```

# Model

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
mongoose.Promise = global.Promise;
const md5 = require('md5');
const validator = require('validator');
const mongodbErrorHandler = require('mongoose-mongodb-errors');
const passportLocalMongoose = require('passport-local-mongoose');

const userSchema = new Schema({
  email: {
    type: String,
    unique: true,
    lowercase: true,
    trim: true,
    validate: [validator.isEmail, 'Invalid Email Address'],
    required: 'Please Supply an email address'
  },
  name: {
    type: String,
    required: 'Please supply a name',
    trim: true
  },
  resetPasswordToken: String,
  resetPasswordExpires: Date,
  hearts: [
    { type: mongoose.Schema.ObjectId, ref: 'Store' }
  ]
});

userSchema.virtual('gravatar').get(function() {
  const hash = md5(this.email);
  return `https://gravatar.com/avatar/${hash}?s=200`;
});

userSchema.plugin(passportLocalMongoose, { usernameField: 'email' });
userSchema.plugin(mongodbErrorHandler);

module.exports = mongoose.model('User', userSchema);


// don't forget to bring in user model in start.js
```



# Register

```javascript
router.get('/register', userController.registerForm);



exports.registerForm = (req, res) => {
  res.render('register', { title: 'Register' });
};
```

### Template

create register template

```jade
extends layout

block content
  .inner
    form.form(action="/register" method="POST")
      h2 Register
      label(for="name") Name
      input(type="text" name="name")
      label(for="email") Email
      input(type="email" name="email")
      label(for="password") Password
      input(type="password" name="password")
      label(for="password-confirm") Confirm Password
      input(type="password" name="password-confirm")
      input.button(type="submit" value="Register →")

```

Before Posting Register Form we need to run some validations so we need to create some middleware

1. Validate the registration data

```javascript
exports.validateRegister = (req, res, next) => {
  req.sanitizeBody('name'); 	// expressValidator has access to sanitizeBody
  req.checkBody('name', 'You must supply a name!').notEmpty();
  req.checkBody('email', 'That Email is not valid!').isEmail();
  req.sanitizeBody('email').normalizeEmail({
    remove_dots: false,
    remove_extension: false,
    gmail_remove_subaddress: false
  });
  req.checkBody('password', 'Password Cannot be Blank!').notEmpty();
  req.checkBody('password-confirm', 'Confirmed Password cannot be blank!').notEmpty();
  req.checkBody('password-confirm', 'Oops! Your passwords do not match').equals(req.body.password);

  const errors = req.validationErrors();
  if (errors) {
    req.flash('error', errors.map(err => err.msg));
    res.render('register', { title: 'Register', body: req.body, flashes: req.flash() });
    return; // stop the fn from running
  }
  next(); // there were no errors!
};

//Visit express validator for more information
```

- register the user

```javascript
// Don't forget to bring in user model UserController

exports.register = async (req, res, next) => {
  const user = new User({ email: req.body.email, name: req.body.name });
  const register = promisify(User.register, User);
  await register(user, req.body.password);  
  next(); // pass to authController.login
};

```

Create new Controller `authController` bring in `passport`

create a middle ware `authenticate`

```javascript
exports.login = passport.authenticate('local', {
  failureRedirect: '/login',
  failureFlash: 'Failed Login!',
  successRedirect: '/',
  successFlash: 'You are now logged in!'
});
```

- we need to log them in

```javascript

router.post('/register',
  userController.validateRegister,
  userController.register,
  authController.login
);
```

## Passport Handler

in handler directory create a file `passport`

```javascript
const passport = require('passport');
const mongoose = require('mongoose');
const User = mongoose.model('User');

passport.use(User.createStrategy());

passport.serializeUser(User.serializeUser());
passport.deserializeUser(User.deserializeUser());

```

import this file in app.js 

``` javascript
require('./handlers/passport');
```

## Log Out

``` javascript
router.get('/logout', authController.logout);

exports.logout = (req, res) => {
  req.logout();
  req.flash('success', 'You are now logged out! 👋');
  res.redirect('/');
};

```



Now create login post route

```javascript
router.post('/login', authController.login);
```



# Avatar

in Schema 

``` javascript
userSchema.virtual('gravatar').get(function() {
  const hash = md5(this.email);
  return `https://gravatar.com/avatar/${hash}?s=200`;
});

// if you copied schema this method already inculded
```

 

you should not be able to add store if you are not logged in so create a middleware in authController

```javascript
exports.isLoggedIn = (req, res, next) => {
  // first check if the user is authenticated
  if (req.isAuthenticated()) {
    next(); // carry on! They are logged in!
    return;
  }
  req.flash('error', 'Oops you must be logged in to do that!');
  res.redirect('/login');
};
```

in index.js route file right before add store add this route

```javascript
router.get('/add', authController.isLoggedIn, storeController.addStore);
```

# User Account

if you click on avatar it will take you to `/account`

```javascript
router.get('/account', authController.isLoggedIn, userController.account);

exports.account = (req, res) => {
  res.render('account', { title: 'Edit Your Account' });
};
```

### Template

```jade
extends layout

block content
  .inner
    h2 Edit Your Account
    form(action="/account" method="POST")
      label(for="name") Name
      input(type="text" name="name" value=user.name)
      label(for="email") Email Address
      input(type="email" name="email" value=user.email)
      input.button(type="submit" value="Update My Account")
      
// user is available to us in every template because we have local set up for that
```

Post Route

```javascript
router.post('/account', catchErrors(userController.updateAccount));

exports.updateAccount = async (req, res) => {
  const updates = {
    name: req.body.name,
    email: req.body.email
  };

  const user = await User.findOneAndUpdate(
    { _id: req.user._id },
    { $set: updates },
    { new: true, runValidators: true, context: 'query' }
  );
  req.flash('success', 'Updated the profile!');
  res.redirect('back');
};


```

# Password Reset

create mixin forgto.pug

```jade
mixin forgotForm()
  form.form(action="/account/forgot" method="POST")
    h2 I forgot my password!
    label(for="email") Email
    input(type="email" name="email")
    input.button(type="submit" value="Send a Reset")


// bring this mixin in to login ( +forgotForm())
```

```javascript
router.post('/account/forgot', catchErrors(authController.forgot));

exports.forgot = async (req, res) => {
  // 1. See if a user with that email exists
  const user = await User.findOne({ email: req.body.email }); // bring in model
  if (!user) {
    req.flash('error', 'No account with that email exists.');
    return res.redirect('/login');
  }
  // 2. Set reset tokens and expiry on their account
  user.resetPasswordToken = crypto.randomBytes(20).toString('hex'); 
   // don't forget to import crypto &in model setup 'resetPasswordToken', 'resetPasswordExpires'
  user.resetPasswordExpires = Date.now() + 3600000; // 1 hour from now
  await user.save();
  // 3. Send them an email with the token
  const resetURL = `http://${req.headers.host}/account/reset/${user.resetPasswordToken}`;
  await mail.send({
    user,
    filename: 'password-reset',
    subject: 'Password Reset',
    resetURL
  }); // require mail
  req.flash('success', `You have been emailed a password reset link.`);
  // 4. redirect to login page
  res.redirect('/login');
};
```

Now Set up the route for token link

```javascript
router.get('/account/reset/:token', catchErrors(authController.reset));


exports.reset = async (req, res) => {
  const user = await User.findOne({
    resetPasswordToken: req.params.token,
    resetPasswordExpires: { $gt: Date.now() }
  });
  if (!user) {
    req.flash('error', 'Password reset is invalid or has expired');
    return res.redirect('/login');
  }
  // if there is a user, show the rest password form
  res.render('reset', { title: 'Reset your Password' });
};
```

create template reset

```jade
extends layout

block content
  .inner
    form.form(method="POST")
      h2 Reset Your Password
      label(for="password") Password
      input(type="password" name="password")
      label(for="password-confirm") Confirm Password
      input(type="password" name="password-confirm")
      input(type="submit" value="Reset Password →")

```

Now Update 

```javascript
router.post('/account/reset/:token',
  authController.confirmedPasswords,
  catchErrors(authController.update)
);


exports.confirmedPasswords = (req, res, next) => {
  if (req.body.password === req.body['password-confirm']) {
    next(); // keepit going!
    return;
  }
  req.flash('error', 'Passwords do not match!');
  res.redirect('back');
};


exports.update = async (req, res) => {
  const user = await User.findOne({
    resetPasswordToken: req.params.token,
    resetPasswordExpires: { $gt: Date.now() }
  });

  if (!user) {
    req.flash('error', 'Password reset is invalid or has expired');
    return res.redirect('/login');
  }

  const setPassword = promisify(user.setPassword, user);
  await setPassword(req.body.password);
  user.resetPasswordToken = undefined;
  user.resetPasswordExpires = undefined;
  const updatedUser = await user.save();
  await req.login(updatedUser); // this is will log them in 
  req.flash('success', '💃 Nice! Your password has been reset! You are now logged in!');
  res.redirect('/');
};

```

Send reset link via email

[sign up to](https://mailtrap.io/) mailtrap

once singed up you will have access to SMTP credentials add to variable.env file

Now create new handler `mail.js` bring in `nodemailer` `pug` `juice` `htmlToText`  `promisify`

set up transporter

``` javascript
const nodemailer = require('nodemailer');
const pug = require('pug');
const juice = require('juice');
const htmlToText = require('html-to-text');
const promisify = require('es6-promisify');

const transport = nodemailer.createTransport({
  host: process.env.MAIL_HOST,
  port: process.env.MAIL_PORT,
  auth: {
    user: process.env.MAIL_USER,
    pass: process.env.MAIL_PASS
  }
});

const generateHTML = (filename, options = {}) => {
  const html = pug.renderFile(`${__dirname}/../views/email/${filename}.pug`, options);
  const inlined = juice(html);
  return inlined;
};

// whenever someone reset their password we nee modile 'send'
exports.send = async (options) => {
  const html = generateHTML(options.filename, options);
  const text = htmlToText.fromString(html);

  const mailOptions = {
    from: `Imran Irshad <noreply@iib.com>`,
    to: options.user.email,
    subject: options.subject,
    html,
    text
  };
  const sendMail = promisify(transport.sendMail, transport);
  return sendMail(mailOptions);
};


```



# Relationship 

user and store relationship 

```javascript
// in Store model

author: {
    type: mongoose.Schema.ObjectId,
    ref: 'User',
    required: 'You must supply an author'
  }
}, {
  toJSON: { virtuals: true },
  toObject: { virtuals: true },
});
```

After add author field to model we need to setup `createStore` route

we need to add `req.body.author = req.user._id;`

so the Whole route will look like this

````javascript
exports.createStore = async (req, res) => {
  req.body.author = req.user._id;
  const store = await (new Store(req.body)).save();
  req.flash('success', `Successfully Created ${store.name}. Care to leave a review?`);
  res.redirect(`/store/${store.slug}`);
};
````

Now we can use 	`populate` to use that data.

go to `getStoreBySlug`

`.populate('author reviews');`

Whole Route 

```javascript
exports.getStoreBySlug = async (req, res, next) => {
  const store = await Store.findOne({ slug: req.params.slug }).populate('author reviews');
  if (!store) return next();
  res.render('store', { store, title: store.name });
};
```

Stop People From editing store that they do not own

create new function

```javascript
const confirmOwner = (store, user) => {
  if (!store.author.equals(user._id)) {
    throw Error('You must own a store in order to edit it!');
  }
};
```

pass that function `editStore`

```javascript
exports.editStore = async (req, res) => {
  // 1. Find the store given the ID
  const store = await Store.findOne({ _id: req.params.id });
  // 2. confirm they are the owner of the store
  confirmOwner(store, req.user);
  // 3. Render out the edit form so the user can update their store
  res.render('editStore', { title: `Edit ${store.name}`, store });
};
```

Note : before testing make sure all store have author (new store)

you should not see the edit button if you are not logged in 

```jade
if user && store.author.equals(user._id)
          .store__action.store__action--edit
            a(href=`/stores/${store._id}/edit`)
              != h.icon('pencil')
```