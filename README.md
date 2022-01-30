# vuejs3-google-maps

> Google Map integration for VueJs

## Demo

You can clone this directory and run the command below to generate a demo Vue 3 project using this plugin!
```
npm run serve:demo
```

![Wall entry demo](https://max.al/uploads/md/vuejs3-google-maps_preview.png)

## Description
This plugin integrates google places into your vue3 application. It uses browser geolocation to track the user. You will have to provide a default fallback address and lat,lng in case the plugin fails to find the user's location. Once the location is found the map is centered and a Marker is placed on the map. The user can search for a different location, move the marker by dragging or clicking.

## Events
1. Search - User can search by query for any kind of address
2. Map clicking - Once a click is triggered on the map, the plugin will try to find the nearest road, business, city.
3. Dragging - Dragging will not trigger any Map API events, it will only update the users latitude and longitude

## Callbacks
1. On each map data change the plugin will return back a list of computed data:

```js
{
  address_description: "Global Records Albania, Rruga Frederik Shiroka 3, Tiranë, Albania"
  city: "Tiranë"
  country: "Albania"
  lat: 41.327455
  lng: 19.80484
  place: Proxy {address_components: Array(6), formatted_address: "Rruga Frederik Shiroka 3, Tiranë, Albania", geometry: {…}, name: "Global Records Albania", html_attributions: Array(0), …} //Google Maps API place object
  query: ""
  state: "Qarku i Tiranës"
  zip_code: "1001" //Sometime comes as null
}
```

## Installation

```
npm i vuejs3-google-maps
```

## Dependencies

This plugin uses geolocation to track users location so **vue3-geolocation** plugin should be installed to:
```
npm i vue3-geolocation
```

## Usage

Before starting you need a Google API key from the [developer console](http://console.developers.google.com/), once you obtained your key, import the module in your application and register it as plugin:

### Needed Google APIS

1. Maps Javascript API
2. Places API
3. GeoCoding API

```js
import VueGeolocation from "vue3-geolocation";
import GMaps from "vuejs3-google-maps";

let app = createApp(App);

app.use(VueGeolocation);
app.use(GMaps, {
  load: {
    apiKey: "your-api-key",
    libraries: ["places"],
  },
});

app.mount("#app");
```

## Components

```html
<PlaceSearch
  v-bind:ready="ready"
  placeholder="Enter a location"
  loading="Map is loading"
  v-bind:fallbackProcedure="fallbackProcedure"
  v-bind:zoom="zoom"
  v-bind:geolocation="geolocation"
  v-bind:gps_timeout="3000"
  v-bind:address="address"
  v-bind:map-only="mapOnly"
  v-bind:location-given="locationGiven"
  v-bind:place-list="placeList"
  @changed="getMapData"
>
</PlaceSearch>
```

```js
export default {
  name: "AddressesCreate",
  data() {
    return {
      ready: false, //Add ready:false to stop map from loading, and then when changed to true map will auto load
      // **GPS** : will trigger geolocation plugin , to find users location by GPS
      // **geolocation** : will try to find the place by lat, lng
      // **address**: will try to find the place by address query
      // **manually**: manually preset values

      // If GPS is selected as a fallbackProcedure and it fails , then address fallback is triggered and if address fails geolocation is triggered
      fallbackProcedure: "location-given", //gps | geolocation | address | manually | location-given
      mapOnly: false,  // remove top search autocomplete bar,
      locationGiven: { lat: 50.7022222, lng: -120.4419444, title: "", info:"", link: "", logo:""}, // only works once fallbackProcedure is location-given.
      placeList: [], // [{coordinates: [49.1950000,-123.1819444],title:'Vancouver (YVR)',logo:'https://ui-avatars.com/api/?name=Vancouver (YVR)', link:'https://ui-avatars.com/api/?name=Vancouver (YVR)', info:'Vancouver Int\'l',color:'red',icon:'circle'},{coordinates:[48.6469444,-123.4258333],title:'Victoria (YYJ)',info:'Victoria Int\'l',color:'red',icon:'triangle'},{coordinates:[50.7022222,-120.4419444],title:'Kamloops (YKA)',info:'YKA, Kamloops, BC',color:'red',icon:'triangle'},{coordinates:[47.4488889,-122.3091667],title:'Seattle (SEA)',info:'Seattle Tacoma Int\'l',color:'cyan',icon:'circle'},{coordinates:[48.7925000,-122.5375000],title:'Bellingham (BLI)',info:'Bellingham Int\'l',color:'cyan',icon:'triangle'},{coordinates:[46.9666667,-122.9000000],title:'Olympia (OLM)',info:'OLM, Olympia, WA',color:'cyan',icon:'triangle'},{coordinates:[45.5894400,-122.6068200],title:'Portland (PDX)',info:'Portland Int\'l',color:'lime',icon:'circle'},{coordinates:[44.1166667,-123.2166667],title:'Eugene (EUG)',info:'Mahlon Sweet Field',color:'lime',icon:'triangle'},{coordinates:[44.2541600,-121.1505100],title:'Redmond (RDM)',info:'Roberts Field',color:'lime',icon:'triangle'}],
      zoom: 17, //Default Zoom
      geolocation: {
        // If GPS and Find by address fails then, map will be positioned by a default geolocation
        lat: 31.73858,
        lng: -35.98628,
        zoom: 2,
      },
      address: {
        query: "Albania, Tirane", //If GPS fails, Find by address is triggered
        zoom: 10,
      },
      manually: {
        address_description: "21 Dhjetori, Tirana, Albania",
        city: "Tirana",
        country: "Albania",
        lat: 41.3267905,
        lng: 19.8060475,
        state: "Tirana County",
        zip_code: "",
        zoom: 17,
      },
      place: {},
    };
  },
  methods: {
    getMapData(place) {
      this.place = place;
    },
  },
  created() {},
};
```

