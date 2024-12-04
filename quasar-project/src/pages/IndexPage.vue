<template>
  <q-page 
    v-touch-pan.vertical.prevent.mouse="handlePan"
    class="flex flex-center text-black"
  >
    <!-- Apartado para mostrar los usos del contador, el nombre y los últimos valores -->
    <div class="absolute-top-right text-red text-h6 q-pa-md shadow-2 rounded" v-if="data.usageCount > 0">
      <div>{{ data.name }} - Usos: {{ data.usageCount }}</div>
      <div>Últimos valores: {{ data.lastValues.join(', ') }}</div>
      <q-btn
        @click="clearUsage"
        flat
        icon="close"
        color="red"
        size="sm"
      />
    </div>

    <!-- Mensajes especiales -->
    <q-banner v-if="showCongrats" class="q-mb-md text-center text-white bg-teal">
      ¡Felicidades! Aumentaste 10 puntos.
    </q-banner>
    <q-banner v-if="showMaxCongrats" class="q-mb-md text-center text-white bg-orange">
      ¡Eres lo máximo! Gracias por usar el contador.
    </q-banner>

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
import { reactive, ref, watch } from 'vue';
import { LocalStorage } from 'quasar';

// Data reactiva
const data = reactive({
  counter: 0,
  name: '',
  usageCount: 0,
  lastValues: []
});

// Mensajes especiales
const showCongrats = ref(false);
const showMaxCongrats = ref(false);

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
  addLastValue(data.counter);
  checkSpecialMessages();
};

const decreaseCounter = () => {
  if (data.counter > 0) {
    data.counter--;
    data.usageCount++;
    addLastValue(data.counter);
    checkSpecialMessages();
  }
};

const resetCounter = () => {
  addLastValue(data.counter);
  data.counter = 0;
  data.usageCount++;
};

// Manejar la lista de últimos valores
const addLastValue = (value) => {
  data.lastValues.unshift(value); // Agregar al inicio
  if (data.lastValues.length > 5) {
    data.lastValues.pop(); // Eliminar el valor más antiguo
  }
};

// Manejar mensajes especiales
const checkSpecialMessages = () => {
  if (data.counter > 0 && data.counter % 10 === 0) {
    showCongrats.value = true;
    setTimeout(() => (showCongrats.value = false), 3000); // Ocultar después de 3 segundos
  }
  if (data.counter === 100) {
    showMaxCongrats.value = true;
    setTimeout(() => (showMaxCongrats.value = false), 5000); // Ocultar después de 5 segundos
  }
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
  data.lastValues = [];
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

.q-banner {
  width: 100%;
}
</style>
