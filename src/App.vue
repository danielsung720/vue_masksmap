<template>
  <div id="app">
    <div class="row no-gutters">
      <div class="col-sm-3">
        <div class="toolbox">
          <div class="sticky-top bg-white shadow-sm p-2">
            <div class="form-group d-flex">
              <label for="cityName" class="mr-2 col-form-label text-right">縣市</label>
              <div class="flex-fill">
                <select name="" id="cityName" class="form-control"
                  v-model="select.city" @change="removeMarker(); updateMap();" >
                  <option value="">--Select One--</option>
                  <option :value="c.CityName" v-for="c in cityName" :key="c.CityName">
                    {{ c.CityName }}
                  </option>
                </select>
              </div>
            </div>
            <div class="form-group d-flex">
              <label for="" class="mr-2 col-form-label text-right">地區</label>
              <div class="flex-fill">
                <select name="" id="area" class="form-control">
                  <option value="">-- Select One --</option>
                </select>
              </div>
            </div>
            <p class="mb-0 small text-muted text-right">請先選擇區域查看，綠色表示有口罩</p>
          </div>
          <ul class="list-group">
            <template>
              <a class="list-group-item text-left">
                <h3>藥局名稱</h3>
                <p class="mb-0">
                  成人口罩：1 ｜ 孩童口罩：2
                </p>
                <p class="mb-0">
                  地址：
                  <a href="https://www.google.com.tw/maps/place/..." target="_blank" title="Google Map">
                    地址
                  </a>
                </p>
              </a>
            </template>
          </ul>
        </div>
      </div>
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
import L from 'leaflet';
import cityName from './assets/cityName.json';

let osmMap = {};

export default {
  name: 'App',
  data: () => ({
    data: [],
    cityName,
    select: {
      city: '臺北市',
    },
  }),
  methods: {
    updateMap() {
      const pharmacies = this.data.filter((pharmacy) => (
        pharmacy.properties.county === this.select.city));
      pharmacies.forEach((pharmacy) => {
        const { properties, geometry } = pharmacy;
        L.marker([
          geometry.coordinates[1],
          geometry.coordinates[0],
        ]).addTo(osmMap).bindPopup(`
            <h4>${properties.name}</h4>
            <p>電話：${properties.phone}</p>
            <p>地址：${properties.address}</p>
            <p>成人口罩數量：${properties.mask_adult}</p>
            <p>孩童口罩數量：${properties.mask_child}</p>
            <p>更新時間：${properties.updated}</p>
        `);
        console.log(properties);
      });
      this.panTo(pharmacies[0]);
    },
    removeMarker() {
      osmMap.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          osmMap.removeLayer(layer);
        }
      });
    },
    panTo(item) {
      const { properties, geometry } = item;
      console.log(properties);
      osmMap.panTo([geometry.coordinates[1], geometry.coordinates[0]]);
    },
  },
  mounted() {
    const url = 'https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json';
    this.$http.get(url).then((response) => {
      this.data = response.data.features;
      this.updateMap();
    });
    osmMap = L.map('map', {
      center: [25.03, 121.55],
      zoom: 15,
    });
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      maxZoom: 18,
    }).addTo(osmMap);

    L.marker([25.03, 121.55]).addTo(osmMap);
  },
};
</script>

<style lang="scss">
@import 'bootstrap/scss/bootstrap';

#map{ height: 100%; }
.highlight{ background: #e9ffe3; }
.toolbox{ height: 100vh; overflow-y: auto; a{ cursor: pointer; } }
.leaflet-popup-content-wrapper{
  width: 350px; h4{width: 200px;} p{font-size: 16px; width: 320px;}
  }
</style>
