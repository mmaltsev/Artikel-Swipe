<template>
  <div class="cards">
    <GameCard
      v-for="(card, index) in cards"
      :key="(card.english_translation + index)"
      :cardWord="card.word"
      :cardArticle="card.article"
      :cardEnTranslation="card.english_translation"
      :is-current="index === 0"
      @cardAccepted="$emit('cardAccepted');"
      @cardRejected="$emit('cardRejected');"
      @cardSkipped="$emit('cardSkipped');"
      @hideCard="$emit('hideCard');"
    />
    <div class="tags">
      <div class="tag tag-der noselect">Der <span class="tag-arrow">→</span></div>
      <div class="tag tag-die noselect"><span class="tag-arrow">←</span> Die</div>
      <div class="tag tag-das noselect">Das <span class="tag-arrow">↑</span></div>
    </div>
  </div>
</template>

<script>
import GameCard from '@/components/GameCard';

export default {
  components: {
    GameCard
  },

  props: {
    cards: {
      type: Array,
      required: true
    }
  }
};
</script>

<style lang='scss' scoped>
.cards {
  position: relative;
  display: flex;
  margin: auto;
  width: 300px;
  padding-top: 100px;
}
.tags {
  position: absolute;
  width: 300px;
  height: 400px;
}
.tag {
  color: white;
  z-index: 3;
  position: relative;
  width: 55px;
  border-radius: 20px;
  display: flex;
  justify-content: space-evenly;
  padding: 3px;
}
.noselect {
  -webkit-touch-callout: none; /* iOS Safari */
  -webkit-user-select: none; /* Safari */
  -khtml-user-select: none; /* Konqueror HTML */
  -moz-user-select: none; /* Old versions of Firefox */
  -ms-user-select: none; /* Internet Explorer/Edge */
  user-select: none; /* Supported by Chrome, Edge, Opera and Firefox */
}
.tag-der {
  background: #4c83ff;
  margin-top: 150px;
  margin-right: -10px;
  float: right;
}
.tag-die {
  background: #c25259;
  margin-top: 150px;
  margin-left: -10px;
  float: left;
}
.tag-das {
  background: #60bd69;
  margin: -10px auto 0 auto;
}
.tag-arrow {
  font-size: 0.8em;
}

@media only screen and (max-width: 768px) {
  .cards {
    width: 83vw;
    padding-top: 10vh;
  }
  .tags {
    width: 83vw;
    height: 60vh;
  }
  .tag-der, .tag-die {
    margin-top: 35vh;
  }
}
</style>
