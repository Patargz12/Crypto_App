<script setup lang="ts">
import { useHead } from "@vueuse/head";

useHead({
  title: "Crypto Tracker - Home",
});

interface Coin {
  id: string;
  name: string;
  symbol: string;
  image: string;
  current_price: number;
  price_change_percentage_24h: number;
}

const isDarkMode = ref(true);

const toggleDarkMode = () => {
  isDarkMode.value = !isDarkMode.value;
};

const coins = ref<Coin[] | null>(null);

const fetchCoins = async () => {
  const { data } = await useFetch<Coin[]>("/api/coins");
  coins.value = data.value;
};

fetchCoins();

const topCoins = computed(() =>
  Array.isArray(coins.value) ? coins.value.slice(0, 5) : []
);
const phCoins = computed(() =>
  Array.isArray(coins.value) ? coins.value.slice(5, 11) : []
);
const leastPerformingCoins = computed(() =>
  Array.isArray(coins.value)
    ? [...coins.value]
        .sort(
          (a, b) =>
            a.price_change_percentage_24h - b.price_change_percentage_24h
        )
        .slice(0, 5)
    : []
);
const risingCoins = computed(() =>
  Array.isArray(coins.value)
    ? [...coins.value]
        .sort(
          (a, b) =>
            b.price_change_percentage_24h - a.price_change_percentage_24h
        )
        .slice(0, 6)
    : []
);

const bitcoinData = computed(() =>
  Array.isArray(coins.value)
    ? coins.value.find((coin) => coin.id === "bitcoin")
    : null
);
</script>



