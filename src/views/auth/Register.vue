<template>
  <Auth :showLogo="false">
    <template #custom-layout>
      <div class="auth-split-wrapper">
        <!-- Columna Izquierda: Hero Informativo -->
        <div class="auth-hero-col">
          <img class="hero-brand-logo" src="@/assets/img/logo/logoibl.png" alt="IBL Inmobiliaria" />
          <div class="hero-accent-dash"></div>
          <h1 class="hero-headline">Forma parte del equipo comercial</h1>
          <p class="hero-tagline">Regístrate para ofrecer membresías IBL.</p>
        </div>

        <!-- Columna Derecha: Tarjeta Blanca de Registro -->
        <div class="register-card-box">
          <h2 class="card-title-main">Registro de vendedor</h2>
          <p class="card-subtitle-main">Únete al equipo comercial de IBL y empieza a ofrecer nuestras membresías.</p>
          <p class="card-account-link">
            ¿Ya tienes cuenta? <router-link to="/login" class="link-bold">Inicia sesión aquí</router-link>
          </p>

          <form @submit.prevent="submit" class="register-form">
            <!-- Fila 1: País y Documento -->
            <div class="form-grid-row">
              <div class="form-item">
                <label class="input-label">País</label>
                <div class="select-container">
                  <select
                    class="form-ctrl select-ctrl"
                    v-model="country"
                    :class="{ 'has-error': error.country }"
                    @change="reset('country')"
                  >
                    <option :value="null" disabled>Selecciona tu país</option>
                    <option value="Perú">Perú</option>
                    <option value="Bolivia">Bolivia</option>
                    <option value="Ecuador">Ecuador</option>
                    <option value="Colombia">Colombia</option>
                  </select>
                  <i class="fas fa-chevron-down select-arrow"></i>
                </div>
              </div>

              <div class="form-item">
                <label class="input-label">Documento de identidad</label>
                <input
                  type="text"
                  class="form-ctrl"
                  placeholder="Ingresa tu documento"
                  oninput="this.value=this.value.replace(/(?![0-9])./gmi,'')"
                  v-model="dni"
                  :class="{ 'has-error': error.dni }"
                  @keydown="reset('dni')"
                />
                <label class="checkbox-subtext">
                  <input type="checkbox" v-model="younger" class="chk-box" />
                  <span>Menor de edad / extranjero</span>
                </label>
              </div>
            </div>

            <!-- Fila 2: Nombre y Apellidos -->
            <div class="form-grid-row">
              <div class="form-item">
                <label class="input-label">Nombre</label>
                <input
                  type="text"
                  class="form-ctrl"
                  placeholder="Tus nombres"
                  v-model="name"
                  :class="{ 'has-error': error.name }"
                  @keydown="reset('name')"
                />
              </div>

              <div class="form-item">
                <label class="input-label">Apellidos</label>
                <input
                  type="text"
                  class="form-ctrl"
                  placeholder="Tus apellidos"
                  v-model="lastName"
                  :class="{ 'has-error': error.lastName }"
                  @keydown="reset('lastName')"
                />
              </div>
            </div>

            <!-- Fila 3: Fecha de nacimiento y Celular -->
            <div class="form-grid-row">
              <div class="form-item">
                <label class="input-label">Fecha de nacimiento</label>
                <div class="date-ctrl-wrapper">
                  <input
                    type="date"
                    class="form-ctrl date-ctrl"
                    placeholder="dd/mm/aaaa"
                    v-model="date"
                  />
                  <i class="far fa-calendar-alt date-icon"></i>
                </div>
              </div>

              <div class="form-item">
                <label class="input-label">Celular</label>
                <input
                  type="text"
                  class="form-ctrl"
                  placeholder="Número de celular"
                  maxlength="12"
                  v-model="phone"
                  :class="{ 'has-error': error.phone }"
                  @keydown="reset('phone')"
                />
              </div>
            </div>

            <!-- Fila 4: Correo electrónico y Contraseña -->
            <div class="form-grid-row">
              <div class="form-item">
                <label class="input-label">Correo electrónico</label>
                <input
                  type="email"
                  class="form-ctrl"
                  placeholder="Tu correo electrónico"
                  v-model.trim="email"
                  :class="{ 'has-error': error.email }"
                  @keydown="reset('email')"
                />
              </div>

              <div class="form-item">
                <label class="input-label">Contraseña</label>
                <div class="pass-ctrl-wrapper">
                  <input
                    :type="show ? 'text' : 'password'"
                    class="form-ctrl pass-ctrl"
                    placeholder="Crea una contraseña"
                    v-model="password"
                    :class="{ 'has-error': error.password }"
                    @keydown="reset('password')"
                  />
                  <i
                    class="pass-toggle-icon"
                    :class="show ? 'far fa-eye-slash' : 'far fa-eye'"
                    @click="show = !show"
                  ></i>
                </div>
              </div>
            </div>

            <!-- Fila 5: Código de quien te invita (Ancho completo) -->
            <div class="form-grid-row single-column">
              <div class="form-item">
                <label class="input-label">Código de quien te invita</label>
                <input
                  type="text"
                  class="form-ctrl invite-code-ctrl"
                  placeholder="Y7PJPB"
                  :disabled="disabled"
                  v-model="code"
                  :class="{ 'has-error': error.code }"
                  @keydown="reset('code')"
                />
              </div>
            </div>

            <!-- Alerta de validación / error -->
            <p v-if="alert" class="register-alert-msg">{{ alert | alert }}</p>

            <!-- Fila 6: Aceptación de términos -->
            <div class="terms-container">
              <label class="terms-chk-label">
                <input type="checkbox" v-model="check" class="chk-box" />
                <span>Acepto los <a href="#" target="_blank" class="terms-hyperlink">términos de uso</a></span>
              </label>
            </div>

            <!-- Fila 7: Botón Registrarme como vendedor -->
            <button type="submit" class="submit-vendor-btn" :disabled="sending">
              {{ sending ? 'Creando cuenta ...' : 'Registrarme como vendedor' }}
            </button>

            <!-- Fila 8: Regresar -->
            <div class="back-link-container">
              <router-link to="/welcome" class="back-nav-link">Regresar</router-link>
            </div>
          </form>
        </div>
      </div>
    </template>
  </Auth>
