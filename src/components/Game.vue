<script>
import SelecteurDifficulte from './SelecteurDifficulte.vue';
import Historique from './game-logic/Historique.vue';
import LigneEssaie from './game-logic/LigneEssaie.vue';

// --- FONCTIONS D'OBFUSCATION (Double Encryptage Léger) ---
const STORAGE_KEY = 'mastermind_secret_code';

function encryptCode(codeArray) {
    const codeString = codeArray.join('|');
    const base64 = btoa(codeString);
    const reversed = base64.split('').reverse().join('');
    return reversed;
}

// ------------------------------------------------------------


// Mappage des jetons (CYAN et GRIS remplacent BLANC et NOIR)
const COLOR_MAP = {
    'Gris': './jeton-Gris.ico', // REMPLACE LE NOIR
    'Bleu': './jeton-Bleu.ico',
    'Brun': './jeton-Brun.ico',
    'Jaune': './jeton-Jaune.ico',
    'Cyan': './jeton-Cyan.ico', // REMPLACE LE BLANC
    'Orange': './jeton-Orange.ico',
    'Rose': './jeton-Rose.ico',
    'Rouge': './jeton-Rouge.ico',
    'Vert': './jeton-Vert.ico',
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
                secretCode: [],
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

            this.gameData.secretCode = code;

            const encrypted = encryptCode(code);
            localStorage.setItem(STORAGE_KEY, encrypted);
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
    <v-card outlined class="game-card custom-shadow d-flex flex-column">

        <v-card-text class="pa-0 flex-grow-1">
            <v-container fluid class="fill-height pa-0">
                <v-row no-gutters class="fill-height">

                    <v-col cols="12" lg="8"
                        class="game-zone fill-height pa-3 d-flex flex-column align-center justify-start">

                        <div class="mobile-controls-area d-flex d-lg-none flex-column align-center w-100 mb-3">

                            <SelecteurDifficulte mode="mobile" :currentDifficulty="currentDifficulty"
                                @difficulty-selected="currentDifficulty = $event"
                                @color-selected="selectColor($event)" />

                            <div class="attempts-display attempts-display-mobile mt-2 mb-3"
                                :class="{ 'selected-mode': selectedColorName }">
                                <span>Tentatives restantes:</span>
                                <span v-if="selectedColorName" class="selected-peg-preview">
                                    <img :src="getJetonPath(selectedColorName)" alt="Pion sélectionné"
                                        class="jeton-icon-preview" />
                                </span>
                                <span v-else class="attempt-count">{{ maxAttempts - gameData.currentRow }}</span>
                            </div>
                        </div>

                        <div class="center-area d-flex align-start w-100 fill-height justify-center">

                            <div class="color-palette-container d-none d-lg-flex flex-column mr-4">
                                <h2>Palette de couleurs</h2>
                                <div class="color-palette-grid">
                                    <div v-for="color in availableColors" :key="color" class="color-peg-wrapper"
                                        :class="{ 'selected': selectedColorName === color }" @click="selectColor(color)">

                                        <img :src="getJetonPath(color)" :alt="`Jeton ${color}`"
                                            class="jeton-icon-desktop" />
                                    </div>
                                </div>
                            </div>

                            <div class="game-board-area flex-grow-1 d-flex flex-column align-center justify-center">

                                <div class="attempts-display attempts-display-desktop d-none d-lg-block mb-4"
                                    :class="{ 'selected-mode': selectedColorName }">
                                    <span>Tentatives restantes:</span>
                                    <span v-if="selectedColorName" class="selected-peg-preview">
                                        <img :src="getJetonPath(selectedColorName)" alt="Pion sélectionné"
                                            class="jeton-icon-preview" />
                                    </span>
                                    <span v-else class="attempt-count">{{ maxAttempts - gameData.currentRow }}</span>
                                </div>

                                <div class="game-board-visual">
                                    <Historique :tentatives="gameData.attempts" :getJetonPath="getJetonPath" />

                                    <LigneEssaie :currentGuess="gameData.currentGuess" :getJetonPath="getJetonPath"
                                        @select-peg="selectPeg" />
                                </div>

                                <v-btn class="validate-button mt-4" large @click="validateCombination"
                                    :disabled="gameData.currentGuess.includes(null) || gameData.currentRow >= maxAttempts">
                                    Valider combinaison
                                </v-btn>
                            </div>
                        </div>

                    </v-col>

                    <v-divider vertical class="d-none d-lg-block"></v-divider>

                    <v-col cols="4"
                        class="difficulty-zone fill-height pa-3 d-none d-lg-flex align-center justify-center">
                        <SelecteurDifficulte mode="desktop" :currentDifficulty="currentDifficulty"
                            @difficulty-selected="currentDifficulty = $event" />
                    </v-col>

                </v-row>
            </v-container>
        </v-card-text>

    </v-card>
</template>

<style scoped>
.game-card {
    width: 95%;
    min-height: 70vh;
    margin-left: auto;
    margin-right: auto;
    margin-top: 15px;
    background-color: #D4C1A5;
    border-radius: 12px;
    box-shadow: 0 5px 20px rgba(0, 0, 0, 0.15);
    border: 1px solid #c9bdae;
}

.custom-shadow {
    box-shadow: 0 5px 20px rgba(0, 0, 0, 0.15) !important;
    border: 1px solid #c9bdae !important;
}

.game-zone {
    overflow-y: auto;
    background-color: #D4C1A5;
    padding: 20px !important;
    border-radius: 9px 0 0 9px;
}

.difficulty-zone {
    background-color: #D4C1A5;
    border-radius: 0 9px 9px 0;
    border-left: 1px solid gray;
}

.game-board-area {
    flex-grow: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}

.game-board-visual {
    flex-grow: 1;
    padding: 15px;
    border: 2px solid #A08D6D;
    background-color: #C9BDAE;
    border-radius: 8px;
    min-width: 280px;
    display: flex;
    flex-direction: column;
    box-shadow: inset 0 0 8px rgba(0, 0, 0, 0.3);
    justify-content: flex-end;
}

.color-palette-container {
    padding: 15px;
    border: 2px solid #6B8E23;
    width: 150px;
    background-color: #E6DCCD;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(107, 142, 35, 0.4);
}

.color-palette-container h2 {
    font-size: 16px;
    margin-bottom: 10px;
    text-align: center;
    color: #333;
    font-weight: 600;
}

.color-palette-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
    width: 100%;
}

