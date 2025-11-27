<script>
import SelecteurDifficulte from './SelecteurDifficulte.vue';
import Historique from './game-logic/Historique.vue'; 
import LigneEssaie from './game-logic/LigneEssaie.vue';

// --- FONCTIONS D'OBFUSCATION (Double Encryptage Léger) ---
// Note: En JavaScript côté client, c'est de l'obfuscation, pas du vrai chiffrement
// car la clé est accessible. Mais c'est efficace contre la triche simple.
const STORAGE_KEY = 'mastermind_secret_code';

// Obfuscation 1: Base64, Obfuscation 2: Inversion de chaîne
function encryptCode(codeArray) {
    // 1. Convertir le tableau en chaîne
    const codeString = codeArray.join('|'); 
    
    // 2. Obfuscation 1 (Base64)
    const base64 = btoa(codeString);
    
    // 3. Obfuscation 2 (Inversion de chaîne)
    const reversed = base64.split('').reverse().join('');
    
    return reversed;
}

function decryptCode(encryptedString) {
    if (!encryptedString) return null;
    
    try {
        // 1. Désobfuscation 2 (Inversion de chaîne)
        const reversed = encryptedString.split('').reverse().join('');
        
        // 2. Désobfuscation 1 (Base64)
        const codeString = atob(reversed);
        
        return codeString.split('|');
    } catch (e) {
        console.error("Erreur de décryptage du code secret:", e);
        return null;
    }
}
// ------------------------------------------------------------


// Mappage des jetons (CHEMIN RELATIF DANS PUBLIC)
const COLOR_MAP = {
    'Blanc': './jeton-Blanc.ico', 'Bleu': './jeton-Bleu.ico', 'Brun': './jeton-Brun.ico',
    'Jaune': './jeton-Jaune.ico', 'Noir': './jeton-Noir.ico', 'Orange': './jeton-Orange.ico',
    'Rose': './jeton-Rose.ico', 'Rouge': './jeton-Rouge.ico', 'Vert': './jeton-Vert.ico',
    'Violet': './jeton-Violet.ico',
};

const ALL_JETON_NAMES = Object.keys(COLOR_MAP);

// DÉFINITION DES NOUVELLES RÈGLES DE DIFFICULTÉ
const DIFFICULTY_SETTINGS = {
    'facile': { colorsCount: 4, attempts: 10, codeLength: 4 },
    'normal': { colorsCount: 5, attempts: 9, codeLength: 5 },
    'moyen': { colorsCount: 7, attempts: 8, codeLength: 5 },
    'difficile': { colorsCount: 9, attempts: 7, codeLength: 5 },
    'expert': { colorsCount: 10, attempts: 6, codeLength: 5 },
};

export default {
    name: 'MastermindGame', 
    
    components: {
        SelecteurDifficulte,
        Historique,
        LigneEssaie,
    },
    
    data() {
        return {
            currentDifficulty: 'normal', 
            selectedColorName: null, 

            gameData: {
                secretCode: [], // Code secret non chiffré (en mémoire)
                currentGuess: Array(4).fill(null), 
                attempts: [], 
                currentRow: 0,
            },
        };
    },

    computed: {
        availableColors() {
            const settings = DIFFICULTY_SETTINGS[this.currentDifficulty] || DIFFICULTY_SETTINGS['normal'];
            return ALL_JETON_NAMES.slice(0, settings.colorsCount);
        },
        maxAttempts() {
            const settings = DIFFICULTY_SETTINGS[this.currentDifficulty] || DIFFICULTY_SETTINGS['normal'];
            return settings.attempts;
        },
        codeLength() {
            const settings = DIFFICULTY_SETTINGS[this.currentDifficulty] || DIFFICULTY_SETTINGS['normal'];
            return settings.codeLength;
        }
    },

    watch: {
        currentDifficulty() {
            this.resetGame();
        }
    },
    
    mounted() {
        this.resetGame(); 
    },
    
    methods: {
        generateSecretCode() {
            const colors = this.availableColors; 
            const code = [];
            for (let i = 0; i < this.codeLength; i++) { 
                const randomIndex = Math.floor(Math.random() * colors.length);
                code.push(colors[randomIndex]); 
            }
            
            // 1. Stocker le code en clair dans les données Vue (pour la logique rapide)
            this.gameData.secretCode = code;
            
            // 2. Chiffrer et stocker le code dans le Local Storage (pour la sécurité)
            const encrypted = encryptCode(code);
            localStorage.setItem(STORAGE_KEY, encrypted);
            
            // (Supprimé: console.log qui révélait le code secret en clair)
        },

        resetGame() {
            this.gameData.currentGuess = Array(this.codeLength).fill(null);
            this.gameData.attempts = [];
            this.gameData.currentRow = 0;
            this.selectedColorName = null;
            this.generateSecretCode();
        },

        selectColor(colorName) {
            this.selectedColorName = colorName;
        },
        
        selectPeg(pegIndex) {
            if (this.selectedColorName && this.gameData.currentRow < this.maxAttempts) {
                const newGuess = [...this.gameData.currentGuess];
                newGuess[pegIndex] = this.selectedColorName;
                this.gameData.currentGuess = newGuess;
            }
        },

        validateCombination() {
            if (this.gameData.currentGuess.includes(null)) {
                alert("Veuillez remplir toute la ligne.");
                return;
            }

            const feedback = this.checkGuess(this.gameData.currentGuess, this.gameData.secretCode);
            this.gameData.attempts.push({
                guess: [...this.gameData.currentGuess], 
                feedback: feedback,
            });

            // LOGIQUE DE REDÉMARRAGE AUTOMATIQUE
            if (feedback.black === this.codeLength) {
                alert("Bravo ! Vous avez trouvé le code ! Une nouvelle partie va commencer.");
                setTimeout(() => {
                    this.resetGame();
                }, 500); 
                return; 
            } else if (this.gameData.currentRow >= this.maxAttempts - 1) {
                 alert(`Partie terminée ! Le code secret était : ${this.gameData.secretCode.join(', ')}.\nUne nouvelle partie va commencer.`);
                 setTimeout(() => {
                    this.resetGame();
                }, 500); 
                return;
            }

            this.gameData.currentRow++;
            this.gameData.currentGuess = Array(this.codeLength).fill(null);
            this.selectedColorName = null;
        },

        checkGuess(guess, secret) {
            // Note: Nous utilisons le 'secret' passé en paramètre, qui est this.gameData.secretCode (non chiffré en mémoire)
            let blackPegs = 0;
            let whitePegs = 0;
            const secretCopy = [...secret];
            const guessCopy = [...guess];

            for (let i = 0; i < this.codeLength; i++) {
                if (guessCopy[i] === secretCopy[i] && guessCopy[i] !== null) {
                    blackPegs++;
                    secretCopy[i] = guessCopy[i] = '__USED__'; 
                }
            }
            for (let i = 0; i < this.codeLength; i++) {
                if (guessCopy[i] !== '__USED__') {
                    const secretIndex = secretCopy.findIndex(color => color === guessCopy[i]);
                    if (secretIndex !== -1) {
                        whitePegs++;
                        secretCopy[secretIndex] = '__USED__';
                    }
                }
            }
            return { black: blackPegs, white: whitePegs };
        },

        getJetonPath(colorName) {
            return COLOR_MAP[colorName] || COLOR_MAP['Blanc']; 
        }
    }
}
</script>

