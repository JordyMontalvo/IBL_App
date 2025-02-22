<template>
  <Auth>
    <section>
      <h1>Registro</h1>
      <p class="forgot-text">
        ¿Ya tienes cuenta? <a href="/login">Inicia sesión aquí</a>
      </p>

      <br />

      <div>
        <div class="input-wrapper">
          <select
            class="input"
            v-model="country"
            :class="{ error: error.country }"
            @change="reset('country')"
          >
            <option value="null" disabled>Selecciona un país</option>
            <option value="Perú">🇵🇪 Perú</option>
            <!-- <option value="Ecuador"   >🇪🇨 Ecuador</option>
            <option value="Argentina" >🇦🇷 Argentina</option>
            <option value="Bolivia"   >🇧🇴 Bolivia</option>
            <option value="Colombia"  >🇨�� Colombia</option>
            <option value="Costa Rica">🇨🇷 Costa Rica</option>
            <option value="Chile"     >🇨🇱 Chile</option> -->
          </select>
          <i class="fa-solid fa-flag icon"></i>
        </div>
      </div>

      <div>
        <div class="input-wrapper">
          <input
            type="text"
            class="input"
            placeholder="Documento de identidad"
            oninput="this.value=this.value.replace(/(?![0-9])./gmi,'')"
            v-model="dni"
            :class="{ error: error.dni }"
            @keydown="reset('dni')"
          />
          <i class="fa-solid fa-id-card icon"></i>
        </div>
      </div>

      <div>
        <label>
          <small>
            <input type="checkbox" v-model="younger" />menor de edad /
            extranjero
          </small>
        </label>
      </div>
      <br />

      <div>
        <div class="input-wrapper">
          <input
            type="text"
            class="input"
            placeholder="Nombre"
            v-model="name"
            :class="{ error: error.name }"
            @keydown="reset('name')"
          />
          <i class="fa-solid fa-user icon"></i>
        </div>
      </div>

      <div>
        <div class="input-wrapper">
          <input
            type="text"
            class="input"
            placeholder="Apellidos"
            v-model="lastName"
            :class="{ error: error.lastName }"
            @keydown="reset('lastName')"
          />
          <i class="fa-solid fa-user icon"></i>
        </div>
      </div>

      <div class="input-wrapper">
        <input
          placeholder="Fecha de nacimiento"
          class="input"
          type="date"
          />

      </div>

      <div>
        <div class="input-wrapper-error">
          <small
            v-if="country"
            style="min-width: 25px; margin-right: 8px; display: inline-block"
            >{{ prefix }}</small
          >
          <input
            type="text"
            class="input-celular"
            placeholder="Celular"
            maxlength="12"
            v-model="phone"
          />
          <i class="fa-solid fa-mobile-alt icon" v-if="!country"></i>
        </div>
      </div>

      <div>
        <div class="input-wrapper">
          <input
            type="email"
            class="input"
            placeholder="Correo"
            v-model.trim="email"
          />
          <i class="fa-solid fa-envelope icon"></i>
        </div>
      </div>

      <div>
        <div class="input-wrapper">
          <input
            :type="show ? 'text' : 'password'"
            class="input"
            placeholder="Contraseña"
            v-model="password"
            :class="{ error: error.password }"
            @keydown="reset('password')"
          />
          <i class="fa-solid fa-lock icon"></i>
          <i class="show far fa-eye" @click="show = !show"></i>
        </div>
      </div>

      <div>
        <div class="input-wrapper">
          <input
            type="text"
            class="input"
            placeholder="Código de patrocinador"
            :disabled="disabled"
            v-model="code"
            :class="{ error: error.code }"
            @keydown="reset('code')"
          />
          <i class="fa-solid fa-paper-plane icon"></i>
        </div>
      </div>

      <p class="alert">{{ alert | alert }}</p>

      <small
        ><input type="checkbox" v-model="check" />Acepto los
        <a href="" target="_blank" style="color: white; font-weight: 600"
          >términos de uso</a
        ></small
      >
      <br />

      <button class="button" v-show="!sending" @click="submit">
        Registrarme
      </button>
      <button class="button" v-show="sending" disabled>
        Creando cuenta ...
      </button>
      <br /><br />
    </section>
    <footer>
      <router-link to="/welcome" class="footer-register">Regresar</router-link>
      <br />
      <header>
        <div class="social">
          <!-- <a class="fab fa-facebook-square" :href="fb" target="_blank"></a>
          <a class="fab fa-instagram"       :href="is" target="_blank"></a>
          <a class="fab fa-tiktok"          :href="tk" target="_blank"></a>
          <a class="fab fa-youtube"         :href="yt" target="_blank"></a> -->
          <a
            class="fab fa-facebook-square"
            target="_blank"
            style="color: white"
          ></a>
          <a
            class="fab fa-instagram"
            target="_blank"
            style="color: white"
          ></a>
          <!-- <a class="fab fa-tiktok"          target="_blank"></a> -->
          <a
            class="fab fa-youtube"
            target="_blank"
            style="color: white"
          ></a>
        </div>
      </header>
    </footer>
  </Auth>
