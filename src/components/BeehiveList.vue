<template>
    <div class="beehive-manager">
        <h2 class="title">Liste des Ruches</h2>
        <div v-if="loading" class="loader">Chargement des ruches...</div>
        <div v-else>
            <!-- Create Beehive Form -->
            <form @submit.prevent="createBeehive" class="beehive-form">
                <input v-model="newBeehive.location" placeholder="Location" required class="input">
                <button type="submit" class="button">Ajouter une ruche</button>
            </form>

            <!-- Beehive List -->
            <ul v-if="beehives.length" class="beehive-list">
                <li v-for="beehive in beehives" :key="beehive.id" class="beehive-card">
                    <div v-if="!beehive.editing">
                        <span class="beehive-location">{{ beehive.location }}</span>
                        <span class="beehive-date">Créée le {{ formatDate(beehive.createdAt) }}</span>
                        <div class="button-group">
                            <button @click="getSensors(beehive.id)" class="button button-secondary">Voir les capteurs</button>
                            <button @click="startEditing(beehive)" class="button button-warning">Modifier</button>
                            <button @click="deleteBeehive(beehive.id)" class="button button-danger">Supprimer</button>
                        </div>
                    </div>
                    <div v-else class="edit-form">
                        <input v-model="beehive.location" class="input">
                        <div class="button-group">
                            <button @click="updateBeehive(beehive)" class="button">Enregistrer</button>
                            <button @click="cancelEditing(beehive)" class="button button-secondary">Annuler</button>
                        </div>
                    </div>
                </li>
            </ul>
            <div v-else class="no-data">Aucune ruche trouvée.</div>

            <h3 class="subtitle">Données des capteurs</h3>
            <ul v-if="sensors.length" class="sensor-list">
                <li v-for="sensor in sensors" :key="sensor.id" class="sensor-item">
                    <span class="sensor-type">{{ sensor.sensorType }}:</span>
                    <span class="sensor-value">{{ sensor.value }}</span>
                    <span class="sensor-date">Enregistré le {{ formatDate(sensor.recordedAt) }}</span>
                </li>
            </ul>
            <div v-else class="no-data">Aucune donnée de capteur disponible.</div>
        </div>
    </div>
</template>

<script setup lang="ts">
    import { ref, onMounted } from 'vue';
    import axios from 'axios';

    const apiBaseUrl = 'http://localhost:5299/api';
    const beehives = ref([]);
    const sensors = ref([]);
    const newBeehive = ref({ location: '' });
    const loading = ref(true);

    const formatDate = (dateString: string) => {
        return new Date(dateString).toLocaleString();
    };

    const getBeehives = async () => {
        try {
            const response = await axios.get(`${apiBaseUrl}/Beehive`);
            beehives.value = response.data.map((beehive: any) => ({ ...beehive, editing: false }));
            console.log(beehives.value);
        } catch (error) {
            console.error('Error fetching beehives:', error);
            alert('Failed to fetch beehives. Please try again.');
        } finally {
            loading.value = false;
        }
    };

    const getSensors = async (beehiveId: number) => {
        try {
            const response = await axios.get(`${apiBaseUrl}/Sensor/${beehiveId}`);
            sensors.value = response.data;
        } catch (error) {
            console.error('Error fetching sensors:', error);
            alert('Failed to fetch sensor data. Please try again.');
        }
    };

    const createBeehive = async () => {
        try {
            const response = await axios.post(`${apiBaseUrl}/Beehive`, newBeehive.value);
            beehives.value.push({ ...response.data, editing: false });
            newBeehive.value.location = '';
        } catch (error) {
            console.error('Error creating beehive:', error);
            alert('Failed to create beehive. Please try again.');
        }
    };

    const startEditing = (beehive: any) => {
        beehive.editing = true;
        beehive.originalLocation = beehive.location;
    };

    const cancelEditing = (beehive: any) => {
        beehive.editing = false;
        beehive.location = beehive.originalLocation;
    };

    const updateBeehive = async (beehive: any) => {
        try {
            await axios.put(`${apiBaseUrl}/Beehive/${beehive.id}`, {
                id: beehive.id,
                location: beehive.location
            });
            beehive.editing = false;
            delete beehive.originalLocation;
        } catch (error) {
            console.error('Error updating beehive:', error);
            alert('Failed to update beehive. Please try again.');
            beehive.location = beehive.originalLocation;
            beehive.editing = false;
        }
    };

    const deleteBeehive = async (id: number) => {
        if (confirm('Are you sure you want to delete this beehive?')) {
            try {
                await axios.delete(`${apiBaseUrl}/Beehive/${id}`);
                beehives.value = beehives.value.filter((beehive: any) => beehive.id !== id);
            } catch (error) {
                console.error('Error deleting beehive:', error);
                alert('Failed to delete beehive. Please try again.');
            }
        }
    };

    onMounted(getBeehives);

    defineExpose({ getBeehives });
