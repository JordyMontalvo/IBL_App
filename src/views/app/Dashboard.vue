<template>
  <App :session="session" :title="title">
    <div class="home">
      <!-- Banner -->
      <figure class="home-banner" v-if="bannerImages.length" :style="bannerStyle">
        <img
          v-for="(img, i) in bannerImages"
          :key="i"
          :src="img"
          alt="IBL Inmobiliaria"
          :class="{ visible: i === slide }"
          @load="i === 0 && onBannerLoad($event)"
        />
      </figure>

      <i class="load" v-if="loading"></i>

      <template v-if="!loading">
        <!-- Fila 1: rango, total ganado, activación -->
        <div class="home-row row-top">
          <div class="home-card rank-card">
            <div class="card-head">
              <i class="fas fa-award head-icon green"></i>
              <h3>Rango alcanzado</h3>
            </div>
            <!-- pin del rango (subido en Admin > Banner > Pines); sin rango se usa la insignia por defecto -->
            <div class="badge">
              <img :src="rankPin || defaultPin" :alt="rankName" />
              <span class="badge-ribbon">{{ rankName }}</span>
            </div>
            <p class="rank-name">{{ rankName }}</p>
          </div>

          <div class="home-card earned-card">
            <div class="earned-info">
              <div class="card-head">
                <i class="fas fa-coins head-icon gold"></i>
                <h3>Total ganado</h3>
              </div>
              <p class="earned-label">Ganancias acumuladas</p>
              <p class="earned-amount">S/ {{ money(totalEarned) }}</p>
            </div>
            <img class="earned-art" src="../../assets/img/total-ganado.svg" alt="" aria-hidden="true" />
          </div>

          <div class="home-card activation-card">
            <div class="card-head">
              <i class="fas fa-check-circle head-icon green"></i>
              <h3>Mi activación</h3>
            </div>
            <div class="activation-box" :class="{ inactive: !isActive }">
              <div class="activation-status">
                <i class="fas" :class="isActive ? 'fa-check-circle' : 'fa-times-circle'"></i>
                <span>{{ isActive ? 'ACTIVO' : 'INACTIVO' }}</span>
              </div>
              <p class="activation-type">Activación anual</p>
              <p class="activation-expiry" v-if="isActive && expiryText">Vence en {{ expiryText }}</p>
              <router-link to="/activation" class="activation-link" v-else-if="!isActive">Activar ahora</router-link>
            </div>
          </div>
        </div>

        <!-- Mi actividad comercial -->
        <div class="section-head">
          <i class="fas fa-chart-bar head-icon green"></i>
          <h3>Mi actividad comercial</h3>
        </div>
        <div class="home-row row-stats">
          <div class="home-card stat-card">
            <i class="fas fa-shopping-cart stat-icon"></i>
            <div>
              <p class="stat-label">Ventas personales</p>
              <p class="stat-value">{{ stats.personalSales }}</p>
              <p class="stat-sub">Membresías vendidas</p>
            </div>
          </div>
          <div class="home-card stat-card">
            <i class="fas fa-users stat-icon"></i>
            <div>
              <p class="stat-label">Ventas grupales</p>
              <p class="stat-value">{{ stats.groupSales }}</p>
            </div>
          </div>
          <div class="home-card stat-card">
            <i class="fas fa-user stat-icon"></i>
            <div>
              <p class="stat-label">Socios directos</p>
              <p class="stat-value">{{ stats.directs }}</p>
            </div>
          </div>
          <div class="home-card stat-card">
            <i class="fas fa-project-diagram stat-icon"></i>
            <div>
              <p class="stat-label">Mi red comercial</p>
              <p class="stat-value">{{ stats.network }}</p>
              <router-link to="/tree" class="card-link">Ver mi red <i class="fas fa-arrow-right"></i></router-link>
            </div>
          </div>
        </div>

        <!-- Fila 3: próxima meta y saldos -->
        <div class="home-row row-bottom">
          <div class="home-card goal-card">
            <div class="card-head">
              <i class="fas fa-bullseye head-icon green"></i>
              <h3>Próxima meta</h3>
            </div>
            <div class="goal-body">
              <div class="badge small">
                <img src="../../assets/img/insignias colores.png" alt="Próximo rango" />
                <span class="badge-ribbon">{{ nextGoal.name }}</span>
              </div>
              <div class="goal-info">
                <p class="goal-name">{{ nextGoal.name }}</p>
                <p class="goal-count">{{ nextGoal.current }} de {{ nextGoal.target }} ventas personales</p>
                <div class="goal-bar">
                  <div class="goal-fill" :style="{ width: goalPercent + '%' }">
                    <span v-if="goalPercent >= 12">{{ goalPercent }}%</span>
                  </div>
                </div>
                <div class="goal-foot">
                  <span v-if="goalMissing > 0">Te faltan {{ goalMissing }} ventas personales</span>
                  <span v-else>¡Meta alcanzada!</span>
                  <router-link to="/resume" class="card-link">Ver requisitos <i class="fas fa-arrow-right"></i></router-link>
                </div>
              </div>
            </div>
          </div>

          <div class="home-card balance-card">
            <div class="card-head">
              <i class="fas fa-wallet head-icon green"></i>
              <h3>Mis saldos</h3>
            </div>
            <div class="balance-item available">
              <i class="fas fa-money-bill-wave balance-icon"></i>
              <div class="balance-info">
                <p class="balance-label">Saldo disponible</p>
                <p class="balance-amount">S/ {{ money(balance) }}</p>
              </div>
              <router-link to="/collect" class="withdraw-btn">
                <i class="fas fa-money-bill-wave"></i> Retirar
              </router-link>
            </div>
            <div class="balance-item locked">
              <i class="fas fa-lock balance-icon"></i>
              <div class="balance-info">
                <p class="balance-label">Saldo no disponible</p>
                <p class="balance-amount">S/ {{ money(_balance) }}</p>
              </div>
            </div>
          </div>
        </div>
      </template>
    </div>
  </App>
