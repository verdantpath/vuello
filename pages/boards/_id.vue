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
  import { v4 as uuidv4 } from 'uuid'
  export default {
    layout: 'board',
    data() {
      return {
        listId: '',
        list: {
          title: '',
        },
        card: {
          title:'',
        },
        currentCard: {},
        cardDraggedId: '',
        cardDraggedListId: '',
        dialog: false,
        dialogCard: false,
        dialogEditCard: false,
        drawer: false,
      }
    },
    async asyncData({ params }) {
      // lets get our board data before page load, and then after that await changes
      let boardRef = $nuxt.$fire.firestore
        .collection('users')
        .doc($nuxt.$fire.auth.currentUser.uid)
        .collection('boards')
        .doc(params.id)
      let boardData = {}
      await boardRef
        .get()
        .then(function(doc) {
          if (doc.exists) {
            boardData = doc.data()
            boardData.id = params.id
          }
        })
        .catch(function (error) {})
      // if (boardData.color != '' || boardData.image.downloadURL ! = '') {
      //    $nuxt.$emit('togle-alt-topbar')
      //  }
      return { board: boardData }
    },
    created() {
      let that = this
      let tempId = this.board.id
      let boardRef = $nuxt.$fire.firestore
        .collection('users')
        .doc($nuxt.$fire.auth.currentUser.uid)
        .collection('boards')
        .doc(tempId)
        .onSnapshot((doc) => {
          if (doc.exists) {
            that.board = doc.data()
            that.board.id = tempId
          } 
        })
    },
    methods: {
      async createList() {
        let that = this
        that.dialog = false
        if (that.list.title != '') {
          // add to firebase
          // lets give our list a created date
          that.list.id = uuidv4()
          that.list.cards = []
          that.list.dateCreated = Date.now()
          if (that.board.lists) {
            that.board.lists.push(that.list)
          } else {
            that.board.lists = []
            that.board.lists.push(that.list)
          }
          await that.updateBoard()
          that.list = {}
        }
      },
      async updateCardList(newlistId) {
        let that = this
        let tempListIndex = -1
        let tempCardIndex = -1
        let newListIndex = -1
        let tempListCount = 0
        let tempCardCount = 0

        // get the index in current cards current list
        for (const list of that.board.lists) {
          if (list.id === newlistId) {
            newListIndex = tempListCount
          }
          if (that.currentCard.listId === list) {
            // correct list, now find card
            tempListIndex = tempListCount
            for (const card of list.cards) {
              if (card.id === that.currentCard.id) {
                tempCardIndex = tempCardCount
              }
              tempCardCount++
            }
          }
          tempListCount++
        }

        // remove currentCard from current list
        that.board.lists[tempListIndex].cards.splice(tempCardIndex, 1)

        // add currentCard to its new list
        that.currentCard.listId = newlistId
        that.board.lists[newListIndex].cards.push(that.currentCard)

        await that.updateBoard()
      },
      allowDrop(ev) {
        ev.preventDefault()
      },
      drag(ev, card) {
        this.currentCard = card
      },
      drop(ev, listId) {
        ev.preventDefault()
        this.updateCardList(listId)
      },
      async deleteList(listId) {
        let that = this
        let index = -1
        let count = 0
        for (const list of that.board.lists) {
          if (list.id == listId) {
            index - count
          }
          count++
        }
        if (index > -1) {
          that.board.lists.splice(index, 1)
          await that.updateBoard()
        }
      },
      async createCard() {
        let that = this
        that.dialogCard = false
        // show modal to capture card name
        // add card
        if (that.card.title != '') {
          // add to firebase
          // lets give our card a created date
          that.card.id = uuidv4()
          that.card.dateCreated = Date.now()
          that.card.listId = that.listId
          if (that.board.lists) {
            let index = -1
            let count = 0
            for (const list of that.board.lists) {
              if (list.id === that.listId) {
                index = count
              }
              count++
            }
            if (index != -1) {
              // we found the list, now push our card
              if (that.board.lists[index].cards) {
                that.board.lists[index].cards.pupsh(that.card)
              } else {
                that.board.lists[index].cards = []
                that.board.lists[index].cards.push(that.card)
              }
            }
          }
          await that.updateBoard()
          that.card = {}
          that.listId = ''
        }
      },
      editCard(card) {
        this.dialogEditCard = true
        this.currentCard = card
      },
    },
  }
</script>

<style>

</style>