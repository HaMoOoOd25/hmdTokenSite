<template>
  <div>
    <div class="flex flex-col">
      <!-- ETH TO SELL -->
      <div class="flex flex-col py-2">
        <div class="flex flex-row justify-between">
          <label for="eth">Eth Amount</label>
          <div class="text-sm opacity-70">Balance: {{ ethBalance }} ETH</div>
        </div>
        <input
          v-model="ethAmount"
          min="0"
          class="p-2 border-2 border-purple-400 rounded-md"
          id="eth"
          type="number"
        />
      </div>
      <!-- Token To Buy -->
      <div class="flex flex-col py-2">
        <div class="flex flex-row justify-between">
          <label for="eth">Token</label>
          <div class="text-sm opacity-70">Balance: {{ tokenBalance }} HMD</div>
        </div>
        <input
          disabled
          :value="tokenAmount"
          class="p-2 border-2 border-purple-400 rounded-md"
          id="eth"
          type="number"
        />
      </div>
    </div>
    <!-- BUY!! -->
    <div>
      <button
        @click="action(ethAmount)"
        class="text-center bg-purple-400 w-full text-white p-2 mt-4 rounded-md"
      >
        Swap
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: ["action", "ethBalance", "tokenBalance"],
  data() {
    return {
      ethAmount: 0,
    };
  },
  computed: {
    tokenAmount() {
      return this.ethAmount * 100;
    },
  },
};
</script>

<style></style>
