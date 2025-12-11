<script setup>
// Importation des composants nécessaires
import { ref } from 'vue';
import Footer from './Footer.vue';
import Game from './Game.vue';
import RulesModal from './RulesModal.vue';

// État pour contrôler la visibilité de la modale des règles
// ref(false) indique que la modale est initialement cachée.
const showRules = ref(false); 
</script>

<template>
  <div class="content">

    <div class="header-content">
      <h1>Mastermind</h1>
      <button @click="showRules = true" class="rules-btn">
        Règle du jeu
      </button>
    </div>
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
  align-items: center;
  background-color: #F8F0E3; 
}

.header-content {
  width: 95%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 25px; 
  padding: 0 15px;
  flex-wrap: wrap;
}

.header-content h1 {
  font-size: 2.5em; 
  margin: 0;
  flex-shrink: 1;
  color: #333;
  font-weight: 700;
  letter-spacing: 1px;
}

.rules-btn {
  padding: 10px 20px;
  background-color: #6B8E23; 
  color: white;
  border: none;
  border-radius: 8px; 
  cursor: pointer;
  font-size: 15px;
  font-weight: 600;
  z-index: 10;
  margin-top: 5px;
  margin-bottom: 5px;
  transition: all 0.3s ease; 
  box-shadow: 0 4px 10px rgba(107, 142, 35, 0.4);
}

.rules-btn:hover {
    background-color: #5A761F; 
    box-shadow: 0 4px 12px rgba(107, 142, 35, 0.6);
}

.content> :deep(.game) {
  flex-grow: 1;
  width: 100%; 
  padding: 0 15px; 
}
</style>