<script>
import Footer from './Footer.vue';
import Game from './Game.vue';
</script>

<template>
  <div class="content">
    <h1>Mastermind</h1>
    <Game />
    <Footer />
  </div>
</template>

<style scoped>
/* Assurez-vous que le conteneur principal prend toute la hauteur de la vue */
.content {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  margin: 0; 
  padding: 0;
}

/* L'élément qui doit prendre l'espace restant (dans ce cas, le contenu du jeu) */
.content > h1,
.content > :deep(.game) {
  flex-grow: 1;
}
</style>