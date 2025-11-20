<script setup>
// Déclare les événements que ce composant peut émettre (ici, l'événement 'close')
const emit = defineEmits(['close']); 
</script>

<template>
  <div class="modal-overlay" @click.self="$emit('close')">
    <div class="modal-content">
      
      <button class="close-btn" @click="$emit('close')">
        ✖
      </button>

      <h2>Mastermind : Les Règles</h2>
      
      <section class="rules-body">
        
        <h3>But du Jeu</h3>
        <p>Découvrir la <strong>combinaison de couleurs secrète</strong> définie par l'ordinateur en un minimum d'essais.</p>

        <hr>
        
        <h3>Comment Jouer</h3>
        <ul>
          <li><strong>Proposer une Combinaison :</strong> Utilisez la palette de couleurs et faites un <strong>glisser-déposer</strong> les pions vers la ligne d'essai active.</li>
          <li><strong>Valider :</strong> Cliquez sur le bouton de validation (✔) pour soumettre votre essai.</li>
        </ul>
        
        <hr>

        <h3>Les Indices</h3>
        <p>Après chaque essai, l'ordinateur fournit des indices :</p>
        
        <div class="feedback-table">
          <div>
            <span class="feedback-peg black"></span>
            <strong>Pion Noir</strong>
          </div>
          <div>
            <p>Couleur <strong>correcte</strong> et à la <strong>bonne place</strong>.</p>
          </div>
          <div>
            <span class="feedback-peg white"></span>
            <strong>Pion Blanc</strong>
          </div>
          <div>
            <p>Couleur <strong>correcte</strong>, mais à la <strong>mauvaise place</strong>.</p>
          </div>
        </div>
        
        <p class="note">L'ordre des indices n'a aucune importance, seul le nombre compte.</p>
        
      </section>

    </div>
  </div>
</template>

<style scoped>
.modal-overlay {
  /* Fixe la modale sur tout l'écran, au-dessus de tout */
  position: fixed; 
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7); /* Arrière-plan sombre et transparent */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000; /* Assurez-vous qu'il est au-dessus du bouton et du jeu */
}

.modal-content {
  background: white;
  padding: 30px;
  border-radius: 8px;
  max-width: 90%;
  max-height: 90%;
  overflow-y: auto; /* Permet de défiler si le contenu est trop grand */
  position: relative;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
  text-align: left;
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  font-size: 1.5em;
  cursor: pointer;
  color: #555;
}

/* Styles pour le contenu des règles */
.rules-body {
  margin-top: 15px;
}

.feedback-table {
  display: grid;
  grid-template-columns: 80px 1fr;
  gap: 10px;
  margin: 15px 0;
  padding: 10px;
  border: 1px solid #eee;
  border-radius: 4px;
}

.feedback-table > div {
  display: flex;
  align-items: center;
}

.feedback-peg {
  width: 15px;
  height: 15px;
  border-radius: 50%;
  display: inline-block;
  margin-right: 5px;
  border: 1px solid #333;
}

.feedback-peg.black {
  background-color: black;
}

.feedback-peg.white {
  background-color: white;
}

.note {
  font-style: italic;
  font-size: 0.9em;
  color: #666;
  text-align: center;
}
</style>