</script>

<style scoped>
    :root {
        --primary-color: #4a90e2;
        --secondary-color: #f5a623;
        --danger-color: #d0021b;
        --text-color: #333;
        --background-color: #f4f4f4;
        --card-background: #ffffff;
        --border-radius: 8px;
        --transition-speed: 0.3s;
    }

    .beehive-manager {
        font-family: Arial, sans-serif;
        max-width: 1200px;
        margin: 0 auto;
        padding: 20px;
        background-color: var(--background-color);
        color: var(--text-color);
    }

    .title {
        font-size: 2.5rem;
        color: var(--primary-color);
        margin-bottom: 30px;
        text-align: center;
    }

    .loader {
        border: 5px solid #f3f3f3;
        border-top: 5px solid var(--primary-color);
        border-radius: 50%;
        width: 50px;
        height: 50px;
        animation: spin 1s linear infinite;
        margin: 50px auto;
    }

    @keyframes spin {
        0% {
            transform: rotate(0deg);
        }

        100% {
            transform: rotate(360deg);
        }
    }

    .beehive-form {
        display: flex;
        gap: 10px;
        margin-bottom: 30px;
    }

    .input {
        flex-grow: 1;
        padding: 10px;
        border: 1px solid #ccc;
        border-radius: var(--border-radius);
        font-size: 1rem;
    }

    .button {
        padding: 10px 20px;
        background-color: var(--primary-color);
        color: white;
        border: none;
        border-radius: var(--border-radius);
        cursor: pointer;
        transition: background-color var(--transition-speed);
    }

        .button:hover {
            background-color: darken(var(--primary-color), 10%);
        }

    .button-secondary {
        background-color: var(--secondary-color);
    }

    .button-danger {
        background-color: var(--danger-color);
    }

    .beehive-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
        gap: 20px;
    }

    .beehive-card {
        background-color: var(--card-background);
        border-radius: var(--border-radius);
        padding: 20px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        transition: transform var(--transition-speed);
    }

        .beehive-card:hover {
            transform: translateY(-5px);
        }

        .beehive-card h2 {
            color: var(--primary-color);
            margin-bottom: 10px;
        }

    .button-group {
        display: flex;
        gap: 10px;
        margin-top: 15px;
    }

    .modal {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.5);
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .modal-content {
        background-color: var(--card-background);
        padding: 20px;
        border-radius: var(--border-radius);
        width: 300px;
    }

    .sensors-data {
        margin-top: 30px;
        background-color: var(--card-background);
        border-radius: var(--border-radius);
        padding: 20px;
    }

        .sensors-data ul {
            list-style-type: none;
            padding: 0;
        }

        .sensors-data li {
            margin-bottom: 10px;
            padding-bottom: 10px;
            border-bottom: 1px solid #eee;
        }

    @media (max-width: 768px) {
        .beehive-form {
            flex-direction: column;
        }

        .beehive-grid {
            grid-template-columns: 1fr;
        }
    }
</style>