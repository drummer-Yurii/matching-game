<script>
import { ref, watch } from 'vue';
import createDeck from './features/createDeck'
import createGame from './features/createGame';
import { launchConfetti } from './utilities/confetti';
import Card from './components/Card.vue';
import halloweenDeck from './data/halloweenDeck.json'


export default {
  name: 'App',
  components: {
    Card
  },
  setup() {
    const {cardList} = createDeck(halloweenDeck)
    const { newPlayer, startGame, restartGame, matchesFound, status } = createGame(cardList)
    const userSelection = ref([])  

    const flipCard = (payload) => {
      cardList.value[payload.position].visible = true

      if (userSelection.value[0]) {
        if ((userSelection.value[0].position === payload.position) &&
          (userSelection.value[0].faceValue === payload.faceValue)
        ) {
          return
        } else {
          userSelection.value[1] = payload
        }
      } else {
        userSelection.value[0] = payload
      }
    }

    watch(matchesFound, currentValue => {
      if (currentValue === 0) {
        launchConfetti()
      }
    })

    watch(userSelection, (currentValue) => {
      if (currentValue.length === 2) {
        const cardOne = currentValue[0]
        const cardTwo = currentValue[1]

        if (cardOne.faceValue === cardTwo.faceValue) {
          cardList.value[cardOne.position].matched = true
          cardList.value[cardTwo.position].matched = true
        } else {
          setTimeout(() => {
            cardList.value[cardOne.position].visible = false
            cardList.value[cardTwo.position].visible = false
          }, 2000)
        }

        userSelection.value.length = 0
      }
    }, { deep: true })

    return {
      cardList,
      flipCard,
      userSelection,
      status,
      restartGame,
      startGame,
      newPlayer
    }
  }
}
</script>

<template>
  <h1 class="sr-only">Peek-a-Vue</h1>
  <img src="/images/peek-a-vue-title.png" 
    alt="Peek-a-Vue" class="title" />
    <section class="description">
      <p>Welcome to Peek-a-Vue!</p>
      <p>A card matching game powered by Vue.js 3!</p>
    </section>
  <transition-group tag="section" class="game-board" name="shuffle-card">
    <Card v-for="(card) in cardList" :key="`${card.value}-${card.variant}`" :matched="card.matched" :value="card.value"
      :visible="card.visible" :position="card.position" @select-card="flipCard" />
  </transition-group>
  <h2 class="status">{{ status }}</h2>
  <button v-if="newPlayer" @click="startGame" class="button"><img src="/images/play.svg" alt="Restart Icon" /> Start Game</button>
  <button v-else @click="restartGame" class="button"><img src="/images/restart.svg" alt="Restart Icon" /> Restart Game</button>
</template>

<style>
html,
body {
  margin: 0;
  padding: 0;
}

h1 {
  margin-top: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background-image: url('../public/images/page-bg.png');
  background-color: #00070c;
  height: 100vh;
  color: #fff;
  padding-top: 60px;
}

.description {
  font-family: 'Titillium Web', sans-serif;
}

.description p {
  margin: 0;
  font-size: 1.2rem;
}

.description p:last-child {
  margin-bottom: 30px; 
}

.status {
  font-family: 'Titillium Web', sans-serif;
}

.button {
  background-color: orange;
  color: white;
  padding: 0.75rem 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
  font-weight: bold;
  font-family: 'Titillium Web', sans-serif;
  font-weight: bold;
  font-size: 1.1rem;
  border: 0;
  border-radius: 10px;
}

.button img {
  padding-right: 5px;
}

.game-board {
  display: grid;
  grid-template-columns: repeat(4, 120px);
  grid-template-rows: repeat(4, 120px);
  grid-column-gap: 24px;
  grid-row-gap: 24px;
  justify-content: center;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  border: 0;
}

.title {
  padding-bottom: 30px;
}

.shuffle-card-move {
  transition: transform 0.8s ease-in;
}
</style>
