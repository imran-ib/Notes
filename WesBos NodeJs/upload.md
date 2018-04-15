# Upload 

To be able to upload files we need need to add enctype attribute `enctype="multipart/form-data"`.

#### Modify the form to take photo

```jade
 form(action=`/add/${store._id || ''}` method="POST" class="card" enctype="multipart/form-data")
//- Image Upload
    label(for="photo") Photo
      input(type="file" name="photo" id="photo" accept="image/gif, image/png, image/jpeg")
      if store.photo
        img(src=`/uploads/${store.photo}`, alt=store.name width=200)
```



we need to add some middleware for upload the file and resize the file. For that we are going to use `multer` and `jimp`. `multer` will help us to upload and `jimp`will help resize the file/photo.

### Multer Options 

- require `multer`, `uuid`and `jimp`in controller file.

- set a variable `multerOption`

  - ```javascript
    const multerOptions = {
      storage: multer.memoryStorage(),
      fileFilter(req, file, next) {
        const isPhoto = file.mimetype.startsWith('image/');
        if(isPhoto) {
          next(null, true);
        } else {
          next({ message: 'That filetype isn\'t allowed!' }, false);
        }
      }
    };
    ```



### Upload

```javascript
exports.upload = multer(multerOptions).single('photo');
```



## Resize

```javascript
exports.resize = async (req, res, next) => {
  // check if there is no new file to resize
  if (!req.file) {
    next(); // skip to the next middleware
    return;
  }
  const extension = req.file.mimetype.split('/')[1];
  req.body.photo = `${uuid.v4()}.${extension}`;
  // now we resize
  const photo = await jimp.read(req.file.buffer);
  await photo.resize(800, jimp.AUTO);
  await photo.write(`./public/uploads/${req.body.photo}`);
  // once we have written the photo to our filesystem, keep going!
  next();
};
```

Before adding new store and editing new store we need to pass upload and resize middleware to routes

```javascript
router.post('/add',
  storeController.upload,
  catchErrors(storeController.resize),
  catchErrors(storeController.createStore)
);

router.post('/add/:id',
  storeController.upload,
  catchErrors(storeController.resize),
  catchErrors(storeController.updateStore)
);
```

Don't forget to setup your Schema for photo `photo :String`

