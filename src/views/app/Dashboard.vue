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
            src="https://s3-alpha-sig.figma.com/img/7802/2e20/5a4b8e6b84fee7f04d9ef27eda097ee1?Expires=1740355200&Key-Pair-Id=APKAQ4GOSFWCW27IBOMQ&Signature=fzkCIOclXtfeMBRARqHw3U4324RsIc49HJWuT5gZaENS5FGF22rzoZm803hywEtzf9kOUOsNO1qFLU0uuC7HGpjb~6AmI5538XhUFHNw~FUzbJ-OTQOJprbPeVvG4pRlT58ArVxElY1gM5rF5RbXvKP5UCtojppAk22GHbBpDuBS6qTorWHV5eE-pOEuvDF-e3EecDYUgpg8hCyiaCPHhLI4z4dnPzVMSJ8p7QKiG3wfftBsbRYOfH-tuAZygeUrILyZgISRYoAJ1ns4hxzkk7jQ619lCafSisSFIHNbf~pmMEeT9pK6ehD3cqjwGHXUTcmUoQc0rTGNna~9Jm3FBA__"
            alt="Rango"
          />
          <h3>Nivel Principiante</h3>
        </div>

        <div class="stats-container">
          <div class="stat-item">
            <div class="stat-icon">
              <i class="fas fa-user-chart"></i>
            </div>
            <div class="stat-info">
              <span class="stat-label">Volumen Personal</span>
              <span class="stat-value">{{ personalVolume }}</span>
            </div>
          </div>

          <div class="stat-item">
            <div class="stat-icon">
              <i class="fas fa-users"></i>
            </div>
            <div class="stat-info">
              <span class="stat-label">Volumen Grupal</span>
              <span class="stat-value">{{ groupVolume }}</span>
            </div>
          </div>

          <div class="stat-item">
            <div class="stat-icon">
              <i class="fas fa-building"></i>
            </div>
            <div class="stat-info">
              <span class="stat-label">Socios</span>
              <span class="stat-value">{{ team }}</span>
            </div>
          </div>

          <div class="stat-item">
            <div class="stat-icon">
              <i class="fas fa-network-wired"></i>
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
        src="https://s3-alpha-sig.figma.com/img/7802/2e20/5a4b8e6b84fee7f04d9ef27eda097ee1?Expires=1740355200&Key-Pair-Id=APKAQ4GOSFWCW27IBOMQ&Signature=fzkCIOclXtfeMBRARqHw3U4324RsIc49HJWuT5gZaENS5FGF22rzoZm803hywEtzf9kOUOsNO1qFLU0uuC7HGpjb~6AmI5538XhUFHNw~FUzbJ-OTQOJprbPeVvG4pRlT58ArVxElY1gM5rF5RbXvKP5UCtojppAk22GHbBpDuBS6qTorWHV5eE-pOEuvDF-e3EecDYUgpg8hCyiaCPHhLI4z4dnPzVMSJ8p7QKiG3wfftBsbRYOfH-tuAZygeUrILyZgISRYoAJ1ns4hxzkk7jQ619lCafSisSFIHNbf~pmMEeT9pK6ehD3cqjwGHXUTcmUoQc0rTGNna~9Jm3FBA__""
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
      <div
        class="box white"
        style="
          text-align: center;
          padding: 1rem;
          border-radius: 8px;
          
        "
      >
        <h3 style="color: #1a3b5d; font-size: 1.5rem; margin-bottom: 0.5rem">
          Total Ganado
        </h3>
        <img
          src="https://s3-alpha-sig.figma.com/img/d6d4/429a/68e4f36017739ce18e2e4f05b1d41531?Expires=1740355200&Key-Pair-Id=APKAQ4GOSFWCW27IBOMQ&Signature=e7rbX~z0sHe5xyWkjj60c-T9eOv-dNefzUkP4m4bIS5jnPed7W5EtQ6Pg3fWVFkY8yRl-akktBybtrCQ2rxxCdCZ0lqLrZ~RabMD0jtk7UXHS9JuaF1V2eBbfp08w8w4VpaNhXom6cpqJgtoRNucVvi~MtL7AeoIlFa-3RoVWcwa~1wzwzndF5H~t-sEj4JJSSHm9UKGQYA6aLqTAKODjQ6WvJTkRdTWe1FODhk5l6fXZkI8ezzgGDMq8R5jacoiDNAUjW0DwVZEvv5f9wvhr84jrQFsIzORj~3eIyhle-IijRhgHJlkubxpUskK9lBBxayi8FmMOpWfB3tW5LxWIw__"
          alt="Trofeo"
          style="width: 100px; height: 100px; margin-bottom: 0.5rem"
          />
          <div class="stat-item">
            <span style="font-weight: bold">
              S/. {{ Number(ins + insVirtual).toFixed(2) }}
        </span>
      </div>
      
    </div>
          <div class="box white">
            <h3>Pack de Activación</h3>
            <span v-if="_activated && !activated">Miembros</span>
            <span v-if="activated">Lotes</span>
          </div>
      <div class="box white">
        <i class="fas fa-wallet"></i>
        <div>
          <p>S/. {{ balance }}</p>
          <span>SALDO</span>
        </div>
      </div>

      <div class="box white">
        <i class="fas fa-hand-holding-usd"></i>
        <div>
          <p>S/. {{ _balance }}</p>
          <span>SALDO NO DISPONIBLE</span>
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

    this.banner = data.banner;
    this.ins = data.ins;
    this.insVirtual = data.insVirtual;
    this.outs = data.outs.toFixed(2);
    this.balance = data.balance.toFixed(2);
    this._balance = data._balance.toFixed(2);
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
</style>
