<script>
import HistoriqueLigne from './HistoriqueLigne.vue';

export default {
    name: 'Historique',
    components: { HistoriqueLigne },
    // Propriétés reçues du parent (Game.vue)
    props: {
        // Le tableau d'historique des tentatives (chaque élément contient guess et feedback)
        tentatives: { // L'array d'historique de Game.vue
            type: Array,
            required: true,
            default: () => []
        },
        // Fonction pour obtenir le chemin de l'icône du jeton
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
    flex-direction: column; 
    max-height: 400px;
    overflow-y: auto;
    padding-bottom: 10px;
}
</style>