<script>
import SelecteurDifficulte from './SelecteurDifficulte.vue';
import Historique from './game-logic/Historique.vue';
import LigneEssaie from './game-logic/LigneEssaie.vue';

// --- FONCTIONS D'OBFUSCATION (Double Encryptage Léger) ---

// Clé de stockage utilisée pour sauvegarder le code secret dans le localStorage
const STORAGE_KEY = 'mastermind_secret_code';

/**
 * Chiffre le tableau de couleurs du code secret pour un stockage léger et non évident
 * dans le localStorage. Cela n'est pas une sécurité, juste une obfuscation.
 * @param {string[]} codeArray - Le tableau de couleurs représentant le code secret.
 * @returns {string} Le code secret obscurci (string, base64, puis inversé).
 */
function encryptCode(codeArray) {
    const codeString = codeArray.join('|'); // Joint les couleurs avec un séparateur
    const base64 = btoa(codeString); // Encode en Base64
    const reversed = base64.split('').reverse().join(''); // Inverse la chaîne pour la "seconde" couche d'obfuscation
    return reversed;
}


// Mappage des jetons
// Définit le chemin d'accès pour l'icône de chaque couleur de jeton disponible.
const COLOR_MAP = {
    'Gris': './jeton-Gris.ico',
    'Bleu': './jeton-Bleu.ico',
    'Brun': './jeton-Brun.ico',
    'Jaune': './jeton-Jaune.ico',
    'Cyan': './jeton-Cyan.ico',
    'Orange': './jeton-Orange.ico',
    'Rose': './jeton-Rose.ico',
    'Rouge': './jeton-Rouge.ico',
    'Vert': './jeton-Vert.ico',
    'Violet': './jeton-Violet.ico',
};

// Tableau contenant tous les noms de couleurs disponibles.
const ALL_JETON_NAMES = Object.keys(COLOR_MAP);

// DÉFINITION DES RÈGLES DE DIFFICULTÉ
// Paramètres de jeu pour chaque niveau de difficulté.
const DIFFICULTY_SETTINGS = {
    'facile': { colorsCount: 4, attempts: 10, codeLength: 4 }, // 4 couleurs disponibles, 10 essais, code de 4 pions
    'normal': { colorsCount: 5, attempts: 9, codeLength: 5 }, // 5 couleurs disponibles, 9 essais, code de 5 pions
    'moyen': { colorsCount: 7, attempts: 8, codeLength: 5 }, // 7 couleurs disponibles, 8 essais, code de 5 pions
    'difficile': { colorsCount: 9, attempts: 7, codeLength: 5 }, // 9 couleurs disponibles, 7 essais, code de 5 pions
    'expert': { colorsCount: 10, attempts: 6, codeLength: 5 }, // 10 couleurs disponibles, 6 essais, code de 5 pions
};

