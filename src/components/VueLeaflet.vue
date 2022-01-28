<template>
  <l-map
    style="width: 100%; height: 600px;"
    :zoom="13"
    :center="center"
    @click="mapClick"
  >
    <l-tile-layer :url="url"></l-tile-layer>
    <l-marker
      v-for="(marker, key) in markers"
      :key="key"
      :lat-lng="marker"
      :id="i"
    >
      <l-popup>
        <h1 v-if="moreInfo">{{ name }}</h1>
        <span v-else
          ><p>
            Широта: {{ latitude }}<br />
            Долгота: {{ longitude }}
          </p>
          <hr />
          <p>
            <b>Подробноая информация:</b>
            <em
              >Lorem ipsum dolor sit amet consectetur adipisicing elit. Veniam,
              quidem.</em
            >
          </p></span
        >
        <button @click="information, (moreInfo = !moreInfo)">Info</button>
        <button @click="removeMarkByIndex(id)">Delete</button>
      </l-popup></l-marker
    >

    <l-control
      :style="{ columnNoBackgr: show }"
      class="column"
      position="topright"
    >
      <a v-if="!show" @click="show = !show"
        ><img src="../assets/next.png" style="max-width: 25px; float: right;"
      /></a>
      <a v-else @click="show = !show"
        ><img src="../assets/menu.png" style="max-width: 25px"
      /></a>

      <button v-if="!show" @click="viewId" class="controlButton">
        Добавить маркер
      </button>
      <button v-if="!show" @click="centerMark" class="controlButton">
        Выравнивание по центру
      </button>
      <button v-if="!show" @click="removeMark" class="controlButton">
        Удалить маркер
      </button>
    </l-control>
  </l-map>
</template>

<script>
import {
  LMap,
  LTileLayer,
  LMarker,
  LControl,
  LPolygon,
  LPopup
} from "vue2-leaflet";

export default {
  name: "VueLeaflet",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LControl,
    LPopup
  },
  data() {
    return {
      center: [54.9179, 37.4265],
      map: null,
      tileLayer: null,
      url: "http://{s}.tile.osm.org/{z}/{x}/{y}.png",
      //url: "http://192.168.1.85/hot/{z}/{x}/{y}.png",
      markers: [],
      popupInfo: [null, null],
      show: false,
      latitude: null,
      longitude: null,
      moreInfo: true,
      name: "Name",
      i: null
    };
  },
  methods: {
    mapClick(event) {
      //console.log(click);
      this.markers.push(event.latlng);
      this.i = this.markers.length - 1;
      //console.log(this.markers);
      this.latitude = event.latlng.lat;
      this.longitude = event.latlng.lng;
      console.log(this.latitude, this.longitude);
    },
    removeMark() {
      // remove a marker
      this.markers.pop();
    },
    centerMark() {
      this.center = [this.latitude, this.longitude];
    },
    logs() {
      console.log("oncl");
    },
    information() {
      console.log("info");
    },
    viewId() {
      console.log(this.i);
    },
    removeMarkByIndex(index) {
      this.markers.splice(index, 1);
    }
  }
};
</script>

<style scoped>
.column {
  display: flex;
  flex-direction: column;
  background: rgba(168, 150, 150, 0.6);
}
.columnNoBackgr {
  display: flex;
  flex-direction: column;
}
.controlButton {
  background-color: #1f1c3b;
  color: #bcbed0;
  font-family: Montserrat-Regular;
  padding: 8px 32px;
  border-radius: 6px;
  border: 1px solid #bcbed0;
  margin: 5px;

  font-weight: bold;
  transition: background-color 0.3s ease-in;
}

.kol-flex {
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
}
.controlButton:hover {
  background-color: #423899;
  color: #ebedff;
  border: 1px solid #e2e5fa;
}
.controlButton:active {
  background-color: #251e6b;
  color: #ffff;
}
</style>
