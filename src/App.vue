<template>
  <div id="app">
		<GameCardsStack
			:cards="visibleCards"
			@cardAccepted="handleCardAccepted"
			@cardRejected="handleCardRejected"
			@cardSkipped="handleCardSkipped"
			@hideCard="removeCardFromDeck"
		/>
  </div>
</template>

<script>
import GameCardsStack from './components/GameCardsStack';
import germanNouns from '../static/german_nouns.json';
const NUMBER_OF_NOUNS = 2000;

export default {
  name: 'App',
  components: {
    GameCardsStack
  },

  data() {
    return {
      visibleCards: [],
    };
  },

  mounted() {
    this.visibleCards = this.pickRandomWords(3);
  },

  methods: {
    handleCardAccepted() {
      console.log('handleCardAccepted');
    },
    handleCardRejected() {
      console.log('handleCardRejected');
    },
    handleCardSkipped() {
      console.log('handleCardSkipped');
    },
    removeCardFromDeck() {
      this.visibleCards.shift();
      this.visibleCards.push(this.pickRandomWords(1)[0]);
    },
    pickRandomWords(numberOfWords) {
      let randomWords = [];
      for (let i = 0; i < numberOfWords; i++) {
        const randomInd = Math.floor(Math.random() * NUMBER_OF_NOUNS);
        randomWords.push(germanNouns[randomInd]);
      }
      return randomWords;
    }
  }
};
</script>

<style lang='scss'>
@import './styles/mixins.scss';

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  text-align: center;
}
</style>
