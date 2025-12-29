<template>
  <div v-if="visible" class="h-screen w-screen flex items-center justify-start pl-20 font-sans text-white">
    <div class="w-96 flex flex-col gap-4 animate-slideIn">
      <div class="mb-4">
        <h1 class="text-4xl font-bold tracking-tighter drop-shadow-md">CHARACTERS</h1>
        <p class="text-gray-300 text-sm drop-shadow-md">Select your persona</p>
      </div>

      <div class="flex flex-col gap-2 max-h-[60vh] overflow-y-auto pr-2">
        <div
          v-for="(char, index) in characters"
          :key="index"
          @click="selectCharacter(char)"
          class="p-4 bg-gray-900/90 border-l-4 border-transparent hover:border-blue-500 hover:bg-gray-800 transition-all cursor-pointer group shadow-lg"
          :class="{ '!border-blue-500 !bg-gray-800': selectedChar?.citizenid === char.citizenid }"
        >
          <div class="flex justify-between items-center">
            <div>
              <h2 class="text-lg font-semibold group-hover:text-blue-400 transition-colors">
                {{ char.charinfo.firstname }} {{ char.charinfo.lastname }}
              </h2>
              <p class="text-xs text-gray-500">{{ char.citizenid }}</p>
            </div>
            <div class="text-right">
                <span class="text-xs text-gray-400 block">{{ char.job.label }}</span>
                <span class="text-xs text-green-500">{{ char.money.bank }} $</span>
            </div>
          </div>
        </div>

        <div
          @click="createNewCharacter"
          class="p-4 bg-gray-900/40 border-2 border-dashed border-gray-600 hover:border-blue-500 hover:bg-gray-900/80 text-center text-gray-300 hover:text-white transition-all cursor-pointer rounded"
        >
          <span class="text-2xl">+</span>
          <p class="text-sm uppercase tracking-wide">New Character</p>
        </div>
      </div>

      <div v-if="selectedChar" class="mt-4 flex gap-2 animate-fadeIn">
        <button
          @click="playCharacter"
          class="flex-1 bg-blue-600 hover:bg-blue-700 text-white py-3 font-bold uppercase tracking-wide transition-colors rounded shadow-lg shadow-blue-900/20"
        >
          Play
        </button>
        <button
          @click="deleteCharacter"
          class="px-4 bg-red-900/80 hover:bg-red-700 text-white font-bold transition-colors rounded"
        >
          <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
            <path stroke-linecap="round" stroke-linejoin="round" d="M14.74 9l-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 01-2.244 2.077H8.084a2.25 2.25 0 01-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 00-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 013.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 00-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 00-7.5 0" />
          </svg>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const visible = ref(false);
const characters = ref([]);
const selectedChar = ref(null);

const selectCharacter = (char) => {
  selectedChar.value = char;
  fetch(`https://${GetParentResourceName()}/selectCharacter`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ citizenid: char.citizenid })
  });
};

const playCharacter = () => {
  if (!selectedChar.value) return;
  fetch(`https://${GetParentResourceName()}/playCharacter`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ citizenid: selectedChar.value.citizenid })
  });
  visible.value = false;
};

const createNewCharacter = () => {
  fetch(`https://${GetParentResourceName()}/createCharacter`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({})
  });
  visible.value = false;
};

const deleteCharacter = () => {
    if (!selectedChar.value) return;
    fetch(`https://${GetParentResourceName()}/deleteCharacter`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ citizenid: selectedChar.value.citizenid })
    });
};

const handleMessage = (event) => {
  const data = event.data;
  if (data?.action === 'openMulticharacter') {
    characters.value = data.characters;
    visible.value = true;
    selectedChar.value = null; // Reset selection
  } else if (data?.action === 'closeMulticharacter') {
    visible.value = false;
  } else if (data?.action === 'refreshCharacters') {
      characters.value = data.characters;
  }
};

onMounted(() => {
  window.addEventListener('message', handleMessage);
});

onUnmounted(() => {
  window.removeEventListener('message', handleMessage);
});
</script>

<style scoped>
.animate-slideIn {
  animation: slideIn 0.5s ease-out forwards;
}

@keyframes slideIn {
  from { opacity: 0; transform: translateX(-50px); }
  to { opacity: 1; transform: translateX(0); }
}

.animate-fadeIn {
    animation: fadeIn 0.3s ease-out forwards;
}
@keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
}
</style>
