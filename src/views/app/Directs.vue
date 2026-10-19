<template>
  <App :session="session" :office_id="office_id" :title="title">
    <div class="invite-container">
      <div class="page-header-text">
        <h1 class="page-title">Invita a tu equipo comercial</h1>
        <p class="page-desc">Comparte tu enlace para registrar a las personas que ofrecerán membresías IBL como parte de tu equipo.</p>
      </div>

      <div class="invite-card">
        <div class="card-header-row">
          <div class="card-icon-badge">
            <i class="fas fa-user-plus"></i>
          </div>
          <div class="card-header-info">
            <h2 class="card-title">Incorpora nuevos vendedores</h2>
            <p class="card-desc">Invita a personas que quieran ofrecer membresías IBL.</p>
          </div>
        </div>

        <div class="field-group">
          <label class="field-label">Tu código de invitación</label>
          <div class="input-box-wrapper">
            <input class="value-display code-font" readonly :value="token" id="token" />
            <button type="button" class="btn-copy-action" @click="copy_token">
              <i :class="c_token ? 'fas fa-check' : 'far fa-copy'"></i>
              <span>{{ c_token ? 'Copiado' : 'Copiar código' }}</span>
            </button>
          </div>
        </div>

        <div class="field-group">
          <label class="field-label">Enlace para incorporar vendedores</label>
          <div class="input-box-wrapper">
            <input class="value-display" readonly :value="link" id="link" />
            <button type="button" class="btn-copy-action" @click="copy_link">
              <i :class="c_link ? 'fas fa-check' : 'far fa-copy'"></i>
              <span>{{ c_link ? 'Copiado' : 'Copiar enlace' }}</span>
            </button>
          </div>
        </div>

        <div class="action-buttons-row">
          <button type="button" class="btn-share-main" @click="share_link">
            <i class="fas fa-share-alt"></i>
            <span>Compartir invitación</span>
          </button>
          <a :href="link" target="_blank" class="btn-register-secondary">
            <i class="fas fa-user-plus"></i>
            <span>Registrar vendedor</span>
          </a>
        </div>

        <div class="info-alert-banner">
          <i class="fas fa-info-circle info-icon"></i>
          <span>Este enlace es para registrar vendedores de membresías.</span>
        </div>
      </div>

      <!-- Vendedores registrados si existen -->
      <div v-if="directs && directs.length" class="directs-table-section">
        <h3 class="table-section-title">Vendedores registrados ({{ directs.length }})</h3>
        <div class="table-responsive">
          <table class="custom-table">
            <thead>
              <tr>
                <th>Nombre</th>
                <th>Email</th>
                <th>Teléfono</th>
                <th>Afiliado</th>
                <th>Activo</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(direct, i) in directs" :key="i">
                <td>{{ direct.name }} {{ direct.lastName }}</td>
                <td>{{ direct.email }}</td>
                <td>{{ direct.phone }}</td>
                <td>
                  <span class="status-pill" :class="direct.affiliated ? 'status-yes' : 'status-no'">
                    {{ direct.affiliated ? 'Sí' : 'No' }}
                  </span>
                </td>
                <td>
                  <span class="status-pill" :class="direct.activated ? 'status-yes' : 'status-no'">
                    {{ direct.activated ? 'Sí' : 'No' }}
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </App>
</template>

<script>
import App from '@/views/layouts/App'
import api from '@/api'
import lib from '@/lib'

const ROOT = process.env.VUE_APP_ROOT

export default {
  components: {
    App,
  },
  data() {
    return {
      id: null,
      directs: [],
      frontals: [],
      loading: true,
      token: '',
      c_token: false,
      c_link: false,
    }
  },
  computed: {
    session() { return this.$store.state.session },
    office_id() { return this.$store.state.office_id },
    link() { return `${ROOT}/register/${this.token}` },
    title() { return 'Equipo comercial' },
  },
  async created() {
    const { data } = await api.directs(this.session)
    this.loading = false

    if(data.error && data.msg === 'invalid session') this.$router.push('/login')
    if(data.error && data.msg === 'unverified user') this.$router.push('/verify')

    this.$store.commit('SET_NAME', data.name)
    this.$store.commit('SET_LAST_NAME', data.lastName)
    this.$store.commit('SET_AFFILIATED', data.affiliated)
    this.$store.commit('SET__ACTIVATED', data._activated)
    this.$store.commit('SET_ACTIVATED', data.activated)
    this.$store.commit('SET_PLAN', data.plan)
    this.$store.commit('SET_COUNTRY', data.country)
    this.$store.commit('SET_PHOTO', data.photo)
    this.$store.commit('SET_TREE', data.tree)

    this.token = data.token || ''
    this.id = data.id
    this.directs = data.directs || []
    this.frontals = data.frontals || []
  },
  methods: {
    copy_token() {
      if (navigator.clipboard) {
        navigator.clipboard.writeText(this.token).catch(() => lib.copy('token'))
      } else {
        lib.copy('token')
      }
      this.c_token = true
      setTimeout(() => this.c_token = false, 3000)
    },
    copy_link() {
      if (navigator.clipboard) {
        navigator.clipboard.writeText(this.link).catch(() => lib.copy('link'))
      } else {
        lib.copy('link')
      }
      this.c_link = true
      setTimeout(() => this.c_link = false, 3000)
    },
    share_link() {
      if (navigator.share) {
        navigator.share({
          title: 'Únete a mi equipo en IBL',
          text: 'Comparte tu enlace para registrar a las personas que ofrecerán membresías IBL como parte de tu equipo.',
          url: this.link
        }).catch(() => {})
      } else {
        this.copy_link()
      }
    },
  },
}
</script>

