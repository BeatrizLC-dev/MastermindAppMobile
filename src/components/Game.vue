<script>
import SelecteurDifficulte from './SelecteurDifficulte.vue';

</script>

<template>
  <v-card outlined class="game-card custom-border d-flex flex-column"> 
    
    <v-card-text class="pa-0 flex-grow-1">
        <v-container fluid class="fill-height pa-0">
            <v-row no-gutters class="fill-height">
                
                <v-col cols="8" class="game-zone fill-height pa-3 d-flex align-center justify-center">
                    <p>test</p>
                </v-col>
                
                <v-divider vertical></v-divider> 

                <v-col cols="4" class="difficulty-zone fill-height pa-3 d-flex align-center justify-center">
                    <SelecteurDifficulte />
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
}

.custom-border {
    box-shadow: 0 0 0 1px black !important;
}

.game-zone {
    overflow-y: auto; 
}
</style>