</template>

<script>
import App from "@/views/layouts/App";
import api from "@/api";

const MONTHS = ["enero", "febrero", "marzo", "abril", "mayo", "junio", "julio", "agosto", "septiembre", "octubre", "noviembre", "diciembre"];

export default {
  components: {
    App,
  },
  data() {
    return {
      banner: {},
      ins: 0,
      insVirtual: 0,
      earned: null,
      balance: 0,
      _balance: 0,

      rankName: "Sin rango",
      rankPin: null,
      defaultPin: require("../../assets/img/insignias colores.png"),
      activationExpires: null,
      stats: {
        personalSales: 0,
        groupSales: 0,
        directs: 0,
        network: 0,
      },
      nextGoal: {
        name: "Emprendedor",
        current: 0,
        target: 1,
      },

      loading: true,
      slide: 0,
      bannerRatio: null,
      timer: null,
    };
  },
  computed: {
    session() {
      return this.$store.state.session;
    },
    _activated() {
      return this.$store.state._activated;
    },
    activated() {
      return this.$store.state.activated;
    },
    title() {
      return "Inicio";
    },
    isActive() {
      return !!(this._activated || this.activated);
    },
    bannerStyle() {
      // en móvil se usa la proporción real de la imagen (acotada) para darle más alto sin deformarla
      if (!this.bannerRatio) return {};
      const ratio = Math.min(3.6, Math.max(3, this.bannerRatio));
      return { "--banner-ratio-mobile": ratio };
    },
    bannerImages() {
      const b = this.banner || {};
      return [b.img, b.img2, b.img3].filter(Boolean);
    },
    totalEarned() {
      if (this.earned != null) return this.earned;
      return Number(this.ins || 0) + Number(this.insVirtual || 0);
    },
    expiryText() {
      if (!this.activationExpires) return null;
      const d = new Date(this.activationExpires);
      if (isNaN(d)) return null;
      return `${MONTHS[d.getMonth()]} de ${d.getFullYear()}`;
    },
    goalPercent() {
      const { current, target } = this.nextGoal;
      if (!target) return 0;
      return Math.min(100, Math.round((current / target) * 100));
    },
    goalMissing() {
      return Math.max(0, this.nextGoal.target - this.nextGoal.current);
    },
  },
  methods: {
    onBannerLoad(e) {
      const { naturalWidth: w, naturalHeight: h } = e.target;
      if (w && h) this.bannerRatio = w / h;
    },
    money(val) {
      return Number(val || 0).toLocaleString("en-US", { minimumFractionDigits: 2, maximumFractionDigits: 2 });
    },
  },
  async created() {
    // GET data
    const { data } = await api.dashboard(this.session);

    this.loading = false;

    // error
    if (data.error && data.msg == "invalid session")
      return this.$router.push("/login");

    // success
    this.$store.commit("SET_NAME", data.name);
    this.$store.commit("SET_LAST_NAME", data.lastName);
    this.$store.commit("SET_AFFILIATED", data.affiliated);
    this.$store.commit("SET__ACTIVATED", data._activated);
    this.$store.commit("SET_ACTIVATED", data.activated);
    this.$store.commit("SET_PLAN", data.plan);
    this.$store.commit("SET_COUNTRY", data.country);
    this.$store.commit("SET_PHOTO", data.photo);
    this.$store.commit("SET_TREE", data.tree);
    this.$store.commit("SET_EMAIL", data.email);
    this.$store.commit("SET_TOKEN", data.token);

    this.banner = data.banner || {};
    this.ins = data.ins;
    this.insVirtual = data.insVirtual;
    this.earned = data.totalEarned != null ? Number(data.totalEarned) : null;
    this.balance = data.balance;
    this._balance = data._balance;

    // campos de la nueva pantalla de inicio (se completan con la lógica del servidor)
    this.rankName = data.rankName || "Sin rango";
    this.rankPin = data.rankPin || null;
    if (data.activationExpires) this.activationExpires = data.activationExpires;
    if (data.stats) this.stats = { ...this.stats, ...data.stats };
    if (data.nextGoal) this.nextGoal = { ...this.nextGoal, ...data.nextGoal };

    this.timer = setInterval(() => {
      const n = this.bannerImages.length;
      if (n > 1) this.slide = (this.slide + 1) % n;
    }, 4000);
  },
  beforeDestroy() {
    clearInterval(this.timer);
  },
};
</script>

