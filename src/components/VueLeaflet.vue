<template>
  <l-map
    style="width: 100%; height: 600px;"
    :zoom="13"
    :center="center"
    @click="mapClick"
    :options="{zoomControl: false}"
  >
    <l-tile-layer :url="url"></l-tile-layer>
    <l-marker
      @click="information(marker), (moreInfo = true)"
      v-for="(marker, key) in markers"
      :key="key"
      :lat-lng="marker"
      :id="i"
    >
      <l-popup>
        <h4 v-if="moreInfo">{{ name }}</h4>
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
        <DxButton
          text="Info"
          type="success"
          styling-mode="contained"
          v-if="!show"
          @click="information(marker), (moreInfo = !moreInfo)"
        />
        <DxButton
          text="Delete"
          type="success"
          styling-mode="contained"
          v-if="!show"
          @click="removeMarkByIndex(marker), (moreInfo = true)"
        />
        <!-- <button @click="information(marker), (moreInfo = !moreInfo)">
          Info
        </button>
        <button @click="removeMarkByIndex(marker), (moreInfo = true)">
          Delete
        </button> -->
      </l-popup></l-marker
    >
<l-control class="column" position="topleft">
        <a v-if="!showSearch" @click="showSearch = !showSearch"
        ><img src="../assets/search.png" style="max-width: 25px; float: right;"
      /></a>
      <a v-else @click="showSearch = !showSearch"
        ><img src="../assets/left.png" style="max-width: 25px"
      /></a>
       <div v-if="showSearch" class="form">
        <div class="dx-fieldset">
          <div class="dx-field">
            <DxButton
              text="Поиск"
              type="success"
              styling-mode="contained"
              @click="postAPI(postCity)"
            />
            <div class="dx-field-value">
              <DxTextBox
                v-model="postCity"
                placeholder="Введите город"
                :show-clear-button="true"
              />
            </div>
          </div>
          <div class="dx-field">
            <DxButton
              text="Поиск"
              type="success"
              styling-mode="contained"
              @click="splitString(message)"
              @keyup.enter="splitString(message)"
            />
            <div class="dx-field-value">
              <DxTextBox
                v-model="message"
                placeholder="Введите координаты"
                :show-clear-button="true"
              />
            </div>
          </div>
        </div>
      </div>
</l-control>
    <l-control class="column" position="topright">
      <a v-if="!show" @click="show = !show"
        ><img src="../assets/next.png" style="max-width: 25px; float: right;"
      /></a>
      <a v-else @click="show = !show"
        ><img src="../assets/menu.png" style="max-width: 25px"
      /></a>

      <DxButton
        class="controlButton"
        text="Выравнивание по центру"
        type="success"
        styling-mode="contained"
        v-if="!show"
        @click="centerMark"
      />
      <DxButton
        class="controlButton"
        text="Удалить маркер"
        type="success"
        styling-mode="contained"
        v-if="!show"
        @click="removeMark"
      />
 
    </l-control>
    <l-control-zoom position="bottomright"></l-control-zoom>
  </l-map>
</template>
<script>
import {
  LMap,
  LTileLayer,
  LMarker,
  LControl,
  LPolygon,
  LPopup,
  LControlZoom
} from "vue2-leaflet";
import DxButton from "devextreme-vue/button";
import DxTextBox from "devextreme-vue/text-box";
export default {
  name: "VueLeaflet",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LControl,
    LPopup,
    LPolygon,
    LControlZoom,
    DxButton,
    DxTextBox
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
      showSearch: true,
      latitude: null,
      longitude: null,
      moreInfo: true,
      name: "Name",
      i: null,
      message: "",
      postCity: "",
      beforeOtvet: "",
      afterOtvet: ""
    };
  },

  methods: {
    mapClick(event) {
      this.markers.push(event.latlng);
      console.log(event.latlng);
      console.log(this.markers);
    },
    removeMark() {
      if (this.markers.length > 0) {
        this.markers.pop();
      } else {
        alert("Markers not found");
      }
    },
    centerMark() {
      //console.log(this.markers[this.markers.length-1]);
      this.center = [
        this.markers[this.markers.length - 1].lat,
        this.markers[this.markers.length - 1].lng
      ];
    },
    logs() {
      console.log("oncl");
    },
    information(index) {
      // console.log(index);
      this.i = this.markers.indexOf(index);
      this.latitude = this.markers[this.i].lat;
      this.longitude = this.markers[this.i].lng;
    },
    removeMarkByIndex(index) {
      //console.log(index);
      this.i = this.markers.indexOf(index);
      this.markers.splice(this.i, 1);
    },
    splitString(message) {
      let razdel = message.split(" ");
      if (razdel[0].indexOf(",") >= 0) {
        razdel[0] = razdel[0].slice(0, -1);
      }
      this.center = [parseInt(razdel[0]), parseInt(razdel[1])];
      this.markers.push(this.center);
    },
    postAPI(postCity) {
      postCity = this.postCity;
      postCity = "http://192.168.1.85/nominatim/search.php?q=" + postCity;
      //console.log(this.postCity);
      fetch(postCity)
        .then(response => {
          return response.json();
        })
        .then(data => {
          console.log(data);
          if (data.length) {
            this.afterOtvet = data;
            console.log(this.afterOtvet[0]);
            this.beforeOtvet = `Обьект находится по координатам: 
      Широта: ${this.afterOtvet[0].lat}
      Долгота: ${this.afterOtvet[0].lon}`;
            this.center = [this.afterOtvet[0].lat, this.afterOtvet[0].lon];
            this.markers.push([this.afterOtvet[0].lat, this.afterOtvet[0].lon]);
            this.name = this.afterOtvet[0].display_name;
          } else {
            alert(`Город ${this.postCity} не найден`);
          }
        });
      //console.log(this.beforeOtvet);
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
  margin: 5px;
}

.kol-flex {
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
}
/* .controlButton:hover {
  background-color: #423899;
  color: #ebedff;
  border: 1px solid #e2e5fa;
}
.controlButton:active {
  background-color: #251e6b;
  color: #ffff;
} */
.column > a > img:hover {
  border: 1px solid black;
  max-width: 23px !important;
  cursor: pointer;
}
.searchInput {
  margin: 5px 5px 15px;
  color: rgb(224, 224, 224);
  background-color: rgba(10, 10, 10, 0.6);
}
input::placeholder {
  color: rgb(224, 224, 224);
}
.labelOtvet {
  height: 100px;
  border: 1px solid black;
  margin: 5px;
  max-width: 217.969px;
}
</style>
