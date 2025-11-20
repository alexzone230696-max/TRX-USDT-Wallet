<template>
  <div class="freeze-page">

    <h2 class="title">Freeze / Unfreeze TRX</h2>

    <div class="card">

      <label>Amount TRX</label>
      <input v-model="amount" type="number" placeholder="Enter amount" />

      <label>Resource</label>
      <select v-model="resource">
        <option value="BANDWIDTH">Bandwidth</option>
        <option value="ENERGY">Energy</option>
      </select>

      <button @click="freeze">Freeze TRX</button>
      <button @click="unfreeze" class="unfreeze">Unfreeze TRX</button>

      <p v-if="message" class="message">{{ message }}</p>

    </div>

  </div>
</template>

<script>
export default {
  name: "Freeze",
  data() {
    return {
      amount: "",
      resource: "ENERGY",
      message: ""
    };
  },
  methods: {
    async freeze() {
      this.message = "Freezing not implemented in demo";
    },
    async unfreeze() {
      this.message = "Unfreezing not implemented in demo";
    }
  }
};
</script>

<style scoped>
.freeze-page {
  padding: 20px;
}
.card {
  background: #111;
  padding: 20px;
  border-radius: 12px;
  margin-top: 10px;
}
button {
  width: 100%;
  padding: 12px;
  margin-top: 10px;
  background: #0084ff;
  border-radius: 8px;
}
.unfreeze {
  background: #ff0044;
}
.message {
  margin-top: 15px;
}
.title {
  font-size: 22px;
  font-weight: bold;
}
</style>