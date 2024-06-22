<template>
  <div
    v-if="isShowing"
    ref="interactElement"
    :class="{
      isAnimating: isInteractAnimating,
      isCurrent: isCurrent
    }"
    class="card"
    :style="{ transform: transformString }"
  >
    <div class="cardTitle">{{ cardWord }}</div>
    <div
      v-if="!isTranslationVisible && isCurrent"
      class="translation show-translation-button"
      @click="isTranslationVisible=true"
      @touchend="isTranslationVisible=true"
    >
      Show translation
    </div>
    <div
      v-if="isTranslationVisible && isCurrent"
      class="translation translated-text"
    >
      {{ cardEnTranslation }}
    </div>
  </div>
</template>

<script>
import interact from 'interact.js';
const DER_ARTICLE = 'cardAccepted';
const DIE_ARTICLE = 'cardRejected';
const DAS_ARTICLE = 'cardSkipped';

export default {
  static: {
    interactMaxRotation: 15,
    interactOutOfSightXCoordinate: 500,
    interactOutOfSightYCoordinate: 600,
    interactYThreshold: 150,
    interactXThreshold: 100
  },

  props: {
    cardWord: {
      type: String,
      required: true
    },
    cardArticle: {
      type: String,
      required: true
    },
    cardEnTranslation: {
      type: String,
      required: true
    },
    isCurrent: {
      type: Boolean,
      required: true
    }
  },

  data() {
    return {
      isShowing: true,
      isTranslationVisible: false,
      isInteractAnimating: true,
      isInteractDragged: null,
      interactPosition: {
        x: 0,
        y: 0,
        rotation: 0
      }
    };
  },

  computed: {
    transformString() {
      if (!this.isInteractAnimating || this.isInteractDragged) {
        const { x, y, rotation } = this.interactPosition;
        return `translate3D(${x}px, ${y}px, 0) rotate(${rotation}deg)`;
      }

      return null;
    }
  },

  mounted() {
    const element = this.$refs.interactElement;

    interact(element).draggable({
      onstart: () => {
        this.isInteractAnimating = false;
      },

      onmove: event => {
        const {
          interactMaxRotation,
          interactXThreshold
        } = this.$options.static;
        const x = this.interactPosition.x + event.dx;
        const y = this.interactPosition.y + event.dy;

        let rotation = interactMaxRotation * (x / interactXThreshold);

        if (rotation > interactMaxRotation) rotation = interactMaxRotation;
        else if (rotation < -interactMaxRotation)
          rotation = -interactMaxRotation;

        this.interactSetPosition({ x, y, rotation });
      },

      onend: () => {
        const { x, y } = this.interactPosition;
        const { interactXThreshold, interactYThreshold } = this.$options.static;
        this.isInteractAnimating = true;
        if (x > interactXThreshold) {
          if (this.cardArticle === 'Der') {
            this.saveAttempt(true);
            this.playCard(DER_ARTICLE);
          } else {
            this.saveAttempt(false);
            this.resetCardPosition();
          }
        }
        else if (x < -interactXThreshold) {
          if (this.cardArticle === 'Die') {
            this.saveAttempt(true);
            this.playCard(DIE_ARTICLE);
          } else {
            this.saveAttempt(false);
            this.resetCardPosition();
          }
        }
        else if (y < -interactYThreshold) {
          if (this.cardArticle === 'Das') {
            this.saveAttempt(true);
            this.playCard(DAS_ARTICLE);
          } else {
            this.saveAttempt(false);
            this.resetCardPosition();
          }
        }
        else this.resetCardPosition();
      }
    });
  },

  beforeDestroy() {
    interact(this.$refs.interactElement).unset();
  },

  methods: {
    hideCard() {
      setTimeout(() => {
        this.isShowing = false;
        this.$emit('hideCard', this.card);
      }, 300);
    },

    playCard(interaction) {
      const {
        interactOutOfSightXCoordinate,
        interactOutOfSightYCoordinate,
        interactMaxRotation
      } = this.$options.static;

      this.interactUnsetElement();

      switch (interaction) {
        case DER_ARTICLE:
          this.interactSetPosition({
            x: interactOutOfSightXCoordinate,
            rotation: interactMaxRotation
          });
          this.$emit(DER_ARTICLE);
          break;
        case DIE_ARTICLE:
          this.interactSetPosition({
            x: -interactOutOfSightXCoordinate,
            rotation: -interactMaxRotation
          });
          this.$emit(DIE_ARTICLE);
          break;
        case DAS_ARTICLE:
          this.interactSetPosition({
            y: -interactOutOfSightYCoordinate
          });
          this.$emit(DAS_ARTICLE);
          break;
      }

      this.hideCard();
    },

    saveAttempt(isRight) {
      let correctAnswers = JSON.parse(localStorage.getItem('correctAnswers') || '[]');
      // if a word is already in wrongAnswers at the last index, no need to add it again
      let wrongAnswers = JSON.parse(localStorage.getItem('wrongAnswers') || '[]');
      if (isRight) {
        correctAnswers.push(this.cardWord);
        localStorage.setItem('correctAnswers', JSON.stringify(correctAnswers));
      } else {
        wrongAnswers.push(this.cardWord);
        localStorage.setItem('wrongAnswers', JSON.stringify(wrongAnswers));
      }
    },

    interactSetPosition(coordinates) {
      const { x = 0, y = 0, rotation = 0 } = coordinates;
      this.interactPosition = { x, y, rotation };
    },

    interactUnsetElement() {
      interact(this.$refs.interactElement).unset();
      this.isInteractDragged = true;
    },

    resetCardPosition() {
      this.interactSetPosition({ x: 0, y: 0, rotation: 0 });
    }
  }
};
</script>

