<script>
import HistoriqueLigne from './HistoriqueLigne.vue';

export default {
    name: 'Historique',
    components: { HistoriqueLigne },
    props: {
        tentatives: { // L'array d'historique de Game.vue
            type: Array,
            required: true,
            default: () => []
        },
        getJetonPath: { // La fonction passée de Game.vue
            type: Function,
            required: true
        }
    }
}
</script>

<template>
  <div class="historique-container">
    <HistoriqueLigne 
        v-for="(tentative, index) in tentatives" 
        :key="index"
        :guess="tentative.guess"
        :feedback="tentative.feedback"
        :getJetonPath="getJetonPath"
    />
  </div>
</template>

<style scoped>
.historique-container {
    display: flex;
    flex-direction: column-reverse;
    max-height: 400px;
    overflow-y: auto;
    padding-bottom: 10px;
}
</style>