<style scoped>
.invite-container {
  max-width: 860px;
  margin: 0 auto;
  padding: 10px 0 40px 0;
}

.page-header-text {
  margin-bottom: 24px;
}

.page-title {
  font-size: 28px;
  font-weight: 700;
  color: #0f2942;
  margin: 0 0 8px 0;
  letter-spacing: -0.3px;
}

.page-desc {
  font-size: 15px;
  color: #64748b;
  margin: 0;
  line-height: 1.5;
}

.invite-card {
  background: #ffffff;
  border-radius: 16px;
  padding: 32px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.04);
  border: 1px solid #edf2f7;
}

.card-header-row {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 28px;
}

.card-icon-badge {
  width: 52px;
  height: 52px;
  background: #e6f3ff;
  border-radius: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.card-icon-badge i {
  color: #2196f3;
  font-size: 22px;
}

.card-header-info {
  display: flex;
  flex-direction: column;
}

.card-title {
  font-size: 19px;
  font-weight: 700;
  color: #0f172a;
  margin: 0 0 4px 0;
}

.card-desc {
  font-size: 14px;
  color: #64748b;
  margin: 0;
}

.field-group {
  margin-bottom: 20px;
}

.field-label {
  display: block;
  font-size: 13.5px;
  font-weight: 700;
  color: #1e293b;
  margin-bottom: 8px;
}

.input-box-wrapper {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #f8fafc;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  padding: 6px 6px 6px 16px;
  transition: border-color 0.2s ease;
}

.input-box-wrapper:focus-within {
  border-color: #2196f3;
}

.value-display {
  flex: 1;
  border: none;
  background: transparent;
  outline: none;
  font-size: 15px;
  color: #334155;
  padding: 6px 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.value-display.code-font {
  font-size: 20px;
  font-weight: 700;
  letter-spacing: 0.8px;
  color: #0f172a;
}

.btn-copy-action {
  background: #ffffff;
  border: 1.5px solid #2196f3;
  color: #2196f3;
  border-radius: 6px;
  padding: 9px 18px;
  font-size: 14px;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
  white-space: nowrap;
}

.btn-copy-action:hover {
  background: #f0f7ff;
}

.action-buttons-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  margin-top: 26px;
}

.btn-share-main {
  background: #2196f3;
  color: #ffffff;
  border: none;
  border-radius: 8px;
  padding: 13px 20px;
  font-size: 15px;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 2px 8px rgba(33, 150, 243, 0.25);
}

.btn-share-main:hover {
  background: #1976d2;
}

.btn-register-secondary {
  background: #ffffff;
  color: #2196f3;
  border: 1.5px solid #2196f3;
  border-radius: 8px;
  padding: 13px 20px;
  font-size: 15px;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-register-secondary:hover {
  background: #f0f7ff;
}

.info-alert-banner {
  margin-top: 24px;
  background: #e6f4fe;
  border-radius: 8px;
  padding: 14px 18px;
  display: flex;
  align-items: center;
  gap: 12px;
  color: #1a568c;
  font-size: 14px;
  font-weight: 500;
}

.info-alert-banner .info-icon {
  font-size: 18px;
  color: #2196f3;
  flex-shrink: 0;
}

/* Directs Table Section */
.directs-table-section {
  margin-top: 36px;
  background: #ffffff;
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.04);
  border: 1px solid #edf2f7;
}

.table-section-title {
  font-size: 18px;
  font-weight: 700;
  color: #0f2942;
  margin: 0 0 16px 0;
}

.table-responsive {
  overflow-x: auto;
}

.custom-table {
  width: 100%;
  border-collapse: collapse;
}

.custom-table th {
  background: #f8fafc;
  color: #475569;
  font-weight: 600;
  font-size: 13px;
  text-align: left;
  padding: 12px 16px;
  border-bottom: 1px solid #e2e8f0;
}

.custom-table td {
  padding: 12px 16px;
  border-bottom: 1px solid #f1f5f9;
  font-size: 14px;
  color: #334155;
}

.status-pill {
  display: inline-block;
  padding: 2px 10px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 600;
}

.status-yes {
  background: #def7ec;
  color: #03543f;
}

.status-no {
  background: #fde8e8;
  color: #9b1c1c;
}

@media (max-width: 600px) {
  .invite-card {
    padding: 20px 16px;
  }
  .action-buttons-row {
    grid-template-columns: 1fr;
  }
  .page-title {
    font-size: 22px;
  }
  .input-box-wrapper {
    flex-direction: column;
    align-items: stretch;
    gap: 8px;
    padding: 10px;
  }
  .btn-copy-action {
    justify-content: center;
  }
}
</style>
