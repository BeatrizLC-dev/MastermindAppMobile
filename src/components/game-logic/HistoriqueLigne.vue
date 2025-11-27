<script>
export default {
    name: 'HistoriqueLigne',
    props: {
        guess: { // Le tableau de couleurs de la tentative
            type: Array,
            required: true
        },
        feedback: { // L'objet {black: N, white: M}
            type: Object,
            required: true
        },
        getJetonPath: { // La fonction passée de Game.vue
            type: Function,
            required: true
        }
    },
    computed: {
        feedbackPegs() {
            const pegs = [];
            // Ajouter les pions noirs (bonne couleur, bonne place)
            for (let i = 0; i < this.feedback.black; i++) {
                pegs.push('black');
            }
            // Ajouter les pions blancs (bonne couleur, mauvaise place)
            for (let i = 0; i < this.feedback.white; i++) {
                pegs.push('white');
            }
            // Remplir avec des pions gris/vides si nécessaire
            while (pegs.length < this.guess.length) {
                pegs.push('empty');
            }
            return pegs;
        }
    }
}
</script>

<template>
  <div class="historique-row">
    <div class="guess-display">
      <div 
          v-for="(colorName, index) in guess" 
          :key="index" 
          class="guess-peg"
      >
          <img :src="getJetonPath(colorName)" :alt="`Jeton ${colorName}`" class="jeton-icon-peg" />
      </div>
    </div>
    
    <div class="feedback-display">
        <div 
            v-for="(peg, index) in feedbackPegs" 
            :key="index" 
            class="feedback-peg"
            :class="`peg-${peg}`"
        ></div>
    </div>
  </div>
</template>

<style scoped>
.historique-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 5px 0;
    border-bottom: 1px solid #eee;
}

.guess-display {
    display: flex;
    gap: 8px;
}

.guess-peg {
    width: 35px;
    height: 35px;
    border-radius: 50%;
    border: 1px solid #ccc;
    overflow: hidden;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.jeton-icon-peg {
    width: 100%;
    height: 100%;
    object-fit: cover;
}

.feedback-display {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 4px;
    width: 40px; /* Espace pour les 4 petits pions */
    height: 40px;
    padding: 2px;
}

.feedback-peg {
    width: 15px;
    height: 15px;
    border-radius: 50%;
    border: 1px solid #aaa;
    background-color: #ddd;
}

.feedback-peg.peg-black {
    background-color: black;
    border-color: black;
}

.feedback-peg.peg-white {
    background-color: white;
    border-color: #666;
}

.feedback-peg.peg-empty {
    /* Fond gris clair par défaut */
    border: 1px dashed #ccc;
    background-color: #f5f5f5;
}
</style>