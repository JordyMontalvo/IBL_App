<template>
  <div class="thanks-overlay" v-if="show">
    <div class="thanks-box" role="dialog" aria-modal="true" aria-labelledby="thanks-title">
      <i class="fas fa-check-circle thanks-icon"></i>
      <h3 id="thanks-title">¡Gracias por su compra!</h3>
      <button class="thanks-btn" @click="home">Volver al inicio</button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    show: { type: Boolean, default: false },
  },
  methods: {
    home() {
      this.$emit('close')
      this.$router.push('/dashboard')
    },
  },
}
</script>

<style lang="stylus">
.thanks-overlay
  position fixed
  top 0
  left 0
  right 0
  bottom 0
  background rgba(0, 0, 0, 0.6)
  display flex
  align-items center
  justify-content center
  padding 16px
  z-index 2000

.thanks-box
  background white
  padding 30px 24px
  border-radius 12px
  text-align center
  max-width 400px
  width 100%
  box-shadow 0 10px 25px rgba(0, 0, 0, 0.2)

  h3
    margin 15px 0 25px
    font-size 22px
    color #2c3e50

  .thanks-icon
    font-size 60px
    color #2ecc71

  .thanks-btn
    padding 12px 24px
    border none
    border-radius 6px
    cursor pointer
    font-weight 600
    background #08385c
    color white
    width 100%
    max-width 240px
    &:hover
      background #062b47
</style>
