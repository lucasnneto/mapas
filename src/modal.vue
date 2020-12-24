<template>
  <v-dialog v-model="dialog" persistent max-width="850px">
    <v-card class="overflow-y-hidden">
      <v-card-title class="pb-0 pr-4 pt-4">
        <v-spacer />
        <v-btn
          icon
          width="30"
          height="30"
          @click="
            () => {
              dialog = false;
            }
          "
        >
          <img width="14" height="14" src="@/assets/exit.svg" alt="" />
        </v-btn>
      </v-card-title>
      <div class="px-8">
        <h2>Editar Terreno</h2>
        <p class="body-3 mt-2 mb-6">
          Selecione um terreno no mapa para editá-lo.
        </p>
      </div>
      <div class="d-flex px-8">
        <div class="pr-8">
          <v-lazy>
            <mapa
              :polygons="poli"
              :zoom="13"
              :telha="telha"
              :center="center"
              height="60vh"
              width="60vh"
              edit
            />
          </v-lazy>
        </div>
        <div>
          <div class="pb-0">
            <p class="body-3 mt-2 mb-8">
              Insira o nome do terreno e selecione a cor.
            </p>
            <v-form>
              <v-text-field label="Nome do terreno" />
              <v-text-field label="Cor do terreno" readonly />
              <v-text-field label="Área aproximada" readonly />
            </v-form>

            <v-btn size="14" class="mt-5" block height="48" outlined>
              <img class="mr-4" src="@/assets/btn_add.svg" alt />
              <span class="font-weight-medium primary--text"
                >Adicionar Terreno</span
              >
            </v-btn>
          </div>
          <v-card-actions class="px-0 py-8">
            <v-btn size="20" block height="48">
              <span class="font-weight-bold">Salvar</span>
            </v-btn>
          </v-card-actions>
        </div>
      </div>
    </v-card>
  </v-dialog>
</template>

<script>
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
  },
  data() {
    return {
      telha: {
        name: "OpenStreetMap",
        url:
          "https://api.maptiler.com/maps/hybrid/{z}/{x}/{y}.jpg?key=Y7PaX0PVfTePQJc5eDua",
      },
      center: [-17.2633538, -46.8311499],
      poli: [
        {
          latlngs: [
            [-17.2633538, -46.8111499],
            [-17.2833538, -46.8311499],
            [-17.2633538, -46.8311499],
          ],
          cor: "blue",
        },
      ],
    };
  },
};
</script>

<style scoped>
</style>