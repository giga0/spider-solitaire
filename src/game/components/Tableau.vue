<template>
  <div class="tableau">
    <div v-for="(cards, i) in tableau"
      :key="`tableau-column_${i}`"
      :id="`tableau-column_${i}`"
      class="tableau__column">
      <div v-for="(card, j) in cards"
        :key="`tableau-column-card_${i}-${j}`"
        :style="[{ top: `${j * 3.7}rem` }, { zIndex: `${j}` }]"
        class="tableau__card"
        :class="{ 'face-up': card.faceUp }"
        :ref="card.faceUp ? `faceUpCard-column_${i}-${j}` : 'faceDownCard'"
        @click="cardClicked(j, card, i, cards)">
        <card :card="card" />
        <div :ref="`move-button_${i}-${j}`"
          class="move-button">
          <btn @click="moveBtnClicked($event, i)">Move here</btn>
        </div>
      </div>
      <!-- case if there is no stock -->
      <div v-if="!cards || !cards.length" class="tableau__card tableau__card--empty">
        <div ref="move-button_of-empty-column"
          class="move-button">
          <btn @click="moveBtnClicked($event, i)">Move here</btn>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import classCleaner from '@/-common-/mixins/ClassCleaner.js'
import Btn from '@/-common-/components/Button'
import Card from '@/game/components/Card'

export default {
  name: 'Tableau',
  components: { Btn, Card },
  mixins: [ classCleaner ],
  props: {
    tableau: { type: Array, required: true }
  },
  data () {
    return {
      cardsToMove: {}
    }
  },
  methods: {
    cardClicked (cardIndexOfCards, card, columnIndex, cards) {
      this.classCleaner()
      if (cardIndexOfCards === cards.length - 1 && !card.faceUp) {
        this.$emit('turnTableauCard', { cardIndexOfCards, columnIndex })
        return
      } else if (cardIndexOfCards < cards.length - 1 && !card.faceUp) return
      let selectionIsValid = true
      let columnFaceUpCards = []
      this.cardsToMove = { fromColumn: columnIndex, clickedCardIndex: cardIndexOfCards, cards: [] }
      for (let i = cardIndexOfCards; i < cards.length; i++) {
        let cardToCheck = cards[i]
        columnFaceUpCards = columnFaceUpCards.concat(this.$refs[`faceUpCard-column_${columnIndex}-${i}`])
        this.cardsToMove.cards.push(cardToCheck)
        let withThisCard = cards[i + 1]
        if (withThisCard) {
          let diff = cardToCheck.id - withThisCard.id
          if (diff === 1) selectionIsValid = true
          else selectionIsValid = false
        } else break
      }
      if (selectionIsValid) {
        for (let card of columnFaceUpCards) {
          card.classList.add('is-clicked')
        }
      } else false
      this.checkAvailableColumns(card, columnIndex)
    },
    checkAvailableColumns (card, columnIndex) {
      for (let i = 0; i < this.tableau.length; i++) {
        if (i !== columnIndex) {
          let col = this.tableau[i]
          let lastColCard = col[col.length - 1]
          if (lastColCard && lastColCard.faceUp) {
            let diff = lastColCard.id - card.id
            let columnFaceUpCards = this.$refs[`faceUpCard-column_${i}-${col.length - 1}`]
            let columnBtn = this.$refs[`move-button_${i}-${col.length - 1}`]
            if (diff === 1 && lastColCard.faceUp) {
              columnFaceUpCards[columnFaceUpCards.length - 1].classList.add('is-available')
              columnBtn[0].classList.add('is-visible')
            }
            else columnFaceUpCards[columnFaceUpCards.length - 1].classList.add('not-available')
          } else {
            // case of empty column
            const btnsOfEmptyColumns = this.$refs['move-button_of-empty-column']
            for (let btn of btnsOfEmptyColumns) {
              btn.classList.add('is-visible')
            }
          }
        }
      }
    },
    moveBtnClicked (e, columnIndex) {
      e.stopPropagation()
      this.$emit('moveCards', { columnIndex, cardsToMove: this.cardsToMove })
    }
  },
  updated () {
    // console.log(this.tableau)
  }
}
</script>

<style lang="scss" scoped>
@import '../../-common-/assets/style/settings/_module-settings.scss';

.tableau {
  display: flex;
  &__column {
    position: relative;
    width: 14rem;
    margin-right: 3rem;
  }
  &__card {
    position: absolute;
    width: 14rem;
    height: 19rem;
    &--empty {
      border: 2px solid $darkgrey;
    }
    &:after {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      opacity: .7;
    }
    &.is-clicked {
      &:after {
        content: "";
        background-color: $yellow;
      }
    }
    &.is-available {
      &:after {
        content: "";
        background-color: $green;
      }
    }
    &.not-available {
      &:after {
        content: "";
        background-color: $red;
      }
    }
    .move-button {
      display: none;
      z-index: 1;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      &.is-visible {
        display: block;
      }
      /deep/ button {
        @include fontSizeRem(10, 16);
        white-space: nowrap;
      }
    }
  }
}
</style>