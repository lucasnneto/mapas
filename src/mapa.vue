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
  },
};
</script>

<style>
.leaflet-control-zoom {
  border: none;
  border-radius: 4px !important;
  background: rgba(38, 51, 87, 0.5);
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
</style>