<template>
  <v-card outlined class="game-card custom-border d-flex flex-column"> 
    
    <v-card-text class="pa-0 flex-grow-1">
      <v-container fluid class="fill-height pa-0">
        <v-row no-gutters class="fill-height">
          
          <v-col 
            cols="12" 
            lg="8" 
            class="game-zone fill-height pa-3 d-flex flex-column align-center justify-start"
          >
            
            <div class="mobile-controls-area d-flex d-lg-none flex-column align-center w-100 mb-3">
                
                <SelecteurDifficulte 
                    mode="mobile" 
                    :currentDifficulty="currentDifficulty" 
                    @difficulty-selected="currentDifficulty = $event" 
                    @color-selected="selectColor($event)" 
                />
                
                <div 
                    class="attempts-display attempts-display-mobile mt-2 mb-3"
                    :class="{'selected-mode': selectedColorName}"
                >
                    <span>Tentatives restantes:</span>
                    <span v-if="selectedColorName" class="selected-peg-preview">
                        <img :src="getJetonPath(selectedColorName)" alt="Pion sélectionné" class="jeton-icon-preview" />
                    </span>
                    <span v-else class="attempt-count">{{ maxAttempts - gameData.currentRow }}</span>
                </div>
            </div>
            
            <div class="center-area d-flex align-start w-100 fill-height justify-center">
                
                <div class="color-palette-container d-none d-lg-flex flex-column mr-4">
                    <h2>Palette de couleurs</h2>
                    <div class="color-palette-grid">
                        <div v-for="color in availableColors" :key="color" 
                             class="color-peg-wrapper"
                             :class="{'selected': selectedColorName === color}"
                             @click="selectColor(color)">
                             
                             <img :src="getJetonPath(color)" :alt="`Jeton ${color}`" class="jeton-icon-desktop" />
                        </div>
                    </div>
                </div>

                <div class="game-board-area flex-grow-1 d-flex flex-column align-center justify-center">
                    
                    <div 
                        class="attempts-display attempts-display-desktop d-none d-lg-block mb-4"
                        :class="{'selected-mode': selectedColorName}"
                    >
                        <span>Tentatives restantes:</span>
                        <span v-if="selectedColorName" class="selected-peg-preview">
                            <img :src="getJetonPath(selectedColorName)" alt="Pion sélectionné" class="jeton-icon-preview" />
                        </span>
                        <span v-else class="attempt-count">{{ maxAttempts - gameData.currentRow }}</span>
                    </div>

                    <div class="game-board-visual">
                        <Historique 
                            :tentatives="gameData.attempts" 
                            :getJetonPath="getJetonPath"
                        />
                        
                        <LigneEssaie 
                            :currentGuess="gameData.currentGuess"
                            :getJetonPath="getJetonPath" 
                            @select-peg="selectPeg"
                        />
                        
                        </div>

                    <v-btn 
                        class="validate-button mt-4" 
                        large
                        @click="validateCombination"
                        :disabled="gameData.currentGuess.includes(null) || gameData.currentRow >= maxAttempts"
                    >
                        Valider combinaison
                    </v-btn>
                </div>
            </div>
            
          </v-col>
          
          <v-divider vertical class="d-none d-lg-block"></v-divider> 

          <v-col 
            cols="4" 
            class="difficulty-zone fill-height pa-3 d-none d-lg-flex align-center justify-center"
          >
            <SelecteurDifficulte 
                mode="desktop" 
                :currentDifficulty="currentDifficulty"
                @difficulty-selected="currentDifficulty = $event"
            />
          </v-col>
          
        </v-row>
      </v-container>
    </v-card-text>
    
  </v-card>
