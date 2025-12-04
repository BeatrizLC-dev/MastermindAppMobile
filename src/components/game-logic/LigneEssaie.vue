<script>
export default {
    props: {
        currentGuess: {
            type: Array,
            required: true
        },
        // Ajouté précédemment pour résoudre l'erreur $parent
        getJetonPath: {
            type: Function,
            required: true
        }
    },
    emits: ['select-peg'],

    // CORRECTION : Déclaration de la méthode handleClick dans l'objet 'methods'
    methods: {
        // Gère le clic sur un emplacement (index du trou)
        handleClick(index) {
            // Émettre l'événement avec l'index pour que Game.vue place la couleur sélectionnée
            this.$emit('select-peg', index);
        }
    }
}
</script>
<template>
    <div class="guess-row current-guess">
        <div v-for="(colorName, index) in currentGuess" :key="index" class="guess-peg" @click="handleClick(index)">
            <img v-if="colorName" :src="colorName && getJetonPath(colorName)" :alt="`Jeton ${colorName}`"
                class="jeton-icon-peg" />
        </div>
    </div>
</template>

<style scoped>
.guess-row {
    display: flex;
    gap: 10px;
    margin-bottom: 10px;
    flex-wrap: nowrap !important;
    min-width: 0;
    justify-content: center;
    width: fit-content;
}

.guess-peg {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    border: 1px solid #A08D6D;
    background-color: #E0E0E0;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.1);
}

.jeton-icon-peg {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    object-fit: cover;
}

.guess-row.current-guess .guess-peg:hover {
    box-shadow: 0 0 0 3px #6B8E23, inset 0 1px 4px rgba(0, 0, 0, 0.1);
}
</style>