<script setup>
import { ref } from "vue";
import { ethers } from "ethers";

const account = ref(null);
const balance = ref(null);

const connectWallet = async () => {
  if (!window.ethereum) {
    alert("MetaMask is not installed. Please install it to continue.");
    return;
  }

  try {
    const provider = new ethers.BrowserProvider(window.ethereum);
    const signer = await provider.getSigner();
    account.value = await signer.getAddress();

    // Fetch balance
    const balanceWei = await provider.getBalance(account.value);
    balance.value = ethers.formatEther(balanceWei); // Convert to ETH
  } catch (error) {
    console.error("Wallet connection failed:", error);
  }
};

const disconnectWallet = () => {
  account.value = null;
  balance.value = null;
};
</script>

<template>
  <div class="flex flex-row bg-gray-900 justify-center text-white">
    <div
      v-if="account"
      class="flex flex-row-reverse mt-2 md:mt-0 items-center space-x-4"
    >
      <div class="flex flex-row space-x-4">
        <p class="rounded-lg ml-6">Balance: {{ balance }} ETH</p>
      </div>
      <button
        @click="disconnectWallet"
        class="bg-red-600 px-4 py-2 mr-4 rounded-lg hover:bg-red-700 transition"
      >
        Disconnect
      </button>
    </div>

    <button
      v-else
      @click="connectWallet"
      class="bg-blue-600 px-4 mt-2 md:mt-0 py-2 rounded-lg place-self-center hover:bg-blue-700 transition"
    >
      Connect MetaMask
    </button>
  </div>
</template>
