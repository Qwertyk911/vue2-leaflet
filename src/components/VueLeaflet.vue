<template>
  <l-map
    style="width: 100%; height: 600px;"
    :zoom="13"
    :center="center"
    @click="mapClick"
    :options="{ zoomControl: false }"
  >
    <l-tile-layer :url="url"></l-tile-layer>
    <l-marker
      @click="information(marker), (moreInfo = true)"
      v-for="(marker, key) in markers"
      :key="key"
      :lat-lng="marker"
    >
      <l-popup>
        <h4 v-if="moreInfo">{{ name }}</h4>
        <span v-else
          ><p>
            Широта: {{ latitude }}<br />
            Долгота: {{ longitude }}
          </p>
          <!-- <hr />
          <p>
            <b>Подробноая информация:</b>
            <em
              >Lorem ipsum dolor sit amet consectetur adipisicing elit. Veniam,
              quidem.</em
            >
          </p> -->
        </span>
        <DxButton
          text="Info"
          type="success"
          styling-mode="contained"
          @click="information(marker), (moreInfo = !moreInfo)"
        />
        <DxButton
          text="Delete"
          type="success"
          styling-mode="contained"
          @click="removeMarkByIndex(marker), (moreInfo = true)"
        /> </l-popup
    ></l-marker>
    <l-control :style="backgrLeft" class="column" position="topleft">
      <a
        v-if="showSearch"
        @click="(showSearch = !showSearch), switchBackLeft(showSearch)"
        ><img src="../assets/search.png" style="max-width: 25px"
      /></a>
      <a v-else @click="(showSearch = !showSearch), switchBackLeft(showSearch)"
        ><img src="../assets/left.png" style="max-width: 25px; float: left;"
      /></a>
      <div v-if="!showSearch" class="form">
        <div class="dx-fieldset">
          <div class="dx-field">
            <div class="dx-field-label">Искать:</div>
            <div class="dx-field-value">
              <DxRadioGroup
                :items="priorities"
                :value="priorities[0]"
                @valueChanged="changeSelection"
              />
            </div>
          </div>
          <div class="dx-field">
            <DxButton
              text="Поиск"
              type="success"
              styling-mode="contained"
              @click="postAPI(postCity, searchOf)"
            />
            <div
              class="dx-field-value"
              @keyup.enter="postAPI(postCity, searchOf)"
            >
              <DxTextBox
                v-model="postCity"
                :placeholder="searchOf"
                :show-clear-button="true"
              />

              <!-- Выпадающий список пока не реализован-->
              <select v-model="list">
                <option
                  v-for="listDat in listData"
                  :value="listDat.display_name"
                  :key="listDat.id"
                  >{{ listDat.display_name }}</option
                >
              </select>
              <!-- Конец выпадающего списка -->

            </div>
          </div>
        </div>
      </div>
    </l-control>
    <l-control :style="backgrRight" class="column" position="topright">
      <a v-if="show" @click="(show = !show), switchBackRignt(show)"
        ><img src="../assets/next.png" style="max-width: 25px; float: right;"
      /></a>
      <a v-else @click="(show = !show), switchBackRignt(show)"
        ><img src="../assets/menu.png" style="max-width: 25px"
      /></a>

      <DxButton
        class="controlButton"
        text="Выравнивание по центру"
        type="success"
        styling-mode="contained"
        v-if="show"
        @click="centerMark"
      />
      <DxButton
        class="controlButton"
        text="Удалить маркер"
        type="success"
        styling-mode="contained"
        v-if="show"
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
import DxRadioGroup from "devextreme-vue/radio-group";

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
    DxTextBox,
    DxRadioGroup
  },
  data() {
    return {
      zoom: 13,
      priorities: ["По названию", "По координатам"],
      searchOf: "Введите название",
      center: [45.17131249, 33.2975006],
      map: null,
      tileLayer: null,
      url: "http://{s}.tile.osm.org/{z}/{x}/{y}.png",
      //url: "http://192.168.1.85/hot/{z}/{x}/{y}.png",
      markers: [],
      bmark: [
        {
          lat: null,
          lng: null,
          display_name: ""
        }
      ],
      popupInfo: [null, null],
      show: true,
      showSearch: true,
      latitude: null,
      longitude: null,
      moreInfo: true,
      name: "Name",
      i: null,
      message: "",
      postCity: "",
      beforeOtvet: "",
      afterOtvet: "",
      backgrLeft: "",
      backgrRight: "background: rgba(168, 150, 150, 0.6);",
      listData: [
        {
          id: null,
          display_name: ""
        }
      ],
      list: null
    };
  },

  methods: {
    mapClick(event) {
      //this.markers.push(event.latlng);

      // Добавление меток по клику через номинатим-------------------------------------------------------------
      let koord = "Введите координаты";
      let zapros = `${event.latlng.lat} ${event.latlng.lng}`;
      this.postAPI(zapros, koord);
    },
    // Удаление  последней добавленной метки
    removeMark() {
      if (this.markers.length > 0) {
        this.markers.pop();
      } else {
        alert("Markers not found");
      }
    },
    // Выровнять карту по центру
    centerMark() {
      try {
        this.center = [
          this.markers[this.markers.length - 1].lat,
          this.markers[this.markers.length - 1].lng
        ];
        this.zoom = 13;
      } catch (err) {
        alert("Сначала установите маркер");
      }
    },
    logs() {
      console.log("oncl");
    },

    // Вывод информации о метке----------------------------------------------------
    information(index) {
      // console.log(index);
      this.i = this.markers.indexOf(index);
      this.latitude = this.markers[this.i].lat;
      this.longitude = this.markers[this.i].lon;
      if (!this.longitude) {
        this.longitude = this.markers[this.i].lng;
      }
      this.name = this.markers[this.i].display_name;
    },

    // Удаление метки по клику на нее
    removeMarkByIndex(index) {
      this.i = this.markers.indexOf(index);
      this.markers.splice(this.i, 1);
    },
    // Формирование запроса в номинатим/парсим ответ/добавляем метку на карту
    postAPI(postCity, searchOf) {
      if (searchOf === "Введите координаты") {

        // Начало блока для запроса по координатам или клика по карте---------------------------------------------------
        if (/[a-zA-ZА-Яа-я]/.test(postCity)) {
          alert(
            "Неверный формат данных \nВведите широту и долготу через запятую или пробел"
          );
        } else {
          let razdel = postCity.split(" ");
          if (razdel[0].indexOf(",") >= 0) {
            razdel[0] = razdel[0].slice(0, -1);
          }
          // For test-----------------------------------------

          // //console.log(razdel);
          // this.center = [razdel[0], razdel[1]];
          // LMarker.latLng = [razdel[0], razdel[1]];
          // console.log(LMarker)
          // //this.bmark.lng = razdel[1];
          // this.markers.push(LMarker.latLng);
          // console.log(this.markers)

          // For Nominatim------------------------------------

          postCity = `http://192.168.1.85/nominatim/reverse.php?lat=${razdel[0]}&lon=${razdel[1]}`;
          fetch(postCity)
            .then(response => {
              return response.text();
            })
            .then(data => {
              if (data.length) {
                let start = data.indexOf("<result ");
                let finish = data.indexOf("</result>");
                let diff = data.slice(start + 8, finish);

                this.center = [razdel[0], razdel[1]];
                this.bmark.lat = razdel[0];
                this.bmark.lng = razdel[1];

                if (data.indexOf("</error") >= 0) {
                  this.bmark.display_name =
                    "Информация о данной метке остутствует";
                }

                // Костыль для Крыма -----------------------------------------------
                else if (data.indexOf("Республика Крым") >= 0) {
                  this.bmark.display_name = diff
                    .slice(diff.indexOf(">") + 1, diff.length)
                    .replace("Украина", "Россия")
                    .split("&quot;")
                    .join('"');
                }
                //----------------------------------------------------------------


                else {
                  this.bmark.display_name = diff
                    .slice(diff.indexOf(">") + 1, diff.length)
                    .split("&quot;")
                    .join('"');
                }
                this.markers.push(Object.assign({}, this.bmark));
              } else {
                this.postCity.length == 0
                  ? alert(`Введите название`)
                  : alert(`Город ${this.postCity} не найден`);
              }
            });
        }
        // Конец блока для запроса по координатам или клика по карте---------------------------------------------------
      } else {
        // Начало блока для запроса по названию---------------------------------------------------
        if (/[a-zA-ZА-Яа-я]/.test(postCity)) {
          postCity = "http://192.168.1.85/nominatim/search.php?q=" + postCity;
          fetch(postCity)
            .then(response => {
              return response.json();
            })
            .then(data => {
              if (data.length) {
                this.listData = data;
                this.center = [data[0].lat, data[0].lon];
                this.markers.push(data[0]);
              } else {
                this.postCity.length == 0
                  ? alert(`Введите название`)
                  : alert(`Город ${this.postCity} не найден`);
              }
            });
        } else {
          alert("Неверно введено название");
        }
        // Конец блока для запроса по названию---------------------------------------------------
      }
    },
    switchBackLeft(showSearch) {
      !showSearch
        ? (this.backgrLeft = "background: rgba(168, 150, 150, 0.6);")
        : (this.backgrLeft = "");
    },
    switchBackRignt(show) {
      show
        ? (this.backgrRight = "background: rgba(168, 150, 150, 0.6);")
        : (this.backgrRight = "");
    },
    changeSelection(e) {
      console.log(e);
      if (e.value === "По координатам") {
        this.searchOf = "Введите координаты";
      } else {
        this.searchOf = "Введите название";
      }
    }
  }
};
</script>

<style scoped>
.column {
  display: flex;
  flex-direction: column;
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
