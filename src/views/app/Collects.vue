<template>
  <App :session="session" :title="title">

    <nav class="withdraw-tabs">
      <router-link class="withdraw-tab" to="/collect">Nuevo retiro</router-link>
      <router-link class="withdraw-tab" to="/collects">Historial de retiros</router-link>
    </nav>

    <i class="load" v-if="loading"></i>

    <div class="withdraw-card" v-if="!loading">
      <h2 class="withdraw-title">Historial de retiros</h2>
      <p class="withdraw-subtitle">Revisa el detalle de tus solicitudes de retiro.</p>

      <div class="scroll">
        <table class="withdraw-table">
          <thead>
            <tr>
              <th>Fecha</th>
              <th>Cuenta bancaria</th>
              <th>Monto</th>
              <th>Oficina</th>
              <th>Estado</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="collect in collects" :key="collect.id">
              <td>{{ collect.date | date }}</td>
              <td>
                <div v-if="!collect.cash" class="bank-name">En efectivo</div>
                <div v-else>
                  <div class="bank-name">{{ collect.bank }}</div>
                  <div class="bank-detail">Cuenta: {{ collect.account || 'No registrada' }}</div>
                  <div class="bank-detail">CCI: {{ collect.ink || 'No registrado' }}</div>
                </div>
              </td>
              <td class="amount">S/ {{ Number(collect.amount).toFixed(2) }}</td>
              <td>{{ collect.office | office }}</td>
              <td>
                <span class="status-badge" :class="collect.status == 'approved' ? 'paid' : 'pending'">
                  <i class="fas" :class="collect.status == 'approved' ? 'fa-check' : 'fa-clock'"></i>
                  {{ collect.status | status }}
                </span>
              </td>
            </tr>
            <tr v-if="!collects.length">
              <td colspan="5" class="withdraw-empty">Aún no tienes retiros.</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="withdraw-note">
        <i class="fas fa-info-circle"></i>
        El estado se actualizará cuando se registre el pago de tu retiro.
      </div>
    </div>

  </App>
</template>

<script>
import App from '@/views/layouts/App'
import api from '@/api'

export default {
  components: {
    App,
  },
  data() {
    return {
      collects: [],
      loading: true,
    }
  },
  computed: {
    session() { return this.$store.state.session },
    title()   { return 'Retiros' },
  },
  filters: {
    date(val) {
      return new Date(val).toLocaleDateString()
      // return new Date(val).toLocaleString()
    },
    status(val) {
      return val == 'approved' ? 'Pagado' : 'Pendiente'
    },
    office(val) {
      if (val == 'central') return 'Central'
      if (val == 'secondary') return 'Ledezma'
      return val
    },
  },
  async created() {
    // GET data
    const { data } = await api.Collects.GET(this.session); console.log({ data })

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

    this.collects = (data.collects || []).reverse()

  },
};
</script>
