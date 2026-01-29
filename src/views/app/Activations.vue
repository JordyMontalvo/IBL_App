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
    <!-- Details Modal (Card Style) -->
    <div class="modal" :class="{'is-active': showModal}">
      <div class="modal-background" @click="closeModal"></div>
      
      <div class="modal-card">
        <!-- Custom Header -->
        <div class="modal-header-custom">
           <div class="modal-title">
             <i class="fas fa-user-plus"></i> Detalles de la Operación
           </div>
           <i class="fas fa-times close-icon" @click="closeModal"></i>
        </div>

        <!-- Custom Body -->
        <div class="modal-body-custom" v-if="selectedActivation">
           <div class="info-grid">
              
              <!-- ID -->
              <div class="info-card">
                 <div class="info-label"><i class="fas fa-hashtag"></i> ID:</div>
                 <div class="info-value">{{ selectedActivation.id }}</div>
              </div>

              <!-- Fecha -->
              <div class="info-card">
                 <div class="info-label"><i class="fas fa-calendar-alt"></i> Fecha:</div>
                 <div class="info-value">{{ selectedActivation.date | date }}</div>
              </div>

              <!-- Usuario (Comprador) -->
              <div class="info-card">
                 <div class="info-label"><i class="fas fa-user"></i> Usuario:</div>
                 <div class="info-value">
                   {{ (selectedActivation.buyer && selectedActivation.buyer.name) ? selectedActivation.buyer.name : 'No registrado' }}
                 </div>
              </div>

              <!-- DNI -->
              <div class="info-card">
                 <div class="info-label"><i class="fas fa-id-card"></i> DNI:</div>
                 <div class="info-value">
                   {{ (selectedActivation.buyer && selectedActivation.buyer.dni) ? selectedActivation.buyer.dni : '-' }}
                 </div>
              </div>

              <!-- Oficina (Placeholder/Dynamic if available) -->
              <div class="info-card">
                 <div class="info-label"><i class="fas fa-building"></i> Oficina:</div>
                 <div class="info-value">{{ selectedActivation.office || 'OFICINA MATRIZ' }}</div>
              </div>

              <!-- Plan (Placeholder/Dynamic) -->
              <div class="info-card">
                 <div class="info-label"><i class="fas fa-cubes"></i> Tipo:</div>
                 <div class="info-value">{{ selectedActivation.type || 'ACTIVACIÓN' }}</div>
              </div>

              <!-- Monto -->
              <div class="info-card">
                 <div class="info-label"><i class="fas fa-money-bill-wave"></i> Monto:</div>
                 <div class="info-value">{{ selectedActivation.price | price }}</div>
              </div>

              <!-- Productos (Full Width) -->
              <div class="info-card full-width">
                 <div class="info-label"><i class="fas fa-shopping-bag"></i> Productos:</div>
                 <div class="info-value">
                    <span v-for="(p, index) in selectedActivation.products" :key="index">
                      {{ p.total }} {{ p.name }}<span v-if="index < selectedActivation.products.length - 1">, </span>
                    </span>
                 </div>
              </div>

              <!-- Vendedor (Full Width if present) -->
               <div class="info-card full-width" v-if="selectedActivation.sellerId">
                 <div class="info-label"><i class="fas fa-hand-holding-usd"></i> Vendedor ID:</div>
                 <div class="info-value">{{ selectedActivation.sellerId }}</div>
              </div>

           </div>
        </div>

        <!-- Custom Footer -->
        <div class="modal-footer-custom">
           <button class="button is-info" @click="closeModal">
             <i class="fas fa-times"></i> &nbsp; Cerrar
           </button>
        </div>

      </div>
    </div>

  </App>
</template>

<style>
/* Global style override to ensure modal visibility */
.modal {
  z-index: 999999 !important;
  display: none; /* Hidden by default */
  align-items: center;
  justify-content: center;
}
.modal.is-active {
  display: flex !important;
  position: fixed !important;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
}
</style>

<style scoped>
/* Card Modal Styling */
.modal-card {
  max-width: 500px;
  width: 95%;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(0,0,0,0.3);
}

.modal-header-custom {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: white;
}
.modal-title {
  font-size: 1.25rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 10px;
}
.close-icon {
  cursor: pointer;
  opacity: 0.8;
  transition: opacity 0.2s;
}
.close-icon:hover {
  opacity: 1;
}

.modal-body-custom {
  background-color: #f7fafc; /* Light gray bg */
  padding: 20px;
  max-height: 80vh;
  overflow-y: auto;
}

.info-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
}

.info-card {
  background: white;
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.05);
  display: flex;
  flex-direction: column;
}

.info-label {
  font-size: 0.85rem;
  color: #667eea; /* Purple-ish text for labels */
  font-weight: 600;
  margin-bottom: 5px;
  display: flex;
  align-items: center;
  gap: 6px;
}

.info-value {
  font-size: 1rem;
  color: #2d3748;
  font-weight: 500;
  word-break: break-word; /* Prevent overflow */
}

/* Make products and Buyer full width */
.full-width {
  grid-column: span 2;
}

.modal-footer-custom {
  padding: 15px 20px;
  background: white;
  display: flex;
  justify-content: flex-end;
  border-top: 1px solid #e2e8f0;
}
</style>

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
      console.log('Opening details for:', item)
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
