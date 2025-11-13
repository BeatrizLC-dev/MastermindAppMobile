<template>
  <v-app>
  <v-app-bar app color="primary" dark>
    <v-toolbar-title>MASTERMIND</v-toolbar-title>
    <v-spacer></v-spacer>
    <v-btn icon><v-icon>mdi-help-circle</v-icon></v-btn>
  </v-app-bar>

  <v-main>
    <v-container>

      <v-row justify="center" class="mb-4">
        <v-col cols="12" class="text-center">
          [ Image of 4 large grey circles with a question mark in the center ]
        </v-col>
      </v-row>
      
      <v-divider></v-divider>

      <v-row class="ma-0 pa-0">
        <v-col cols="12" class="pa-0">
          <div style="max-height: 40vh; overflow-y: auto;">
            <v-row align="center" class="my-2 pa-2 border-bottom">
              <v-col cols="7">
                [ Image of 4 colored circles: Blue, Green, Red, Yellow ]
              </v-col>
              <v-col cols="5">
                [ Image of 1 black circle and 2 white circles ]
              </v-col>
            </v-row>
          </div>
        </v-col>
      </v-row>

      <v-divider></v-divider>
      
      <v-row justify="center" class="mt-4">
        <v-col cols="12" class="text-center">
          <p class="font-weight-bold">Votre essai :</p>
          [ Image of 4 bordered empty circles ready for color selection ]
        </v-col>
        
        <v-col cols="12" class="text-center">
          <p class="font-weight-bold">Couleurs disponibles :</p>
          [ Image of 6 small buttons/circles with different colors: Red, Blue, Green, Yellow, Orange, Purple ]
        </v-col>

        <v-col cols="12" class="d-flex justify-space-around">
          <v-btn color="error">Effacer</v-btn>
          <v-btn color="success" x-large>Valider</v-btn>
        </v-col>
      </v-row>

    </v-container>
  </v-main>
</v-app>
</template>