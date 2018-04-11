- V1- 	Grab starter files and npm install 
- v2-  	Setup Mongodb on mLab  Create a new db. in mLab and copy URL and set up 			variable.env Database and setup MongoDB compass
- V3- 	
- V4- 	define all routes in 'index.js' , require it in app.js . 
- V7- 	in controller folder create a storeController.js. import it in routes/index.js.
- V9- create models/Store.js. require mongoose, set Promise require slug.

​	create store Schema and export it 
	fields: name , slug  , description , tags[]
	import it to start.js
	slug will be auto generated for us so we need to define a pre function

```javascript
	storeSchma.pre('save' , function(next){
		if (!this.isModefied('name'){
			next() // skip it 
			return;
		})

		this.slug = slug(this.name)
		next();
	}); 
```

- V-10 create new route ('/add' , storecontroller.addStore) and export addStore

​	 render editStore pass {title : 'Add Store'}

```javascript
 define a mixin  => mixin storeForm(store = {}) use it +storeForm()
 in form filed add attr enctype="multipart/form-data"

 create a variable in pug file that contains an array [ ' wifi' , 'open late' , 'Faimly fiendly' , "Vegatagrian" , 'Lincensed']
 loop through them s

 create a post rout ('/add' , storecontroller.addStore) and export createStore)

 you can get all of your form data = req.body.
 	=> consts store = new Store(req.body) create a new store 
 	=> Store.save(); save data
```

V-11 require mongoose and store model to store controller
	 import errorhandler in storeController  with es6 { catchError}
	  wrap our post rout in catchError

	  res.redirect('/store')
V-12  we are using app.use(flash ) in app.js which gives us req.flash. we have a 			local variable flashes that is available to us in all templates
		we have block messagesin layout 
		it only works if we have sessions

V-13	create a controller getStore. set home and store route that getStore (async)
		 create stores.pug

		 query all stores => const stores = await Store.find();
	
		 pass stores to template
		 create a mixin storeCard.pug
		  in description only show 25 words (substring(0, 100))

V-14  	Add edit button that goest to that store id edit page.
		on click that edit button find that perticular store.
		render edit template

		in edit template fill name and description fields 
		for tags 
			create a const tag = store.tags || []
			loop each choice in choices 
			inside loop add input(name tags and checked = tags.includes(choice))
	
		set action `/add/${store._id|| ""}` 
	
		create add route post => 
		await findOneAndUpdate({_id: req.params.id} , req.body , {
			new : true ,
		 	runValidator: tru
		 }).exec()
		 flash them => redirect edit page
V-15	add created fields in schema and location 
		
		location :{
			type : {
				type: String,
				default: point
			},
	
			coordinate : 
			address {[{
			type: number,
			required : 'msg'
			}}]
		}
	
		add input for location name location[address]  value = (store.location && store.location.address)
		 add label and input for lng lat
		  	input name location[coordinates][0] add value same like location => lng
		  	input name location[coordinates][1] => lat

V-16  make autocomplete in js modules
		create autoComplete and exports pass ids 




V-17	remove all stores and add new ones so they all have location data.
		in updateStore controller => req.body.location.type = 'Point'

V-18 	uploads multer => helps uploads file
				jimp => resise the image

			add enctype='multipart/form-data'  in storeform.pug
		bring multer in storeController
			
			cont multerOption ={
				storage : multer.memoryStorage(),
				fileFilter = funtion(req , file , next){
					const isPhoto = file.mimetype.startsWith('/image');
	
					if(isPhoto) {
						next(null , true)
					} else {
						next({message : 'invalid file type'})
					}
				}
			}
	
			middleware => exports.upload = multer(multerOptionss).single('photo');
	
			add lable and input field in storeform
			if store.photo 
				img(src=`/uploads/${store.photo}` width = 200)
	
		bring in jimp  and uuid 
	
			exports.resise = async (req , res , next) => {
				if(!req.file) {
					next()
					return;
				} 
				const estension = req.file.mimetype.split('/')[1]
				req.body.photo = 	`${uuid.v4()}.${extension}`;
				const photo = await jimp.ready(req.file.buffer);
				await photo.resize(800 , jimp.AUTO);
				await phtot.write(`./public/uploads/${req.body.photo}`);
	
				next();
			}
	
			in post route for add and update stores add upload and resize 
	
			add photo : String in Schema

V-19	set route for getStoreBySlug => query store
		findOne({sulg : req.params.slug})
		if no stere retuen next  => render store pass store and title

V-20	in store schems pre=>save copy whole function from solution

V-21	set up two tags route getStoreByTag one tag and other tage/:tag

		get all tags create getTaglist in store Schema  
			render tag file

V-22 	promisAll


V-23	create login controller => UserController create login pug
		ceate mixin loginForm pug 
		create a user model => bring in mongoose,md5, validator, mongodbHAndller ,passportlocalMongoose

		set Refister Route 
		add post route register => 
	
			valideRegistet middleware in userController
					req.sanitizeBody('name')
					req.checkBody('name ' , 'message')
					req.checkBody('email ' , 'message')
					===>>>  see docs
- V-24
  - in register route bring in user and promisify
  - create new user  req.body.emai req.body.name
  - create authController bring passport
  - configure the local statugy
  - create passport.js in handdlers brin passport mongoose user
  - in app.js require this file
- V-25 	Logout routes 

​		router post login
		=>set up gravatar 	
			userSchema.virtual('gravatar').get(function()){
			}
		create middleware isLoggedin

- v-26	account route => account pug 

​		account post (isloggedin) 

- V-27  forgot password 

  create fowling files

  ​	`mixin/forgot.pug`

  - create forgot route in authCotroller
    - Route`/account/forgot`
    - ​

  ​