<script>
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

        <v-card v-if="mode === 'desktop'" outlined
            class="desktop-sidebar fill-height d-flex flex-column align-center elevation-0">
            <v-card-title class="sidebar-title">Veuillez sélectionner la difficulté</v-card-title>
            <v-card-text class="difficulty-list-desktop pa-0 flex-grow-1 d-flex flex-column align-center">
                <v-btn v-for="level in difficultyLevels" :key="level.key"
                    :class="{ 'active': selectedDifficulty === level.key }" @click="selectLevel(level.key)" class="my-2"
                    block depressed>
                    {{ level.label }}
                </v-btn>
            </v-card-text>
        </v-card>

        <div v-else-if="mode === 'mobile'" class="mobile-bar-container">

            <div class="difficulty-bar-mobile">
                <v-btn v-for="level in difficultyLevels" :key="level.key"
                    :class="{ 'active': selectedDifficulty === level.key }" @click="selectLevel(level.key)" small tile
                    depressed class="flex-grow-1 mx-0">
                    {{ level.label }}
                </v-btn>
            </div>

            <div class="color-palette-mobile mt-2">
                <div v-for="color in availableJetonNames" :key="color" class="color-peg-mobile"
                    @click="selectColor(color)">

                    <img :src="getJetonPath(color)" :alt="`Jeton ${color}`" class="jeton-icon" />

                </div>
            </div>

        </div>

    </div>
</template>

<style scoped>
.desktop-sidebar {
    width: 100%;
    padding: 20px;
    background-color: transparent !important;
    border: none !important;
    box-shadow: none !important;
    height: 100%;
}

.sidebar-title {
    font-size: 18px;
    padding-bottom: 15px;
    border-bottom: 2px solid #A08D6D;
    margin-bottom: 20px;
    width: 100%;
    text-align: center;
    font-weight: 700;
    color: #333;
}

.difficulty-list-desktop {
    width: 90%;
    max-width: 200px;
    flex-grow: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.difficulty-list-desktop .v-btn {
    padding: 12px !important;
    border: 1px solid #ccc !important;
    background-color: white !important;
    color: #444 !important;
    text-transform: none;
    height: auto !important;
    font-size: 15px;
    border-radius: 8px;
    width: 100%;
    transition: all 0.2s ease;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.difficulty-list-desktop .v-btn:hover {
    background-color: #f0f0f0 !important;
    transform: translateY(-1px);
}

.difficulty-list-desktop .v-btn.active {
    background-color: #6B8E23 !important;
    color: white !important;
    font-weight: bold;
    box-shadow: 0 4px 10px rgba(107, 142, 35, 0.4);
    border-color: #6B8E23 !important;
}

.mobile-bar-container {
    width: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.difficulty-bar-mobile {
    display: flex;
    gap: 0;
    border: 1px solid #ccc;
    background-color: #fff;
    border-radius: 8px;
    overflow: hidden;
}

.difficulty-bar-mobile .v-btn {
    font-size: 10px !important;
    padding: 8px 0 !important;
    border-radius: 0 !important;
    background-color: #f0f0f0 !important;
    color: #333 !important;
    text-transform: none;
    min-width: auto !important;
    height: auto !important;
    flex-basis: 0;
}

.difficulty-bar-mobile .v-btn.active {
    background-color: #6B8E23 !important;
    color: white !important;
    font-weight: bold;
    box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.4);
}

.color-palette-mobile {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    gap: 8px;
    width: 100%;
    padding: 10px 5px;
    border: 1px solid #ccc;
    background-color: #fff;
    border-radius: 8px;
    margin-top: 10px;
    max-width: 400px;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.color-peg-mobile {
    width: 35px;
    height: 35px;
    border-radius: 50%;
    cursor: pointer;
    overflow: hidden;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
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