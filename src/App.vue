<template>
  <div class="container">
    <div class="pt-2 font-size-xl mb-3">Create Address</div>

    <button class="btn btn-success" @click="triggerReady">
      Trigger map ready event
    </button>
    <hr />

    <div class="row">
      <div class="col-md-8">
        <div class="map_holder">
          <place-search
            v-bind:ready="ready"
            placeholder="Enter a location"
            loading="Map is loading"
            v-bind:gps_timeout="7000"
            v-bind:fallbackProcedure="fallbackProcedure"
            v-bind:zoom="zoom"
            v-bind:geolocation="geolocation"
            v-bind:address="address"
            v-bind:manually="manually"
            v-bind:map-only="mapOnly"
            v-bind:location-given="locationGiven"
            v-bind:place-list="placeList"
            @changed="getMapData"
          >
          </place-search>
        </div>
      </div>
      <div class="col-md-4">
        <div class="text-uppercase color-secondary mb-1">Address</div>
        <div class="mb-3 font-weight-bold">
          {{
            place.country != null
              ? place.country + ", " + place.city
              : "Please search for an address before procceeding"
          }}
        </div>

        <div class="text-uppercase color-secondary mb-1">Zip code</div>
        <div class="mb-3">
          <input type="text" v-model="place.zip_code" class="form-control" />
        </div>

        <div class="text-uppercase color-secondary mb-1">
          Address descriotion
        </div>
        <div class="mb-3">
          <textarea
            class="form-control"
            cols="30"
            rows="5"
            v-model="place.address_description"
          ></textarea>
        </div>

        <div class="mb-2">
          <div class="form-check">
            <input
              class="form-check-input"
              id="primary_address"
              type="checkbox"
              v-model="form_data.primary"
            />
            <label class="form-check-label" for="primary_address">
              Make Primary Address
            </label>
          </div>
        </div>
        <div class="mt-3">
          <button class="btn btn-primary w-100">Save address</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      ready: false, //Add ready:false to stop map from loading, and then when changed to true map will auto load
      fallbackProcedure: "location-given", //gps | geolocation | address | manually | location-given
      mapOnly: true,
      locationGiven: { lat: 50.7022222, lng: -120.4419444, title: "benim", info:"canım", link: "", logo:"" },
      placeList: [], //  [{coordinates: [49.1950000,-123.1819444],title:'Vancouver (YVR)',logo:'https://ui-avatars.com/api/?name=Vancouver (YVR)', link:'https://ui-avatars.com/api/?name=Vancouver (YVR)', info:'Vancouver Int\'l',color:'red',icon:'circle'},{coordinates:[48.6469444,-123.4258333],title:'Victoria (YYJ)',info:'Victoria Int\'l',color:'red',icon:'triangle'},{coordinates:[50.7022222,-120.4419444],title:'Kamloops (YKA)',info:'YKA, Kamloops, BC',color:'red',icon:'triangle'},{coordinates:[47.4488889,-122.3091667],title:'Seattle (SEA)',info:'Seattle Tacoma Int\'l',color:'cyan',icon:'circle'},{coordinates:[48.7925000,-122.5375000],title:'Bellingham (BLI)',info:'Bellingham Int\'l',color:'cyan',icon:'triangle'},{coordinates:[46.9666667,-122.9000000],title:'Olympia (OLM)',info:'OLM, Olympia, WA',color:'cyan',icon:'triangle'},{coordinates:[45.5894400,-122.6068200],title:'Portland (PDX)',info:'Portland Int\'l',color:'lime',icon:'circle'},{coordinates:[44.1166667,-123.2166667],title:'Eugene (EUG)',info:'Mahlon Sweet Field',color:'lime',icon:'triangle'},{coordinates:[44.2541600,-121.1505100],title:'Redmond (RDM)',info:'Roberts Field',color:'lime',icon:'triangle'}],
      zoom: 15, //Default Zoom
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
      form_data: {},
    };
  },
  methods: {
    getMapData(place) {
      this.place = place;
      console.log(place);
    },
    triggerReady() {
      this.fallbackProcedure = "location-given";
      this.ready = true;
    },
  },
  created() {},
};
</script>

<style lang="scss" scoped>
.map_holder {
  width: 100%;
  height: 450px;
  float: left;
}
</style>


