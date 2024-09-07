<script setup lang="ts">
    import { ref } from 'vue';
    import BeehiveList from '../components/BeehiveList.vue';

    defineProps<{
        msg: string
    }>();

    const beehiveListRef = ref<InstanceType<typeof BeehiveList> | null>(null);

    // Si vous avez besoin de rafraîchir les données depuis le parent
    const refreshBeehives = () => {
        beehiveListRef.value?.getBeehives();
    };
</script>

<template>
    <div class="greetings">
        <h1 class="green">{{ msg }}</h1>
        <h3>
            You've successfully created a project with
            <a href="https://vitejs.dev/" target="_blank" rel="noopener">Vite</a> +
            <a href="https://vuejs.org/" target="_blank" rel="noopener">Vue 3</a>. What's next?
        </h3>
        <BeehiveList ref="beehiveListRef" />
    </div>
</template>

<style scoped>
    /* Vos styles restent inchangés */
</style>