<template>
  <div
    :class="{ dark: isDarkMode }"
    class="min-h-screen bg-gray-100 dark:bg-gray-900 transition-colors duration-300"
  >
    <!-- Navbar -->
    <nav class="bg-white dark:bg-gray-800 shadow-md">
      <div
        class="container mx-auto px-8 md:px-24 py-4 flex justify-between items-center"
      >
        <div class="flex flex-row ml-5">
          <CryptoLogo class="absolute top-0 mr-4" />
          <h1
            class="text-xl md:text-2xl font-bold text-gray-800 ml-6 dark:text-white"
          >
            Crypto Tracker
          </h1>
        </div>
        <div className="flex flex-row space-x-5">
          <button
            @click="toggleDarkMode"
            class="p-2 rounded-full bg-gray-200 dark:bg-gray-700"
          >
            <span
              v-if="isDarkMode"
              class="text-yellow-400"
              style="font-size: 1.3rem; line-height: 1.3rem"
              >☀️</span
            >
            <span
              v-else
              class="text-gray-700"
              style="font-size: 1.3rem; line-height: 1.3rem"
              >🌙</span
            >
          </button>
        </div>
      </div>
    </nav>

    <!-- Main Content -->
    <main class="container mx-auto px-6 md:px-32 py-8">
      <!-- Top Coins -->
      <div
        class="flex flex-col md:flex-row justify-end items-center space-x-0 md:space-x-12"
      >
        <section class="space-y-4">
          <CryptoChart v-if="bitcoinData" :bitcoinData="bitcoinData" />
          <MetaMask />
        </section>
        <section class="mb-12 w-full mt-6 md:mt-0">
          <h2 class="text-2xl font-semibold text-gray-800 dark:text-white mb-4">
            Top Coins
          </h2>

          <div
            class="bg-white dark:bg-gray-800 rounded-lg shadow-md overflow-hidden"
          >
            <table class="w-full">
              <thead class="bg-gray-50 dark:bg-gray-700">
                <tr>
                  <th
                    class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase tracking-wider"
                  >
                    Coin
                  </th>
                  <th
                    class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase tracking-wider"
                  >
                    Price
                  </th>
                  <th
                    class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase tracking-wider"
                  >
                    24h Change
                  </th>
                </tr>
              </thead>
              <tbody class="divide-y divide-gray-200 dark:divide-gray-600">
                <tr
                  v-for="coin in topCoins"
                  :key="coin.id"
                  class="hover:bg-gray-50 dark:hover:bg-gray-700"
                >
                  <td class="px-6 py-4 whitespace-nowrap">
                    <div class="flex items-center">
                      <img
                        :src="coin.image"
                        :alt="coin.name"
                        class="w-8 h-8 rounded-full mr-3"
                      />
                      <div>
                        <div
                          class="text-sm font-medium text-gray-900 dark:text-white"
                        >
                          {{ coin.name }}
                        </div>
                        <div class="text-sm text-gray-500 dark:text-gray-400">
                          {{ coin.symbol.toUpperCase() }}
                        </div>
                      </div>
                    </div>
                  </td>
                  <td class="px-6 py-4 whitespace-nowrap">
                    <div class="text-sm text-gray-900 dark:text-white">
                      ${{ coin.current_price.toFixed(2) }}
                    </div>
                  </td>
                  <td class="px-6 py-4 whitespace-nowrap">
                    <span
                      :class="[
                        'px-2 inline-flex text-xs leading-5 font-semibold rounded-full',
                        coin.price_change_percentage_24h > 0
                          ? 'bg-green-100 text-green-800'
                          : 'bg-red-100 text-red-800',
                      ]"
                    >
                      {{ coin.price_change_percentage_24h.toFixed(2) }}%
                    </span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </section>
      </div>

      <!-- Best Coins in the Philippines -->
      <section class="mb-12">
        <h2 class="text-2xl font-semibold text-gray-800 dark:text-white mb-4">
          Best Coins in the Philippines
        </h2>
        <div class="bg-white dark:bg-gray-800 rounded-lg shadow-md p-6">
          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            <div
              v-for="coin in phCoins"
              :key="coin.id"
              class="bg-gray-50 dark:bg-gray-700 rounded-lg p-4"
            >
              <div class="flex items-center mb-2">
                <img
                  :src="coin.image"
                  :alt="coin.name"
                  class="w-10 h-10 rounded-full mr-3"
                />
                <div>
                  <div class="font-medium text-gray-900 dark:text-white">
                    {{ coin.name }}
                  </div>
                  <div class="text-sm text-gray-500 dark:text-gray-400">
                    {{ coin.symbol.toUpperCase() }}
                  </div>
                </div>
              </div>
              <div class="text-lg font-semibold text-gray-900 dark:text-white">
                ${{ coin.current_price.toFixed(2) }}
              </div>
              <div
                :class="[
                  'text-sm font-medium',
                  coin.price_change_percentage_24h > 0
                    ? 'text-green-600'
                    : 'text-red-600',
                ]"
              >
                {{ coin.price_change_percentage_24h.toFixed(2) }}%
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- Least Performing Coins -->
      <section class="mb-12">
        <h2 class="text-2xl font-semibold text-gray-800 dark:text-white mb-4">
          Least Performing Coins
        </h2>
        <div
          class="bg-white dark:bg-gray-800 rounded-lg shadow-md overflow-hidden"
        >
          <table class="w-full">
            <thead class="bg-gray-50 dark:bg-gray-700">
              <tr>
                <th
                  class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase tracking-wider"
                >
                  Coin
                </th>
                <th
                  class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase tracking-wider"
                >
                  Price
                </th>
                <th
                  class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase tracking-wider"
                >
                  24h Change
                </th>
              </tr>
            </thead>
            <tbody class="divide-y divide-gray-200 dark:divide-gray-600">
              <tr
                v-for="coin in leastPerformingCoins"
                :key="coin.id"
                class="hover:bg-gray-50 dark:hover:bg-gray-700"
              >
                <td class="px-6 py-4 whitespace-nowrap">
                  <div class="flex items-center">
                    <img
                      :src="coin.image"
                      :alt="coin.name"
                      class="w-8 h-8 rounded-full mr-3"
                    />
                    <div>
                      <div
                        class="text-sm font-medium text-gray-900 dark:text-white"
                      >
                        {{ coin.name }}
                      </div>
                      <div class="text-sm text-gray-500 dark:text-gray-400">
                        {{ coin.symbol.toUpperCase() }}
                      </div>
                    </div>
                  </div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <div class="text-sm text-gray-900 dark:text-white">
                    ${{ coin.current_price.toFixed(2) }}
                  </div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <span
                    class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-red-100 text-red-800"
                  >
                    {{ coin.price_change_percentage_24h.toFixed(2) }}%
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Rising Coins -->
      <section class="mb-12">
        <h2 class="text-2xl font-semibold text-gray-800 dark:text-white mb-4">
          Rising Coins
        </h2>
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          <div
            v-for="coin in risingCoins"
            :key="coin.id"
            class="bg-white dark:bg-gray-800 rounded-lg shadow-md p-6"
          >
            <div class="flex items-center mb-4">
              <img
                :src="coin.image"
                :alt="coin.name"
                class="w-12 h-12 rounded-full mr-4"
              />
              <div>
                <div
                  class="text-lg font-semibold text-gray-900 dark:text-white"
                >
                  {{ coin.name }}
                </div>
                <div class="text-sm text-gray-500 dark:text-gray-400">
                  {{ coin.symbol.toUpperCase() }}
                </div>
              </div>
            </div>
            <div class="text-2xl font-bold text-gray-900 dark:text-white mb-2">
              ${{ coin.current_price.toFixed(2) }}
            </div>
            <div class="text-lg font-semibold text-green-600">
              +{{ coin.price_change_percentage_24h.toFixed(2) }}%
            </div>
          </div>
        </div>
      </section>
    </main>
  </div>
</template>


<style>
@import "tailwindcss/base";
@import "tailwindcss/components";
@import "tailwindcss/utilities";
</style>