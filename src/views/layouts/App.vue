<template>
  <div class="app">
    <header>
      <!--<h3 class="slogan">
        <span v-if="country == 'Perú'"       style="font-size: 28px;">🇵🇪</span>
        <span v-if="country == 'Bolivia'"    style="font-size: 28px;">🇧🇴</span>
        <span v-if="country == 'Ecuador'"    style="font-size: 28px;">🇪🇨</span>
        <span v-if="country == 'Colombia'"   style="font-size: 28px;">🇨🇴</span>
        <span v-if="country == 'Argentina'"  style="font-size: 28px;">🇦🇷</span>
        <span v-if="country == 'Chile'"      style="font-size: 28px;">🇨🇱</span>
        <span v-if="country == 'Costa Rica'" style="font-size: 28px;">🇨🇷</span>
          &nbsp;&nbsp;&nbsp;SUEÑA SIN LIMITES
      </h3>-->
      <img src="../../assets/img/logo/logoibl.png" alt="" class="logo" />
      <i
        class="burger fas fa-bars"
        style="margin-left: 310px"
        @click.stop="toggleMenu"
      ></i>

      <!-- <h4>{{ name }} {{ lastName }} <i class=""
                      :class="{'yellow': affiliated, 'blue': _activated, 'green': activated}"></i>
      </h4>-->
    </header>
    <section :class="{ open: open }">
      <div class="menu">
        <div class="text-header">
          <label v-if="office_id == null">
            <img v-if="photoState == 'default'" class="photo" :src="photo" />
            <img v-if="photoState == 'changed'" class="photo" :src="newPhoto" />

            <input type="file" @change="changePhoto" />
          </label>

          <div v-if="photoState == 'changed'" class="controls">
            <i @click="cancelNewPhoto" class="fas fa-times"></i>
            <i @click="changeNewPhoto" class="fas fa-check"></i>
          </div>
          <div>
            <p style="color: white; font-size: 18px">
              {{ name }} {{ lastName }}
            </p>
            <p style="color: white; font-size: 12px">{{ email }}</p>
          </div>
        </div>
        <!-- <div class="social" style="display: flex;" v-if="office_id == null">
          <a class="fab fa-facebook-square" :href="fb" target="_blank" style="font-size: 18px;color: #4267B2;"></a>
          <a class="fab fa-instagram"       :href="is" target="_blank" style="font-size: 18px;color: #e95950;"></a>
          <a class="fab fa-youtube"         :href="yt" target="_blank" style="font-size: 18px;color: #ff0050;"></a>
        </div>-->

        <img src="../../assets/img/logo/logoibl.png" class="photo-logo" alt="IBL Inmobiliaria" />

        <div class="menu-category">Menú Principal</div>

        <router-link to="/dashboard" exact class="menu-link" @click.native="close" v-if="office_id == null">
          <div class="menu-link-left">
            <i class="fas fa-home menu-icon"></i>
            <span>Inicio</span>
          </div>
        </router-link>

        <!-- Productos -->
        <div class="menu-group" :class="{ 'is-open': openSections.products }">
          <a class="menu-link menu-parent" @click.stop="toggleSection('products')">
            <div class="menu-link-left">
              <i class="fas fa-cube menu-icon"></i>
              <span>Productos</span>
            </div>
            <i :class="openSections.products ? 'fas fa-chevron-up' : 'fas fa-chevron-down'" class="chevron-icon"></i>
          </a>
          <div class="sub-menu" v-show="openSections.products">
            <router-link to="/activation" exact class="sub-link" @click.native="close" v-if="affiliated">
              <i class="fas fa-tag sub-icon"></i>
              <span>Activaciones</span>
            </router-link>
            <router-link to="/membership" exact class="sub-link" @click.native="close" v-if="affiliated">
              <i class="fas fa-users sub-icon"></i>
              <span>Venta de Membresías</span>
            </router-link>
          </div>
        </div>

        <!-- Equipo comercial -->
        <div class="menu-group" :class="{ 'is-open': openSections.network }" v-if="tree">
          <a class="menu-link menu-parent" @click.stop="toggleSection('network')">
            <div class="menu-link-left">
              <i class="fas fa-users menu-icon"></i>
              <span>Equipo comercial</span>
            </div>
            <i :class="openSections.network ? 'fas fa-chevron-up' : 'fas fa-chevron-down'" class="chevron-icon"></i>
          </a>
          <div class="sub-menu" v-show="openSections.network">
            <router-link to="/tree" exact class="sub-link" @click.native="close">
              <i class="fas fa-users sub-icon"></i>
              <span>Mi equipo</span>
            </router-link>
            <router-link to="/directs" exact class="sub-link" @click.native="close">
              <i class="fas fa-user-plus sub-icon"></i>
              <span>Incorporar vendedores</span>
            </router-link>
          </div>
        </div>

        <!-- Comisiones -->
        <div class="menu-group" :class="{ 'is-open': openSections.commissions }" v-if="tree">
          <a class="menu-link menu-parent" @click.stop="toggleSection('commissions')">
            <div class="menu-link-left">
              <i class="fas fa-coins menu-icon"></i>
              <span>Comisiones</span>
            </div>
            <i :class="openSections.commissions ? 'fas fa-chevron-up' : 'fas fa-chevron-down'" class="chevron-icon"></i>
          </a>
          <div class="sub-menu" v-show="openSections.commissions">
            <router-link
              to="/transfer"
              class="sub-link"
              :class="{ 'router-link-active': $route.path === '/transfer' || $route.path === '/transfers' }"
              @click.native="close"
            >
              <i class="fas fa-wallet sub-icon"></i>
              <span>Monedero</span>
            </router-link>
            <router-link to="/transactions" exact class="sub-link" @click.native="close">
              <i class="fas fa-dollar-sign sub-icon"></i>
              <span>Movimientos</span>
            </router-link>
            <router-link
              to="/collect"
              class="sub-link"
              :class="{ 'router-link-active': $route.path === '/collect' || $route.path === '/collects' }"
              @click.native="close"
            >
              <i class="fas fa-hand-holding-usd sub-icon"></i>
              <span>Retiros</span>
            </router-link>
          </div>
        </div>

        <!-- Resumen -->
        <div class="menu-group" :class="{ 'is-open': openSections.resume }" v-if="tree && office_id == null">
          <a class="menu-link menu-parent" @click.stop="toggleSection('resume')">
            <div class="menu-link-left">
              <i class="fas fa-file-alt menu-icon"></i>
              <span>Resumen</span>
            </div>
            <i :class="openSections.resume ? 'fas fa-chevron-up' : 'fas fa-chevron-down'" class="chevron-icon"></i>
          </a>
          <div class="sub-menu" v-show="openSections.resume">
            <router-link to="/bonuses" exact class="sub-link" @click.native="close" v-if="affiliated">
              <i class="fas fa-shopping-bag sub-icon"></i>
              <span>Bonificaciones</span>
            </router-link>
            <router-link to="/resume" exact class="sub-link" @click.native="close">
              <i class="fas fa-users sub-icon"></i>
              <span>Personal</span>
            </router-link>
            <router-link to="/closeds" exact class="sub-link" @click.native="close">
              <i class="fas fa-calendar-check sub-icon"></i>
              <span>Cierres</span>
            </router-link>
          </div>
        </div>

        <!-- Proyectos -->
        <router-link
          to="/tools"
          exact
          class="menu-link"
          @click.native="close"
          v-if="office_id == null && affiliated"
        >
          <div class="menu-link-left">
            <i class="fas fa-building menu-icon"></i>
            <span>Proyectos</span>
          </div>
        </router-link>

        <!-- Perfil -->
        <router-link
          to="/profile"
          exact
          class="menu-link"
          @click.native="close"
          v-if="office_id == null"
        >
          <div class="menu-link-left">
            <i class="fas fa-user menu-icon"></i>
            <span>Perfil</span>
          </div>
        </router-link>

        <div class="menu-divider"></div>

        <!-- Cerrar sesión -->
        <a class="menu-link menu-logout" @click="logout">
          <div class="menu-link-left">
            <i class="fas fa-sign-out-alt menu-icon"></i>
            <span>Cerrar sesión</span>
          </div>
        </a>
      </div>

      <div class="content">
        <header>
          <div class="page-heading">
            <p class="page-heading__title">{{ title }}</p>
            <p class="page-heading__crumbs" v-if="crumbs.length">
              <span>{{ crumbs[0] }}</span>
              <span v-if="crumbs[1]"> › {{ crumbs[1] }}</span>
            </p>
          </div>
          <div
            style="
              display: flex;
              align-items: center;
              gap: 20px;
              margin-top: 25px;
            "
          >
            <img class="logo" style="height: 50px" />
            <button class="share-button" @click="copy_affiliation_link">
              <i class="fas fa-share-alt" style="margin-right: 8px"></i
              >Compartir Afiliación
              <input
                readonly
                id="link-global"
                @click="copy_affiliation_link"
                v-model="affiliationLink"
                style="opacity: 0; position: absolute"
              />
            </button>
            <i class="copy" v-if="c_affiliation_link"
              >link copiado <i class="fas fa-check"></i
            ></i>

            <!-- <img class="logo-text" src="@/assets/img/logo/text.svg" style="margin-left: 12px;"> -->

            <label v-if="office_id == null">
              <img
                v-if="photoState == 'default'"
                class="photo-header"
                :src="photo"
              />
              <img
                v-if="photoState == 'changed'"
                class="photo-header"
                :src="newPhoto"
              />

              <input type="file" @change="changePhoto" />
            </label>

            <div v-if="photoState == 'changed'" class="controls">
              <i @click="cancelNewPhoto" class="fas fa-times"></i>
              <i @click="changeNewPhoto" class="fas fa-check"></i>
            </div>
            <div>
              <p style="color: rgba(8, 56, 92, 1); font-size: 18px">
                {{ name }} {{ lastName }}
              </p>
              <p style="color: rgba(8, 56, 92, 1); font-size: 12px">
                {{ email }}
              </p>
            </div>
          </div>

          <!-- <div class="social">
            <a class="fab fa-facebook-square" :href="fb" target="_blank" style="color: #4267B2;"></a>
            <a class="fab fa-instagram"       :href="is" target="_blank" style="color: #e95950;"></a>
            <a class="fab fa-tiktok"          :href="tk" target="_blank" style="color: #ff0050;"></a>
            <a class="fab fa-youtube"         :href="yt" target="_blank" style="color: #ff0050;"></a>
            
          </div> -->
        </header>
        <section style="overflow: auto"
        :class="{ slide: open }"
        @touchstart="startTouch"
        @touchmove="moveTouch"
        @touchend="endTouch">
          <slot />
        </section>
      </div>
    </section>

    <footer class="footer-Dashboard">
      <router-link to="/dashboard">
        <i class="fa-solid fa-house"></i>
        Inicio
      </router-link>
      <router-link to="/activation" v-if="affiliated">
        <i class="fas fa-bolt"></i>
        Activar
      </router-link>
      <router-link to="/affiliation" v-if="!affiliated">
        <i class="fas fa-shopping-bag"></i>
        Plan
      </router-link>
      <router-link to="/tree" v-if="tree">
        <i class="fas fa-project-diagram"></i>
        Red
      </router-link>
      <router-link to="/collect" v-if="tree">
        <i class="fas fa-hand-holding-usd"></i>
        Retiros
      </router-link>
    </footer>

    <a :href="wsp" target="_blank" class="wsp fab fa-whatsapp"></a>
  </div>
