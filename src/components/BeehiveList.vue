<template>
    <div>
        <h1>Ruches</h1>
        <ul>
            <li v-for="beehive in beehives" :key="beehive.id">
                {{ beehive.location }} - Créée le {{ beehive.createdAt }}
                <button @click="getSensors(beehive.id)">Voir les capteurs</button>
            </li>
        </ul>
      
        <h2>Donn&eacute;es des capteurs</h2>
        <ul>
            <li v-for="sensor in sensors" :key="sensor.id">
                {{ sensor.sensorType }}: {{ sensor.value }} - Enregistré le {{ sensor.recordedAt }}
            </li>
        </ul>
    </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      beehives: [],
      sensors: []
    };
  },
  mounted() {
    this.getBeehives();
  },
  methods: {
    getBeehives() {
      axios.get('https://localhost:5001/api/Beehive')
        .then(response => {
          this.beehives = response.data;
        });
    },
    getSensors(beehiveId) {
      axios.get(`https://localhost:5001/api/Sensor/${beehiveId}`)
        .then(response => {
          this.sensors = response.data;
        });
    }
  }
};
</script>
