<script setup lang="ts">
import {Ref, ref, watch} from "vue";
import {debounce} from "lodash";

let url: Ref<string | undefined> = ref(undefined);
let error = ref(false);

let coords = ref("");
let zoom = ref(19);

watch([coords, zoom], debounce(updateURL, 250));

function onTileError() {
  error.value = true;
}

function updateURL() {
  error.value = false;

  const [lat, long] = coords.value.split(",").map(it => it.trim()).map(it => parseFloat(it));
  const [x, y] = getGlobalCoordinates(lat, long, zoom.value);
  url.value = `https://core-carparks-renderer-lots.maps.yandex.net/maps-rdr-carparks/tiles?l=carparks&x=${x}&y=${y}&z=${zoom.value}&scale=1&lang=ru_RU`;
}

function getGlobalCoordinates(lat: number, long: number, z: number) {
  const e = 0.0818191908426;

  const rho = Math.pow(2, z + 8) / 2;
  const beta = Math.PI * lat / 180;
  const phi = (1 - e * Math.sin(beta)) / (1 + e * Math.sin(beta));
  const theta = Math.tan(Math.PI / 4 + beta / 2) * Math.pow(phi, e / 2);

  const x = rho * (1 + long / 180);
  const y = rho * (1 - Math.log(theta) / Math.PI);

  const tileX = Math.floor(x / 256);
  const tileY = Math.floor(y / 256);

  return [tileX, tileY];
}

</script>

<template>
  <div class="tile-calculator">
    <form>
      <div class="lat-long">
        <input type="text" placeholder="Координаты" v-model="coords"/>
      </div>
      <div class="zoom">
        <input type="number" name="z" step="any" min="0" placeholder="Зум" v-model="zoom"/>
      </div>
    </form>

    <div class="preview">
      <span v-if="error">Не удалось загрузить тайл.</span>
      <span v-else-if="!url">Ожидание ввода координат...</span>
      <img v-else :src="url" @error="onTileError"/>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.tile-calculator {
  display: flex;
  flex-direction: column;
  align-items: center;
}

form {
  display: flex;
  flex-direction: column;

  input {
    margin-right: 1rem;

    &:last-child {
      margin-right: 0;
    }

    &:invalid {
      border-color: red;
    }
  }

  .zoom {
    margin-top: 1rem;
    align-self: center;
  }
}

.preview {
  margin-top: 1rem;

  img {
    box-shadow: 0 0 5px black;
  }
}
</style>