<style scoped>
.home {
  --navy: #10284a;
  --green: #1f8a3b;
  --green-soft: #e9f6ec;
  --muted: #5d6b7c;
  --card-shadow: 0 4px 16px rgba(16, 40, 74, 0.07);
  color: var(--navy);
  display: flex;
  flex-direction: column;
  gap: 18px;
  max-width: 1200px;
}

p {
  margin: 0;
}

/* Banner */
.home-banner {
  position: relative;
  margin: 0;
  width: 100%;
  aspect-ratio: 1200 / 290;
  border-radius: 14px;
  overflow: hidden;
  box-shadow: var(--card-shadow);
  background: #0b1f3a;
}

.home-banner img {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0;
  transition: opacity 1.5s;
}

.home-banner img.visible {
  opacity: 1;
}

/* Layout de filas */
.home-row {
  display: grid;
  gap: 18px;
}

.row-top {
  grid-template-columns: 1fr 1.6fr 1fr;
}

.row-top .home-card {
  padding: 14px 20px;
}

.row-stats {
  grid-template-columns: repeat(4, 1fr);
}

.row-bottom {
  grid-template-columns: 2fr 1.2fr;
}

/* Tarjeta base */
.home-card {
  background: #fff;
  border-radius: 14px;
  box-shadow: var(--card-shadow);
  padding: 20px 22px;
  min-width: 0;
}

.card-head,
.section-head {
  display: flex;
  align-items: center;
  gap: 12px;
}

.card-head h3,
.section-head h3 {
  margin: 0;
  font-size: 1.3rem;
  font-weight: 700;
  color: var(--navy);
}

.section-head {
  margin-bottom: -6px;
}

.head-icon {
  font-size: 1.5rem;
}

.head-icon.green {
  color: var(--green);
}

.head-icon.gold {
  color: #f2b705;
}

.card-link {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  color: #1c6fd1;
  font-weight: 600;
  font-size: 0.95rem;
  text-decoration: none;
}

/* Insignia */
.badge {
  position: relative;
  width: 150px;
  margin: 6px auto 2px;
  text-align: center;
}

.badge img {
  width: 92px;
  height: 92px;
  object-fit: contain;
  filter: drop-shadow(0 4px 6px rgba(0, 0, 0, 0.18));
}

