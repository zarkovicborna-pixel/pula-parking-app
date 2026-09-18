<template>
  <div class="map-container">
    <div id="map"></div>

    <!-- Bottom sheet kartica s detaljima odabranog parkinga -->
    <div v-if="selected" class="bottom-sheet p-3 bg-white shadow">
      <h5 class="fw-bold mb-1">{{ selected.name }}</h5>
      <p class="mb-1">Cijena: <strong>{{ selected.pricePerHour.toFixed(2) }} €/h</strong></p>
      <p class="mb-1">Slobodno: <strong>{{ selected.total - selected.occupied }}</strong> / {{ selected.total }}</p>
      <p class="mb-3" v-if="selected.minutes">Vožnja: <strong>{{ selected.minutes }} min</strong></p>
      
      <button class="btn btn-primary w-100 fw-bold" @click="$emit('select-parking', selected)">
        Idi na plaćanje
      </button>
    </div>
  </div>
</template>

<script>
import L from 'leaflet';
import parkingsData from '../data/parkings.json';

export default {
  data() {
    return {
      parkings: parkingsData,
      selected: null,
      map: null,
      markersLayer: null
    };
  },
  mounted() {
    // 1. Pokretanje karte usmjerene na Pulu
    this.map = L.map('map').setView([44.8683, 13.8481], 14);

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: 'OpenStreetMap'
    }).addTo(this.map);

    // Sloj u koji spremamo sve znakove radi lakšeg ponovnog crtanja
    this.markersLayer = L.layerGroup().addTo(this.map);

    // Prvo crtanje znakova bez GPS-a
    this.renderMarkers();

    // 2. Dohvat GPS-a i izračun udaljenosti
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(pos => {
        const uLat = pos.coords.latitude;
        const uLng = pos.coords.longitude;

        this.parkings.forEach(p => {
          const meters = L.latLng(uLat, uLng).distanceTo([p.lat, p.lng]);
          p.minutes = Math.max(1, Math.round(meters / 500));
        });

        // Ponovno nacrtaj znakove s izračunatim minutama
        this.renderMarkers();
      });
    }
  },
  methods: {
    renderMarkers() {
      // Obriši stare znakove
      this.markersLayer.clearLayers();

      this.parkings.forEach(p => {
        // Izračun postotka popunjenosti i odabir boje (crvena ako je > 80%, inače zelena)
        const percent = Math.round((p.occupied / p.total) * 100);
        const color = percent > 80 ? '#dc3545' : '#198754';
        const minText = p.minutes ? `${p.minutes} min` : '-- min';

        // Čisti bijeli pravokutnik zaobilazeći standardne ikone
        const customIcon = L.divIcon({
          className: 'white-marker',
          html: `
            <div style="
              background: #ffffff;
              border: 1px solid #cccccc;
              border-radius: 8px;
              padding: 5px 8px;
              text-align: center;
              font-family: sans-serif;
              box-shadow: 0 2px 6px rgba(0,0,0,0.15);
              white-space: nowrap;
            ">
              <div style="font-size: 11px; font-weight: bold; color: #0d6efd;">${minText}</div>
              <div style="font-size: 12px; font-weight: bold; color: #212529;">${p.name}</div>
              <div style="font-size: 10px; font-weight: bold; color: ${color};">${percent}% popunjeno</div>
            </div>
          `,
          iconSize: [120, 50],
          iconAnchor: [60, 25]
        });

        const marker = L.marker([p.lat, p.lng], { icon: customIcon });
        
        marker.on('click', () => {
          this.selected = p;
        });

        this.markersLayer.addLayer(marker);
      });
    }
  }
};
</script>

<style scoped>
.map-container { position: relative; width: 100vw; height: 100vh; }
#map { width: 100%; height: 100%; }
.bottom-sheet { position: absolute; bottom: 0; left: 0; right: 0; z-index: 1000; border-top-left-radius: 15px; border-top-right-radius: 15px; }
</style>