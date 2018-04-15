# Tags

video 21 22

When we complex query we use aggregation. Aggregate is just a method like `findOne ` or `findByID`

First we need to set our Schema  and create our own static method . Its important to proper function here because we need to use `this` inside this function

```javascript
storeSchema.statics.getTagsList = function() {
  return this.aggregate([
    { $unwind: '$tags' },
    { $group: { _id: '$tags', count: { $sum: 1 } } },
    { $sort: { count: -1 } }
  ]);
};
```

`aggregate` takes an array of operator that we are looking for. You can google `mogodb opratoe`



Setup Route

```javascript
router.get('/tags', catchErrors(storeController.getStoresByTag));
router.get('/tags/:tag', catchErrors(storeController.getStoresByTag));


exports.getStoresByTag = async (req, res) => {
  const tag = req.params.tag;
  const tagQuery = tag || { $exists: true };

  const tagsPromise = Store.getTagsList();
  const storesPromise = Store.find({ tags: tagQuery });
  const [tags, stores] = await Promise.all([tagsPromise, storesPromise]);


  res.render('tag', { tags, title: 'Tags', tag, stores });
};

```

 Create template

```jade
extends layout

include mixins/_storeCard

block content
  .inner
    h2 #{tag || 'Tags'}
    ul.tags
      each t in tags
        li.tag
          a.tag__link(href=`/tags/${t._id}` class=(t._id === tag ? 'tag__link--active' : ''))
            span.tag__text= t._id
            span.tag__count= t.count
    .stores
      each store in stores
        +storeCard(store)
```

