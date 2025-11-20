<script setup>
import { ref } from 'vue'; // Importez 'ref' pour gérer l'état
import Footer from './Footer.vue';
import Game from './Game.vue';
import RulesModal from './RulesModal.vue'; // Importez le nouveau composant de règles

// État pour contrôler la visibilité de la modale des règles
const showRules = ref(false); 
</script>

<template>
  <div class="content">
    
    <button @click="showRules = true" class="rules-btn">
      ❓ Afficher les Règles
    </button>
    
    <h1>Mastermind</h1>
    
    <Game />
    
    <RulesModal v-if="showRules" @close="showRules = false" /> 
    
    <Footer />
  </div>
</template>

<style scoped>
.content {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  margin: 0; 
  padding: 0;
}

/* Positionnement du bouton */
.rules-btn {
  position: absolute; /* Place le bouton dans le coin */
  top: 10px;
  right: 10px;
  padding: 8px 15px;
  background-color: #f8f9fa;
  border: 1px solid #ccc;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  z-index: 10; /* Assurez-vous qu'il est au-dessus du reste du contenu */
}

/* L'élément qui doit prendre l'espace restant (dans ce cas, le contenu du jeu) */
.content > h1,
.content > :deep(.game) {
  flex-grow: 1;
}
</style>