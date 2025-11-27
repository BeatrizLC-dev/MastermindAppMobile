<script setup>
import { ref } from 'vue';
import Footer from './Footer.vue';
import Game from './Game.vue';
import RulesModal from './RulesModal.vue';

// État pour contrôler la visibilité de la modale des règles
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
}

.header-content {
    width: 95%; 
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 15px;
    padding: 0 10px;
    flex-wrap: wrap; 
}

.header-content h1 {
    font-size: 2em;
    margin: 0;
    flex-shrink: 1; 
}

.rules-btn {
    padding: 8px 15px;
    background-color: #f0f0f0; 
    border: 1px solid #aaa;
    border-radius: 4px;
    cursor: pointer;
    font-size: 14px;
    z-index: 10;
    margin-top: 5px; 
    margin-bottom: 5px; 
    
    flex-shrink: 0; 
}

.content > :deep(.game) {
  flex-grow: 1; 
}
</style>