</template>

<style scoped>
/* -------------------------------------- */
/* --- Styles généraux du Game.vue --- */
/* -------------------------------------- */
.game-card {
    width: 95%; 
    min-height: 70vh; 
    margin-left: auto;
    margin-right: auto;
    margin-top: 15px;
    background-color: #f5f5f5; 
}

.custom-border {
    box-shadow: 0 0 0 1px black !important; 
}

.game-zone {
    overflow-y: auto; 
    background-color: white; 
    padding: 15px !important; 
}

/* -------------------------------------- */
/* --- PLATEAU DE JEU & CENTRAGE --- */
/* -------------------------------------- */
.game-board-area {
    /* Clé du centrage: S'assure que tout est centré verticalement et horizontalement */
    flex-grow: 1; 
    display: flex;
    flex-direction: column;
    align-items: center; 
    justify-content: center; /* CENTRE VERTICALEMENT */
}

.game-board-visual {
    padding: 10px;
    border: 1px solid #ddd; 
    background-color: #f9f9f9; 
    border-radius: 4px;
    /* La largeur sera déterminée par le contenu dynamique, mais gardons une min-width visuelle */
    min-width: 280px; 
}

/* -------------------------------------- */
/* --- Placeholders pour le centrage --- */
/* -------------------------------------- */
.placeholder-peg-grid {
    display: grid;
    /* Utilise la variable CSS pour la longueur du code (4 ou 5) */
    grid-template-columns: repeat(var(--code-length), 40px);
    grid-auto-rows: 40px;
    gap: 10px;
    padding: 10px;
    width: fit-content; 
    margin: 0 auto;
}

.placeholder-peg-grid::before {
    content: '';
    grid-column: 1 / span var(--code-length);
    background-image: radial-gradient(circle, #e0e0e0 60%, transparent 60%);
    background-size: 50px 50px;
    height: 390px;
}

/* -------------------------------------- */
/* --- Styles de la Palette et Tentatives --- */
/* -------------------------------------- */
.color-palette-container {
    padding: 15px;
    border: 1px solid #ccc; 
    width: 150px; 
    background-color: #fff;
    border-radius: 4px;
}
.color-palette-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
    width: 100%;
}
.color-peg-wrapper {
    width: 40px; height: 40px; border-radius: 50%; cursor: pointer;
    overflow: hidden; box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
    display: flex; align-items: center; justify-content: center; transition: all 0.1s ease;
}
.color-peg-wrapper.selected {
    box-shadow: 0 0 0 4px #007bff, 0 1px 5px rgba(0, 0, 0, 0.4); 
    transform: scale(1.05);
}
.jeton-icon-desktop {
    width: 100%; height: 100%; object-fit: contain; 
}

.attempts-display {
    padding: 5px 10px; border: 1px solid #ccc; background-color: #e0e0e0;
    border-radius: 4px; width: fit-content; display: flex; align-items: center;
}
.attempt-count {
    padding: 2px 6px; background-color: #fff; border-radius: 50%; border: 1px solid #888;
    margin-left: 8px; font-weight: bold; min-width: 25px; text-align: center;
}
.selected-peg-preview {
    display: flex; align-items: center; justify-content: center; width: 25px; height: 25px;
    margin-left: 8px; overflow: hidden; border-radius: 50%;
}
.jeton-icon-preview {
    width: 100%; height: 100%; object-fit: contain;
}
.attempts-display.selected-mode {
    border-color: #007bff; background-color: #e6f3ff;
}

.validate-button {
    background-color: #e0e0e0 !important; color: black !important; text-transform: none;
    border: 1px solid #aaa; margin-top: 20px !important; font-weight: bold; padding: 0 30px !important;
    transition: all 0.2s;
}

.validate-button[disabled] {
    opacity: 0.5; cursor: not-allowed;
}

/* ******************************************* */
/* ******** MEDIA QUERY (RESPONSIVE) **** */
/* ******************************************* */
@media (min-width: 1264px) { 
    .center-area {
        align-items: flex-start !important; /* Décalage vers le haut */
    }
}
@media (max-width: 1263px) { 
    .center-area {
        flex-direction: column !important;
        align-items: center !important;
    }
    .game-board-area {
        width: 100%;
        margin-top: 15px; 
    }
}
</style>