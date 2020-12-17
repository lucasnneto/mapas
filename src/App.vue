<template>
  <v-app>
    <v-main>
      <div class="d-flex justify-space-around">
        <l-map
          style="height: 500px; width: 500px"
          :zoom="zoom"
          :center="center"
          ref="map"
        >
          <l-tile-layer :url="url"></l-tile-layer>
        </l-map>
        <v-simple-table style="width: 50%">
          <thead>
            <tr>
              <th>id</th>
              <th>Nome</th>
              <th>Qtd de pontos</th>
              <th>Área</th>
              <th>Cor</th>
            </tr>
          </thead>
          <tbody>
            <tr @click="select(item)" v-for="item in polygon" :key="item.id">
              <td>{{ item.id }}</td>
              <td>{{ item.nome }}</td>
              <td>{{ item.latlng.length }}</td>
              <td>{{ item.area }}</td>
              <td>{{ item.cor }}</td>
            </tr>
          </tbody>
        </v-simple-table>
      </div>
      <v-simple-table>
        <thead>
          <tr>
            <th>Lat</th>
            <th>Lng</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, pos) in selecionado" :key="pos">
            <td>{{ item.lat }}</td>
            <td>{{ item.lng }}</td>
          </tr>
        </tbody>
      </v-simple-table>
    </v-main>
  </v-app>
</template>

<script>
import L from "leaflet";
import "leaflet-draw";
import { LMap, LTileLayer } from "vue2-leaflet";
export default {
  name: "App",

  components: {
    LMap,
    LTileLayer,
  },

  mounted() {
    this.$nextTick(() => {
      const map = this.$refs.map.mapObject;
      var drawnItems = new L.FeatureGroup();
      map.addLayer(drawnItems);
      const drawControl = new window.L.Control.Draw({
        position: "topright",
        edit: {
          featureGroup: drawnItems,
          edit: {
            // this property shouldn't be needed
            selectedPathOptions: {
              // this property should be one level up
              color: "#000",
              fillColor: "#000",
            },
          },
        },
        draw: {
          polygon: {
            allowIntersection: false,
            showArea: true,
          },
          polyline: false,
          rectangle: false,
          circle: false,
          circlemarker: false,
          marker: false,
        },
      });

      map.addControl(drawControl);
      let vm = this;

      map.on(L.Draw.Event.CREATED, function (e) {
        var layer = e.layer;
        drawnItems.addLayer(layer);
        layer.on("click", function (e) {
          // do something here like display a popup
          const index = vm.polygon.findIndex((el) => {
            return el.id == e.target._leaflet_id;
          });
          if (index != -1) vm.select(vm.polygon[index]);
        });
        vm.polygon.push({
          id: layer._leaflet_id,
          nome: `Area ${vm.polygon.length}`,
          latlng: layer.getLatLngs()[0],
          area: L.GeometryUtil.formattedNumber(
            L.GeometryUtil.geodesicArea(layer.getLatLngs()[0]),
            2
          ),
          cor: layer.options.color,
        });
      });

      map.on("draw:edited", function (e) {
        var layers = e.layers;
        layers.eachLayer(function (layer) {
          const index = vm.polygon.findIndex((el) => {
            return el.id == layer._leaflet_id;
          });
          if (index != -1) {
            vm.polygon.splice(index, 1, {
              id: layer._leaflet_id,
              nome: `Area ${vm.polygon.length}`,
              latlng: layer.getLatLngs()[0],
              area: L.GeometryUtil.formattedNumber(
                L.GeometryUtil.geodesicArea(layer.getLatLngs()[0]),
                2
              ),
              cor: layer.options.color,
            });
          }
        });
      });

      map.on("draw:deleted", function (e) {
        var layers = e.layers;
        layers.eachLayer(function (layer) {
          const index = vm.polygon.findIndex((el) => {
            return el.id == layer._leaflet_id;
          });
          if (index != -1) {
            vm.polygon.splice(index, 1);
          }
        });
      });

      map.on("draw:markercontext", function (e) {
        console.log(e);
      });
    });
  },

  data: () => ({
    url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
    //url: "http://{s}.tile.osm.org/{z}/{x}/{y}.png",
    zoom: 14,
    center: [-19.7532845, -47.9363265],
    polygon: [],
    selecionado: [],
  }),
  methods: {
    select(item) {
      this.selecionado = item.latlng;
    },
  },
};
</script>
