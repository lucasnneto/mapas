<template>
  <v-app>
    <v-main>
      <div class="mt-4 mx-6">
        <div class="d-flex justify-space-between">
          <h2 class="font-weight-bold">Fazenda Olhos d’Água</h2>
          <v-btn outlined>Adicionar</v-btn>
        </div>
        <v-row>
          <v-col>
            <mapa
              :polygons.sync="lista"
              @layer="Layer"
              :zoom="13"
              :telha="telhas[1]"
              :center="center"
              height="500px"
            />
            <!-- :createPolygon.sync="createPoli"
              :createSubPolygon.sync="createPoli2"
              edit -->
          </v-col>
          <v-col>
            <div class="d-flex align-center mb-4">
              <h3 class="mr-6">Informações</h3>
              <v-btn
                outlined
                small
                icon
                tile
                class="rounded"
                color="blue"
                @click="edit(1)"
                ><v-icon>mdi-plus</v-icon></v-btn
              >
            </div>
            <div class="d-flex justify-start mb-4">
              <div class="mr-6">
                <span class="body-2 font-font-weight-medium">Área Total</span>
                <div>
                  <span class="body-3">547,21 </span
                  ><span class="body-2 font-font-weight-medium">ha</span>
                </div>
              </div>
              <div class="mr-6">
                <span class="body-2 font-font-weight-medium"
                  >Área Consolidada</span
                >
                <div>
                  <span class="body-3">435 </span
                  ><span class="body-2 font-font-weight-medium">ha</span>
                </div>
              </div>
              <div>
                <span class="body-2 font-font-weight-medium">Reserva</span>
                <div>
                  <span class="body-3">112,21 </span
                  ><span class="body-2 font-font-weight-medium">ha</span>
                </div>
              </div>
            </div>
            <div class="d-flex justify-start mb-4">
              <div class="mr-6 d-flex flex-column">
                <span class="body-2 font-font-weight-medium">Matrícula</span>
                <span class="body-3">27.002</span>
                <span class="body-3">209234</span>
              </div>
              <div class="mr-6 d-flex flex-column">
                <div>
                  <span class="body-2 font-font-weight-medium"
                    >Núm. INCRA
                  </span>
                  <v-btn icon x-small
                    ><v-img
                      class="ml-2"
                      x-small
                      height="16px"
                      width="14px"
                      src="@/assets/download.svg"
                    >
                    </v-img
                  ></v-btn>
                </div>
                <v-icon></v-icon>
                <span class="body-3">4141230105453</span>
                <span class="body-3">4141230132507</span>
              </div>
              <div class="mr-6 d-flex flex-column">
                <div>
                  <span class="body-2 font-font-weight-medium">NIRF</span>
                  <v-btn icon x-small
                    ><v-img
                      class="ml-2"
                      x-small
                      height="16px"
                      width="14px"
                      src="@/assets/download.svg"
                    >
                    </v-img
                  ></v-btn>
                </div>
                <span class="body-3">12345678900</span>
              </div>
            </div>
            <div class="d-flex flex-column justify-start mb-4">
              <div class="d-flex align-center">
                <span class="body-2 font-font-weight-medium">CAR</span>
                <v-btn icon x-small
                  ><v-img
                    class="ml-2"
                    x-small
                    height="16px"
                    width="14px"
                    src="@/assets/download.svg"
                  >
                  </v-img
                ></v-btn>
              </div>
              <span class="body-3"
                >UF-1231234-1234.1234.1234.1234.1234.1234.1234.1234</span
              >
            </div>
            <div class="d-flex align-center mb-4">
              <h3 class="mr-6">Talhão</h3>
              <v-btn
                @click="edit(2)"
                outlined
                small
                icon
                tile
                class="rounded"
                color="blue"
                ><v-icon>mdi-plus</v-icon></v-btn
              >
            </div>
            <div class="d-flex align-center mb-4">
              <v-data-table
                dense
                :headers="headers"
                :items="talhao"
                item-key="id"
                hide-default-footer
                style="width: 100%"
              >
                <template v-slot:[`item.detalhes`]="{ item }">
                  <span
                    class="blue--text"
                    style="cursor: pointer"
                    @click="dtl(item)"
                    >Detalhes
                  </span>
                </template>
              </v-data-table>
            </div>
          </v-col>
        </v-row>
      </div>

      <!-- <div class="d-flex justify-space-around">
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
      </v-simple-table> -->

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
      <modal
        v-if="d"
        :dialog="d"
        :type="type"
        @dialog="d = false"
        @lista="att"
        :lista.sync="listamodal"
      />
      <detalhes :dialog="d2" v-if="d2" :dados="dados" @dialog="d2 = false" />
    </v-main>
  </v-app>
