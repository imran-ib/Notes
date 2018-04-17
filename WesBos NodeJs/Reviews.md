# Reviews

Create New model called `Review.js`

```javascript
const mongoose = require('mongoose');
mongoose.Promise = global.Promise;

const reviewSchema = new mongoose.Schema({
  created: {
    type: Date,
    default: Date.now
  },
  author: {
    type: mongoose.Schema.ObjectId,
    ref: 'User',
    required: 'You must supply an author!'
  },
  store: {
    type: mongoose.Schema.ObjectId,
    ref: 'Store',
    required: 'You must supply a store!'
  },
  text: {
    type: String,
    required: 'Your review must have text!'
  },
  rating: {
    type: Number,
    min: 1,
    max: 5
  }
});

function autopopulate(next) {
  this.populate('author');
  next();
}

reviewSchema.pre('find', autopopulate);
reviewSchema.pre('findOne', autopopulate);

module.exports = mongoose.model('Review', reviewSchema);

```

In `start.js`import it  `require('./models/Review');`

create a mixin  `reviewForm`

```jade
mixin reviewForm(store)
  form.reviewer(action=`/reviews/${store._id}` method="POST")
    textarea(name="text" placeholder="Did you try this place? Have something to say? Leave a review...")
    .reviewer__meta
      .reviewer__stars
        each num in [5,4,3,2,1]
          input(type="radio" required id=`star${num}` name="rating" value=num)
          label(for=`star${num}`) #{num} Stars
      input.button(type="submit" value="Submit Review →")


```

In 	`store.pug` ring in `reviewFrom`

```jade
include mixins/_reviewForm
include mixins/_review


if user
      +reviewForm(store)

    if store.reviews
      .reviews
        each review in store.reviews
          .review
            +review(review)
```

Create Route  new file `reviewController.js`

```javascript
router.post('/reviews/:id',
  authController.isLoggedIn,
  catchErrors(reviewController.addReview)
);


// new controller file

const mongoose = require('mongoose');
const Review = mongoose.model('Review');

exports.addReview = async (req, res) => {
  req.body.author = req.user._id;
  req.body.store = req.params.id;
  const newReview = new Review(req.body);
  await newReview.save();
  req.flash('success', 'Revivew Saved!');
  res.redirect('back');
};


```

Now in `Store.js` model create a virtual field 

```javascript
// find reviews where the stores _id property === reviews store property
storeSchema.virtual('reviews', {
  ref: 'Review', // what model to link?
  localField: '_id', // which field on the store?
  foreignField: 'store' // which field on the review?
});
```

Now populate this `storeControler.js` to `getStoreBySlug`

```javascript
exports.getStoreBySlug = async (req, res, next) => {
  const store = await Store.findOne({ slug: req.params.slug }).populate('author reviews');
  if (!store) return next();
  res.render('store', { store, title: store.name });
};
```

Reviews properties lives on `store.reviews` they are not saved in database .

we need to set up our `store.js` model so it understands this 

```javascript
// right before closing paranthesy 
{
  toJSON: { virtuals: true },
  toObject: { virtuals: true },
}
```

in template

```jade
 if store.reviews
      .reviews
        each review in store.reviews
          .review
            +review(review)
```

make a new review mixin

```jade
mixin review(review)
  .review__header
    .review__author
      img.avatar(src=review.author.gravatar)
      p= review.author.name
    .review__stars(title=`Rated ${review.rating} our of 5 stars`)
      = `★`.repeat(review.rating)
      = `☆`.repeat(5 - review.rating)
    time.review__time(datetime=review.created)= h.moment(review.created).fromNow()
  .review__body
    p= review.text

```

to auto populate our author in model review 

```javascript
function autopopulate(next) {
  this.populate('author');
  next();
}

reviewSchema.pre('find', autopopulate);
reviewSchema.pre('findOne', autopopulate);
```

# Top Page

Add Route Top

```javascript
router.get('/top', catchErrors(storeController.getTopStores));


exports.getTopStores = async (req, res) => {
  const stores = await Store.getTopStores();
  res.render('topStores', { stores, title:'⭐ Top Stores!'});
}
```

Whenever you have a complex query it is best to do it model instead of doing it controller

```javascript
storeSchema.statics.getTopStores = function() {
  return this.aggregate([
    // Lookup Stores and populate their reviews
    { $lookup: { from: 'reviews', localField: '_id', foreignField: 'store', as: 'reviews' }},
    // filter for only items that have 2 or more reviews
    { $match: { 'reviews.1': { $exists: true } } },
    // Add the average reviews field
    { $project: {
      photo: '$$ROOT.photo',
      name: '$$ROOT.name',
      reviews: '$$ROOT.reviews',
      slug: '$$ROOT.slug',
      averageRating: { $avg: '$reviews.rating' }
    } },
    // sort it by our new field, highest reviews first
    { $sort: { averageRating: -1 }},
    // limit to at most 10
    { $limit: 10 }
  ]);
}

```

remove all stores and add them from data



Template

```jade
extends layout

block content
  .inner
    h2 Top #{stores.length} Stores
    table.table
      thead
        td photo
        td ranking
        td name
        td reviews
        td Average Rating
      each store, i in stores
        tr
          td
            a(href=`/store/${store.slug}`)
              img(width=200 src=`/uploads/${store.photo || 'store.png'}` alt=store.name)
          td #{i + 1}
          td: a(href=`/store/${store.slug}`)= store.name
          td= store.reviews.length
          td #{Math.round(store.averageRating * 10) / 10} / 5

```

if you dump the store each store should not have reviews to fix that 

go to `storeController.js` and find `getStores` function  where you are finding all stores populate it with reviews

```javascript
const stores = await Store.find().populate('reviews')
```

but we should not it in here 

do it in `autopopulate` function 

Store.js in model 

```javascript
function autopopulate(next) {
  this.populate('reviews');
  next();
}

storeSchema.pre('find', autopopulate);
storeSchema.pre('findOne', autopopulate);
```



add icon in `storeCard`

```jade
 if store.reviews
          .store__action.store__action--count
            != h.icon('review')
            span= store.reviews.length
```

