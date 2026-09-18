<template>
  <div class="container py-4" style="max-width: 400px;">
    <button class="btn btn-sm btn-outline-secondary mb-3" @click="$emit('back')">← Natrag</button>
    
    <h4>Plaćanje: {{ parking.name }}</h4>
    <hr />

    <form @submit.prevent="submitForm">
      <!-- Registracija -->
      <div class="mb-3">
        <label class="form-label fw-bold">Registracija</label>
        <input type="text" class="form-control text-uppercase" v-model="plate" placeholder="PU-1234-AB" required />
      </div>

      <!-- Odabir sati -->
      <div class="mb-3">
        <label class="form-label fw-bold">Broj sati</label>
        <input type="number" class="form-control" v-model.number="hours" min="1" max="8" required />
      </div>

      <!-- Kartica -->
      <div class="mb-3">
        <label class="form-label fw-bold">Broj kartice</label>
        <input type="text" class="form-control" v-model="cardNumber" placeholder="1234 5678 9012 3456" maxlength="19" required />
      </div>

      <!-- Ukupna cijena -->
      <div class="d-flex justify-content-between my-3">
        <h5>Ukupno:</h5>
        <h5 class="text-success fw-bold">{{ totalPrice }} €</h5>
      </div>

      <button type="submit" class="btn btn-success w-100 fw-bold">Plati odmah</button>
    </form>
  </div>
</template>

<script>
export default {
  props: ['parking'],
  data() {
    return {
      plate: '',
      hours: 1,
      cardNumber: ''
    };
  },
  computed: {
    // Automatski preračunava cijenu čim se promjeni broj sati
    totalPrice() {
      return (this.hours * this.parking.pricePerHour).toFixed(2);
    }
  },
  methods: {
    submitForm() {
      // Slanje podataka roditeljskoj komponenti (App.vue)
      this.$emit('confirm-payment', {
        plate: this.plate,
        hours: this.hours,
        price: this.totalPrice
      });
    }
  }
};
</script>