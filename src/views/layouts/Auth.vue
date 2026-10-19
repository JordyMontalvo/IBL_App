<template>
  <div class="auth">
    <slot name="custom-layout">
      <section>
        <header v-if="showLogo">
          <div class="logo-container">
            <img src="@/assets/img/logo/logoibl.png" alt="IBL Inmobiliaria">
            <slot name="hero"></slot>
          </div>
        </header>

        <slot/>
      </section>
    </slot>
  </div>
</template>

<script>

export default {
  props: {
    showLogo: {
      type: Boolean,
      default: true
    }
  },
  
  computed: {
    // msn()  { return this.$store.state.msn },
    // wsp()  { return this.$store.state.wsp_bo },
    wsp()  { return this.$store.state.wsp_ec },
    site() { return this.$store.state.site },

    // social
    fb() { return this.$store.state.fb },
    is() { return this.$store.state.is },
    tk() { return this.$store.state.tk },
    yt() { return this.$store.state.yt },
  },
};
</script>
