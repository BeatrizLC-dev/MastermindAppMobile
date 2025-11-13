<script>

</script>

<template>
  <v-card outlined class="game-card custom-border d-flex flex-column"> 
    
    <v-card-title class="pa-3 text-h6 justify-center">
        Mastermind - Niveau: 
    </v-card-title>
    
    <v-divider></v-divider>
    
    <v-card-text class="pa-0 flex-grow-1">
        <v-container fluid class="fill-height pa-0">
            <v-row no-gutters class="fill-height">
                
                <v-col cols="6" class="historique-zone fill-height pa-3">
                    
                </v-col>
                
                <v-divider vertical></v-divider> <v-col cols="6" class="controls-zone fill-height pa-3 d-flex align-center justify-center">
                    
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

.historique-zone {
    overflow-y: auto; 
}
</style>