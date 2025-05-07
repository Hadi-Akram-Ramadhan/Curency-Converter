<template>
  <div class="min-h-screen bg-gradient-to-br from-indigo-100 to-purple-100 py-6 flex flex-col justify-center sm:py-12">
    <div class="relative py-3 sm:max-w-xl sm:mx-auto">
      <div class="relative px-4 py-10 bg-white/80 backdrop-blur-lg shadow-xl rounded-3xl sm:p-16">
        <div class="max-w-md mx-auto">
          <div class="flex items-center justify-center mb-8">
            <svg class="w-10 h-10 text-indigo-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>
            <h1 class="text-3xl font-bold text-gray-800 ml-3">Currency Converter</h1>
          </div>

          <div class="space-y-6">
            <!-- Amount Input -->
            <div class="relative">
              <label class="text-sm font-medium text-gray-700 mb-1 block">Amount</label>
              <input
                v-model="amount"
                type="number"
                min="0"
                step="0.01"
                class="w-full px-4 py-3 rounded-lg border border-gray-200 focus:ring-2 focus:ring-indigo-400 focus:border-transparent transition duration-200 bg-white/50 backdrop-blur-sm"
                placeholder="Enter amount..."
              >
            </div>

            <!-- Currency Selectors -->
            <div class="grid grid-cols-[1fr,auto,1fr] gap-4 items-center">
              <div>
                <label class="text-sm font-medium text-gray-700 mb-1 block">From</label>
                <select
                  v-model="fromCurrency"
                  class="w-full px-4 py-3 rounded-lg border border-gray-200 focus:ring-2 focus:ring-indigo-400 focus:border-transparent transition duration-200 bg-white/50 backdrop-blur-sm"
                >
                  <option v-for="currency in currencies" :key="currency" :value="currency">
                    {{ currency }}
                  </option>
                </select>
              </div>

              <!-- Swap Button -->
              <button
                @click="swapCurrencies"
                class="mt-6 p-2 rounded-full hover:bg-gray-100 transition duration-200"
              >
                <svg class="w-6 h-6 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7h12m0 0l-4-4m4 4l-4 4m0 6H4m0 0l4 4m-4-4l4-4" />
                </svg>
              </button>

              <div>
                <label class="text-sm font-medium text-gray-700 mb-1 block">To</label>
                <select
                  v-model="toCurrency"
                  class="w-full px-4 py-3 rounded-lg border border-gray-200 focus:ring-2 focus:ring-indigo-400 focus:border-transparent transition duration-200 bg-white/50 backdrop-blur-sm"
                >
                  <option v-for="currency in currencies" :key="currency" :value="currency">
                    {{ currency }}
                  </option>
                </select>
              </div>
            </div>

            <!-- Convert Button -->
            <button
              @click="convertCurrency"
              class="w-full py-3 px-4 bg-gradient-to-r from-indigo-600 to-purple-600 text-white rounded-lg font-medium hover:from-indigo-700 hover:to-purple-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transform transition duration-200 hover:scale-[1.02]"
            >
              Convert
            </button>

            <!-- Result -->
            <div v-if="result" class="mt-6 p-4 rounded-lg bg-gradient-to-r from-green-50 to-emerald-50">
              <p class="text-xl font-semibold text-gray-800 text-center">
                {{ amount }} {{ fromCurrency }} = {{ result }} {{ toCurrency }}
              </p>
              <p class="text-sm text-gray-500 text-center mt-1">
                1 {{ fromCurrency }} = {{ (result / amount).toFixed(4) }} {{ toCurrency }}
              </p>
            </div>

            <!-- Error Message -->
            <div v-if="error" class="mt-6 p-4 rounded-lg bg-red-50 border border-red-100">
              <p class="text-sm font-medium text-red-800 text-center">
                {{ error }}
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const amount = ref(1)
const fromCurrency = ref('USD')
const toCurrency = ref('EUR')
const result = ref(null)
const error = ref(null)
const currencies = ref([])

const fetchCurrencies = async () => {
  try {
    const response = await axios.get('https://api.frankfurter.app/currencies')
    currencies.value = Object.keys(response.data)
  } catch (error) {
    console.error('Error fetching currencies:', error)
    error.value = 'Failed to load currencies. Please try again later.'
  }
}

const convertCurrency = async () => {
  if (fromCurrency.value === toCurrency.value) {
    result.value = amount.value
    return
  }

  try {
    error.value = null
    const response = await axios.get(
      `https://api.frankfurter.app/latest?amount=${amount.value}&from=${fromCurrency.value}&to=${toCurrency.value}`
    )
    result.value = response.data.rates[toCurrency.value].toFixed(2)
  } catch (error) {
    console.error('Error converting currency:', error)
    error.value = 'Failed to convert currency. Please try again later.'
  }
}

const swapCurrencies = () => {
  [fromCurrency.value, toCurrency.value] = [toCurrency.value, fromCurrency.value]
  if (result.value) convertCurrency()
}

onMounted(() => {
  fetchCurrencies()
})
</script>
