# Mongo DB

  Important `command`
```mongo db
mongod 	| starts the mongo demon

mongo  	| opens mongo shell

help   	| gives some usefull features

show db | will show all the databases availabe 

use  	| use.nameOfDb |will switch between dbs. it will create if not found

Insert 	| db.name.insert ({prop1 : "prop1" , prop2 :"prop2"}) | inserts contnt 			 to dara base

find  	| db.name.find({})

update 	| db.name.update({prop1 : "prop1" , prop2 :"prop2"}) | this will 			  overwrite the dab


			 db.name.update({}, {$set:{prop1 : "prop1" , prop2 :"prop2"}})  this will insert content without overwrting
	
	remove  | db.name.remove 

# Mongoose

~~~javascript
   * npm install and require it
   * then connect mongodb server with server with the flowing command. if the database does not 	exist it will create it.
   * mongoose.connect("mongodb://localhost/nameOf Db")
   * Create a Schema and save it to variavle.
```
		var nameSchema = new mongoose.Schema({
			name: String.
			age : Number,
			MarriedStatus: Boolean
			});
			```
	* compile the Schema to a model and save it variable
	
			```var Name = mongoose.model("Name" , nameSchema)```
	* Now we can use all our mongo properties to our Name (model) Variable
	* we can create a new databse by using the keyword "new" folwed by name(model variable) and 		pass it an object like so.
	```
			var Imran = new Name ({
				name : "Imran",
				age  : 30,
				MarriedStatus: true
	
			})
```
	and then we can save it to database by using

			Imran.save();

	we can also pass it a anonymus function
```
			Imran.save(function(err , name){
				if(err){
					console.log(err)
				}else {
					console.log("Succes")
				}
			})
```
	we use create() to whch will create and save data to database with one step
```
			Name.creat({
				name : "Imran",
				age  : 30,
				MarriedStatus: true
			} , (err) => {if(err){console.log(err)}})
```
~~~

		we can retrive data from database by using find method on the model name.
		the find will take parameters first is an empty object and other is callback function

~~~javascript
			```Name.find({} , function(err , res){
				if(err){console.log(err)}
				else{console.lof(res)}
			})```
~~~



~~~javascript
Update Increament => $inc 
	this operator will increament by one. below is the exapmle of its use
```
		User.update({name : 'imran'},{$inc : {posts : 1}});
```
Mongoose Validator:
Validation is defined in the SchemaType
	Validation is middleware. Mongoose registers validation as a pre('save') hook on every schema by default.
	You can manually run validation using ```doc.validate(callback) or doc.validateSync()```
	Validators are not run on undefined values. The only exception is the required validator.
	Validation is asynchronously recursive; when you call Model save, sub-document validation is executed as well. If an error occurs, your Model save callback receives it
	Validation is customizable

	require the field in Schema 
	we can use ```validateSync()``` on model
	we can access the error message by
		```User.validateSync().error.name.message```

	in our Schema 
```
		name :{
			validator => (name) name.lenght > 2
			message : 'Number id greater then 2'
		}
```
		OR Another complecated exapmle would be
```
			var userSchema = new Schema({
		      phone: {
		        type: String,
		        validate: {
		          validator: function(v) {
		            return /\d{3}-\d{3}-\d{4}/.test(v);
		          },
		          message: '{VALUE} is not a valid phone number!'
		        },
		        required: [true, 'User phone number required']
		      }
		    });
```javascript
		 we use validateSync message error to users
~~~

Nesting Posts: => (subdocuments) => (Data Association)
		
	Mainly there are three types of data association
	One To One 
		one entity is related to only one entity  
			example  onebook one publisher
	
	One To Many 
		One entity is related to many entities.
			example one user can have multiple photos but photos can only have on user.
	
	Many To Many 
		It Goes Both ways.
			example multiple Student can have multple course 

Embedding Data:
	if we have two schemas "user" and "post". we can embed in userSchema the postschema as an Array.
	when we sare embading data we dont create seprate model for each schema , we just create schema and embed them 

~~~javascript
		posts : [postSchema]
that will create link between both schemas. the order is importent. postSchema must be defined before 
Now we can push new users post by using
```
		newUser.push({ add props})
			newUser.save();```
	we can also push user post by finding it by id(findById or findOne) and push it in appropriate Route.
~~~javascript

Object Refrencing Data:
		instead of passing and array inside of user shema we pass  object inside that array.
		that object will have type and refrence.

​~~~javascript
			posts : [{
					type : mongoose.Schema.type.ObjectId,
					ref : "Post"
				}]
~~~

create a user and post seratly
when saving post pass a function inside this function push "found.posts.push(post)" and save that 

~~~javascript
		User.findOne({name : "Irshad"}).populate("posts").exec(function(err , user) {
					if(err) throw err; 
					else {
						console.log(user)
					}
			});

	.populate => load up all the associations of that query 

	Nested Population:
		Project.find(query)
		  .populate({ 
		     path: 'pages',
		     populate: {
		       path: 'components',
		       model: 'Component'
		     } 
		  })
		  .exec(function(err, docs) {});

~~~

Virtuald Type: 
	does not get sved on the db . It gets calculated on the way
	Mongoose supports virtual attributes. Virtual attributes are attributes
that are convenient to have around but that do not get persisted to mongodb.
	Virtual fields are awesome for creating aggregate fields.  For example, if our system requires to have first name, last name and the full name (which is just a concatenation of first two names) – there’s no need to store the full name values in addition to the first and last name values! All we need to do is concatenate the first and last name in a full name virtual.

```javascript
In Schema 

	UserSchema.virtual('posts')
		.get(function(){
			return this.post.length
		})
```

Mongoose middleware 
	Middleware (also called pre and post hooks) are functions which are passed control during execution of asynchronous functions.

	pre 
	post 
	
	The $in operator selects the documents where the value of a field equals any value in the specified array. 

​	