export default {
    name: 'MastermindGame',

    components: {
        SelecteurDifficulte,
        Historique,
        LigneEssaie,
    },

    // Données réactives du composant
    data() {
        return {
            currentDifficulty: 'normal', // Niveau de difficulté sélectionné
            selectedColorName: null, // Nom de la couleur sélectionnée par l'utilisateur pour placer un pion

            // Objet contenant l'état actuel de la partie
            gameData: {
                secretCode: [], // Le code secret généré par l'ordinateur
                currentGuess: Array(4).fill(null), // Le tableau représentant la ligne d'essai actuelle de l'utilisateur
                attempts: [], // L'historique des tentatives (guess et feedback)
                currentRow: 0, // Le numéro de la ligne d'essai actuelle (compteur de tentatives)
            },
        };
    },

    // Propriétés calculées
    computed: {
        /**
         * Retourne les noms des couleurs de jetons disponibles pour la difficulté actuelle.
         * @returns {string[]} Liste des noms de couleurs.
         */
        availableColors() {
            const settings = DIFFICULTY_SETTINGS[this.currentDifficulty] || DIFFICULTY_SETTINGS['normal'];
            // Utilise les 'colorsCount' pour prendre les N premières couleurs du tableau ALL_JETON_NAMES
            return ALL_JETON_NAMES.slice(0, settings.colorsCount);
        },
        /**
         * Retourne le nombre maximum de tentatives autorisées pour la difficulté actuelle.
         * @returns {number} Nombre maximum d'essais.
         */
        maxAttempts() {
            const settings = DIFFICULTY_SETTINGS[this.currentDifficulty] || DIFFICULTY_SETTINGS['normal'];
            return settings.attempts;
        },
        /**
         * Retourne la longueur du code (nombre de pions par ligne) pour la difficulté actuelle.
         * @returns {number} Longueur du code.
         */
        codeLength() {
            const settings = DIFFICULTY_SETTINGS[this.currentDifficulty] || DIFFICULTY_SETTINGS['normal'];
            return settings.codeLength;
        }
    },

    // Observateur de propriétés (Watchers)
    watch: {
        /**
         * Déclenche un redémarrage du jeu (resetGame) lorsque le niveau de difficulté change.
         */
        currentDifficulty() {
            this.resetGame();
        }
    },

    // Cycle de vie : exécuté après le montage du composant
    mounted() {
        // Initialise une nouvelle partie au chargement
        this.resetGame();
    },

    // Méthodes
    methods: {
        /**
         * Génère un nouveau code secret aléatoire basé sur les paramètres de difficulté actuels.
         * Le code est ensuite stocké de manière obscurcie dans le localStorage.
         */
        generateSecretCode() {
            const colors = this.availableColors;
            const code = [];
            for (let i = 0; i < this.codeLength; i++) {
                const randomIndex = Math.floor(Math.random() * colors.length);
                code.push(colors[randomIndex]);
            }

            this.gameData.secretCode = code;

            // Obfusque et stocke dans le localStorage (principalement à des fins de débogage/vérification côté client)
            const encrypted = encryptCode(code);
            localStorage.setItem(STORAGE_KEY, encrypted);
        },

        /**
         * Réinitialise l'état du jeu pour commencer une nouvelle partie.
         * Cela inclut la réinitialisation du tableau d'essais, de l'historique, du compteur de lignes
         * et la génération d'un nouveau code secret.
         */
        resetGame() {
            this.gameData.currentGuess = Array(this.codeLength).fill(null); // Vide la ligne d'essai actuelle
            this.gameData.attempts = []; // Vide l'historique
            this.gameData.currentRow = 0; // Réinitialise le compteur de tentatives
            this.selectedColorName = null; // Désélectionne la couleur
            this.generateSecretCode(); // Génère un nouveau code secret
        },

        /**
         * Met à jour la couleur actuellement sélectionnée par l'utilisateur.
         * @param {string} colorName - Le nom de la couleur sélectionnée.
         */
        selectColor(colorName) {
            this.selectedColorName = colorName;
        },

        /**
         * Place le pion de la couleur sélectionnée à l'index spécifié dans la ligne d'essai actuelle.
         * @param {number} pegIndex - L'index (position) où placer le pion.
         */
        selectPeg(pegIndex) {
            // Vérifie qu'une couleur est sélectionnée et qu'il reste des tentatives
            if (this.selectedColorName && this.gameData.currentRow < this.maxAttempts) {
                // Crée une copie du tableau d'essai pour le modifier de manière immuable
                const newGuess = [...this.gameData.currentGuess];
                newGuess[pegIndex] = this.selectedColorName;
                this.gameData.currentGuess = newGuess;
            }
        },

        /**
         * Valide la combinaison d'essai actuelle.
         * Vérifie si la ligne est complète, calcule les indices (feedback), met à jour l'historique,
         * gère la fin de partie (victoire ou défaite) et passe à la ligne suivante.
         */
        validateCombination() {
            // Vérifie si tous les emplacements ont été remplis
            if (this.gameData.currentGuess.includes(null)) {
                alert("Veuillez remplir toute la ligne.");
                return;
            }

            // Calcule les indices (pions noirs et blancs)
            const feedback = this.checkGuess(this.gameData.currentGuess, this.gameData.secretCode);
            
            // Ajoute la tentative et les indices à l'historique
            this.gameData.attempts.push({
                guess: [...this.gameData.currentGuess],
                feedback: feedback,
            });

            // --- LOGIQUE DE FIN DE PARTIE ---
            
            // Condition de Victoire : Si le nombre de pions noirs est égal à la longueur du code
            if (feedback.black === this.codeLength) {
                alert("Bravo ! Vous avez trouvé le code ! Une nouvelle partie va commencer.");
                // Redémarre le jeu après un court délai
                setTimeout(() => {
                    this.resetGame();
                }, 500);
                return;
            } 
            // Condition de Défaite : Si le joueur a utilisé toutes ses tentatives
            else if (this.gameData.currentRow >= this.maxAttempts - 1) {
                alert(`Partie terminée ! Le code secret était : ${this.gameData.secretCode.join(', ')}.\nUne nouvelle partie va commencer.`);
                // Redémarre le jeu après un court délai
                setTimeout(() => {
                    this.resetGame();
                }, 500);
                return;
            }

            // Si le jeu continue :
            this.gameData.currentRow++; // Incrémente le compteur de tentatives
            this.gameData.currentGuess = Array(this.codeLength).fill(null); // Réinitialise la ligne d'essai pour la prochaine tentative
            this.selectedColorName = null; // Désélectionne la couleur
        },

        /**
         * Calcule les indices (pions noirs et blancs) pour une tentative donnée.
         * Le calcul suit les règles standard du Mastermind.
         * @param {string[]} guess - La combinaison d'essai de l'utilisateur.
         * @param {string[]} secret - Le code secret.
         * @returns {{black: number, white: number}} L'objet contenant le nombre de pions noirs et blancs.
         */
        checkGuess(guess, secret) {
            let blackPegs = 0;
            let whitePegs = 0;
            // Crée des copies pour pouvoir modifier les tableaux sans altérer l'état d'origine
            const secretCopy = [...secret];
            const guessCopy = [...guess];

            // Première passe : Compte les pions noirs (bonne couleur, bonne place)
            for (let i = 0; i < this.codeLength; i++) {
                if (guessCopy[i] === secretCopy[i] && guessCopy[i] !== null) {
                    blackPegs++;
                    // Marque les pions trouvés comme '__USED__' pour ne pas les compter à nouveau
                    secretCopy[i] = guessCopy[i] = '__USED__';
                }
            }
            
            // Deuxième passe : Compte les pions blancs (bonne couleur, mauvaise place)
            for (let i = 0; i < this.codeLength; i++) {
                if (guessCopy[i] !== '__USED__') { // Traite seulement les pions non encore utilisés (ni noirs, ni null)
                    // Recherche la couleur du pion d'essai dans le code secret (qui contient encore les pions non noirs)
                    const secretIndex = secretCopy.findIndex(color => color === guessCopy[i]);
                    if (secretIndex !== -1) {
                        whitePegs++;
                        // Marque le pion du code secret comme utilisé pour éviter les doublons
                        secretCopy[secretIndex] = '__USED__';
                    }
                }
            }
            
            return { black: blackPegs, white: whitePegs };
        },

        /**
         * Fournit le chemin d'accès à l'icône du jeton pour un nom de couleur donné.
         * Utilisé pour l'affichage de tous les jetons dans l'application.
         * @param {string} colorName - Le nom de la couleur du jeton (e.g., 'Bleu').
         * @returns {string} Le chemin d'accès au fichier icône.
         */
        getJetonPath(colorName) {
            // Utilise 'Blanc' par défaut si la couleur n'est pas trouvée (fallback)
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