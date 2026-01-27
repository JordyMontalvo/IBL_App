<template>
  <App :session="session" :title="title">
    <!-- <figure class="slide" style="display: flex; margin: 0">
      <img
        src="../../assets/img/banner/banner1ibl.webp"
        style="width: 100%; max-width: 850px; transition: all 2s"
        :style="{ opacity: op }"
      />
      <img
        src="../../assets/img/banner/banner2ibl.webp"
        style="
          width: 100%;
          max-width: 850px;
          transition: all 2s;
          transform: translate(-100%);
          opacity: 0;
        "
        :style="{ opacity: op2 }"
      />
      <img
        src="../../assets/img/banner/banner3ibl.webp"
        style="
          width: 100%;
          max-width: 850px;
          transition: all 2s;
          transform: translate(-200%);
          opacity: 0;
        "
        :style="{ opacity: op3 }"
      />
    </figure>
    <br /> -->


    <figure class="slide" style="display: flex; margin: 0;">
      <img :src="banner.img" style="width: 100%; max-width: 850px; transition: all 2s;" :style="{ opacity: op }" >
      <img :src="banner.img2" style="width: 100%; max-width: 850px; transition: all 2s; transform: translate(-100%); opacity:0;" :style="{ opacity: op2 }">
      <img :src="banner.img3" style="width: 100%; max-width: 850px; transition: all 2s; transform: translate(-200%); opacity:0;" :style="{ opacity: op3 }">
    </figure> <br>

    
    <i class="load" v-if="loading"></i>

    <div class="boxes" v-if="!loading">
      <div class="box white nivel-actual-card">
        <h2>Nivel Actual</h2>
        <div class="rank-image">
          <img
            src="../../assets/img/insignias colores.png"
            alt="Rango"
          />
          <h3>Nivel Principiante</h3>
        </div>

        <div class="stats-container">
          <div class="stat-item">
            <div class="stat-icon">
              <i class="fas fa-user" style="font-size: 20px;"></i>
            </div>
            <div class="stat-info">
              <span class="stat-label">Volumen Personal</span>
              <span class="stat-value">{{ personalVolume }}</span>
            </div>
          </div>

          <div class="stat-item">
            <div class="stat-icon">
              <i class="fas fa-users" style="font-size: 20px;"></i>
            </div>
            <div class="stat-info">
              <span class="stat-label">Volumen Grupal</span>
              <span class="stat-value">{{ groupVolume }}</span>
            </div>
          </div>

          <div class="stat-item">
            <div class="stat-icon">
              <i class="fas fa-building " style="font-size: 20px;"></i>
            </div>
            <div class="stat-info">
              <span class="stat-label">Socios</span>
              <span class="stat-value">{{ team }}</span>
            </div>
          </div>

          <div class="stat-item">
            <div class="stat-icon">
              <i class="fas fa-network-wired" style="font-size: 20px;"></i>
            </div>
            <div class="stat-info">
              <span class="stat-label">Frontales</span>
              <span class="stat-value">{{ frontales }}</span>
            </div>
          </div>
        </div>
      </div>

      <div
        class="box white nivel-alcanzado-card"
        style="text-align: center; padding: 1rem; border-radius: 8px"
      >
        <h3 style="color: #1a3b5d; font-size: 1.5rem; margin-bottom: 0.5rem">
          Rango Alcanzado
        </h3>
        <img
        src="../../assets/img/insignias colores.png"
        alt="Rango Diamante" style="width: 100px; height: 100px; margin-bottom:
        0.5rem" />
        <p style="font-weight: bold">{{ achievedRank }}</p>

        <div class="stats-container" style="margin: 0.5rem 0">
          <div
            class="stat-item"
            style="
              background-color: #f8faff;
              padding: 0.5rem;
              border-radius: 8px;
            "
          >
            <i class="fas fa-users"></i>
            <span
              >Puntos Grupales: <strong>{{ groupPoints }}</strong></span
            >
          </div>
          <div
            class="stat-item"
            style="
              background-color: #f8faff;
              padding: 0.5rem;
              border-radius: 8px;
            "
          >
            <i class="fas fa-users"></i>
            <span
              >Frontales: <strong>{{ frontales }}</strong></span
            >
          </div>
          <div
            class="stat-item"
            style="
              background-color: #f8faff;
              padding: 0.5rem;
              border-radius: 8px;
            "
          >
            <i class="fas fa-users"></i>
            <span
              >Volumen Grupal: <strong>{{ groupVolume }}</strong></span
            >
          </div>
        </div>
      </div>
      <div class="box white earnings-card">
        <div class="earnings-header">
          <i class="fas fa-coins header-icon"></i>
          <h2>GANANCIAS</h2>
        </div>
        
        <div class="earnings-content">
          <!-- Total Ganado Section -->
          <div class="earnings-main-section">
            <div class="section-title">
              <i class="fas fa-trophy"></i>
              <span>Total Ganado</span>
            </div>
            <div class="amount-row">
              <span class="currency">S/.</span>
              <span class="amount">{{ (Number(ins) + Number(insVirtual)).toFixed(2) }}</span>
            </div>
            <button class="retirar-btn">
              <i class="fas fa-money-bill-wave"></i> Retirar
            </button>
          </div>

          <!-- Saldo Disponible Section -->
          <div class="earnings-sub-section">
            <div class="section-title">
              <i class="fas fa-wallet"></i>
              <span>Saldo Disponible</span>
            </div>
            <div class="amount-row small">
              <span class="currency">S/.</span>
              <span class="amount">{{ balance }}</span>
            </div>
          </div>

          <!-- Saldo No Disponible Section -->
          <div class="earnings-sub-section no-border">
            <div class="section-title">
              <i class="fas fa-hourglass-half"></i>
              <span>Saldo No Disponible</span>
            </div>
            
            <div class="sub-items">
              <div class="sub-item">
                <div class="sub-item-info">
                  <i class="fas fa-home sub-icon" style="color: #4caf50;"></i>
                  <span>No disponible - Lotes</span>
                </div>
                <span class="sub-amount">S/. {{ _balance_lote }}</span>
              </div>
              
              <div class="sub-item">
                <div class="sub-item-info">
                  <i class="fas fa-id-card sub-icon" style="color: #2196f3;"></i>
                  <span>No disponible - Membresías</span>
                </div>
                <span class="sub-amount">S/. {{ _balance_membresia }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
          <div class="box white activation-pack-card">
            <h3 style="color: #1a3b5d; margin-bottom: 1rem;">Pack de Activación</h3>
            
            <div class="activation-status">
              <!-- Membresía Status -->
              <div class="activation-item" :class="{ 'active': _activated }">
                <div class="activation-icon">
                  <i class="fas fa-id-card"></i>
                </div>
                <div class="activation-info">
                  <span class="activation-label">Membresía</span>
                  <span class="activation-badge" v-if="_activated">
                    <i class="fas fa-check-circle"></i> Activo
                  </span>
                  <span class="activation-badge inactive" v-else>
                    <i class="fas fa-times-circle"></i> No activo
                  </span>
                </div>
              </div>

              <!-- Lotes Status -->
              <div class="activation-item" :class="{ 'active': activated }">
                <div class="activation-icon">
                  <i class="fas fa-map-marked-alt"></i>
                </div>
                <div class="activation-info">
                  <span class="activation-label">Lotes</span>
                  <span class="activation-badge" v-if="activated">
                    <i class="fas fa-check-circle"></i> Activo
                  </span>
                  <span class="activation-badge inactive" v-else>
                    <i class="fas fa-times-circle"></i> No activo
                  </span>
                </div>
              </div>
            </div>

            <!-- Overall Package Status -->
            <div class="package-summary" v-if="activated">
              <i class="fas fa-crown"></i>
              <span>Pack Full Completo</span>
            </div>
            <div class="package-summary partial" v-else-if="_activated">
              <i class="fas fa-star-half-alt"></i>
              <span>Pack Simple</span>
            </div>
            <div class="package-summary none" v-else>
              <i class="fas fa-exclamation-circle"></i>
              <span>Sin activación</span>
            </div>
          </div>



      <div class="box white" v-if="node">
        <i class="fas fa-gem"></i>
        <div>
          <p>{{ node.rank | _rank }}</p>
          <span>RANGO ACTUAL</span>
        </div>
      </div>

      <div class="box white" v-if="node">
        <i class="fa fa-tachometer"></i>
        <div>
          <p>{{ node.next_rank.name | _rank }}</p>
          <span>SIGUIENTE RANGO</span>
        </div>
      </div>
    </div>


  </App>
</template>

<script>
import App from "@/views/layouts/App";
import api from "@/api";

export default {
  components: {
    App,
  },
  data() {
    return {
      banner: { img: null },
      ins: null,
      outs: null,
      balance: null,
      _balance: null,
      _balance_lote: null,
      _balance_membresia: null,
      team: null,
      rank: null,
      points: null,

      loading: true,

      op: 1,
      op2: 0,
      op3: 0,
    };
  },
  computed: {
    session() {
      return this.$store.state.session;
    },
    plan() {
      return this.$store.state.plan;
    },
    _activated() {
      return this.$store.state._activated;
    },
    activated() {
      return this.$store.state.activated;
    },
    title() {
      return "Dashboard";
    },
  },
  filters: {
    _rank(val) {
      if (val == "none") return "Ninguno";
      if (val == "active") return "ACTIVO";
      if (val == "star") return "MASTER";
      if (val == "master") return "PLATA";
      if (val == "silver") return "PLATINO";
      if (val == "gold") return "ORO";
      if (val == "sapphire") return "ZAFIRO";
      if (val == "RUBI") return "DIAMANTE RUBI";
      if (val == "DIAMANTE") return "DIAMANTE ESTRELLA";
      if (val == "DOBLE DIAMANTE") return "DIAMANTE DOS ESTRELLAS";
      if (val == "TRIPLE DIAMANTE") return "DIAMANTE TRES ESTRELLAS";
      if (val == "DIAMANTE ESTRELLA") return "DIAMANTE CBM";
    },
  },
  async created() {
    // GET data
    const { data } = await api.dashboard(this.session);
    console.log({ data });

    this.loading = false;

    // error
    if (data.error && data.msg == "invalid session")
      this.$router.push("/login");

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

    this.banner = data.banner;
    this.ins = data.ins;
    this.insVirtual = data.insVirtual;
    this.outs = data.outs.toFixed(2);
    this.balance = data.balance.toFixed(2);
    this._balance = data._balance.toFixed(2);
    this._balance_lote = data._balance_lote ? data._balance_lote.toFixed(2) : "0.00";
    this._balance_membresia = data._balance_membresia ? data._balance_membresia.toFixed(2) : "0.00";
    this.team = data.team;
    this.rank = data.rank;
    this.points = data.points;
    this.node = data.node;
    this.n_affiliates = data.n_affiliates;

    const time = 4000;
    let i = 0;
    const n = 3;

    setInterval(() => {
      i += 1;
      console.log(i);

      if (i == 0) {
        // reset all
        this.op = 0;
        this.op2 = 0;
        this.op3 = 0;
        // show 0
        this.op = 1;
      }

      if (i == 1) {
        // reset all
        this.op = 0;
        this.op2 = 0;
        this.op3 = 0;
        // show 0
        this.op2 = 1;
      }

      if (i == 2) {
        // reset all
        this.op = 0;
        this.op2 = 0;
        this.op3 = 0;
        // show 0
        this.op3 = 1;
      }

      if (i == n - 1) i = -1;
    }, time);

    setTimeout(() => {
      var elem = document.querySelector('.boxes');
      new Masonry( elem, {
        itemSelector: '.box',
      });
    }, 200);
  },
};
</script>

<style scoped>

.nivel-actual-card {
  padding: 2rem;
  text-align: center;
}

.rank-image {
  margin: 2rem 0;
}

.rank-image img {
  width: 150px;
  height: 150px;
  margin-bottom: 1rem;
}

.rank-image h3 {
  color: #1a3b5d;
  font-size: 1.2rem;
  margin: 0.5rem 0;
}

.stats-container {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.stat-item {
  display: flex;
  align-items: center;
  background-color: #f8faff;
  padding: 1rem;
  border-radius: 8px;
}

.stat-icon {
  background-color: #4caf50;
  color: white;
  width: 40px;
  height: 40px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 1rem; 
}

.stat-info {
  flex: 1;
  text-align: left;
}

.stat-label {
  display: block;
  color: #666;
  font-size: 0.9rem;
}

.stat-value {
  display: block;
  color: #1a3b5d;
  font-weight: bold;
  font-size: 1.1rem;
}

/* Activation Pack Card Styles */
.activation-pack-card {
  padding: 1.5rem !important;
}

.activation-status {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-bottom: 1rem;
}

.activation-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
  background: #f8faff;
  border-radius: 8px;
  border-left: 4px solid #e0e0e0;
  transition: all 0.3s ease;
}

.activation-item.active {
  background: linear-gradient(135deg, #e8f5e9 0%, #f1f8f4 100%);
  border-left-color: #4caf50;
}

.activation-icon {
  width: 45px;
  height: 45px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.3rem;
  background: #e0e0e0;
  color: #666;
  transition: all 0.3s ease;
}

.activation-item.active .activation-icon {
  background: linear-gradient(135deg, #4caf50 0%, #66bb6a 100%);
  color: white;
  box-shadow: 0 4px 8px rgba(76, 175, 80, 0.3);
}

.activation-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
}

.activation-label {
  font-weight: 600;
  color: #1a3b5d;
  font-size: 1rem;
}

.activation-badge {
  display: inline-flex;
  align-items: center;
  gap: 0.3rem;
  font-size: 0.85rem;
  color: #4caf50;
  font-weight: 500;
}

.activation-badge i {
  font-size: 0.9rem;
}

.activation-badge.inactive {
  color: #999;
}

.package-summary {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  padding: 0.8rem 1rem;
  border-radius: 8px;
  font-weight: 600;
  font-size: 0.95rem;
  background: linear-gradient(135deg, #ffd700 0%, #ffed4e 100%);
  color: #1a3b5d;
  box-shadow: 0 3px 8px rgba(255, 215, 0, 0.3);
}

.package-summary i {
  font-size: 1.1rem;
}

.package-summary.partial {
  background: linear-gradient(135deg, #64b5f6 0%, #81c784 100%);
  color: white;
  box-shadow: 0 3px 8px rgba(100, 181, 246, 0.3);
}

.package-summary.none {
  background: linear-gradient(135deg, #e0e0e0 0%, #f5f5f5 100%);
  color: #757575;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

/* Earnings Card Styling */
.earnings-card {
  padding: 1.5rem !important;
  display: flex !important;
  flex-direction: column !important;
  gap: 1.5rem;
}

.earnings-header {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.earnings-header .header-icon {
  font-size: 1.8rem;
  color: #ffd700;
}

.earnings-header h2 {
  font-size: 1.4rem !important;
  font-weight: 700 !important;
  color: #1a3b5d !important;
  margin: 0 !important;
}

.earnings-content {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.earnings-main-section, .earnings-sub-section {
  display: flex;
  flex-direction: column;
  gap: 0;
  padding: 0.75rem 1rem;
  border: 1px solid #eef2f7;
  border-radius: 12px;
  background: white;
}

.section-title {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  color: #1a3b5d;
  font-weight: 600;
  font-size: 1rem;
}

.section-title i {
  color: #4a5568;
  font-size: 1.2rem;
}

.amount-row {
  display: flex;
  align-items: baseline;
  gap: 0.4rem;
  justify-content: flex-start;
  padding-left: 2rem;
  margin-top: -0.2rem;
}

.amount-row .currency {
  font-size: 1rem;
  font-weight: 600;
  color: #1a3b5d;
}

.amount-row .amount {
  font-size: 1.3rem;
  font-weight: 700;
  color: #1a3b5d;
}

.amount-row.small .amount {
  font-size: 1.3rem;
}

.retirar-btn {
  background: #4caf50;
  color: white;
  border: none;
  padding: 0 1rem;
  border-radius: 6px;
  font-weight: 700;
  font-size: 1.1rem;
  height: 48px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  width: 100%;
  margin-top: 0.6rem;
  transition: background 0.2s;
}

.retirar-btn:hover {
  background: #43a047;
}

.sub-items {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.sub-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 0.5rem;
  border-top: 1px solid #f0f4f8;
}

.sub-item:first-child {
  border-top: none;
  padding-top: 0;
}

.sub-item-info {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  color: #4a5568;
  font-size: 0.9rem;
}

.sub-icon {
  font-size: 0.7rem;
  width: 16px;
  text-align: center;
  opacity: 0.8;
}

.sub-amount {
  font-weight: 600;
  color: #1a3b5d;
}
</style>
