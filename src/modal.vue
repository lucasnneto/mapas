<template>
  <v-dialog v-model="dialog" persistent max-width="850px">
    <v-card class="overflow-y-hidden">
      <v-card-title class="pb-0 pr-4 pt-4">
        <v-spacer />
        <v-btn icon width="30" height="30" @click="$emit('dialog', false)">
          <img width="14" height="14" src="@/assets/exit.svg" alt="" />
        </v-btn>
      </v-card-title>
      <div class="px-8">
        <h2>Editar {{ type == "TALHAO" ? "Talão" : "Terreno" }}</h2>
        <p class="body-3 mt-2 mb-6">
          Selecione um {{ type == "TALHAO" ? "Talão" : "Terreno" }} no mapa para
          editá-lo.
        </p>
      </div>
      <div class="d-flex px-8">
        <div class="pr-8 mb-8">
          <v-lazy>
            <mapa
              @created="
                (e) => {
                  created = !created;
                }
              "
              :polygons="lista"
              :zoom="13"
              :telha="telha"
              :center="center"
              @clickPolygon="clickPolygon"
              :createPolygon.sync="newPoli"
              :createSubPolygon.sync="newSubPoli"
              @lista="
                (e) => {
                  this.poli = e;
                }
              "
              height="50vh"
              width="60vh"
              edit
            />
          </v-lazy>
        </div>
        <div
          class="d-flex flex-column justify-space-between"
          style="width: 100%"
        >
          <v-flex class="pb-0">
            <v-flex v-if="tela == 0">
              <v-data-table
                class="overflow-y-auto"
                dense
                fixed-header
                :headers="headers"
                :items="poli"
                item-key="id"
                hide-default-footer
                style="width: 100%; height: 28vh"
                @click:row="clickPolygon"
              >
                <template v-slot:[`item.area`]="{ item }">
                  {{ area(item) }}
                </template>
              </v-data-table>
            </v-flex>
            <v-form v-else>
              <p class="body-3 mt-2 mb-8">
                <v-icon @click="tela = 0">mdi-chevron-left </v-icon>
                Insira o nome do terreno e selecione a cor.
              </p>
              <v-text-field label="Nome do terreno" v-model="selected.name" />
              <v-menu
                ref="menu"
                :close-on-content-click="false"
                :close-on-click="false"
                :return-value.sync="selected.cor"
                v-model="menu"
                min-width="290px"
                class="white"
              >
                <template v-slot:activator="{ on, attrs }">
                  <div class="d-flex align-center">
                    <v-sheet
                      v-bind="attrs"
                      v-on="on"
                      :color="selected.cor"
                      class="rounded-circle mr-3 mb-2"
                      elevation="1"
                      height="25"
                      width="25"
                    ></v-sheet>
                    <v-text-field
                      v-bind="attrs"
                      v-on="on"
                      label="Cor"
                      v-model="selected.cor"
                      readonly
                    ></v-text-field>
                  </div>
                </template>
                <v-color-picker v-model="selected.cor" mode="hexa">
                </v-color-picker>
                <div class="white pb-2 d-flex justify-space-between">
                  <v-btn text color="primary" @click="menu = false">
                    Cancelar
                  </v-btn>
                  <v-btn
                    text
                    color="primary"
                    @click="$refs.menu.save(selected.cor)"
                  >
                    OK
                  </v-btn>
                </div>
              </v-menu>
              <!-- <v-text-field label="Cor do terreno" v-model="selected.cor" /> -->
              <v-text-field
                v-if="!isNew"
                label="Área aproximada"
                v-model="selected.area"
                readonly
              />
            </v-form>
          </v-flex>
          <v-card-actions v-if="tela == 0" class="px-0 py-8 d-flex flex-column">
            <v-btn
              @click="novoTerreno()"
              size="14"
              class="mb-5"
              block
              height="48"
              outlined
            >
              <img class="mr-4" src="@/assets/btn_add.svg" alt />
              <span class="font-weight-medium primary--text"
                >Adicionar {{ type == "TALHAO" ? "Talão" : "Terreno" }}</span
              >
            </v-btn>
            <v-btn size="20" block height="48" @click="saveAll">
              <span class="font-weight-bold">Salvar Tudo</span>
            </v-btn>
          </v-card-actions>
          <v-card-actions v-else class="px-0 py-8">
            <v-btn @click="salvar()" size="20" block height="48">
              <span class="font-weight-bold">{{
                isNew ? "Criar Pontos" : "Salvar"
              }}</span>
            </v-btn>
          </v-card-actions>
        </div>
      </div>
    </v-card>
  </v-dialog>
</template>

<script>
import L from "leaflet";
import mapa from "./mapa.vue";
export default {
  name: "modal",
  components: {
    mapa,
  },
  props: {
    dialog: {
      type: Boolean,
      default: false,
    },
    type: {
      type: String,
      default: "",
    },
    lista: {
      type: Array,
      default: () => {
        return [];
      },
    },
  },
  watch: {
    created() {
      this.tela = 0;
    },
  },
  mounted() {
    this.poli = this.lista.map((e) => {
      return {
        ...e,
        type: this.type,
      };
    });
  },
  data() {
    return {
      created: false,
      menu: false,
      newPoli: {},
      newSubPoli: {},
      headers: [
        { text: "Id", value: "id", sortable: false },
        { text: "Nome", value: "name", sortable: false },
        { text: "Cor", value: "cor", sortable: false },
        { text: "Área", value: "area", sortable: false },
      ],
      tela: 0,
      isNew: false,
      selected: {
        id: "",
        name: "",
        cor: "#FFFFFF",
        area: "",
        latlngs: [],
      },
      telha: {
        name: "OpenStreetMap",
        url:
          "https://api.maptiler.com/maps/hybrid/{z}/{x}/{y}.jpg?key=Y7PaX0PVfTePQJc5eDua",
      },
      center: [-17.2633538, -46.8311499],
      poli: [],
    };
  },
  methods: {
    clickPolygon(a) {
      this.selected = {
        id: a.id,
        name: a.name,
        cor: a.cor,
        area: a.area,
        latlngs: a.latlngs,
      };
      this.isNew = false;
      this.tela = 1;
    },
    area(item) {
      let a;
      if (Array.isArray(item.latlngs[0])) {
        a = item.latlngs.map((e) => {
          return L.latLng(e[0], e[1]);
        });
      } else a = item.latlngs;
      return L.GeometryUtil.formattedNumber(L.GeometryUtil.geodesicArea(a), 2);
    },
    novoTerreno() {
      this.selected = {
        id: "",
        name: "",
        cor: "#FFFFFF",
        area: "",
        latlngs: [],
      };
      this.isNew = true;
      this.tela = 1;
    },
    salvar() {
      if (this.type == "TALHAO") this.newSubPoli = this.selected;
      else this.newPoli = this.selected;
      if (!this.isNew) this.tela = 0;
    },
    saveAll() {
      this.$emit(
        "lista",
        this.poli.map((e) => {
          return {
            ...e,
            type: this.type,
          };
        })
      );
      this.$emit("dialog", false);
    },
  },
};
</script>

<style scoped>
</style>