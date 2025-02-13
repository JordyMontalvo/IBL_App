<template>
  <App :session="session" :title="title">

    <figure class="slide" style="display: flex; margin: 0;">
      <img src="../../assets/img/banner/banner1ibl.webp" style="width: 100%; max-width: 850px; transition: all 2s;" :style="{ opacity: op }" >
      <img src="../../assets/img/banner/banner2ibl.webp" style="width: 100%; max-width: 850px; transition: all 2s; transform: translate(-100%); opacity:0;" :style="{ opacity: op2 }">
      <img src="../../assets/img/banner/banner3ibl.webp" style="width: 100%; max-width: 850px; transition: all 2s; transform: translate(-200%); opacity:0;" :style="{ opacity: op3 }">
    </figure> <br>

    

    <i class="load" v-if="loading"></i>

    <div class="grid">
      <div class="grid-item">
        <div class="card">
          <h3> Mis Ganancias Mis Ganancias Mis Ganancias Mis Ganancias Mis Ganancias Mis Ganancias Mis Ganancias Mis Ganancias </h3>
          <div class="card-content">
            <i class="fas fa-dollar-sign"></i>
            <p>{{ balance }}</p>
          </div>
        </div>
      </div>

      <div class="grid-item">
        <div class="card">
          <h3>Comisiones Comisiones Comisiones Comisiones </h3>
          <div class="card-content">
            <i class="fas fa-hand-holding-usd"></i>
            <p>{{ _balance }}</p>
          </div>
        </div>
      </div>

      <div class="grid-item">
        <div class="card">
          <h3>Puntos</h3>
          <div class="card-content">
            <i class="fas fa-star"></i>
            <p>{{ points }}</p>
          </div>
        </div>
      </div>

      <div class="grid-item">
        <div class="card">
          <h3>Rango</h3>
          <div class="card-content">
            <i class="fas fa-trophy"></i>
            <p>{{ rank }}</p>
          </div>
        </div>
      </div>
    </div>



    <!-- <div class="boxes grid" v-if="!loading">

      <div class="box white nivel-actual-card grid-item">
        <h2>Nivel Actual</h2>
        <div class="rank-image">
          <img src="https://s3-alpha-sig.figma.com/img/7802/2e20/5a4b8e6b84fee7f04d9ef27eda097ee1?Expires=1740355200&Key-Pair-Id=APKAQ4GOSFWCW27IBOMQ&Signature=fzkCIOclXtfeMBRARqHw3U4324RsIc49HJWuT5gZaENS5FGF22rzoZm803hywEtzf9kOUOsNO1qFLU0uuC7HGpjb~6AmI5538XhUFHNw~FUzbJ-OTQOJprbPeVvG4pRlT58ArVxElY1gM5rF5RbXvKP5UCtojppAk22GHbBpDuBS6qTorWHV5eE-pOEuvDF-e3EecDYUgpg8hCyiaCPHhLI4z4dnPzVMSJ8p7QKiG3wfftBsbRYOfH-tuAZygeUrILyZgISRYoAJ1ns4hxzkk7jQ619lCafSisSFIHNbf~pmMEeT9pK6ehD3cqjwGHXUTcmUoQc0rTGNna~9Jm3FBA__" alt="Rango" />
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

      <div class="box white grid-item">
        <h3>Rango Alcanzado</h3>
        <p>{{ achievedRank }}</p>
        <p>Puntos Grupales: {{ groupPoints }}</p>
        <p>Frontales: {{ frontales }}</p>
      </div>

      <div class="box white grid-item">
        <i class="fas fa-wallet"></i>
        <div>
          <p>S/. {{ balance }}</p>
          <span>SALDO</span>
        </div>
      </div>

      <div class="box white grid-item">
        <i class="fas fa-hand-holding-usd"></i>
        <div>
          <p>S/. {{ _balance }}</p>
          <span>SALDO NO DISPONIBLE</span>
        </div>
      </div>

      <div class="box white grid-item">
        <i class="fas fa-hand-holding-usd"></i>
        <div>
          <p>S/. {{ Number(ins + insVirtual).toFixed(2) }}</p>
          <span>TOTAL GANADO</span>
        </div>
      </div>

      <div class="box white grid-item">
        <i class="fas fa-medal"></i>
        <div>
          <p>{{ points }}</p>
          <span>PUNTOS</span>
        </div>
      </div>


      <div class="box white grid-item" v-if="node">
        <i class="fas fa-gem"></i>
        <div>
          <p>{{ node.rank | _rank }}</p>
          <span>RANGO ACTUAL</span>
        </div>
      </div>


      <div class="box white grid-item" v-if="node">
        <i class="fa fa-tachometer"></i>
        <div>
          <p>{{ node.next_rank.name | _rank }}</p>
          <span>SIGUIENTE RANGO</span>
        </div>
      </div>

      <div class="box white grid-item">
        <h3>Pack de Afiliación</h3>
        <p></p>
      </div>

    </div> -->

  </App>
