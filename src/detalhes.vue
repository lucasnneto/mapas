<template>
  <v-dialog v-model="dialog" persistent max-width="500px">
    <v-card>
      <v-card-title class="pb-0 pr-4 pt-4 pb-4">
        Cultura: {{ dados.name }}
        <v-spacer />
        <v-btn icon width="30" height="30" @click="$emit('dialog', false)">
          <img width="14" height="14" src="@/assets/exit.svg" alt="" />
        </v-btn>
      </v-card-title>
      <v-card-text>
        <h4 class="mb-2">{{ dados.area }} ha</h4>
        <v-lazy>
          <mapa
            :polygons="lista"
            :zoom="13"
            :telha="{
              name: 'OpenStreetMap',
              url:
                'https://api.maptiler.com/maps/hybrid/{z}/{x}/{y}.jpg?key=Y7PaX0PVfTePQJc5eDua',
            }"
            :center="[-17.2633538, -46.8311499]"
            height="400px"
          />
        </v-lazy>
      </v-card-text>
    </v-card>
  </v-dialog>
</template>

<script>
import mapa from "./mapa.vue";
export default {
  name: "detalhes",
  components: {
    mapa,
  },
  props: {
    dialog: Boolean,
    dados: Object,
  },
  computed: {
    lista() {
      return [this.dados];
    },
  },
};
</script>