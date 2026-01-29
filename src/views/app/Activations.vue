<template>
  <App :session="session">

    <h4 class="tabs">
      <router-link class="tab" to="/activation">
        Comprar
      </router-link> &nbsp;&nbsp;
      <router-link class="tab" to="/activations">
        Historial
      </router-link>
    </h4>

    <i class="load" v-if="loading"></i> <br>

    <!-- <table>
      <thead>
        <tr>
          <th>1</th>
          <th>2</th>
          <th>3</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td :class="{'green': arr[0]}">250</td>
          <td :class="{'green': arr[1]}">450</td>
          <td :class="{'green': arr[2]}">650</td>
        </tr>
      </tbody>
    </table> -->

    <div class="scroll" v-if="!loading">
      <table>
        <thead>
          <tr>
            <th>Fecha</th>
            <th>Productos</th>
            <th>Monto</th>
            <th>Puntos</th>
            <th>Voucher</th>
            <th>Estado</th>
            <th>Boleta</th>
            <th>Detalle</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="activation in activations">
            <td>{{ activation.date | date }}</td>
            <td>
              <div v-for="product in activation.products" v-if="product.total != 0" style="display: flex; align-items: center; gap: 8px;">
                <img v-if="product.image" :src="product.image" style="width: 40px; height: 40px; object-fit: cover; border-radius: 4px;">
                <span>{{ product.name }} - {{ product.total }}</span>
              </div>
              <div style="margin-top: 4px;">
                <span class="tag is-info is-light" style="font-size: 0.7em; padding: 0 6px;" v-if="activation.type === 'MEMBRESÍA' || (activation.products && activation.products.some(p => p.name.includes('Membresía')))">Membresía</span>
                <span class="tag is-warning is-light" style="font-size: 0.7em; padding: 0 6px;" v-if="activation.type === 'LOTE' || (activation.products && activation.products.some(p => p.name.includes('Lote')))">Lote</span>
              </div>
            </td>
            <td>{{ activation.price | price }}</td>
            <td>{{ activation.points }}</td>
            <td>
              <a :href="activation.voucher" target="_blank" v-if="activation.voucher">
                <img :src="activation.voucher" style="max-height: 80px; max-width: 80px">
              </a>
              <span v-else>-</span>
            </td>
            <td>{{ activation.status | status }}</td>
            <td><a :href="`${INVOICE_ROOT}?id=${activation.id}`" target="_blank" style="color: gray;"
                v-if="activation.status == 'approved'">boleta</a>
            </td>
            <!-- Detail Button -->
            <td>
               <button class="button is-small is-info is-light" @click="openDetail(activation)">
                 <i class="fas fa-eye"></i>
               </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Details Modal -->
    <div class="modal" :class="{'is-active': showModal}">
      <div class="modal-background" @click="closeModal"></div>
      <div class="modal-card">
        <header class="modal-card-head">
          <p class="modal-card-title">Detalles de Operación</p>
          <button class="delete" aria-label="close" @click="closeModal"></button>
        </header>
        <section class="modal-card-body" v-if="selectedActivation">
           
           <h5 class="title is-5">Información General</h5>
           <div class="columns is-mobile is-multiline">
              <div class="column is-6"><strong>ID:</strong> <br> {{ selectedActivation.id }}</div>
              <div class="column is-6"><strong>Fecha:</strong> <br> {{ selectedActivation.date | date }}</div>
              <div class="column is-6"><strong>Estado:</strong> <br> {{ selectedActivation.status | status }}</div>
              <div class="column is-6"><strong>Tipo:</strong> <br> {{ selectedActivation.type || 'ACTIVACIÓN' }}</div>
           </div>

           <hr>

           <h5 class="title is-5">Datos del Comprador</h5>
           <div v-if="selectedActivation.buyer" class="content is-small">
              <div class="columns is-mobile is-multiline">
                <div class="column is-12" v-if="selectedActivation.buyer.name"><strong>Nombre:</strong> {{ selectedActivation.buyer.name }}</div>
                <div class="column is-6" v-if="selectedActivation.buyer.dni"><strong>DNI/Cédula:</strong> {{ selectedActivation.buyer.dni }}</div>
                <div class="column is-6" v-if="selectedActivation.buyer.email"><strong>Email:</strong> {{ selectedActivation.buyer.email }}</div>
                <div class="column is-6" v-if="selectedActivation.buyer.phone"><strong>Teléfono:</strong> {{ selectedActivation.buyer.phone }}</div>
                <div class="column is-12" v-if="selectedActivation.buyer.address"><strong>Dirección:</strong> {{ selectedActivation.buyer.address }}</div>
              </div>
           </div>
           <div v-else class="notification is-light">
             No hay datos de comprador registrados en este objeto.
           </div>

           <hr>
           <h5 class="title is-5">Datos del Vendedor / Patrocinador</h5>
           <div class="content is-small">
              <p><strong>Seller ID:</strong> {{ selectedActivation.sellerId || 'Sistema/Admin' }}</p>
           </div>
           
        </section>
        <footer class="modal-card-foot">
          <button class="button" @click="closeModal">Cerrar</button>
        </footer>
      </div>
    </div>

  </App>
</template>

<script>
import App from '@/views/layouts/App'
import api from '@/api'

const INVOICE_ROOT = process.env.VUE_APP_INVOICE_ROOT
console.log({ INVOICE_ROOT })

export default {
  components: {
    App,
  },
  data() {
    return {
      activations: null,
      // arr: [0,0,0],
      loading: true,
      INVOICE_ROOT,
      showModal: false,
      selectedActivation: null
    }
  },
  computed: {
    session() { return this.$store.state.session },
  },
  filters: {
    date(val) {
      if (!val) return '-'
      return new Date(val).toLocaleDateString() + ' ' + new Date(val).toLocaleTimeString()
    },
    price(val) {
      return `S/. ${(val || 0).toFixed(2)}`
    },
    status(val) {
      if (val == 'pending') return 'Pendiente'
      if (val == 'approved') return 'Aprobada'
      if (val == 'rejected') return 'Rechazada'
      return val
    },
  },
  methods: {
    openDetail(item) {
      this.selectedActivation = item
      this.showModal = true
    },
    closeModal() {
      this.showModal = false
      this.selectedActivation = null
    }
  },
  async created() {
    // GET data
    const { data } = await api.Activations.GET(this.session); console.log({ data })

    this.loading = false

    // error
    if (data.error && data.msg == 'invalid session') this.$router.push('/login')
    if (data.error && data.msg == 'unverified user') this.$router.push('/verify')

    // success
    this.$store.commit('SET_NAME', data.name)
    this.$store.commit('SET_LAST_NAME', data.lastName)
    this.$store.commit('SET_AFFILIATED', data.affiliated)
    this.$store.commit('SET_ACTIVATED', data.activated)
    this.$store.commit('SET__ACTIVATED', data._activated)
    this.$store.commit('SET_PLAN', data.plan)
    this.$store.commit('SET_COUNTRY', data.country)
    this.$store.commit('SET_PHOTO', data.photo)
    this.$store.commit('SET_TREE', data.tree)

    this.activations = data.activations
    // this.arr         = data.arr

  },
};
</script>
