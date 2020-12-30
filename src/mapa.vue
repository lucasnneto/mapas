<template>
  <div>
    <l-map
      style="
        height: 100vh;
        z-index: 0;
        width: 80vh;
        border-radius: 6px !important;
      "
      :style="{ height: height + '!important', width: width + '!important' }"
      :zoom="zoom"
      :center="center"
      :options="{ zoomControl: false, attributionControl: false }"
      ref="map"
    >
      <l-tile-layer
        :name="telha.name"
        :url="telha.url"
        layer-type="base"
      ></l-tile-layer>
      <l-control-zoom
        :position="zoomControl.position"
        zoomInText=""
        :zoomInTitle="zoomControl.inTitle"
        zoomOutText=""
        :zoomOutTitle="zoomControl.outTitle"
      ></l-control-zoom>
    </l-map>
  </div>
</template>

<script>
import L from "leaflet";
import "leaflet-draw";
import { LMap, LTileLayer, LControlZoom } from "vue2-leaflet";
export default {
  name: "mapa",
  components: {
    LMap,
    LTileLayer,
    LControlZoom,
  },
  props: {
    polygons: {
      type: Array,
      default: () => {
        return [];
      },
    },
    zoom: {
      type: Number,
      default: 15,
    },
    telha: {
      required: true,
      default: {
        name: "",
        url: "",
      },
    },
    center: {
      required: true,
    },
    height: String,
    width: String,
    zoomControl: {
      type: [Array, Object],
      default() {
        return {
          position: "bottomright",
          inTitle: "Mais Zoom",
          outTitle: "Menos Zoom",
        };
      },
    },
    edit: {
      type: Boolean,
      default: false,
    },
    createPolygon: {
      type: Object,
      default: () => {
        return {};
      },
    },
    createSubPolygon: {
      type: Object,
      default: () => {
        return {};
      },
    },
  },
  data() {
    return {
      map: {},
      drawControl: {},
      drawnItems: {},
      polygon: null,
      subpolygon: null,
      lista: [],
      item: {},
      deletable: false,
    };
  },
  watch: {
    polygons() {
      for (let el in this.drawnItems._layers) {
        // console.log(this.drawnItems._layers[el]);
        this.map.removeLayer(this.drawnItems._layers[el]);
      }
      this.polygons.forEach((e) => {
        var polygon = L.polygon(e.latlngs, {
          color: e.cor,
          opacity: e.type == "TERRENO" ? 0.8 : 0.5,
          fillOpacity: e.type == "TERRENO" ? 0 : 0.2,
        });
        polygon.addTo(this.drawnItems);
        //   const a = {
        //     ...e,
        //     id: layer._leaflet_id,
        //     cor: layer.options.color,
        //     area: L.GeometryUtil.formattedNumber(
        //       L.GeometryUtil.geodesicArea(layer._latlngs[0]),
        //       2
        //     ),
        //   };
        //   this.lista.push(a);
        //   this.$emit("lista", this.lista);
        //   polygon.on("click", (e) => {
        //     if (!this.deletable) {
        //       const index = this.lista.findIndex((el) => {
        //         return el.id == e.target._leaflet_id;
        //       });
        //       if (index != -1) {
        //         this.$emit("clickPolygon", this.lista[index]);
        //       }
        //     }
        //   });
      });
    },
    createPolygon(item) {
      this.item = item;
      if (item.id == "" || item.id == undefined) {
        if (this.polygon == null) {
          //https://stackoverflow.com/questions/15775103/leaflet-draw-mapping-how-to-initiate-the-draw-function-without-toolbar
          this.polygon = new L.Draw.Polygon(
            this.map,
            this.drawControl.options.polygon
          );
          this.polygon.enable();
        }
        // new L.Draw.Polygon(this.map, this.drawControl.options.polygon).disable();
      } else {
        this.lista = this.lista.map((e) => {
          if (e.id == this.item.id) return this.item;
          else return e;
        });
        this.$refs.map.mapObject._layers[this.item.id].setStyle({
          color: this.item.cor,
        });
        this.$emit("lista", this.lista);
      }
    },
    createSubPolygon(item) {
      this.item = item;
      if (item.id == "" || item.id == undefined) {
        if (this.subpolygon == null) {
          //https://stackoverflow.com/questions/15775103/leaflet-draw-mapping-how-to-initiate-the-draw-function-without-toolbar
          this.subpolygon = new L.Draw.Polygon(
            this.map,
            this.drawControl.options.polygon
          );
          this.subpolygon.enable();
        }
      } else {
        this.lista = this.lista.map((e) => {
          if (e.id == this.item.id) return this.item;
          else return e;
        });
        this.$refs.map.mapObject._layers[this.item.id].setStyle({
          color: this.item.cor,
        });
        this.$emit("lista", this.lista);
      }
    },
  },
  mounted() {
    this.$nextTick(() => {
      this.map = this.$refs.map.mapObject;
      this.drawnItems = new L.FeatureGroup();
      this.map.addLayer(this.drawnItems);
      this.drawControl = new window.L.Control.Draw({
        position: this.zoomControl.position,
        edit: {
          featureGroup: this.drawnItems,
        },
        draw: {
          polygon: false,
          polyline: false,
          rectangle: false,
          circle: false,
          circlemarker: false,
          marker: false,
        },
      });
      L.drawLocal.draw.toolbar = {
        actions: {
          title: "Cancelar",
          text: "Cancelar",
        },
        finish: {
          title: "Finalizar",
          text: "Finalizar",
        },
        undo: {
          title: "Desfazer",
          text: "Desfazer ultimo ponto",
        },
        buttons: {
          polygon: "Desenhar um polígono",
        },
      };
      L.drawLocal.draw.handlers.polygon = {
        tooltip: {
          start: "Clique para começar a desenhar.",
          cont: "Clique para continuar a desenhar.",
          end: "Clique no primeiro ponto para finalizar.",
        },
      };
      (L.drawLocal.draw.handlers.simpleshape = {
        tooltip: {
          end: "Solte o mouse para terminar de desenhar.",
        },
      }),
        (L.drawLocal.edit = {
          toolbar: {
            actions: {
              save: {
                title: "Salvar Mudanças",
                text: "Salvar",
              },
              cancel: {
                title: "Cancelar edição e desfazer mudanças",
                text: "Cancelar",
              },
              clearAll: {
                title: "Limpar tudo",
                text: "Limpar tudo",
              },
            },
            buttons: {
              edit: "Editar Layers",
              editDisabled: "Sem Layers para edição",
              remove: "Deletar Layers",
              removeDisabled: "Sem Layers para deleção",
            },
          },
          handlers: {
            edit: {
              tooltip: {
                text: "Arraste pontas para editar recursos.",
                subtext: "Clique para cancelar mudanças",
              },
            },
            remove: {
              tooltip: {
                text: "Clique para remover.",
              },
            },
          },
        });
      if (this.edit) this.map.addControl(this.drawControl);
      this.map.on("draw:created", (e) => {
        var layer = e.layer;
        if (this.polygon != null)
          layer.setStyle({
            color: this.item.cor,
            opacity: 0.8,
            fillOpacity: 0.0,
          });
        if (this.subpolygon != null) layer.setStyle({ color: this.item.cor });
        this.drawnItems.addLayer(layer);
        const a = {
          name: this.item.name,
          id: layer._leaflet_id,
          cor: layer.options.color,
          area: L.GeometryUtil.formattedNumber(
            L.GeometryUtil.geodesicArea(layer._latlngs[0]),
            2
          ),
          latlngs: layer._latlngs[0],
        };
        layer.on("click", (e) => {
          if (!this.deletable) {
            const index = this.lista.findIndex((el) => {
              return el.id == e.target._leaflet_id;
            });
            if (index != -1) {
              this.$emit("clickPolygon", this.lista[index]);
            }
          }
        });
        this.lista.push(a);
        this.$emit("lista", this.lista);
        this.polygon = null;
        this.subpolygon = null;
        this.$emit("created", true);
      });
      this.map.on("draw:edited", (e) => {
        var layers = e.layers;
        this.$emit("layer", layers);
      });
      this.map.on("draw:deletestop", () => {
        this.deletable = false;
      });
      this.map.on("draw:deleted", (e) => {
        this.deletable = false;
        var layers = e.layers;
        layers.eachLayer((layer) => {
          const index = this.lista.findIndex((el) => {
            return el.id == layer._leaflet_id;
          });
          if (index != -1) {
            this.lista.splice(index, 1);
          }
        });
        this.$emit("lista", this.lista);
      });
      this.map.on("draw:edited", (e) => {
        this.editable = false;
        var layers = e.layers;
        layers.eachLayer((layer) => {
          const index = this.lista.findIndex((el) => {
            return el.id == layer._leaflet_id;
          });
          if (index != -1) {
            const n = this.lista[index].name;
            this.lista.splice(index, 1, {
              id: layer._leaflet_id,
              name: n,
              latlngs: layer.getLatLngs()[0],
              area: L.GeometryUtil.formattedNumber(
                L.GeometryUtil.geodesicArea(layer.getLatLngs()[0]),
                2
              ),
              cor: layer.options.color,
            });
          }
        });
        this.$emit("lista", this.lista);
      });
      this.map.on("draw:deletestart", () => {
        this.deletable = true;
      });
      this.polygons.forEach((e) => {
        var polygon = L.polygon(e.latlngs, {
          color: e.cor,
          opacity: e.type == "TERRENO" ? 0.8 : 1,
          fillOpacity: e.type == "TERRENO" ? 0 : 0.5,
        });
        const layer = polygon.addTo(this.drawnItems);
        // this.drawnItems.addLayer(polygon);
        // console.log(layer);
        const a = {
          ...e,
          id: layer._leaflet_id,
          cor: layer.options.color,
          area: L.GeometryUtil.formattedNumber(
            L.GeometryUtil.geodesicArea(layer._latlngs[0]),
            2
          ),
        };
        this.lista.push(a);
        this.$emit("lista", this.lista);
        polygon.on("click", (e) => {
          if (!this.deletable) {
            const index = this.lista.findIndex((el) => {
              return el.id == e.target._leaflet_id;
            });
            if (index != -1) {
              this.$emit("clickPolygon", this.lista[index]);
            }
          }
        });
      });
    });
  },
};
</script>

