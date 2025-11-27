<script>
// Mappage des noms de couleurs/jetons aux chemins d'accès (CHEMIN RELATIF DANS PUBLIC)
const COLOR_MAP = {
    'Blanc': './jeton-Blanc.ico', 'Bleu': './jeton-Bleu.ico', 'Brun': './jeton-Brun.ico',
    'Jaune': './jeton-Jaune.ico', 'Noir': './jeton-Noir.ico', 'Orange': './jeton-Orange.ico',
    'Rose': './jeton-Rose.ico', 'Rouge': './jeton-Rouge.ico', 'Vert': './jeton-Vert.ico',
    'Violet': './jeton-Violet.ico',
};

// DÉFINITION DES COULEURS DISPONIBLES PAR NIVEAU
const DIFFICULTY_COLORS = {
    'facile': Object.keys(COLOR_MAP).slice(0, 4), 
    'normal': Object.keys(COLOR_MAP).slice(0, 5), 
    'moyen': Object.keys(COLOR_MAP).slice(0, 7), 
    'difficile': Object.keys(COLOR_MAP).slice(0, 9), 
    'expert': Object.keys(COLOR_MAP).slice(0, 10), 
};

export default {
    name: 'SelecteurDifficulte',
    
    props: {
        mode: { 
            type: String,
            required: true,
            validator: (value) => ['desktop', 'mobile'].includes(value)
        },
        currentDifficulty: {
            type: String,
            default: 'normal', 
        }
    },
    
    data() {
        return {
            difficultyLevels: [
                { label: 'Facile', key: 'facile' },
                { label: 'Normal', key: 'normal' },
                { label: 'Moyen', key: 'moyen' },
                { label: 'Difficile', key: 'difficile' },
                { label: 'Expert', key: 'expert' },
            ],
            selectedDifficulty: this.currentDifficulty, 
        };
    },
    
    computed: {
        availableJetonNames() {
            const key = this.selectedDifficulty.toLowerCase();
            return DIFFICULTY_COLORS[key] || DIFFICULTY_COLORS['normal'];
        }
    },
    
    methods: {
        selectLevel(level) {
            this.selectedDifficulty = level;
            this.$emit('difficulty-selected', level);
        },
        selectColor(colorName) {
            this.$emit('color-selected', colorName);
        },
        getJetonPath(colorName) {
             return COLOR_MAP[colorName]; 
        }
    }
}
</script>

<template>
  <div class="difficulty-selector-wrapper" :class="`mode-${mode}`">
      
      <v-card v-if="mode === 'desktop'" outlined class="desktop-sidebar fill-height d-flex flex-column align-center justify-center elevation-0">
          <v-card-title class="sidebar-title">Veuillez sélectionner la difficulté</v-card-title>
          <v-card-text class="difficulty-list-desktop pa-0 flex-grow-1 d-flex flex-column">
              <v-btn 
                  v-for="level in difficultyLevels" 
                  :key="level.key" 
                  :class="{ 'active': selectedDifficulty === level.key }"
                  @click="selectLevel(level.key)"
                  class="my-2"
                  block
                  depressed
              >
                  {{ level.label }}
              </v-btn>
          </v-card-text>
      </v-card>
      
      <div v-else-if="mode === 'mobile'" class="mobile-bar-container">
          
          <div class="difficulty-bar-mobile">
              <v-btn 
                  v-for="level in difficultyLevels" 
                  :key="level.key" 
                  :class="{ 'active': selectedDifficulty === level.key }"
                  @click="selectLevel(level.key)"
                  small
                  tile
                  depressed
                  class="flex-grow-1 mx-0"
              >
                  {{ level.label }}
              </v-btn>
          </div>
          
          <div class="color-palette-mobile mt-2">
              <div v-for="color in availableJetonNames" :key="color" 
                   class="color-peg-mobile"
                   @click="selectColor(color)">
                   
                   <img :src="getJetonPath(color)" :alt="`Jeton ${color}`" class="jeton-icon" />

              </div>
          </div>
          
      </div>
      
  </div>
</template>

<style scoped>
/* -------------------------------------- */
/* --- Styles de la Sidebar Desktop (SelecteurDifficulte) --- */
/* -------------------------------------- */
.desktop-sidebar {
    width: 100%; 
    padding: 15px;
    background-color: #fff !important;
    border: none !important; 
    box-shadow: none !important;
}

.sidebar-title {
    font-size: 16px;
    padding-bottom: 10px;
    border-bottom: 1px solid #eee;
    margin-bottom: 10px;
    width: 100%;
    text-align: center;
    font-weight: bold;
}

.difficulty-list-desktop {
    width: 90%; 
    max-width: 200px; 
}

.difficulty-list-desktop .v-btn {
    padding: 10px !important;
    border: 1px solid #aaa !important;
    background-color: #f0f0f0 !important;
    color: black !important;
    text-transform: none;
    height: auto !important;
    font-size: 14px;
    border-radius: 4px;
}

.difficulty-list-desktop .v-btn.active {
    background-color: #ccc !important;
    font-weight: bold;
    box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.2);
}

/* -------------------------------------- */
/* --- Styles de la Barre Mobile (SelecteurDifficulte) --- */
/* -------------------------------------- */
.mobile-bar-container {
    width: 100%; 
    display: flex;
    flex-direction: column;
    align-items: center;
}

/* Barre de difficulté mobile */
.difficulty-bar-mobile {
    display: flex;
    gap: 0;
    width: 100%;
    border: 1px solid #ccc;
    background-color: #fff; 
    border-radius: 4px;
    overflow: hidden; 
}

.difficulty-bar-mobile .v-btn {
    font-size: 10px !important;
    padding: 8px 0 !important;
    border-radius: 0 !important;
    background-color: #f0f0f0 !important;
    color: black !important;
    text-transform: none;
    min-width: auto !important;
    height: auto !important;
    flex-basis: 0; 
}

.difficulty-bar-mobile .v-btn.active {
    background-color: #ccc !important;
    font-weight: bold;
    box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.2);
}

/* Palette de couleurs mobile (Images des jetons) */
.color-palette-mobile {
    display: flex;
    justify-content: center; 
    flex-wrap: wrap; 
    gap: 8px; 
    width: 100%;
    padding: 10px 5px; 
    background-color: #fff; 
    border: 1px solid #ccc; 
    border-radius: 4px;
    margin-top: 10px; 
    max-width: 400px; 
}

.color-peg-mobile {
    width: 35px; 
    height: 35px;
    border-radius: 50%;
    cursor: pointer;
    overflow: hidden; 
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
    display: flex; 
    align-items: center;
    justify-content: center;
}

.jeton-icon {
    width: 100%; 
    height: 100%;
    object-fit: cover; 
}
</style>