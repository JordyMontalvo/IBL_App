<template>
  <Auth :showLogo="false">
    <section class="welcome-container">
      <div class="logo-container2">
        <img class="logo" src="@/assets/img/logo/logoibl.png">
      </div>
      
      <router-link to="/login" class="button">Ingresar a Oficina</router-link>
      <router-link to="/register" class="button">Registrar Distribuidor</router-link>
      
      <a  target="_blank" class="whatsapp-link">
        <i class="fa-brands fa-whatsapp"></i>
        WhatsApp de ayuda
      </a>
    </section>
  </Auth>
</template>

<script>
import Auth from '@/views/layouts/Auth'
import '@/assets/style/welcome.styl'

export default {
  components: { Auth },
  computed: {
    wsp() { return this.$store.state.wsp_pe }
  },
}
</script>

