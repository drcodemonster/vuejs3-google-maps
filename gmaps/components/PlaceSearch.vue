<template>
  <div id="mapHolder">
    <div id="map">
      <div class="loading">
        <div class="text">{{ this.loading }}</div>
      </div>
    </div>
    <div id="pac-card">
      <input
        id="pac-input"
        v-bind:placeholder="placeholder"
        type="text"
        v-model="query"
      />
      <div
        class="location-by-gps-btn"
        :class="{ searching: searching_gps }"
        @click.prevent="getLocationViaGPS"
      ></div>
    </div>
    <div id="infowindow-content">
      <!-- /.prova -->
      <span id="place-name" class="title">{{
        place != null ? place.name : ""
      }}</span
      ><br />
      <span id="place-address">{{
        place != null ? place.formatted_address : ""
      }}</span>
    </div>
  </div>
  <!-- /#mapHolder -->
</template>

<script>
import loader from "./../lib-loader";

export default {
  name: "PlaceSearch",
  props: {
    ready: {
      type: Boolean,
      default: undefined,
    },
    loading: String,
    placeholder: String,
    fallbackProcedure: String, // extra location-given
    zoom: Number,
    geolocation: Object,
    gps_timeout: {
      type: Number,
      default: 5000,
    },
    address: Object,
    manually: Object,
    mapOnly: {
      type: Boolean,
      default: false,
    },
    placeList: {
      type: Array,
      default: () => [],
    },
    locationGiven: Object, // {lat: 0, lng: 0 }
  },
  data() {
    return {
      searching_gps: false,
      //Default Coordinates (Somewhere in North Atlantic Ocean) if GPS and Find By Country Fails
      lat: this.geolocation != null ? this.geolocation.lat : null,
      lng: this.geolocation != null ? this.geolocation.lng : null,
      query_address: this.address && this.address.query ? this.address.query: null,
      default_zoom: this.zoom != undefined ? this.zoom : 9,
      query: "",
      map: null,
      marker: null,
      place: null,
      zip_code: null,
      country: null,
      state: null,
      city: null,
      town: null,
      area: null,
      nearest_road: "route",
      address_description: null,
      placeAddresCompoponent: {
        zip_code: "postal_code",
        country: "country",
        state: "administrative_area_level_1",
        city: "administrative_area_level_2",
        town: "sublocality_level_1",
        area: "sublocality_level_2",
        nearest_road: "route",
      },
    };
  },
  watch: {
    place: function () {
      if (this.place.manually == undefined) {
        this.watchCityAndCountryChange();

        this.lat = this.place.geometry.location.lat();
        this.lng = this.place.geometry.location.lng();

        for (const key in this.placeAddresCompoponent) {
          this.$set;
          this[key] = this.getAddressComponent(
            this.place.address_components,
            this.placeAddresCompoponent[key]
          );
        }

        this.emitData();
      }
    },
    geolocation: function (new_value) {
      this.lat = new_value != null ? new_value.lat : null;
      this.lng = new_value != null ? new_value.lng : null;
    },
    ready: function (new_value) {
      //Add timeout to wait for any other watchers
      setTimeout(() => {
        if (new_value == true) {
          this.buildApplication();
        }
      }, 200);
    },
  },
  methods: {
    returnData() {
      return {
        query: this.query,
        lat: this.lat,
        lng: this.lng,
        place: this.place,
        address_description: this.address_description,
        city: this.city,
        state: this.state,
        country: this.country,
        zip_code: this.zip_code,
      };
    },
    createMap(geolocation, zoom) {
      return new Promise((resolve) => {
        if (geolocation && geolocation.lat && geolocation.lng) {
          this.map = new window.google.maps.Map(document.getElementById("map"), {
            center: geolocation,
            zoom: zoom,
            mapTypeId: google.maps.MapTypeId.ROADMAP,
            mapTypeControlOptions: {
              position: window.google.maps.ControlPosition.BOTTOM_LEFT,
            },
          });
          resolve(this.map);
        }
      });
    },
    findNearestPlace() {
      this.findPlaces(this.lat, this.lng, 10, (results, status) => {
        if (status === window.google.maps.places.PlacesServiceStatus.OK) {
          //Get place data for first result
          let firstResult = results[0];
          if (results[0] != undefined) {
            if (results[0].types.includes("route")) {
              firstResult = results[0];
            } else {
              if (results[1] != undefined) {
                firstResult = results[1];
              }
            }
          }
          this.getPlaceDetails(firstResult.place_id, (place, status) => {
            if (status === window.google.maps.places.PlacesServiceStatus.OK) {
              this.place = place;
            }
          });
        }
      });
    },
    findPlaces(lat, lng, radius, callback) {
      var center_point = new window.google.maps.LatLng(lat, lng);
      var request = {
        location: center_point,
        radius: radius,
        // types: ["establishment"],
        fields: ["address_components", "formatted_address", "geometry", "name"],
      };

      var service = new window.google.maps.places.PlacesService(this.map);
      service.nearbySearch(request, callback);
    },
    getPlaceDetails(id, callback) {
      var service = new window.google.maps.places.PlacesService(this.map);
      service.getDetails(
        {
          placeId: id,
          fields: [
            "address_components",
            "formatted_address",
            "geometry",
            "name",
          ],
        },
        callback
      );
    },
    watchCityAndCountryChange() {
      if (this.place != null) {
        if (this.place.name == undefined || this.place.name == null) {
          this.address_description = this.place.formatted_address;
        } else {
          if (this.place.formatted_address.includes(this.place.name)) {
            this.address_description = this.place.formatted_address;
          } else {
            this.address_description =
              this.place.name + ", " + this.place.formatted_address;
          }
        }
      }
    },
    getAddressComponent(address_components, key) {
      var value = "";
      var postalCodeType = address_components.filter((aComp) =>
        aComp.types.some((typesItem) => typesItem === key)
      );
      if (postalCodeType != null && postalCodeType.length > 0)
        value = postalCodeType[0].long_name;
      return value;
    },
    async initMapByCoordinates(lat, lng, override_zoom = null) {
      let zoom =
        this.geolocation.zoom != undefined
          ? this.geolocation.zoom
          : this.default_zoom;

      if (override_zoom != null) {
        zoom = override_zoom;
      }

      await this.createMap({ lat: lat, lng: lng }, zoom);
      this.findNearestPlace();
      this.prepareMap();
      //Create Marker
      this.createMarker();

      this.marker.setPosition({
        lat: lat,
        lng: lng,
      });
    },
    async initMapManually() {
      this.address_description = this.manually.address_description;
      this.city = this.manually.city;
      this.country = this.manually.country;
      this.lat = this.manually.lat;
      this.lng = this.manually.lng;
      this.state = this.manually.state;
      this.zip_code = this.manually.zip_code;
      this.lat = this.manually.lat;
      this.lng = this.manually.lng;

      this.emitData();

      await this.createMap({ lat: this.lat, lng: this.lng }, this.manually.zoom);
      this.prepareMap();
      //Create Marker
      this.createMarker();

      this.marker.setPosition({
        lat: this.lat,
        lng: this.lng,
      });
    },
    initMapByAddress() {
      let geocoder = new window.google.maps.Geocoder();
      let zoom =
        this.address.zoom != undefined ? this.address.zoom : this.default_zoom;

      geocoder.geocode({ address: this.query_address }, async (results, status) => {
        if (status == window.google.maps.GeocoderStatus.OK) {
          await this.createMap(results[0].geometry.location, zoom);
          this.place = results[0];
          this.prepareMap();
        } else {
          //Use default lat long
          this.initMapByCoordinates(this.lat, this.lng, 5);
        }
      });
    },
    createMarker(latlng = null, draggable = true) {
      if (this.marker == null) {
        this.marker = new window.google.maps.Marker({
          map: this.map,
          draggable: draggable,
        });

        if (latlng != null) {
          this.marker.setPosition(latlng);
        }

        //Get markes position on move
        window.google.maps.event.addListener(this.marker, "dragend", () => {
          this.lat = this.marker.getPosition().lat();
          this.lng = this.marker.getPosition().lng();
          this.emitData();
        });

        //Get markes position on move
        window.google.maps.event.addListener(this.marker, "dragstart", () => {
          this.hideInfoWindow();
        });
      }
    },

    placeMarkersOnMap(places) {
      const map = new window.google.maps.Map(document.getElementById("map"), {
        mapTypeId: google.maps.MapTypeId.ROADMAP,
        mapTypeControlOptions: {
          position: window.google.maps.ControlPosition.BOTTOM_LEFT,
        },
      });
      // Loop through our array of places each one on the map
      // places [{title: "", coordinates: [lat,lng], info: ""}...]
      if (places.length > 0) {
        const bounds = new window.google.maps.LatLngBounds();
        places.forEach((place) => {
          const position = {
            lat: place.coordinates[0],
            lng: place.coordinates[1],
          };
          // Stretch our bounds to the newly found marker position
          bounds.extend(position);

          const marker = new window.google.maps.Marker({
            position: position,
            map: map,
            title: place.title,
            draggable: false,
          });

          this.newMerkerBubble(marker, place);
          // Automatically center the map fitting all markers on the screen
          if (places.length > 1) {
            map.fitBounds(bounds);
          } else {
            map.setCenter({
              lat: places[0].coordinates[0],
              lng: places[0].coordinates[1],
            });
            map.setZoom(13);
          }
        });
      } else {
        console.warn("No places given.. check placeList prop...");
      }
    },
    newMerkerBubble(marker, place) {
      const infoWindow = new window.google.maps.InfoWindow();
      // Allow each marker to have an info window
      window.google.maps.event.addListener(marker, "click", () => {
        infoWindow.setContent(`<div class="card border-0 d-flex flex-row">
            ${
              place.logo
                ? `<img src="${place.logo}" class="card-img-top rounded-circle w-25 h-25" alt="logo">`
                : ""
            }
            <div class="card-body">
              <h5 class="card-title">${
                place.link
                  ? `<a href="${place.link}" class="card-link">${place.title}</a>`
                  : `${place.title}`
              }</h5>
              <p class="card-text">${place.info}</p>
            </div></div>`);
        infoWindow.open({ anchor: marker, map, shouldFocus: false });
      });
    },
    repositionMarker(latlng) {
      this.lat = latlng.lat();
      this.lng = latlng.lng();

      this.findNearestPlace();

      if (this.marker == null) {
        this.createMarker(latlng);
      } else {
        this.marker.setPosition(latlng);
      }
    },
    createInfoWindow() {
      this.infowindow = new window.google.maps.InfoWindow();
      this.infowindowContent = document.getElementById("infowindow-content");
      this.infowindow.setContent(this.infowindowContent);
    },
    showInfoWindow() {
      this.infowindow.open(this.map, this.marker);
    },
    hideInfoWindow() {
      if (this.infowindow) {
        this.infowindow.close();
      }
    },
    generateSearchCard() {
      const card = document.getElementById("pac-card");
      const input = document.getElementById("pac-input");

      const options = {
        // componentRestrictions: { country: "us" },
        fields: ["address_components", "formatted_address", "geometry", "name"],
        origin: this.map.getCenter(),
        strictBounds: false,
        // types: ["address"],
      };
      this.map.controls[window.google.maps.ControlPosition.TOP_RIGHT].push(
        card
      );
      const autocomplete = new window.google.maps.places.Autocomplete(
        input,
        options
      );

      // Bind the map's bounds (viewport) property to the autocomplete object,
      // so that the autocomplete requests use the current map bounds for the
      // bounds option in the request.
      autocomplete.bindTo("bounds", this.map);

      //Create marker
      this.createMarker();

      autocomplete.addListener("place_changed", () => {
        this.hideInfoWindow();
        this.marker.setVisible(false);

        const place = autocomplete.getPlace();

        if (!place.geometry || !place.geometry.location) {
          // User entered the name of a Place that was not suggested and
          // pressed the Enter key, or the Place Details request failed.
          window.alert("No details available for input: '" + place.name + "'");
          return;
        }

        // If the place has a geometry, then present it on a map.
        if (place.geometry.viewport) {
          this.map.fitBounds(place.geometry.viewport);
        } else {
          this.map.setCenter(place.geometry.location);
          this.map.setZoom(17);
        }
        this.marker.setPosition(place.geometry.location);
        this.marker.setVisible(true);
        this.place = place;
        this.showInfoWindow();
      });
    },
   async  prepareMap() {
      if (this.mapOnly) {
        if (this.placeList.length === 0) {
          if (this.locationGiven) {
            const placeList = [
              {
                coordinates: [this.locationGiven.lat, this.locationGiven.lng],
                title: this.locationGiven.title,
                info: this.locationGiven.info,
                link: this.locationGiven.link,
                logo: this.locationGiven.logo,
              },
            ];
            this.placeMarkersOnMap(placeList);
          } else {
            console.warn(
              "if mapOnly prop set to true, then you should provide either placeList or locationGiven prop..."
            );
          }
        } else {
          this.placeMarkersOnMap(this.placeList);
        }
      } else {
        //Adds the serach card, created a marked and creates the info window
        //Bind map events
        if (!this.map) {
           await this.createMap({ lat: this.lat, lng: this.lng }, this.manually.zoom);
        }
        window.google.maps.event.addListener(this.map, "click", (e) => {
          //Marker re position
          this.repositionMarker(e["latLng"]);
          //Hide info window
          this.hideInfoWindow();
        });
        this.generateSearchCard();
        this.createInfoWindow();
      }
    },

    emitData() {
      this.$emit("changed", this.returnData());
    },

    //Get location via GPS
    getLocationViaGPS() {
      this.searching_gps = true;
      this.$getLocation({ timeout: this.gps_timeout })
        .then((coordinates) => {
          this.lat = coordinates.lat;
          this.lng = coordinates.lng;

          this.findNearestPlace();

          this.marker.setPosition({
            lat: this.lat,
            lng: this.lng,
          });

          this.map.panTo(new window.google.maps.LatLng(this.lat, this.lng));
        })
        .catch((error) => {
          console.log(error);
        })
        .finally(() => {
          this.searching_gps = false;
        });
    },
    //Get location via props
    async getLocationViaGivenLatLong() {
      let location = this.locationGiven;
      if (!location || (!location.lat && !location.lng)) {
        location = this.geolocation;
      }
      await this.createMap(location, this.zoom);

      if (this.placeList.length === 0 && location) {
        if (
          location.lat !== undefined &&
          location.lng !== undefined &&
          location.lat !== null &&
          location.lng !== null
        ) {
          this.lat = location.lat;
          this.lng = location.lng;
          this.prepareMap();
          this.createMarker(location, false);
          this.marker.setPosition(location);
          this.newMerkerBubble(this.marker, location);
          this.map.panTo(new window.google.maps.LatLng(location));
        } else {
          console.warn("Given Location {lat: lng: } value missing");
        }
      } else {
        this.prepareMap();
      }
    },
    async buildApplication() {
      if (this.fallbackProcedure === "manually") {
        this.initMapManually();
      } else if (this.fallbackProcedure === "geolocation") {
        this.initMapByCoordinates(this.lat, this.lng);
      } else if (this.fallbackProcedure === "address") {
        this.initMapByAddress();
      } else if (this.fallbackProcedure === "location-given") {
        this.getLocationViaGivenLatLong();
      } else {
        this.searching_gps = true;
        this.$getLocation({ timeout: this.gps_timeout })
          .then((coordinates) => {
            this.lat = coordinates.lat;
            this.lng = coordinates.lng;
            this.initMapByCoordinates(this.lat, this.lng, this.zoom);
            //Create Marker
            this.createMarker();

            this.marker.setPosition({
              lat: this.lat,
              lng: this.lng,
            });
          })
          .catch(() => {
            this.initMapByAddress();
          })
          .finally(() => {
            this.searching_gps = false;
          });
      }
    },
  },
  async mounted() {
    await loader.ensureReady();
    if (this.ready != undefined && this.ready == false) {
      return;
    }
    this.buildApplication();
  },
};
</script>