</template>

<script>
import api from "@/api";
import lib from "@/lib";

const ROOT = process.env.VUE_APP_ROOT;
console.log({ ROOT });

export default {
  props: {
    session: String,
    office_id: String,
    title: String,
    crumbs: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      // photo: 'https://ik.imagekit.io/asu/Lehaim/avatar_bEyc3MFLf.png',
      newPhoto: null,
      photoState: "default",
      photoFile: null,
      c_affiliation_link: false,
      startX: 0,
      endX: 0,
      openSections: {
        products: false,
        network: false,
        commissions: false,
        resume: false,
      },
    };
  },
  watch: {
    "$route.path"() {
      this.syncOpenSections();
    },
  },
  mounted() {
    this.syncOpenSections();
  },
  computed: {
    // user
    name() {
      return this.$store.state.name;
    },
    lastName() {
      return this.$store.state.lastName;
    },
    affiliated() {
      return this.$store.state.affiliated;
    },
    activated() {
      return this.$store.state.activated;
    },
    _activated() {
      return this.$store.state._activated;
    },
    plan() {
      return this.$store.state.plan;
    },
    country() {
      return this.$store.state.country;
    },
    photo() {
      return this.$store.state.photo;
    },
    tree() {
      return this.$store.state.tree;
    },
    email() {
      return this.$store.state.email;
    },

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

    // help
    wsp() {
      if (this.country == "Perú") return this.$store.state.wsp_pe;
      if (this.country == "Bolivia") return this.$store.state.wsp_bo;
      if (this.country == "Ecuador") return this.$store.state.wsp_ec;
    },

    // menú
    open() {
      return this.$store.state.open;
    },
    resume() {
      return this.$store.state.resume;
    },
    buys() {
      return this.$store.state.buys;
    },
    network() {
      return this.$store.state.network;
    },
    commissions() {
      return this.$store.state.commissions;
    },
    affiliationLink() {
      return `${ROOT}/register/${this.token}`;
    },
    token() {
      return this.$store.state.token;
    },
    isProducts() {
      return this.$route.path === "/activation" || this.$route.path === "/membership";
    },
  },
  methods: {
    toggleMenu() {
      this.$store.commit("SET_OPEN");
    },
    closeMenu() {
      if (this.open) {
        this.$store.commit("SET_OPEN");
      }
    },
    close() {
      if (this.open) {
        this.$store.commit("SET_OPEN");
      }
    },
    syncOpenSections() {
      const path = this.$route.path;
      if (path === "/activation" || path === "/membership") {
        this.openSections.products = true;
      } else if (path === "/tree" || path === "/directs") {
        this.openSections.network = true;
      } else if (
        path === "/transfer" ||
        path === "/transfers" ||
        path === "/transactions" ||
        path === "/collect" ||
        path === "/collects"
      ) {
        this.openSections.commissions = true;
      } else if (path === "/bonuses" || path === "/resume" || path === "/closeds") {
        this.openSections.resume = true;
      }
    },
    toggleSection(section) {
      const willOpen = !this.openSections[section];
      this.openSections.products = false;
      this.openSections.network = false;
      this.openSections.commissions = false;
      this.openSections.resume = false;
      this.openSections[section] = willOpen;
    },
    changePhoto(e) {
      this.photoFile = e.target.files[0];

      if (!this.photoFile) return;

      const reader = new FileReader();

      reader.onload = (e) => {
        this.newPhoto = e.target.result;
        this.photoState = "changed";
      };

      reader.readAsDataURL(this.photoFile);
    },
    async changeNewPhoto() {
      const ret = await lib.upload(this.photoFile, this.photoFile.name, "photos");

      this.$store.commit("SET_PHOTO", ret);

      this.photoState = "default";

      await api.photo(this.session, { photo: this.photo });
    },
    cancelNewPhoto() {
      this.photoState = "default";
    },
    copy_affiliation_link() {
      console.log("Copying Affiliation Link:", this.affiliationLink);
      lib.copy("link-global");
      this.c_affiliation_link = true;
      setTimeout(() => (this.c_affiliation_link = false), 4000);
    },
    startTouch(event) {
      this.startX = event.touches[0].clientX; // Guardar la posición inicial del toque
    },
    moveTouch(event) {
      this.endX = event.touches[0].clientX; // Actualizar la posición del toque
    },
    endTouch() {
      // Si el deslizamiento es hacia la izquierda y el menú está abierto, cerrarlo
      if (this.startX - this.endX > 100 && this.open) {
        this.closeMenu();
      }
    },
  },
};
</script>
<style scoped>
.menu {
  transition: transform 0.3s ease;
  transform: translateX(-100%); /* Oculta el menú fuera de la vista */
}

.menu.slide {
  transform: translateX(0); /* Muestra el menú */
}
</style>