<style>
/*TODO Limpar CSS */
.leaflet-control-zoom {
  border: none;
  border-radius: 4px !important;
  background: rgba(38, 51, 87, 0.5);
}
.leaflet-bar a.leaflet-disabled {
  background-color: rgba(38, 51, 87, 0.5) !important;
}
.leaflet-touch .leaflet-control-zoom-in,
.leaflet-touch .leaflet-control-zoom-in:hover {
  width: 24px !important;
  background: url(./assets/zoomin.svg) no-repeat center center;
  text-decoration: none;
  border-bottom: none;
  margin-bottom: 5px;
}
.leaflet-touch .leaflet-control-zoom-out,
.leaflet-touch .leaflet-control-zoom-out:hover {
  width: 24px !important;
  background: url(./assets/zoomout.svg) no-repeat center center;
  text-decoration: none;
}
.leaflet-touch .leaflet-control-layers,
.leaflet-touch .leaflet-bar {
  border: none !important;
}
.leaflet-bar a,
.leaflet-bar a:hover {
  border-bottom: 0px !important;
}
.leaflet-draw-toolbar .leaflet-draw-edit-edit {
  border: 0px;
  border-top-left-radius: 4px !important;
  border-top-right-radius: 4px !important;
  background: rgba(38, 51, 87, 0.5) !important;
  background-image: url(./assets/edit.svg) !important;
  width: 25px !important;
  height: 25px !important;
  background-repeat: no-repeat !important;
  background-position: center !important;
}
.leaflet-draw-toolbar .leaflet-draw-edit-remove {
  border-bottom-left-radius: 4px !important;
  border-bottom-right-radius: 4px !important;
  background: rgba(38, 51, 87, 0.5) !important;
  background-image: url(./assets/delete.svg) !important;
  width: 25px !important;
  height: 25px !important;
  background-repeat: no-repeat !important;
  background-position: center !important;
}

.leaflet-draw-toolbar a.leaflet-draw-draw-polygon {
  border-radius: 4px !important;
  background-color: rgba(38, 51, 87, 0.5) !important;
  background-image: url(./assets/polygon.svg) !important;
  width: 25px !important;
  height: 25px !important;
  background-repeat: no-repeat !important;
  background-position: center !important;
  background-size: 300px 22px !important;
}
ul.leaflet-draw-actions li a {
  color: white;
}
</style>