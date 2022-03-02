<template>
  <div class="d-flex flex-column board" :style="board.image.downloadURL != '' ? `background:url(${board.image.downloadURL});background-size:cover;` : board.color ? `background-color:${board.color}` : ''">
    <div class="d-block">
      <v-container fluid class="jello-topbar">
        <div class="d-flex justify-space-between">
          <v-icon @click="drawer = true">mdi-menu</v-icon>
          <nuxt-link to="/">
            <v-row no-gutters align="center" justify="space-between">
              <h3 class="logo">Jello</h3>
            </v-row>
          </nuxt-link>
          <v-icon small @click="deleteBoard()">mdi-delete-outline</v-icon>
        </div>
      </v-container>
      <v-navigation-drawer v-model="drawer" fixed left class="d-block px-3 py-3">
        <v-container fluid class="jello-topbar">
          <v-row no-gutters align="center" justify="space-between">
            <v-icon @click="drawer = false">mdi-close</v-icon>
            <v-row no-gutters align="center" justify="end">
              <p class="jello-user">
                Signed in as<br />
                {{ $nuxt.$fire.auth.currentUser.email }}
              </p>
              &nbsp;
              <v-icon>mdi-account-circle-outline</v-icon>
            </v-row>
          </v-row>
        </v-container>
        <v-container class="d-block menu-items">
          <div class="d-flex flex-column">
            <div class="d-flex">
              <br />
            </div>
            <div class="d-flex">
              <nuxt-link to="/">
                <v-icon>mdi-view-dashboard-variant-outline</v-icon>
                &nbsp;&nbsp;
                <b>My Boards</b>
              </nuxt-link>
            </div>
            <div class="d-flex">
              <nuxt-link to="/auth/signout">
                <v-icon>mdi-exit-to-app</v-icon>
                &nbsp;&nbsp;
                <b>Sign out</b>
              </nuxt-link>
            </div>
          </div>
        </v-container>
      </v-navigation-drawer>
    </div>
    <h1>{{ board.title }}</h1>
    <small>created {{ board.dateCreated | formatDate }} </small>
    <div class="d-flex flex-row pr-6 pt-3">
      <div v-for="list in board.lists" @drop="drop($event, list.id)" @dragover="allowDrop($event)" class="d-flex flex-column pt-3 mr-6 list" :key="list.id">
        <div class="d-flex flex-row justify-space-between">
          <h4>{{ list.title }}</h4>
          <v-icon small @click="deleteList(list.id)">mdi-delete-outline</v-icon>
        </div>

        <!-- display cards -->
        <v-card v-for="card in list.cards" :draggable="true" @dragover.prevent @dragstart="drag($event, card)" class="mt-5" @click="editCard(card)" :key="card.id">
          <v-card-text>{{ card.title }}</v-card-text>
        </v-card>

        <v-btn depressed @click="dialogCard = true; listId = list.id" class="mt-auto">Add card</v-btn>
      </div>
      <v-dialog v-model="dialogCard" persistent max-width="600px">
        <v-card elevation="0">
          <v-card-title>
            <span class="headline">Card name</span>
          </v-card-title>
          <v-card-text>
            <v-container>
              <v-row>
                <v-col cols="12">
                  <v-text-field label="Stuff to do" v-model="card.title" required></v-text-field>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="dialogCard = false">Close</v-btn>
            <v-btn color="blue darken-1" text @click="createCard()">Save</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <div class="d-flex flex-row">
        <v-btn depressed @click="dialog = true" class="create-list">Create new list</v-btn>
        <v-dialog v-model="dialog" persistent max-width="600px">
          <v-card elevation="0">
            <v-card-title>
              <span class="headline">List name</span>
            </v-card-title>
            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12">
                    <v-text-field label="Stuff to do" v-model="list.title" required></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="dialog=false">Close</v-btn>
              <v-btn color="blue darken-1" text @click="createList()">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </div>
      <v-dialog v-model="dialogEditCard" persistent max-width="600px">
        <v-card elevation="0">
          <v-card-title>
            <span class="headline">{{ currentCard.title }}</span>
          </v-card-title>
          <v-card-text>
            <v-container>
              <v-row>
                <v-col cols="12">
                  <v-text-field label="Edit title" v-model="currentCard.title" required></v-text-field>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="red darken-1" text @click="deleteCard()">Delete</v-btn>
            <v-btn color="red darken-1" text @click="dialogEditCard = false">Close</v-btn>
            <v-btn color="red darken-1" text @click="updateCard()">Save</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </div>
  </div>
</template>
<script>
  export default {
    
  }
</script>
<style>

</style>