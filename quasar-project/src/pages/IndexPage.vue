<template>
  <q-page 
    v-touch-pan.vertical.prevent.mouse="handlePan"
    class="flex flex-center text-black"
  >
    <!-- Apartado para mostrar los usos del contador y el nombre -->
    <div class="absolute-top-right text-red text-h6 q-pa-md bg-white shadow-2 rounded" v-if="data.usageCount > 0">
      <div>{{ data.name }} - Usos: {{ data.usageCount }}</div>
      <q-btn
        @click="clearUsage"
        flat
        icon="close"
        color="red"
        size="sm"
      />
    </div>

    <div class="row">
      <q-input
        v-model="data.name"
        input-class="text-center text-h5 text-red"
        color="teal"
        placeholder="Counter"
        filled
      />
    </div>
    <div class="row full-width items-center">
      <div class="col text-center">
        <q-btn
          @click="decreaseCounter"
          v-touch-repeat:300:300:300:300:50.mouse.enter.space="decreaseCounter"
          icon="remove" 
          size="xl"
          round 
        />
      </div>
      <div class="col text-center text-h2">
        {{ data.counter }}
      </div>
      <div class="col text-center">
        <q-btn
          @click="increaseCounter"
          v-touch-repeat:300:300:300:300:50.mouse.enter.space="increaseCounter"
          icon="add" 
          size="xl"
          round 
        />
      </div>
    </div>
    <div class="row">
      <q-btn
        @click="resetCounter"
        icon="restart_alt" 
        size="xl"
        round 
      />
    </div>
  </q-page>
</template>

<script setup>
import { reactive, watch } from 'vue';
import { LocalStorage } from 'quasar';

// Data reactiva
const data = reactive({
  counter: 0,
  name: '',
  usageCount: 0
});

// Recuperar datos almacenados
const savedData = LocalStorage.getItem('data');
if (savedData) Object.assign(data, savedData);

// Vigilar cambios y guardar datos
watch(
  () => data,
  (value) => {
    LocalStorage.set('data', value);
  },
  { deep: true }
);

// Métodos del contador
const increaseCounter = () => {
  data.counter++;
  data.usageCount++;
};

const decreaseCounter = () => {
  if (data.counter > 0) {
    data.counter--;
    data.usageCount++;
  }
};

const resetCounter = () => {
  data.counter = 0;
  data.usageCount++;
};

const handlePan = (e) => {
  if (e.delta.y < 0) {
    increaseCounter();
  } else {
    decreaseCounter();
  }
};

const clearUsage = () => {
  data.usageCount = 0;
};
</script>

<style scoped>
.q-page {
  max-width: 700px;
  margin: 0 auto;
}

.absolute-top-right {
  position: absolute;
  top: 0;
  right: 0;
}
</style>
