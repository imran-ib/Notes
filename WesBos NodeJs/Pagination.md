# Pagination

in order to apply pagination we need to modify our `getStore` route to instead of finding all stores 

find only number of stores

but doing that we need to first create a route 

```javascript
router.get('/stores/page/:page', catchErrors(storeController.getStores));
```



```javascript
exports.getStores = async (req, res) => {
  const page = req.params.page || 1;
  const limit = 4;
  const skip = (page * limit) - limit;

  // 1. Query the database for a list of all stores
  const storesPromise = Store
    .find()
    .skip(skip)
    .limit(limit)
    .sort({ created: 'desc' });

  const countPromise = Store.count();

  const [stores, count] = await Promise.all([storesPromise, countPromise]);
  const pages = Math.ceil(count / limit);
  if (!stores.length && skip) {
    req.flash('info', `Hey! You asked for page ${page}. But that doesn't exist. So I put you on page ${pages}`);
    res.redirect(`/stores/page/${pages}`);
    return;
  }

  res.render('stores', { title: 'Stores', stores, page, pages, count });
};
```



template 

mixin pagination

```jade
mixin pagination(page, pages, count)
  .pagination
    .pagination__prev
      if page > 1
        a(href=`/stores/page/${page - 1}`) Prev
    .pagination__text
      p Page #{page} of #{pages} — #{count} total results
    .pagination__next
      if page < pages
        a(href=`/stores/page/${parseFloat(page) + 1}`) Next

```



in `store.pug`

```jade
extends layout

include mixins/_storeCard
include mixins/_pagination

block content
  .inner
    h2= title
    .stores
      each store in stores
        +storeCard(store)
    +pagination(page, pages, count)

```

