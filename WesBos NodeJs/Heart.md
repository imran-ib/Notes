# Hearts

Modify User Schema

```javascript
hearts: [
    { type: mongoose.Schema.ObjectId, ref: 'Store' }
  ]
```

Template or Interface  `storForm`

```jade
if user
          .store__action.store__action--heart
            form.heart(method="POST" action=`/api/stores/${store._id}/heart`)
              button.heart__button(type="submit" name="heart" class=heartClass)
                != h.icon('heart')
```

Set up Route

```javascript
router.post('/api/stores/:id/heart', catchErrors(storeController.heartStore));


exports.heartStore = async (req, res) => {
  const hearts = req.user.hearts.map(obj => obj.toString());

  const operator = hearts.includes(req.params.id) ? '$pull' : '$addToSet';
  const user = await User  // import User
    .findByIdAndUpdate(req.user._id,
      { [operator]: { hearts: req.params.id } }, //ES6 
      { new: true }
    );
  res.json(user);
};
```

The [`$pull`](https://docs.mongodb.com/manual/reference/operator/update/pull/#up._S_pull) operator removes from an existing array all instances of a value or values that match a specified condition

The [`$addToSet`](https://docs.mongodb.com/manual/reference/operator/update/addToSet/#up._S_addToSet) operator adds a value to an array unless the value is already present, in which case [`$addToSet`](https://docs.mongodb.com/manual/reference/operator/update/addToSet/#up._S_addToSet) does nothing to that array.

At this point we should be able to add and removes hearts to MongoDB. when we click on heart on store icon it increases the number the number by one and display total numbers of hearts that is because it setup in `layout.pug`

```jade
 if user
              li.nav__item: a.nav__link(href="/hearts", class=(currentPath.startsWith('/hearts') ? 'nav__link--active' : ''))
                != h.icon('heart')
                span.heart-count #{user.hearts && user.hearts.length}
              li.nav__item: a.nav__link(href="/logout", class=(currentPath.startsWith('/logout') ? 'nav__link--active' : ''))
```

 We need to get list of heart string in template  `storeCard`

```javascript
 - const heartStrings = user.hearts.map(obj => obj.toString()) // list of hearats
 - const heartClass = heartStrings.includes(store._id.toString()) ? 'heart__button--hearted' : ''
// add remove heart
```

Complete template code for heart

```javascript
if user
          .store__action.store__action--heart
            form.heart(method="POST" action=`/api/stores/${store._id}/heart`)
              - const heartStrings = user.hearts.map(obj => obj.toString())
              - const heartClass = heartStrings.includes(store._id.toString()) ? 'heart__button--hearted' : ''
              button.heart__button(type="submit" name="heart" class=heartClass)
                != h.icon('heart')
```



Now make it happen real time so user don't have to refresh the page to see the changes. 

make new javascript file in `modules/heart.js`

```javascript
import axios from 'axios';
import { $ } from './bling';

function ajaxHeart(e) {
}

export default ajaxHeart;
```

Import it main file

```javascript
import ajaxHeart from './modules/heart';

const heartForms = $$('form.heart'); // any form that has class of heart
heartForms.on('submit', ajaxHeart);
```



```javascript
function ajaxHeart(e) {
  e.preventDefault(); // stop from submiting itself
  console.log('HEART ITTT!!!!!!!!!!!!!!!!');
  console.log(this);
  axios
    .post(this.action)
    .then(res => {
      const isHearted = this.heart.classList.toggle('heart__button--hearted');
      $('.heart-count').textContent = res.data.hearts.length;
      if (isHearted) {
        this.heart.classList.add('heart__button--float');
        setTimeout(() => this.heart.classList.remove('heart__button--float'), 2500);
      }
    })
    .catch(console.error);
}
```

Displaying heart page

Route 

```javascript
router.get('/hearts', authController.isLoggedIn, catchErrors(storeController.getHearts));

exports.getHearts = async (req, res) => {
  const stores = await Store.find({
    _id: { $in: req.user.hearts }
  });
  res.render('stores', { title: 'Hearted Stores', stores });
};


// we already have stores.pug 
```

`$in` Returns a Boolean indicating whether a specified value is in an array.

