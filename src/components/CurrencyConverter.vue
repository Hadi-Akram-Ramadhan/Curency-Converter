<template>
  <div class="min-h-screen bg-gray-100 py-6 flex flex-col justify-center sm:py-12">
    <div class="relative py-3 sm:max-w-xl sm:mx-auto">
      <div class="relative px-4 py-10 bg-white shadow-lg sm:rounded-3xl sm:p-20">
        <div class="max-w-md mx-auto">
          <div class="divide-y divide-gray-200">
            <div class="py-8 text-base leading-6 space-y-4 text-gray-700 sm:text-lg sm:leading-7">
              <h1 class="text-3xl font-bold text-center mb-8">Currency Converter</h1>

              <div class="space-y-4">
                <div>
                  <label class="block text-sm font-medium text-gray-700">Amount</label>
                  <input
                    v-model="amount"
                    type="number"
                    class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                  >
                </div>

                <div class="grid grid-cols-2 gap-4">
                  <div>
                    <label class="block text-sm font-medium text-gray-700">From</label>
                    <select
                      v-model="fromCurrency"
                      class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                    >
                      <option v-for="currency in currencies" :key="currency" :value="currency">
                        {{ currency }}
                      </option>
                    </select>
                  </div>

                  <div>
                    <label class="block text-sm font-medium text-gray-700">To</label>
                    <select
                      v-model="toCurrency"
                      class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                    >
                      <option v-for="currency in currencies" :key="currency" :value="currency">
                        {{ currency }}
                      </option>
                    </select>
                  </div>
                </div>

                <div class="mt-4">
                  <button
                    @click="convertCurrency"
                    class="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
                  >
                    Convert
                  </button>
                </div>

                <div v-if="result" class="mt-4 p-4 bg-gray-50 rounded-md">
                  <p class="text-lg font-medium text-gray-900">
                    {{ amount }} {{ fromCurrency }} = {{ result }} {{ toCurrency }}
                  </p>
                </div>

                <div v-if="error" class="mt-4 p-4 bg-red-50 rounded-md">
                  <p class="text-lg font-medium text-red-900">
                    {{ error }}
                  </p>
                </div>
              </div>
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

onMounted(() => {
  fetchCurrencies()
})
</script>
