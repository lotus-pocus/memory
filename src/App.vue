<template>
  <h1>Memory Game</h1>
  <transition-group tag="section" class="game-board" name="shuffle-card">
    <card
      v-for="card in cardList"
      :key="`${card.value}-${card.variant}`"
      :matched="card.matched"
      :value="card.value"
      :visible="card.visible"
      :position="card.position"
      @select-card="flipCard"
    />
  </transition-group>
  <h2>{{ status }}</h2>
  <button v-if="newPlayer" @click="startGame" class="button"><img src="../public/images/play.svg" alt="start Icon">  Start Game</button>
  <button v-else @click="restartGame" class="button"><img src="../public/images/restart.svg" alt="Restart Icon">  Restart Game</button>
</template>

<script>
import _ from "lodash";
import { computed, ref, watch } from "vue";
import { launchConfetti } from '../src/utilities/confetti'
import card from "./components/card_comp.vue";

export default {
  name: "App",
  components: {
    card,
  },
  setup() {
    const cardList = ref([]);
    const userSelection = ref([]);
    const newPlayer = ref(true)

    const startGame = () => {
      newPlayer.value = false

      restartGame()
    }

    const status = computed(() => {
      if (remainingPairs.value === 0) {
        return "Player Wins!";
      } else {
        return `Remaining Pairs: ${remainingPairs.value}`;
      }
    });

    const remainingPairs = computed(() => {
      const remainingCards = cardList.value.filter(
        (card) => card.matched === false
      ).length;

      return remainingCards / 2;
    });

    const restartGame = () => {

      cardList.value = _.shuffle(cardList.value)

      cardList.value = cardList.value.map((card, index) => {
        return {
          ...card,
          matched: false,
          position: index,
          visible: false,
        };
      });
    };

    const cardItems = [
      "hamster",
      "dog",
      "axe",
      "soda",
      "bread",
      "moose",
      "storm",
      "accordian",
    ];

    cardItems.forEach((item) => {
      cardList.value.push({
        value: item,
        variant: 1,
        visible: false,
        position: null,
        matched: false,
      });
      cardList.value.push({
        value: item,
        variant: 2,
        visible: true,
        position: null,
        matched: false,
      });
    });

    cardList.value = cardList.value.map((card, index) => {
      return {
        ...card,
        position: index,
      };
    });

    const flipCard = (payload) => {
      cardList.value[payload.position].visible = true;

      if (userSelection.value[0]) {
        if (
          userSelection.value[0].position === payload.position &&
          userSelection.value[0].faceValue === payload.faceValue
        ) {
          return;
        } else {
          userSelection.value[1] = payload;
        }
      } else {
        userSelection.value[0] = payload;
      }
    };

    watch (remainingPairs, currentValue => {
      if (currentValue === 0) {
        launchConfetti()
      }
    })

    watch(
      userSelection,
      (currentValue) => {
        if (currentValue.length === 2) {
          const cardOne = currentValue[0];
          const cardTwo = currentValue[1];

          if (cardOne.faceValue === cardTwo.faceValue) {
            cardList.value[cardOne.position].matched = true;
            cardList.value[cardTwo.position].matched = true;
          } else {
            setTimeout(() => {
              cardList.value[cardOne.position].visible = false;
              cardList.value[cardTwo.position].visible = false;
            }, 2000);
          }

          userSelection.value.length = 0;
        }
      },
      { deep: true }
    );

    return {
      cardList,
      flipCard,
      userSelection,
      status,
      restartGame,
      startGame,
      newPlayer
    };
  },
};
</script>

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
  color: white;
  background-image: url(../public/images/game.png);
  background-repeat: no-repeat;
  background-attachment: fixed;
  background-size: 100% 100%;
  height: 100vh;
  padding-top: 60px;
}

.button {
  background-color: rgb(231, 173, 17);
  color: white;
  padding: 1rem 3rem;
  text-align: center;
  text-decoration: none;
  outline: none;
  border: none;
  border-radius: 10px;
  border: 2px white;
  font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  font-weight:bolder;
  cursor: pointer;
  font-size: 3vh;
  margin: 0 auto;
}

.button:hover {
  background-color: #a143e9;
}

.button img {
  padding-right: 10px;
}


.game-board {
  display: grid;
  grid-template-columns: repeat(4, 120px);
  grid-template-rows: repeat(4, 120px);
  grid-column-gap: 24px;
  grid-row-gap: 24px;
  justify-content: center;
}

.shuffle-card-move {
  transition: transform 0.8s ease-in;
}
</style>
