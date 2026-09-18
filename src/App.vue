<template>
  <div>
    <!-- Ekran 1: Karta -->
    <MapView 
      v-if="screen === 'map'" 
      @select-parking="goToPayment" 
    />

    <!-- Ekran 2: Forma -->
    <PaymentView 
      v-else-if="screen === 'payment'" 
      :parking="selectedParking" 
      @back="screen = 'map'" 
      @confirm-payment="goToConfirmation" 
    />

    <!-- Ekran 3: Potvrda -->
    <div v-else-if="screen === 'confirmation'" class="container py-5 text-center style-box">
      <div class="alert alert-success">
        <h4>Plaćanje Uspješno!</h4>
        <p>Parking: <strong>{{ selectedParking.name }}</strong></p>
        <p>Vozilo: <strong>{{ details.plate }}</strong></p>
        <p>Trajanje: <strong>{{ details.hours }} h</strong></p>
        <h5>Plaćeno: {{ details.price }} €</h5>
      </div>
      <button class="btn btn-primary" @click="screen = 'map'">Natrag na kartu</button>
    </div>
  </div>
</template>

<script>
import MapView from './components/MapView.vue';
import PaymentView from './components/PaymentView.vue';

export default {
  components: { MapView, PaymentView },
  data() {
    return {
      screen: 'map', // 'map' | 'payment' | 'confirmation'
      selectedParking: null,
      details: null
    };
  },
  methods: {
    goToPayment(parking) {
      this.selectedParking = parking;
      this.screen = 'payment';
    },
    goToConfirmation(paymentDetails) {
      this.details = paymentDetails;
      this.screen = 'confirmation';
    }
  }
};
</script>

<style>
.style-box { max-width: 400px; }
</style>