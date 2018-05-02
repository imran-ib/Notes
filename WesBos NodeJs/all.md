`pre= h.dump(store)`

# Helpers.js

in this file file we have anything external liabery that we need in all templates

in `app.js` we have local variables that are available to us in all templates



if you are using node greater then version seven then you can delete the code that check node version 

in `start.js`

```javascript
// Make sure we are running node 7.6+
const [major, minor] = process.versions.node.split('.').map(parseFloat);
if (major < 7 || (major === 7 && minor <= 5)) {
  console.log('🛑 🌮 🐶 💪 💩\nHey You! \n\t ya you! \n\t\tBuster! \n\tYou\'re on an older version of node that doesn\'t support the latest and greatest things we are learning (Async + Await)! Please go to nodejs.org and download version 7.6 or greater. 👌\n ');
  process.exit();
}

```



here we are importing `mongoose` `dotenv ` setting up mongoose Promise bringing in models and server



# MVC

### M	Model

​	in model we write the code that has access to data base 

### v  	View

 In Views We have template 

### c	Controller

Controller in kind of inter connection between model and views



# Middle Ware



# Template Mixins

in Our mixin folder when we create  a form mixin for add and edit form so that we dont have to wite it again 

for that we need to know few things

- in the top of mixin `mixin storeForm(store = {})`what it will do is if we pass the store it will use that store data but if there is no store it will fall back to an empty object  we can include 	`include mixins/_storeCard` to use it `        +storeCard(store)`
- in mixin we can also pass data` +storeCard(name : 'store')` 
- ​

