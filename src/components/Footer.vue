<script>
export default {
  // Données
  data() {
    // Obtenir l'année en cours pour l'affichage dans le footer
    return {
      currentYear: new Date().getFullYear()
    };
  }
}
</script>

<template>
  <footer class="footer">
    <p>© {{ currentYear }} MastermindAppMobile. All rights reserved.</p>
  </footer>
</template>

<style scoped>
.footer {
  width: 100%;
  background-color: #F8F0E3;
  color: #555;
  padding: 20px 0;
  margin-top: auto;
  text-align: center;
  border-top: 1px solid #ccc; 
}

.footer p {
  margin: 0;
  font-size: 0.85rem;
}
</style>