<style lang="scss">
#mapHolder {
  width: 100%;
  height: 100%;
  position: relative;
  float: left;

  #map {
    width: 100%;
    height: 100%;
    float: left;

    .loading {
      width: 100%;
      height: 100%;
      float: left;
      background-color: #ededed;
      position: realtive;

      .text {
        width: 100%;
        padding: 15px;
        text-align: center;
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
      }
    }

    iframe {
      width: 100%;
      height: 100%;
    }

    #pac-card,
    #infowindow-content {
      display: block;
    }
  }

  @keyframes gps-searching-animation {
    from {
      width: 50%;
      height: 50%;
    }
    to {
      width: 100%;
      height: 100%;
    }
  }

  #pac-card {
    background: #ededed;
    padding: 10px;
    width: 100%;
    height: auto;
    display: none;
    position: relative;

    input {
      width: 100%;
      line-height: 30px;
      border-radius: 5px;
      background-color: #fff;
      border: 0;
      outline: 0;
      padding: 5px 40px 5px 40px;
      font-size: 14px;
      background-image: url("~@/assets/search.png");
      background-size: auto 15px;
      background-repeat: no-repeat;
      background-position: left 15px center;
    }

    .location-by-gps-btn {
      position: absolute;
      width: 30px;
      height: 30px;
      top: 50%;
      transform: translateY(-50%);
      right: 15px;

      &:hover {
        cursor: pointer;
      }

      &:before {
        display: block;
        content: "";
        width: 100%;
        height: 100%;
        position: absolute;
        top: 0;
        left: 0;
        z-index: 2;
        background-image: url("~@/assets/gps.png");
        background-position: center;
        background-repeat: no-repeat;
        background-size: 18px auto;
        border-radius: 30px;
      }

      &.searching {
        &:after {
          display: block;
          border-radius: 30px;
          background-color: #fdf6e3;
          content: "";
          width: 100%;
          height: 100%;
          position: absolute;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
          z-index: 1;
          animation-name: gps-searching-animation;
          animation-iteration-count: infinite;
          animation-duration: 1s;
        }
      }
    }
  }

  #infowindow-content {
    width: 100%;
    float: left;
    display: none;
  }
}
</style>
