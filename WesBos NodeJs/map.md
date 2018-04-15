# Set up Schema 

in Schema add 

```javascript
location: {
    type: {
      type: String,
      default: 'Point'
    },
    coordinates: [{
      type: Number,
      required: 'You must supply coordinates!'
    }],
    address: {
      type: String,
      required: 'You must supply an address!'
    }
```



# Auto Complete 

first we need to set up an input field where we type name of place and it will automatically  give us options in drop down.

in js modules create autoComplete.js and import in main.js 

in our autoComplete.js create a function named autoComplete 

```javascript
function autocomplete(input, latInput, lngInput) {
  if(!input) return; // skip this fn from running if there is not input on the page
  const dropdown = new google.maps.places.Autocomplete(input); //must setup key first

  dropdown.addListener('place_changed', () => {
    const place = dropdown.getPlace();
    latInput.value = place.geometry.location.lat();
    lngInput.value = place.geometry.location.lng();
  });
  // if someone hits enter on the address field, don't submit the form
  input.on('keydown', (e) => {
    if (e.keyCode === 13) e.preventDefault();
  });
}

export default autocomplete;
```

Now require it in main.js 

```javascript
import autocomplete from './modules/autocomplete';

autocomplete( $('#address'), $('#lat'), $('#lng') );
```

Note: 

​	ids('#') are coming from template and $ is coming from blign.js

# Template 

```jade
 label(for="address") Address
    input(type="text" id="address" name="location[address]" value=(store.location && store.location.address))
    label(for="lng") Address Lng
    input(type="text" id="lng" name="location[coordinates][0]" value=(store.location && store.location.coordinates[0]) required)
    label(for="lat") Address Lat
    input(type="text" id="lat" name="location[coordinates][1]" value=(store.location && store.location.coordinates[1]) required)
```

- `location[address] `will render by body parser as `location.address`  we can access with dot.

