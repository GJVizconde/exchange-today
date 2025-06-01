<script setup lang="ts">
import axios from 'axios';
import { format } from 'date-fns';
import { es } from 'date-fns/locale';
import { computed, onMounted, ref, type Ref } from 'vue';


const averagePrice: Ref<number> = ref(0)
const formattedDate: Ref<string> = ref('')

const dollars: Ref<number | null> = ref(1)





const getData = async () => {
  try {
    const { data } = await axios.get(`${import.meta.env.VITE_API_URL}/api/seed`)

    averagePrice.value = data.averagePrice

    // Convertimos el timestamp en objeto Date
    const localDate = new Date(data.date)

    // Formateamos a hora local del usuario
    const formatted = format(localDate, 'EEEE, dd/MM/yyyy hh:mm a', { locale: es })


    // Capitalizamos
    formattedDate.value = formatted.charAt(0).toUpperCase() + formatted.slice(1)
  } catch (err) {
    console.error('Error al obtener datos', err)
  }
}

onMounted(async () => {
  await getData()
})

const formattedBs = computed(() => {
  if (dollars.value) {
    const raw = dollars.value * averagePrice.value
    return new Intl.NumberFormat('es-VE', {
      style: 'decimal',
      minimumFractionDigits: 2,
      maximumFractionDigits: 2,
      useGrouping: true
    }).format(raw)
  } else {
    throw new Error(`No parameters`)
  }
})


</script>

<template>
  <div class="min-h-screen w-screen bg-black text-white flex flex-col items-center justify-center px-4 py-6">
    <!-- Logo -->
    <div class="mb-6 flex flex-col items-center">
      <!-- <img src="/logo.jpg" alt="Freddy Torres" class="w-24 h-24 rounded-full object-cover mb-2" /> -->
      <h1 class="text-xl font-semibold">Johns Today</h1>
    </div>

    <!-- Card -->
    <div class="bg-gray-900 rounded-2xl w-full max-w-md p-6 shadow-md">
      <!-- Botón de tipo -->
      <div class="mb-4">
        <button class="bg-lime-500 text-black font-bold w-full py-2 rounded-lg">Paralelo</button>
      </div>

      <!-- Montos -->
      <div class="space-y-4 mb-4">
        <div class="flex justify-between items-center border-b border-gray-700 pb-2">
          <span>$</span>
          <input v-model.number="dollars" type="number" min="0" @focus="dollars = null" @blur="dollars = dollars || 0"
            class="bg-transparent border-none text-right w-20 focus:outline-none focus:ring-0 appearance-none" />
        </div>
        <div class="flex justify-between items-center border-b border-gray-700 pb-2">
          <span>Bs</span>
          <span>{{ formattedBs }}</span>
        </div>
      </div>

      <!-- Variación -->
      <!-- <div class="text-red-500 text-center font-semibold text-sm mb-2">
        ↓ -135.74 Bs (-99.99%)
      </div> -->
    </div>

    <!-- Fecha -->
    <div class="mt-6 text-sm text-gray-400 text-center">
      {{ formattedDate }}
    </div>
  </div>
</template>
