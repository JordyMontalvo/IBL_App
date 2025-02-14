<template>
  <Auth>
    
    <section>
      <span class="separator"></span>

      <h1>Iniciar sesion</h1>
      <br>
    
        
        <div class="form-group">
          <label>Correo Electrónico:</label>
          <div class="input-wrapper">
            <input 
              type="email"
              class="input" 
              placeholder="Correo Electronico"
              v-model="dni"
              :class="{'error': error.dni}"
              @keydown="reset('dni')"
            >
            <i class="fa-solid fa-user icon"></i>
          </div>
        </div>

        <div class="form-group" v-if="!office_id">
          <label>Contraseña:</label>
          <div class="input-wrapper">
            <input 
              :type="show ? 'text' : 'password'" 
              class="input" 
              placeholder="************"
              v-model="password"
              :class="{'error': error.password}"
              @keydown="reset('password')"
            >
            <i class="fa-solid fa-lock icon" @click="show = !show"></i>
          </div>
        </div>

        <p class="forgot-text">¿Olvidaste tu contraseña? <a href="#">Ingresa aquí</a></p>

        <button class="button" v-show="!sending" @click="submit">Iniciar Sesión</button>
        <button class="button" v-show="sending" disabled>Validando datos ...</button>

        <!-- Nuevo botón de registro -->
        <div class="register-container">
    <span class="register-text">¿No tienes cuenta? <router-link to="/register" class="register-link">Regístrate</router-link></span>
  </div>
    </section>
    <footer>
      <!-- <router-link to="/welcome" class="route">Regresar</router-link>-->
      <br>
      <header>
          <!-- <div class="social">
          <!-- <a class="fab fa-facebook-square" :href="fb" target="_blank"></a>
          <a class="fab fa-instagram"       :href="is" target="_blank"></a>
          <a class="fab fa-tiktok"          :href="tk" target="_blank"></a>
          <a class="fab fa-youtube"         :href="yt" target="_blank"></a> 
          <a class="fab fa-facebook-square" :href="fb" target="_blank"></a>
          <a class="fab fa-instagram"       :href="is" target="_blank"></a>
          <!-- <a class="fab fa-tiktok"          target="_blank"></a> 
          <a class="fab fa-youtube"         :href="yt" target="_blank"></a>  -->
        
      </header>

    </footer>
  </Auth>
</template>

<script>
import Auth from '@/views/layouts/Auth'
import api  from '@/api'

export default {
  components: { Auth },
  data() {
    return {
      // username:    null,
      dni:      null,
      password: null,
      error: {
        // username:     false,
        dni:      false,
        password: false,
      },
      sending: false,
      alert:   null,
      show:    false,

      office_id: null,
      path:      null,
    }
  },
  computed: {
    // social
    fb() { return this.$store.state.fb },
    is() { return this.$store.state.is },
    tk() { return this.$store.state.tk },
    yt() { return this.$store.state.yt },
  },
  filters: {
    alert(msg) {
      // if (msg == 'user not found')   return 'El usuario no existe'
      if (msg == 'dni not found')    return 'El documento no existe'
      if (msg == 'invalid password') return 'Contraseña incorrecta'
    },
  },
  created() {
    this.office_id = this.$route.params.id
    this.path      = this.$route.query.path
    console.log({ office_id: this.office_id, path: this.path })

    if(this.office_id) {
      this.password  = '8QfghvCxuzxrbvii4w'
    } else {
      
      localStorage.removeItem('office')
      localStorage.removeItem('path')
    }
  },
  methods: {
    async submit() {

      // const { username, password } = this
      const { dni, password, office_id, path } = this
      console.log({ dni, password, office_id, path })

      // valid fields
      // if(!username) { return this.error.username = true }
      if(!dni) { return this.error.dni = true }
      if(!password) { return this.error.password = true }

      // POST Login
      this.sending = true

      // const { data } = await api.login({ username, password }); console.log({ data })
      const { data } = await api.login({ dni, password, office_id }); console.log({ data })

      this.sending = false

      // error
      if(data.error) return this.alert = data.msg

      // login
                    this.$store.commit('SET_SESSION',   data.session)
      if(office_id) this.$store.commit('SET_OFFICE_ID', {office_id, path})

      // routing
      if(office_id)
        this.$router.push(`/${this.path}`)
      else 
        this.$router.push('/dashboard')
    },
    reset(name) {
      this.alert = null

      // if(name == 'username') this.error.username = false
      if(name == 'dni')      this.error.dni      = false
      if(name == 'password') this.error.password = false
    },
  },
};
</script>