.color-peg-wrapper {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    cursor: pointer;
    overflow: hidden;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.1s ease;
}

.color-peg-wrapper.selected {
    box-shadow: 0 0 0 4px #6B8E23, 0 1px 5px rgba(0, 0, 0, 0.4);
    transform: scale(1.1);
}

.jeton-icon-desktop {
    width: 100%;
    height: 100%;
    object-fit: contain;
}

.attempts-display {
    padding: 5px 10px;
    border: 1px solid #ccc;
    background-color: #FFFFFF;
    color: #333;
    border-radius: 4px;
    width: fit-content;
    display: flex;
    align-items: center;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.attempt-count {
    padding: 2px 6px;
    background-color: #6B8E23;
    color: white;
    border-radius: 50%;
    border: none;
    margin-left: 8px;
    font-weight: bold;
    min-width: 25px;
    text-align: center;
    font-size: 1em;
}

.selected-peg-preview {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 25px;
    height: 25px;
    margin-left: 8px;
    overflow: hidden;
    border-radius: 50%;
}

.jeton-icon-preview {
    width: 100%;
    height: 100%;
    object-fit: contain;
}

.attempts-display.selected-mode {
    border-color: #6B8E23;
    background-color: #F8FFF0;
}

.validate-button {
    background-color: #4CAF50 !important;
    color: white !important;
    text-transform: uppercase;
    font-size: 1.1em;
    letter-spacing: 1px;
    border: none;
    border-radius: 8px;
    margin-top: 30px !important;
    font-weight: bold;
    padding: 12px 40px !important;
    box-shadow: 0 4px 15px rgba(76, 175, 80, 0.5);
    transition: all 0.2s;
}

.validate-button:hover:not([disabled]) {
    background-color: #45a049 !important;
    box-shadow: 0 6px 20px rgba(76, 175, 80, 0.7);
}

.validate-button[disabled] {
    opacity: 0.5;
    background-color: #ccc !important;
    color: #888 !important;
    box-shadow: none;
}

@media (min-width: 1264px) {
    .center-area {
        align-items: flex-start !important;
    }
}

@media (max-width: 1263px) {
    .game-zone {
        border-radius: 12px 12px 0 0;
    }

    .game-card {
        padding-bottom: 15px;
    }

    .game-board-area {
        width: 100%;
        margin-top: 0;
    }

    .game-board-visual {
        min-width: 95%;
    }
}
</style>