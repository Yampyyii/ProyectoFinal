<template>
  <q-page 
    v-touch-pan.vertical.prevent.mouse="handlePan"
    class="flex flex-center text-black"
  >
    <!-- Contador del usuario actual -->
    <div class="absolute-top-left text-red text-h6 q-pa-md shadow-2 rounded">
      <div>{{ user.name || 'Usuario' }} - Usos: {{ user.usageCount }}</div>
      <div>Últimos valores: {{ user.lastValues.join(', ') }}</div>
      <q-btn
        @click="clearUsage"
        flat
        icon="close"
        color="red"
        size="sm"
        label="Reset"
      />
    </div>

    <!-- Mensajes especiales -->
    <div class="absolute-top-left q-ml-md q-mt-xl">
      <q-banner v-if="showCongrats" class="q-mb-md text-center text-white bg-teal">
        ¡Felicidades! Aumentaste 10 puntos.
      </q-banner>
      <q-banner v-if="showMaxCongrats" class="q-mb-md text-center text-white bg-orange">
        ¡Eres lo máximo! Gracias por usar el contador.
      </q-banner>
    </div>

    <!-- Panel de control -->
    <div class="row">
      <q-input
        v-model="user.name"
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
        {{ user.counter }}
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
        label="Reset Counter"
      />
    </div>

    <!-- Historial de uso -->
    <div class="absolute-bottom-left text-black q-pa-md shadow-2 rounded" :style="{ backgroundColor: 'transparent' }">
      <div class="text-h6">Historial</div>
      <div v-for="(record, index) in history" :key="index" class="q-mb-sm">
        <div><strong>{{ record.name || 'Sin nombre' }}</strong></div>
        <div>Hasta: {{ record.counter }}</div>
        <div>Mensaje: {{ record.message || 'N/A' }}</div>
      </div>
    </div>

    <!-- Botón general de reset -->
    <div class="absolute-bottom-right q-pa-md">
      <q-btn
        @click="resetAll"
        icon="delete"
        color="red"
        label="Reset All"
        flat
      />
    </div>
  </q-page>
</template>

<script setup>
import { reactive, ref, watch } from 'vue';
import { LocalStorage } from 'quasar';

const user = reactive({
  name: '',
  counter: 0,
  usageCount: 0,
  lastValues: [],
  messages: [],
});

const history = reactive([]);

const showCongrats = ref(false);
const showMaxCongrats = ref(false);

// Guardar datos en almacenamiento local
watch(
  () => ({ user, history }),
  (value) => LocalStorage.set('data', value),
  { deep: true }
);

const savedData = LocalStorage.getItem('data');
if (savedData) {
  Object.assign(user, savedData.user);
  Object.assign(history, savedData.history);
}

const increaseCounter = () => {
  user.counter++;
  user.usageCount++;
  addLastValue(user.counter);
  checkSpecialMessages();
};

const decreaseCounter = () => {
  if (user.counter > 0) {
    user.counter--;
    user.usageCount++;
    addLastValue(user.counter);
    checkSpecialMessages();
  }
};

const resetCounter = () => {
  addLastValue(user.counter);
  addToHistory(user.counter, user.messages.at(-1));
  user.counter = 0;
  user.usageCount++;
};

const addLastValue = (value) => {
  user.lastValues.unshift(value);
  if (user.lastValues.length > 5) user.lastValues.pop();
};

const addToHistory = (counter, message) => {
  history.unshift({
    name: user.name,
    counter,
    message,
  });
  if (history.length > 5) history.pop();
};

const checkSpecialMessages = () => {
  if (user.counter % 10 === 0 && user.counter > 0) {
    showCongrats.value = true;
    user.messages.push('¡Felicidades! Aumentaste 10 puntos.');
    setTimeout(() => (showCongrats.value = false), 3000);
  }
  if (user.counter === 100) {
    showMaxCongrats.value = true;
    user.messages.push('¡Eres lo máximo! Gracias por usar el contador.');
    setTimeout(() => (showMaxCongrats.value = false), 5000);
  }
};

const clearUsage = () => {
  user.usageCount = 0;
  user.lastValues = [];
  user.messages = [];
};

const resetAll = () => {
  user.name = '';
  user.counter = 0;
  user.usageCount = 0;
  user.lastValues = [];
  user.messages = [];
  history.splice(0, history.length);
};

const handlePan = (e) => {
  if (e.delta.y < 0) increaseCounter();
  else decreaseCounter();
};
</script>

<style scoped>
.q-page {
  max-width: 700px;
  margin: 0 auto;
}

.absolute-top-left {
  position: absolute;
  top: 0;
  left: 0;
}

.absolute-bottom-left {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 300px;
  max-height: 200px;
  overflow-y: auto;
}

.absolute-bottom-right {
  position: absolute;
  bottom: 0;
  right: 0;
}
</style>
