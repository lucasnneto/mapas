<template>
  <v-app>
    <v-main>
      <div class="d-flex justify-space-around">
        <l-map
          style="height: 500px; z-index: 0"
          :zoom="zoom"
          :center="center"
          ref="map"
        >
          <l-control-layers position="bottomright"></l-control-layers>
          <l-tile-layer
            v-for="tileProvider in telhas"
            :key="tileProvider.name"
            :name="tileProvider.name"
            :visible="tileProvider.visible"
            :url="tileProvider.url"
            layer-type="base"
          ></l-tile-layer>
        </l-map>
      </div>

      <v-simple-table>
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
          <tr
            style="cursor: pointer"
            @click="detalhes(item)"
            v-for="item in polygon"
            :key="item.id"
          >
            <td>{{ item.id }}</td>
            <td>{{ item.nome }}</td>
            <td>{{ item.latlng.length }}</td>
            <td>{{ item.area }} m²</td>
            <td>{{ item.cor }}</td>
          </tr>
        </tbody>
      </v-simple-table>

      <v-dialog v-model="dialog" persistent max-width="450">
        <v-card>
          <v-card-title primary-title>
            {{ editable ? "Editar Polígono" : "Criar Polígono" }}
          </v-card-title>
          <v-card-text>
            <v-text-field label="Nome" v-model="item.nome"></v-text-field>
            <v-menu
              ref="menu"
              :close-on-content-click="false"
              :close-on-click="false"
              :return-value.sync="item.cor"
              v-model="menu"
              min-width="290px"
              class="white"
            >
              <template v-slot:activator="{ on, attrs }">
                <div class="d-flex align-center">
                  <v-sheet
                    v-bind="attrs"
                    v-on="on"
                    :color="item.cor"
                    class="rounded-circle mr-3 mb-2"
                    elevation="1"
                    height="25"
                    width="25"
                  ></v-sheet>
                  <v-text-field
                    v-bind="attrs"
                    v-on="on"
                    label="Cor"
                    v-model="item.cor"
                    readonly
                  ></v-text-field>
                </div>
              </template>
              <v-color-picker v-model="item.cor" mode="hexa"> </v-color-picker>
              <div class="white pb-2 d-flex justify-space-between">
                <v-btn text color="primary" @click="menu = false">
                  Cancelar
                </v-btn>
                <v-btn text color="primary" @click="$refs.menu.save(item.cor)">
                  OK
                </v-btn>
              </div>
            </v-menu>
          </v-card-text>
          <v-card-actions class="d-flex justify-space-between">
            <!-- <v-btn text color="primary" @click="dialog = false">
              Cancelar
            </v-btn> -->
            <v-spacer></v-spacer>
            <v-btn text color="primary" @click="salvar()"> OK </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-dialog v-model="dialog2" max-width="450">
        <v-card>
          <v-card-title primary-title>
            {{ selecionado.id }} - {{ selecionado.nome }}
          </v-card-title>
          <v-card-text>
            <v-simple-table>
              <thead>
                <tr>
                  <th>Lat</th>
                  <th>Lng</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(item, pos) in selecionado.latlng" :key="pos">
                  <td>{{ item.lat }}</td>
                  <td>{{ item.lng }}</td>
                </tr>
              </tbody>
            </v-simple-table>
          </v-card-text>
          <v-card-actions>
            <v-btn text color="primary" @click="dialog2 = false">
              Fechar
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-main>
  </v-app>
</template>

<script>
import L from "leaflet";
import "leaflet-draw";
import { LMap, LTileLayer, LControlLayers } from "vue2-leaflet";
export default {
  name: "App",

  components: {
    LMap,
    LTileLayer,
    LControlLayers,
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
            // selectedPathOptions: {
            //   color: "#000",
            //   fillColor: "#000",
            // },
          },
        },
        draw: {
          polygon: {
            allowIntersection: false,
            showArea: true,
            metric: true,
            feet: false,
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
        layer.on("click", (e) => {
          if (!vm.deletable) {
            const index = vm.polygon.findIndex((el) => {
              return el.id == e.target._leaflet_id;
            });
            if (index != -1) {
              if (!vm.editable) {
                vm.select(vm.polygon[index]);
                vm.dialog2 = true;
              } else {
                vm.item = vm.polygon[index];
                vm.layers = drawnItems._leaflet_id;
                vm.dialog = true;
              }
            }
          }
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
        vm.item = vm.polygon[vm.polygon.length - 1];
        vm.layers = drawnItems._leaflet_id;
        vm.dialog = true;
      });
      map.on("draw:editstart", function () {
        vm.editable = true;
        vm.deletable = false;
      });
      map.on("draw:editstop", function () {
        vm.polygon.forEach((e) => {
          vm.$refs.map.mapObject._layers[e.id].setStyle({
            color: e.cor,
          });
        });
        vm.editable = false;
        vm.deletable = false;
      });
      map.on("draw:deletestop", function () {
        vm.editable = false;
        vm.deletable = false;
      });
      map.on("draw:deletestart", function () {
        vm.editable = false;
        vm.deletable = true;
      });
      map.on("draw:edited", function (e) {
        vm.editable = false;
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
        vm.deletable = false;
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
    });
  },

  data: () => ({
    telhas: [
      {
        name: "OpenStreetMap",
        visible: true,
        url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      },
      {
        name: "Toner",
        visible: false,
        url:
          "https://api.maptiler.com/maps/toner/{z}/{x}/{y}.png?key=Y7PaX0PVfTePQJc5eDua",
      },
      {
        name: "Dark",
        visible: false,
        url:
          "https://api.maptiler.com/maps/darkmatter/{z}/{x}/{y}.png?key=Y7PaX0PVfTePQJc5eDua",
      },
    ],

    zoom: 16,
    center: [-19.7532845, -47.9363265],
    polygon: [],
    selecionado: {},
    dialog: false,
    dialog2: false,
    menu: false,
    item: {},
    editable: false,
    deletable: false,
    layersid: 0,
  }),
  methods: {
    select(item) {
      this.selecionado = item;
    },
    salvar() {
      this.$refs.map.mapObject._layers[this.item.id].setStyle({
        color: this.item.cor,
      });

      this.dialog = false;
    },
    detalhes(e) {
      const index = this.polygon.findIndex((el) => {
        return el.id == e.id;
      });
      if (index != -1) {
        this.select(this.polygon[index]);
        this.dialog2 = true;
      }
    },
  },
};
</script>
