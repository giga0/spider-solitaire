<template>
  <div class="foundations">
    <div v-for="(suit, i) in foundations"
      :key="`foundations-suit_${i}`"
      class="foundations__suit">
      <div v-for="(card, j) in suit"
        :key="`foundations-suit-card_${i}-${j}`"
        :style="[{ top: `-${j / 10}rem` }, { right: `${j / 10}rem` }, { zIndex: `${j}` }]"
        class="foundations__card">
        <card :card="card" />
      </div>
      <!-- case if there is no stock -->
      <div v-if="!suit || !suit.length" class="foundations__card foundations__card--empty"></div>
    </div>
  </div>
</template>

<script>
import Card from '@/game/components/Card'

export default {
  name: 'Foundations',
  components: { Card },
  props: {
    foundations: { type: Array, required: true }
  }
}
</script>

<style lang="scss" scoped>
@import '../../-common-/assets/style/settings/_module-settings.scss';

.foundations {
  display: flex;
  &__suit {
    position: relative;
    width: 9.8rem;
    margin-right: 2.442rem;
  }
  &__card {
    position: absolute;
    width: 9.8rem;
    height: 13.3rem;
    &--empty {
      border: 2px solid $darkgrey;
    }
  }
}
</style>
