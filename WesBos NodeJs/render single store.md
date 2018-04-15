# Single Store 

video 19

- find the store that has a slug 
- Display name , photo, description , map , tags , 
- Show user login button if they are not logged in 
- show list of all the reviews 

Setup the route 

```javascript
router.get('/store/:slug', catchErrors(storeController.getStoreBySlug));
```

```javascript
exports.getStoreBySlug = async (req, res, next) => {
  const store = await Store.findOne({ slug: req.params.slug }).populate('author reviews');
  if (!store) return next();
  res.render('store', { store, title: store.name });
};
```

Create template 	`store.pug` and setup image , title with link

```jade
 img.single__image(src=`/uploads/${store.photo || 'store.png'}`)
      h2.title.title--single
        a(href=`/stores/${store.slug}`) #{store.name}
```

Now the Google Map Image. 

in helpers.js we have a helper function `staticMap` 

```javascript
exports.staticMap = ([lng, lat]) => `https://maps.googleapis.com/maps/api/staticmap?center=${lat},${lng}&zoom=14&size=800x150&key=${process.env.MAP_KEY}&markers=${lat},${lng}&scale=2`;
```

in template 

```jade
img.single__map(src=h.staticMap(store.location.coordinates))
    p.single__location= store.location.address
    p= store.description
```

Display tags 

```jade
  if store.tags
      ul.tags
        each tag in store.tags
          li.tag
            a.tag__link(href=`/tags/${tag}`)
              span.tag__text  ##{tag}
```