</template>

<script>
// import L from "leaflet";
// import "leaflet-draw";
// import { LMap, LTileLayer, LControlLayers } from "vue2-leaflet";
import modal from "./modal.vue";
import detalhes from "./detalhes.vue";
import mapa from "./mapa.vue";
export default {
  name: "App",
  computed: {
    talhao() {
      return this.lista.filter((e) => {
        return e.type == "TALHAO";
      });
    },
  },
  components: {
    mapa,
    modal,
    detalhes,
    // LMap,
    // LTileLayer,
    // LControlLayers,
  },

  // mounted() {
  //   this.$nextTick(() => {
  //     const map = this.$refs.map.mapObject;
  //     var drawnItems = new L.FeatureGroup();
  //     map.addLayer(drawnItems);
  //     const drawControl = new window.L.Control.Draw({
  //       position: "topright",
  //       edit: {
  //         featureGroup: drawnItems,
  //         edit: {},
  //       },
  //       draw: {
  //         polygon: {
  //           allowIntersection: false,
  //           showArea: true,
  //           metric: true,
  //           feet: false,
  //         },
  //         polyline: false,
  //         rectangle: false,
  //         circle: false,
  //         circlemarker: false,
  //         marker: false,
  //       },
  //     });

  //     map.addControl(drawControl);
  //     //
  //     //https://stackoverflow.com/questions/48661724/leaflet-draw-edit-control-custom-text
  //     //
  //     L.drawLocal.draw.toolbar = {
  //       actions: {
  //         title: "Cancelar",
  //         text: "Cancelar",
  //       },
  //       finish: {
  //         title: "Finalizar",
  //         text: "Finalizar",
  //       },
  //       undo: {
  //         title: "Desfazer",
  //         text: "Desfazer ultimo ponto",
  //       },
  //       buttons: {
  //         polygon: "Desenhar um polígono",
  //       },
  //     };
  //     L.drawLocal.draw.handlers = {
  //       polygon: {
  //         tooltip: {
  //           start: "Clique para começar a desenhar.",
  //           cont: "Clique para continuar a desenhar.",
  //           end: "Clique no primeiro ponto para finalizar.",
  //         },
  //       },
  //       simpleshape: {
  //         tooltip: {
  //           end: "Solte o mouse para terminar de desenhar.",
  //         },
  //       },
  //     };

  //     L.drawLocal.edit = {
  //       toolbar: {
  //         actions: {
  //           save: {
  //             title: "Salvar Mudanças",
  //             text: "Salvar",
  //           },
  //           cancel: {
  //             title: "Cancelar edição e desfazer mudanças",
  //             text: "Cancelar",
  //           },
  //           clearAll: {
  //             title: "Limpar tudo",
  //             text: "Limpar tudo",
  //           },
  //         },
  //         buttons: {
  //           edit: "Editar Layers",
  //           editDisabled: "Sem Layers para edição",
  //           remove: "Deletar Layers",
  //           removeDisabled: "Sem Layers para deleção",
  //         },
  //       },
  //       handlers: {
  //         edit: {
  //           tooltip: {
  //             text: "Arraste pontas para editar recursos.",
  //             subtext: "Clique para cancelar mudanças",
  //           },
  //         },
  //         remove: {
  //           tooltip: {
  //             text: "Clique para remover.",
  //           },
  //         },
  //       },
  //     };

  //     let vm = this;

  //     map.on(L.Draw.Event.CREATED, function (e) {
  //       var layer = e.layer;
  //       drawnItems.addLayer(layer);
  //       layer.on("click", (e) => {
  //         if (!vm.deletable) {
  //           const index = vm.polygon.findIndex((el) => {
  //             return el.id == e.target._leaflet_id;
  //           });
  //           if (index != -1) {
  //             if (!vm.editable) {
  //               vm.select(vm.polygon[index]);
  //               vm.dialog2 = true;
  //             } else {
  //               vm.item = vm.polygon[index];
  //               vm.layers = drawnItems._leaflet_id;
  //               vm.dialog = true;
  //             }
  //           }
  //         }
  //       });
  //       vm.polygon.push({
  //         id: layer._leaflet_id,
  //         nome: `Area ${vm.polygon.length}`,
  //         latlng: layer.getLatLngs()[0],
  //         area: L.GeometryUtil.formattedNumber(
  //           L.GeometryUtil.geodesicArea(layer.getLatLngs()[0]),
  //           2
  //         ),
  //         cor: layer.options.color,
  //       });
  //       vm.item = vm.polygon[vm.polygon.length - 1];
  //       vm.layers = drawnItems._leaflet_id;
  //       vm.dialog = true;
  //     });
  //     map.on("draw:editstart", function () {
  //       vm.editable = true;
  //       vm.deletable = false;
  //     });
  //     map.on("draw:editstop", function () {
  //       vm.polygon.forEach((e) => {
  //         vm.$refs.map.mapObject._layers[e.id].setStyle({
  //           color: e.cor,
  //         });
  //       });
  //       vm.editable = false;
  //       vm.deletable = false;
  //     });
  //     map.on("draw:deletestop", function () {
  //       vm.editable = false;
  //       vm.deletable = false;
  //     });
  //     map.on("draw:deletestart", function () {
  //       vm.editable = false;
  //       vm.deletable = true;
  //     });
  //     map.on("draw:edited", function (e) {
  //       vm.editable = false;
  //       var layers = e.layers;
  //       layers.eachLayer(function (layer) {
  //         const index = vm.polygon.findIndex((el) => {
  //           return el.id == layer._leaflet_id;
  //         });
  //         if (index != -1) {
  //           vm.polygon.splice(index, 1, {
  //             id: layer._leaflet_id,
  //             nome: `Area ${vm.polygon.length}`,
  //             latlng: layer.getLatLngs()[0],
  //             area: L.GeometryUtil.formattedNumber(
  //               L.GeometryUtil.geodesicArea(layer.getLatLngs()[0]),
  //               2
  //             ),
  //             cor: layer.options.color,
  //           });
  //         }
  //       });
  //     });

  //     map.on("draw:deleted", function (e) {
  //       vm.deletable = false;
  //       var layers = e.layers;
  //       layers.eachLayer(function (layer) {
  //         const index = vm.polygon.findIndex((el) => {
  //           return el.id == layer._leaflet_id;
  //         });
  //         if (index != -1) {
  //           vm.polygon.splice(index, 1);
  //         }
  //       });
  //     });
  //   });
  // },

  data: () => ({
    headers: [
      { text: "Cultura", value: "name", sortable: false },
      { text: "Área", value: "area", sortable: false },
      { text: "Safra", value: "safra", sortable: false },
      { text: "", value: "detalhes", sortable: false, align: "end" },
    ],
    items: [
      {
        cultura: "Soja",
        area: "152,45 ha",
        safra: "20/21",
      },
      {
        cultura: "Soja",
        area: "152,45 ha",
        safra: "20/21",
      },
    ],
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
          "https://api.maptiler.com/maps/hybrid/{z}/{x}/{y}.jpg?key=Y7PaX0PVfTePQJc5eDua",
      },
      {
        name: "Dark",
        visible: false,
        url:
          "https://api.maptiler.com/maps/darkmatter/{z}/{x}/{y}.png?key=Y7PaX0PVfTePQJc5eDua",
      },
    ],

    zoom: 16,
    center: [-17.2633538, -46.8311499],
    polygon: [],
    selecionado: {},
    dialog: false,
    dialog2: false,
    menu: false,
    item: {},
    editable: false,
    deletable: false,
    layersid: 0,
    createPoli: false,
    createPoli2: false,
    type: "",
    lista: [],
    listamodal: [],
    d: false,
    d2: false,
    dados: {},
  }),
  methods: {
    Layer(e) {
      console.log(e);
    },
    newPolygon(t) {
      if (t == 1) this.createPoli = !this.createPoli;
      else this.createPoli2 = !this.createPoli2;
    },
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
    edit(a) {
      if (a == 1) {
        this.type = "TERRENO";
        this.listamodal = this.lista.filter((e) => {
          return e.type == "TERRENO";
        });
      } else {
        this.type = "TALHAO";
        this.listamodal = this.lista.filter((e) => {
          return e.type == "TALHAO";
        });
      }
      this.d = true;
    },
    att(e) {
      this.lista = this.lista.filter((el) => {
        return el.type != this.type;
      });
      this.lista = this.lista.concat(e);
    },
    dtl(item) {
      this.dados = item;
      this.d2 = true;
    },
  },
};
</script>