</template>

<script>
import App from '@/views/layouts/App'
import api from '@/api'
import Masonry from 'masonry-layout'

export default {
  components: {
    App,
  },
  data() {
    return {
      banner:  { img: null },
      ins:     null,
      outs:    null,
      balance: null,
     _balance: null,
      team:    null,
      rank:    null,
      points:  null,

      loading: true,

      op: 1,
      op2: 0,
      op3: 0,
    }
  },
  computed: {
    session () { return this.$store.state.session },
    plan    () { return this.$store.state.plan },
    title() { return 'Dashboard' },
  },
  filters: {
    _rank(val) {
      if(val == 'none')              return 'Ninguno'
      if(val == 'active')            return 'ACTIVO'
      if(val == 'star')              return 'MASTER'
      if(val == 'master')            return 'PLATA'
      if(val == 'silver')            return 'PLATINO'
      if(val == 'gold')              return 'ORO'
      if(val == 'sapphire')          return 'ZAFIRO'
      if(val == 'RUBI')              return 'DIAMANTE RUBI'
      if(val == 'DIAMANTE')          return 'DIAMANTE ESTRELLA'
      if(val == 'DOBLE DIAMANTE')    return 'DIAMANTE DOS ESTRELLAS'
      if(val == 'TRIPLE DIAMANTE')   return 'DIAMANTE TRES ESTRELLAS'
      if(val == 'DIAMANTE ESTRELLA') return 'DIAMANTE CBM'
    },
  },
  async created() {
    // GET data
    const { data } = await api.dashboard(this.session); console.log({ data })

    this.loading = false

    // error
    if(data.error && data.msg == 'invalid session') this.$router.push('/login')

    // success
    this.$store.commit('SET_NAME',       data.name)
    this.$store.commit('SET_LAST_NAME',  data.lastName)
    this.$store.commit('SET_AFFILIATED', data.affiliated)
    this.$store.commit('SET__ACTIVATED',  data._activated)
    this.$store.commit('SET_ACTIVATED',  data.activated)
    this.$store.commit('SET_PLAN',       data.plan)
    this.$store.commit('SET_COUNTRY',    data.country)
    this.$store.commit('SET_PHOTO',      data.photo)
    this.$store.commit('SET_TREE',       data.tree)
    this.$store.commit('SET_EMAIL',       data.email)

    this.banner   = data.banner
    this.ins      = data.ins
    this.insVirtual = data.insVirtual
    this.outs     = data.outs.toFixed(2)
    this.balance  = data.balance.toFixed(2)
    this._balance = data._balance.toFixed(2)
    this.team     = data.team
    this.rank     = data.rank
    this.points   = data.points
    this.node     = data.node
    this.n_affiliates = data.n_affiliates

    const time = 4000
    let   i = 0
    const n = 3

    setInterval(() => {

      i += 1
      console.log(i)

      if(i == 0){
        // reset all
        this.op = 0; this.op2 = 0; this.op3 = 0
        // show 0
        this.op = 1
      }

      if(i == 1) {
        // reset all
        this.op = 0; this.op2 = 0; this.op3 = 0
        // show 0
        this.op2 = 1
      }

      if(i == 2) {
        // reset all
        this.op = 0; this.op2 = 0; this.op3 = 0
        // show 0
        this.op3 = 1
      }


      if(i == n - 1) i = -1

    }, time)

  },
  mounted() {
    this.$nextTick(() => {
      // Initialize Masonry after the content is loaded
      new Masonry('.grid', {
        itemSelector: '.grid-item',
        columnWidth: '.grid-item',
        percentPosition: true,
        gutter: 20
      })
    })
  }
};
</script>

<!-- <style scoped>
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
  background-color: #4CAF50;
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

.boxes {
  padding: 1rem;
  width: 100%;
}

.grid-item {
  width: calc(33.333% - 16px);
  margin-bottom: 16px;
}

@media (max-width: 1200px) {
  .grid-item {
    width: calc(50% - 16px);
  }
}

@media (max-width: 768px) {
  .grid-item {
    width: 100%;
  }
}
</style> -->
<style scoped>
.grid {
  width: 100%;
  padding: 20px;
}

.grid-item {
  width: calc(33.333% - 20px);
  margin-bottom: 20px;
}

.card {
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.card h3 {
  margin: 0 0 15px 0;
  color: #333;
  font-size: 1.2em;
}

.card-content {
  display: flex;
  align-items: center;
  gap: 15px;
}

.card-content i {
  font-size: 24px;
  color: #4a90e2;
}

.card-content p {
  font-size: 1.5em;
  font-weight: bold;
  margin: 0;
  color: #2c3e50;
}

/* @media (max-width: 1200px) {
  .grid-item {
    width: calc(50% - 20px);
  }
} */

@media (max-width: 768px) {
  .grid-item {
    width: 100%;
  }
}
</style>