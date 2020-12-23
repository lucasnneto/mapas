<template>
  <div>
    <l-map
      style="height: 100vh; z-index: 0"
      :style="{ height: height + '!important' }"
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
      type: Boolean,
      default: false,
    },
    createSubPolygon: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      map: {},
      drawControl: {},
      polygon: null,
      subpolygon: null,
    };
  },
  watch: {
    createPolygon() {
      if (this.polygon == null) {
        //https://stackoverflow.com/questions/15775103/leaflet-draw-mapping-how-to-initiate-the-draw-function-without-toolbar
        this.polygon = new L.Draw.Polygon(
          this.map,
          this.drawControl.options.polygon
        );
        this.polygon.enable();
      }
      // new L.Draw.Polygon(this.map, this.drawControl.options.polygon).disable();
    },
    createSubPolygon() {
      if (this.subpolygon == null) {
        //https://stackoverflow.com/questions/15775103/leaflet-draw-mapping-how-to-initiate-the-draw-function-without-toolbar
        this.subpolygon = new L.Draw.Polygon(
          this.map,
          this.drawControl.options.polygon
        );
        this.subpolygon.enable();
      }
    },
  },
  mounted() {
    this.$nextTick(() => {
      this.map = this.$refs.map.mapObject;
      var drawnItems = new L.FeatureGroup();
      this.map.addLayer(drawnItems);
      this.drawControl = new window.L.Control.Draw({
        position: "bottomright",
        edit: {
          featureGroup: drawnItems,
        },
        draw: {
          polygon: false,
          // polygon: {
          //   allowIntersection: false,
          //   showArea: true,
          //   metric: true,
          //   feet: false,
          // },
          polyline: false,
          rectangle: false,
          circle: false,
          circlemarker: false,
          marker: false,
        },
      });
      if (this.edit) this.map.addControl(this.drawControl);
      this.map.on("draw:created", (e) => {
        var layer = e.layer;
        this.$emit("layer", layer);
        if (this.polygon != null)
          layer.setStyle({ color: "#ff0000", opacity: 0.8, fillOpacity: 0 });
        if (this.subpolygon != null) layer.setStyle({ color: "#ffff00" });
        drawnItems.addLayer(layer);
        this.polygon = null;
        this.subpolygon = null;
      });
      this.map.on("draw:edited", (e) => {
        var layers = e.layers;
        this.$emit("layer", layers);
      });
    });
  },
};
</script>

<style>
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
</style>