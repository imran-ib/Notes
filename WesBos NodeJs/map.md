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


Now lets make the map center and put markers on the map 

we need to index our data 

```javascript
storeSchema.index({ location: '2dsphere' });

```

make route

```javascript
router.get('/api/stores/near', catchErrors(storeController.mapStores));

exports.mapStores = async (req, res) => {
  const coordinates = [req.query.lng, req.query.lat].map(parseFloat);
  const q = {
    location: {
      $near: {
        $geometry: {
          type: 'Point',
          coordinates
        },
        $maxDistance: 10000 // 10km
      }
    }
  };

  const stores = await Store.find(q).select('slug name description location photo').limit(10);
  res.json(stores); // all the string in .select name for search
};
```

# Front End

First add rout for `map`

```javascript
router.get('/map', storeController.mapPage);

exports.mapPage = (req, res) => {
  res.render('map', { title: 'Map' });
};

```

Template

```jade
extends layout

block content
  .inner
    h2= title
    .map
      .autocomplete
        input.autocomplete__input(type="text" placeholder="Search for Anything" name="geolocate")
      #map
        p Loading Map...

```



Client Side `map.js`

​	Imports

```javascript
import axios from 'axios';
import { $ } from './bling';

function makeMap(){
    
}

export default makeMap;
```

Import this in main file

```javascript
import makeMap from './modules/map';

makeMap( $('#map') );
```



we are going to make two function `makeMap` and  `loadPlaces`

if there are no maps we return 

```javascript
function makeMap(mapDiv) {
  if (!mapDiv) return;
}


```

we need some map options

```javascript
const mapOptions = {
  center: { lat: 43.2, lng: -79.8 },
  zoom: 10
};
```



now make map

``` javascript
function makeMap(mapDiv) {
  if (!mapDiv) return;
  // make our map
  const map = new google.maps.Map(mapDiv, mapOptions);
  loadPlaces(map);

  const input = $('[name="geolocate"]'); //
  const autocomplete = new google.maps.places.Autocomplete(input);
  autocomplete.addListener('place_changed', () => {
    const place = autocomplete.getPlace();
    loadPlaces(map, place.geometry.location.lat(), place.geometry.location.lng());
  });
}
```

After this you should have map and working search input

Now load places

```javascript
function loadPlaces(map, lat = 43.2, lng = -79.8) {
  axios.get(`/api/stores/near?lat=${lat}&lng=${lng}`)
    .then(res => {
      const places = res.data;
      if (!places.length) {
        alert('no places found!');
        return;
      }
      // create a bounds
      const bounds = new google.maps.LatLngBounds();
      const infoWindow = new google.maps.InfoWindow();

      const markers = places.map(place => {
        const [placeLng, placeLat] = place.location.coordinates;
        const position = { lat: placeLat, lng: placeLng };
        bounds.extend(position);
        const marker = new google.maps.Marker({ map, position });
        marker.place = place;
        return marker;
      });

      // when someone clicks on a marker, show the details of that place
      markers.forEach(marker => marker.addListener('click', function() {
        console.log(this.place);
        const html = `
          <div class="popup">
            <a href="/store/${this.place.slug}">
              <img src="/uploads/${this.place.photo || 'store.png'}" alt="${this.place.name}" />
              <p>${this.place.name} - ${this.place.location.address}</p>
            </a>
          </div>
        `;
        infoWindow.setContent(html);
        infoWindow.open(map, this);
      }));

      // then zoom the map to fit all the markers perfectly
      map.setCenter(bounds.getCenter());
      map.fitBounds(bounds);
    });

}

```

Whole File `map.js`

```javascript
import axios from 'axios';
import { $ } from './bling';

const mapOptions = {
  center: { lat: 43.2, lng: -79.8 },
  zoom: 10
};

function loadPlaces(map, lat = 43.2, lng = -79.8) {
  axios.get(`/api/stores/near?lat=${lat}&lng=${lng}`)
    .then(res => {
      const places = res.data;
      if (!places.length) {
        alert('no places found!');
        return;
      }
      // create a bounds
      const bounds = new google.maps.LatLngBounds();
      const infoWindow = new google.maps.InfoWindow();

      const markers = places.map(place => {
        const [placeLng, placeLat] = place.location.coordinates;
        const position = { lat: placeLat, lng: placeLng };
        bounds.extend(position);
        const marker = new google.maps.Marker({ map, position });
        marker.place = place;
        return marker;
      });

      // when someone clicks on a marker, show the details of that place
      markers.forEach(marker => marker.addListener('click', function() {
        console.log(this.place);
        const html = `
          <div class="popup">
            <a href="/store/${this.place.slug}">
              <img src="/uploads/${this.place.photo || 'store.png'}" alt="${this.place.name}" />
              <p>${this.place.name} - ${this.place.location.address}</p>
            </a>
          </div>
        `;
        infoWindow.setContent(html);
        infoWindow.open(map, this);
      }));

      // then zoom the map to fit all the markers perfectly
      map.setCenter(bounds.getCenter());
      map.fitBounds(bounds);
    });

}

function makeMap(mapDiv) {
  if (!mapDiv) return;
  // make our map
  const map = new google.maps.Map(mapDiv, mapOptions);
  loadPlaces(map);

  const input = $('[name="geolocate"]');
  const autocomplete = new google.maps.places.Autocomplete(input);
  autocomplete.addListener('place_changed', () => {
    const place = autocomplete.getPlace();
    loadPlaces(map, place.geometry.location.lat(), place.geometry.location.lng());
  });
}

export default makeMap;

```

