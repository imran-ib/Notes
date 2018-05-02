we can pull in data from u`url` with `req.query.name` we can use `req.query` to get all data

whenever express hit the route it will check if there is any form data input by user and then body parser will parse that data in  request object . And we can access that data by using`req.query` or `req.body` 

or `req.params ` => that will have all the variables defined in url

Response has on the other hand has all the data sending back.

 

in short `req.query` to get query parameters 

`req.body` => we will use for posted parameters

`req.params`to access variables from url 



For more you must go to express docs



 Name in form name filed must line up with your Schema

set Up the form mixin and add store

Add these methods

```javascript
router.get('/', storeController.homePage);
router.get('/add', storeController.addStore);
router.post('/add', storeController.createStore);




exports.homePage = (req, res) => {
  console.log(req.name);
  res.render('index');
};

exports.addStore = (req, res) => {
  res.render('editStore', { title: 'Add Store' });
};

exports.createStore = (req, res) => {
      req.flash('success', `Successfully Created ${store.name}. Care to leave a review?`);
      //res.redirect(`/store/${store.slug}`);
  res.json(req.body);
};

//You should get josn data name description and tags 

exports.getStores = async (req, res) => {
  // 1. Query the database for a list of all stores
  const stores = await Store.find();
  res.render('stores', { title: 'Stores', stores });
};

```

## One Thing to keep in mind that javascript will not wait for database to respond it will keep executing. That's we use async methods like promises or callbacks or async await.  



Whenever we are interacting with database we need to use async await



# Edit & Update

Our Edit must go to `/stores/:id:/edit



 ```javascript
router.get('/stores/:id/edit', catchErrors(storeController.editStore));


exports.editStore = async (req, res) => {
  // 1. Find the store given the ID
  const store = await Store.findOne({ _id: req.params.id });
  // 2. confirm they are the owner of the store
  // TODO
  // 3. Render out the edit form so the user can update their store
  res.render('editStore', { title: `Edit ${store.name}`, store });
};


router.post('/add/:id', catchErrors(storeController.updateStore));


exports.updateStore = async (req, res) => {
  // find and update the store
  const store = await Store.findOneAndUpdate({ _id: req.params.id }, req.body, {
    new: true, // return the new store instead of the old one
    runValidators: true
  }).exec();
  req.flash('success', `Successfully updated <strong>${store.name}</strong>. <a href="/stores/${store.slug}">View Store →</a>`);
  res.redirect(`/stores/${store._id}/edit`);
  // Redriect them the store and tell them it worked
};
 ```



After setting up edit route dump the data to see if you are editing right store

in edit mixin we can pass store to store form 

in store form at the top we have store ={} . if there is store it will populate that store otherwise it will fall back empty 

we can use that to fill input fields for edit route

in form tag we can you `action=/add/:id || ""`



# Order

```javascript
const express = require('express');
const router = express.Router();
const storeController = require('../controllers/storeController');
const { catchErrors } = require('../handlers/errorHandlers');

router.get('/', catchErrors(storeController.getStores));
router.get('/stores', catchErrors(storeController.getStores));
router.get('/add', storeController.addStore);
router.post('/add', catchErrors(storeController.createStore));
router.post('/add/:id', catchErrors(storeController.updateStore));
router.get('/stores/:id/edit', catchErrors(storeController.editStore));

module.exports = router;

```