</template>

<script>
import Auth from "@/views/layouts/Auth";
import api from "@/api";

export default {
  components: { Auth },
  data() {
    return {
      // country: 'Perú',
      // country: 'Bolivia',
      // country: 'Ecuador',
      country: null,

      younger: false,
      dni: null,
      name: null,
      lastName: null,
      // username: null,
      date: null,
      email: null,
      phone: null,
      password: "123456",
      code: null,
      check: false,
      error: {
        country: false,
        dni: false,
        name: false,
        lastName: false,
        // username: false,
        email: false,
        phone: false,
        password: false,
        code: false,
      },
      sending: false,
      alert: null,
      disabled: false,
      show: false,
    };
  },
  filters: {
    alert(msg) {
      if (msg == "dni already use") return "El documento ya existe";
      if (msg == "code not found") return "El código de invitación no existe";
      if (msg == "email already use")
        return "Este correo electrónico ya está registrado";
      if (msg == "invalid email")
        return "Por favor ingrese un correo electrónico válido";
    },
  },
  computed: {
    // social
    fb() {
      return this.$store.state.fb;
    },
    is() {
      return this.$store.state.is;
    },
    tk() {
      return this.$store.state.tk;
    },
    yt() {
      return this.$store.state.yt;
    },

    prefix() {
      if (this.country == "Ecuador") return "+593";
      if (this.country == "Perú") return "+51";
      if (this.country == "Argentina") return "+54";
      if (this.country == "Bolivia") return "+591";
      if (this.country == "Colombia") return "+57";
      if (this.country == "Costa Rica") return "+506";
      if (this.country == "Chile") return "+56";
    },
  },
  created() {
    this.code = this.$route.params.code;

    if (this.code) this.disabled = true;
  },
  watch: {
    /* async dni(dni) {
      if(this.country != 'Perú' || this.younger) return

      if(dni.length < 8) return

      const response = await fetch(`https://dniruc.apisperu.com/api/v1/dni/${dni}?token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6ImhhbnNldmFuZ2VsaXN0YUBnbWFpbC5jb20ifQ.Fa2w9sYLXtcgMbtM58YRiwYvomChYwwMAoIDmYmo1H8`)

      var data = await response.json()
      console.log(data)

      if(data.success == false) {
        console.log('person not found ...')
        this.error.name = true
        return
      }

      this.error.name = false

      this.name     = data.nombres
      this.lastName = data.apellidoPaterno + ' ' + data.apellidoMaterno
    } */
  },
  methods: {
    async submit() {
      const {
        country,
        dni,
        name,
        lastName,
        date,
        email,
        password,
        phone,
        code,
        check,
      } = this;

      // valid fields
      if (!country) {
        return (this.error.country = true);
      }
      if (!dni) {
        return (this.error.dni = true);
      }
      if (!name) {
        return (this.error.name = true);
      }
      if (!lastName) {
        return (this.error.lastName = true);
      }
      if (!email) {
        this.error.email = true;
        this.alert = "invalid email";
        return;
      }
      if (!password) {
        return (this.error.password = true);
      }
      // if(!phone)    { return this.error.phone    = true }
      if (!code) {
        return (this.error.code = true);
      }
      if (!check) {
        return;
      }

      // Validar formato de email
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (!emailRegex.test(email)) {
        this.error.email = true;
        this.alert = "invalid email";
        return;
      }

      // POST Register
      this.sending = true;

      try {
        const { data } = await api.register({
          country,
          dni,
          name,
          lastName,
          date,
          email: email.toLowerCase().trim(),
          password,
          phone,
          code,
        });

        console.log("Respuesta del registro:", data);

        if (data.error) {
          if (data.msg === "email already use") {
            this.error.email = true;
            this.alert = data.msg;
          } else if (data.msg === "dni already use") {
            this.error.dni = true;
            this.alert = data.msg;
          } else {
            this.alert = data.msg;
          }
          this.sending = false;
          return;
        }

        // Si todo está bien, proceder con el login
        this.$store.commit("SET_SESSION", data.session);
        this.$router.push("/dashboard");
      } catch (error) {
        console.error("Error en registro:", error);
        this.alert = "Error en el registro";
      } finally {
        this.sending = false;
      }
    },
    reset(name) {
      this.alert = null;
      if (name == "country") this.error.country = false;
      if (name == "dni") this.error.dni = false;
      if (name == "name") this.error.name = false;
      if (name == "lastName") this.error.lastName = false;
      if (name == "email") this.error.email = false;
      if (name == "password") this.error.password = false;
      if (name == "code") this.error.code = false;
    },
  },
};
</script>