<style lang='scss' scoped>
@import '../styles/index.scss';

$cardsTotal: 3;
$cardsWidth: 300px;
$cardsPositionOffset: 55vh * 0.06;
$cardsScaleOffset: 0.08;
$defaultTranslation: $cardsPositionOffset * $cardsTotal;
$defaultScale: 1 - ($cardsScaleOffset * $cardsTotal);
$fs-card-title: 1.925em;

.card {
  @include card();
  @include absolute(0);
  @include sizing(100% 80vw);
  @include flex-center();

  @include after() {
    @include sizing(21px 3px);
    @include absolute(right 0 bottom 11px left 0);

    margin: auto;
    border-radius: 100px;
    background: rgba($c-black, 0.3);
  }

  display: flex;
  max-height: 400px;
  margin: auto;
  font-size: $fs-h1;
  font-weight: $fw-bold;
  color: $c-black;
  background-image: linear-gradient(
    -180deg,
    $primary-gradient-start 2%,
    $primary-gradient-end 100%
  );
  opacity: 0;
  transform: translateY($defaultTranslation) scale($defaultScale);
  transform-origin: 50%, 100%;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  user-select: none;
  pointer-events: none;
  will-change: transform, opacity;

  height: 60vh;

  &.isCurrent {
    pointer-events: auto;
  }

  &.isAnimating {
    transition: transform 0.7s $ease-out-back;
  }
}

.cardTitle {
  margin-top: 105px;
  font-size: $fs-card-title;
}

@for $i from 1 through $cardsTotal {
  $index: $i - 1;
  $translation: $cardsPositionOffset * $index;
  $scale: 1 - ($cardsScaleOffset * $index);

  .card:nth-child(#{$i}) {
    z-index: $cardsTotal - $index;
    opacity: 1;
    transform: translateY($translation) scale($scale);

    @if $i == 3 {
      color: $c-gray-25;
      background-color: $c-gray-25;
    } @else if $i == 2 {
      color: $c-gray-50;
      background-color: $c-gray-50;
    }

    @if $i != 1 {
      background-image: none;

      @include after() {
        @include sizing(0 0);
      }
    }
  }
}
.translation {
  position: fixed;
  margin-top: 230px;
  font-weight: 100;
}
.show-translation-button {
  background: #cfcfcf;
  color: #fff;
  padding: 3px 10px;
  border-radius: 20px;
  cursor: pointer;
}
@media only screen and (max-width: 768px) {
  .card {
    max-height: 600px;
  }
  .cardTitle {
    margin-top: 30vh;
  }
  .translation {
    margin-top: 45vh;
  }
  .show-translation-button {
    background: #bbb;
  }
}
</style>