</template>

<script>
import Auth from "@/views/layouts/Auth";
import api from "@/api";

export default {
  components: { Auth },
  data() {
    return {
      country: "Perú",
      younger: false,
      dni: null,
      name: null,
      lastName: null,
      date: null,
      email: null,
      phone: null,
      password: null,
      code: null,
      check: false,
      error: {
        country: false,
        dni: false,
        name: false,
        lastName: false,
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
      return "";
    },
  },
  created() {
    this.code = this.$route.params.code;
    if (this.code) this.disabled = true;
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

      if (!country) return (this.error.country = true);
      if (!dni) return (this.error.dni = true);
      if (!name) return (this.error.name = true);
      if (!lastName) return (this.error.lastName = true);
      if (!email) {
        this.error.email = true;
        this.alert = "invalid email";
        return;
      }
      if (!password) return (this.error.password = true);
      if (!code) return (this.error.code = true);
      if (!check) return;

      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (!emailRegex.test(email)) {
        this.error.email = true;
        this.alert = "invalid email";
        return;
      }

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

<style scoped>
.auth-split-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 70px;
  width: 100%;
  max-width: 1180px;
  margin: 0 auto;
  padding: 30px 20px;
  box-sizing: border-box;
}

.auth-hero-col {
  flex: 0 0 380px;
  max-width: 400px;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.hero-brand-logo {
  width: 220px;
  height: auto;
  display: block;
}

.hero-accent-dash {
  width: 34px;
  height: 3px;
  background: #38bdf8;
  border-radius: 2px;
  margin: 22px auto 18px auto;
}

.hero-headline {
  font-size: 34px;
  font-weight: 800;
  color: #ffffff;
  line-height: 1.18;
  margin: 0 0 16px 0;
  letter-spacing: -0.5px;
}

.hero-tagline {
  font-size: 18px;
  color: rgba(255, 255, 255, 0.92);
  line-height: 1.45;
  margin: 0;
}

.register-card-box {
  flex: 1;
  max-width: 580px;
  background: #ffffff;
  border-radius: 20px;
  padding: 40px 44px;
  box-shadow: 0 25px 60px rgba(0, 0, 0, 0.28);
  box-sizing: border-box;
}

.card-title-main {
  font-size: 28px;
  font-weight: 800;
  color: #0f2844;
  text-align: center;
  margin: 0 0 8px 0;
  letter-spacing: -0.4px;
}

.card-subtitle-main {
  font-size: 14px;
  color: #64748b;
  text-align: center;
  line-height: 1.45;
  margin: 0 0 10px 0;
}

.card-account-link {
  font-size: 13.5px;
  color: #64748b;
  text-align: center;
  margin: 0 0 24px 0;
}

.link-bold {
  color: #1e88e5;
  font-weight: 600;
  text-decoration: underline;
}

.register-form {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.form-grid-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.form-grid-row.single-column {
  grid-template-columns: 1fr;
}

.form-item {
  display: flex;
  flex-direction: column;
  text-align: left;
}

.input-label {
  font-size: 12.5px;
  font-weight: 700;
  color: #1e293b;
  margin-bottom: 5px;
}

.form-ctrl {
  width: 100%;
  height: 42px;
  background: #ffffff;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  padding: 8px 14px;
  font-size: 13.5px;
  color: #334155;
  outline: none;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
  box-sizing: border-box;
}

.form-ctrl:focus {
  border-color: #2196f3;
  box-shadow: 0 0 0 3px rgba(33, 150, 243, 0.12);
}

.form-ctrl.has-error {
  border-color: #ef4444;
  background: #fff5f5;
}

.select-container {
  position: relative;
}

.select-ctrl {
  appearance: none;
  -webkit-appearance: none;
  cursor: pointer;
  padding-right: 32px;
}

.select-arrow {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 11px;
  color: #64748b;
  pointer-events: none;
}

.checkbox-subtext {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 11.5px;
  color: #64748b;
  margin-top: 5px;
  cursor: pointer;
}

.chk-box {
  cursor: pointer;
}

.pass-ctrl-wrapper {
  position: relative;
}

.pass-ctrl {
  padding-right: 36px;
}

.pass-toggle-icon {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: #64748b;
  font-size: 14px;
  cursor: pointer;
}

.date-ctrl-wrapper {
  position: relative;
}

.date-ctrl {
  padding-right: 36px;
  cursor: pointer;
}

.date-icon {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: #64748b;
  font-size: 14px;
  pointer-events: none;
}

.date-ctrl::-webkit-calendar-picker-indicator {
  opacity: 0;
  cursor: pointer;
  position: absolute;
  right: 0;
  top: 0;
  width: 36px;
  height: 100%;
}

.invite-code-ctrl {
  font-weight: 600;
  letter-spacing: 0.5px;
}

.register-alert-msg {
  color: #ef4444;
  font-size: 13px;
  font-weight: 600;
  margin: 0;
  text-align: center;
}

.terms-container {
  margin-top: 2px;
}

.terms-chk-label {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12.5px;
  color: #475569;
  cursor: pointer;
}

.terms-hyperlink {
  color: #1e88e5;
  text-decoration: underline;
}

.submit-vendor-btn {
  width: 100%;
  height: 46px;
  background: #1e88e5;
  color: #ffffff;
  font-size: 15px;
  font-weight: 700;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.2s ease, transform 0.1s ease;
  margin-top: 6px;
  box-shadow: 0 4px 14px rgba(30, 136, 229, 0.35);
}

.submit-vendor-btn:hover:not(:disabled) {
  background: #1976d2;
}

.submit-vendor-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.back-link-container {
  text-align: center;
  margin-top: 4px;
}

.back-nav-link {
  color: #1e88e5;
  font-size: 13.5px;
  font-weight: 600;
  text-decoration: none;
}

.back-nav-link:hover {
  text-decoration: underline;
}

/* Responsive Layout */
@media (max-width: 960px) {
  .auth-split-wrapper {
    flex-direction: column;
    gap: 32px;
    padding: 16px;
  }

  .auth-hero-col {
    flex: auto;
    max-width: 100%;
  }

  .hero-headline {
    font-size: 26px;
  }

  .register-card-box {
    width: 100%;
    padding: 28px 20px;
  }
}

@media (max-width: 600px) {
  .form-grid-row {
    grid-template-columns: 1fr;
    gap: 12px;
  }

  .register-card-box {
    padding: 24px 16px;
  }

  .card-title-main {
    font-size: 24px;
  }
}
</style>