.badge-ribbon {
  position: absolute;
  left: 50%;
  bottom: 4px;
  transform: translateX(-50%);
  white-space: nowrap;
  background: linear-gradient(180deg, #f7d460 0%, #d9a520 100%);
  color: #10284a;
  font-weight: 800;
  font-size: 0.78rem;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  padding: 4px 16px;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.badge.small {
  width: 140px;
  margin: 0;
  flex-shrink: 0;
}

.badge.small img {
  width: 120px;
  height: 120px;
}

.rank-card {
  text-align: center;
}

.rank-card .card-head {
  justify-content: flex-start;
}

.rank-name {
  font-size: 1.15rem;
  font-weight: 700;
}

/* Total ganado */
.earned-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.earned-label {
  margin-top: 14px;
  color: var(--muted);
  font-size: 1.15rem;
}

.earned-amount {
  margin-top: 6px;
  color: var(--green);
  font-size: clamp(1.8rem, 3vw, 3.2rem);
  font-weight: 800;
  line-height: 1.1;
  white-space: nowrap;
}

.earned-info {
  min-width: 0;
}

.earned-art {
  width: 120px;
  height: auto;
  flex-shrink: 0;
}

/* Activación */
.activation-box {
  margin-top: 10px;
  background: var(--green-soft);
  border-radius: 12px;
  padding: 10px 12px;
  text-align: center;
}

.activation-status {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 14px;
  color: var(--green);
}

.activation-status i {
  font-size: 2.2rem;
}

.activation-status span {
  font-size: 1.6rem;
  font-weight: 800;
}

.activation-type {
  margin-top: 4px;
  font-weight: 700;
  font-size: 1.05rem;
}

.activation-expiry {
  margin-top: 2px;
  color: var(--muted);
}

.activation-box.inactive {
  background: #f3f4f6;
}

.activation-box.inactive .activation-status {
  color: #8a94a3;
}

.activation-link {
  display: inline-block;
  margin-top: 2px;
  color: #1c6fd1;
  font-weight: 600;
}

/* Actividad comercial */
.stat-card {
  display: flex;
  gap: 18px;
  align-items: flex-start;
}

.stat-icon {
  font-size: 1.7rem;
  color: var(--green);
  margin-top: 4px;
  width: 30px;
  text-align: center;
}

.stat-label {
  font-weight: 600;
  font-size: 0.95rem;
}

.stat-value {
  font-size: 2.4rem;
  font-weight: 800;
  line-height: 1.2;
}

.stat-sub {
  color: var(--muted);
  font-size: 0.9rem;
}

/* Próxima meta */
.goal-body {
  display: flex;
  align-items: center;
  gap: 24px;
  margin-top: 12px;
}

.goal-info {
  flex: 1;
  min-width: 0;
}

.goal-name {
  font-size: 1.4rem;
  font-weight: 700;
}

.goal-count {
  color: var(--muted);
  margin: 6px 0 10px;
}

.goal-bar {
  height: 30px;
  background: #e8ebef;
  border-radius: 8px;
  overflow: hidden;
}

.goal-fill {
  height: 100%;
  background: linear-gradient(90deg, #1f8a3b 0%, #2ea44f 100%);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
  font-weight: 700;
  font-size: 0.9rem;
  transition: width 1s ease;
}

.goal-foot {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 10px;
  color: var(--muted);
}

/* Saldos */
.balance-card {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.balance-item {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 14px 18px;
  border-radius: 12px;
}

.balance-item.available {
  background: var(--green-soft);
}

.balance-item.locked {
  background: #f0f3f8;
}

.balance-icon {
  font-size: 1.5rem;
  color: #7d8aa0;
  width: 26px;
  text-align: center;
}

.balance-info {
  flex: 1;
}

.balance-label {
  font-weight: 600;
  font-size: 0.95rem;
}

.balance-amount {
  font-size: 1.4rem;
  font-weight: 800;
  color: var(--navy);
}

.withdraw-btn {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  background: var(--green);
  color: #fff;
  font-weight: 700;
  padding: 10px 24px;
  border-radius: 8px;
  text-decoration: none;
  white-space: nowrap;
}

.withdraw-btn:hover {
  background: #197432;
}

/* Responsive */
@media (max-width: 1100px) {
  .row-top {
    grid-template-columns: 1fr 1fr;
  }

  .earned-card {
    grid-column: 1 / -1;
    order: -1;
  }

  .row-stats {
    grid-template-columns: repeat(2, 1fr);
  }

  .row-bottom {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 600px) {
  .home-banner {
    aspect-ratio: var(--banner-ratio-mobile, 3.4);
  }

  .row-top,
  .row-stats {
    grid-template-columns: 1fr;
  }

  .earned-art {
    width: 72px;
  }

  .earned-label {
    margin-top: 16px;
  }

  .goal-body {
    flex-direction: column;
    text-align: center;
  }

  .goal-foot {
    justify-content: center;
  }

  .balance-item {
    flex-wrap: wrap;
  }

  .withdraw-btn {
    width: 100%;
    justify-content: center;
  }
}
</style>
