<template>
  <App :session="session" :title="title">
    <div class="history">
      <div class="history-head">
        <div>
          <h2>Historial de ventas de membresías</h2>
          <p>Consulta tus ventas personales de membresías.</p>
        </div>
        <router-link to="/membership" class="sell-btn">
          <i class="fas fa-plus"></i> Vender membresía
        </router-link>
      </div>

      <div class="filter-bar">
        <label for="period">Período</label>
        <select id="period" v-model="period">
          <option value="all">Todos (agrupado por mes)</option>
          <option v-for="p in periods" :key="p" :value="p">{{ periodLabel(p) }}</option>
        </select>
      </div>

      <i class="load" v-if="loading"></i>

      <p class="error" v-if="error">{{ error }}</p>

      <template v-if="!loading && !error">
        <div class="empty" v-if="!filtered.length">
          <i class="fas fa-receipt"></i>
          <p v-if="sales.length">No tienes ventas en este período.</p>
          <p v-else>Todavía no has vendido membresías.</p>
        </div>

        <div class="table-card" v-else>
        <div class="table-wrap">
          <table>
            <thead>
              <tr>
                <th>Fecha</th>
                <th>Membresía</th>
                <th>Período</th>
                <th>Detalle del comprador</th>
                <th class="num">Monto</th>
                <th class="center">Voucher</th>
                <th class="center">Estado</th>
                <th class="center">Boleta</th>
              </tr>
            </thead>
            <tbody v-for="group in groups" :key="group.period">
              <tr class="group-row">
                <td colspan="8">{{ periodLabel(group.period) }}</td>
              </tr>
              <tr v-for="(sale, i) in group.sales" :key="sale.id" :class="{ alt: i % 2 === 1 }">
                <td class="nowrap strong">{{ sale.date | date }}</td>
                <td class="strong">{{ sale.name }}</td>
                <td class="nowrap strong">{{ periodLabel(sale.period) }}</td>
                <td class="buyer">
                  <strong>{{ sale.buyer.name || '—' }}</strong>
                  <span>DNI: {{ sale.buyer.dni || '—' }}</span>
                  <span>Cel.: {{ sale.buyer.phone || '—' }}</span>
                </td>
                <td class="num nowrap">S/ {{ money(sale.price) }}</td>
                <td class="center">
                  <div class="vouchers" v-if="sale.voucher || sale.voucher2">
                    <a v-for="(v, i) in [sale.voucher, sale.voucher2].filter(Boolean)" :key="i" :href="v" target="_blank" rel="noopener">
                      <img :src="v" alt="Voucher" />
                      <span>Ver</span>
                    </a>
                  </div>
                  <span class="muted" v-else>—</span>
                </td>
                <td class="center">
                  <span class="status" :class="sale.status">
                    <i class="fas" :class="statusIcon(sale.status)"></i>
                    {{ sale.status | status }}
                  </span>
                </td>
                <td class="center">
                  <!-- la boleta se habilitará más adelante -->
                  <button v-if="sale.status === 'approved'" type="button" class="invoice-btn" disabled title="Disponible próximamente">
                    <i class="fas fa-file-alt"></i> Boleta
                  </button>
                  <span class="muted" v-else>—</span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="table-foot">
          <span v-if="pages === 1">Mostrando {{ filtered.length }} {{ filtered.length === 1 ? 'venta' : 'ventas' }}</span>
          <span v-else>Mostrando {{ pageStart + 1 }}–{{ pageStart + pageSales.length }} de {{ filtered.length }} ventas</span>
          <div class="pager">
            <button type="button" :disabled="page === 1" @click="page--" aria-label="Página anterior"><i class="fas fa-chevron-left"></i></button>
            <button type="button" v-for="n in pages" :key="n" :class="{ active: n === page }" @click="page = n">{{ n }}</button>
            <button type="button" :disabled="page === pages" @click="page++" aria-label="Página siguiente"><i class="fas fa-chevron-right"></i></button>
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

const MONTHS = ["Enero", "Febrero", "Marzo", "Abril", "Mayo", "Junio", "Julio", "Agosto", "Septiembre", "Octubre", "Noviembre", "Diciembre"];
const PAGE_SIZE = 20;

export default {
  components: {
    App,
  },
  data() {
    return {
      sales: [],
      period: "all",
      page: 1,
      loading: true,
      error: null,
    };
  },
  computed: {
    session() {
      return this.$store.state.session;
    },
    title() {
      return "Historial de ventas";
    },
    periods() {
      return [...new Set(this.sales.map((s) => s.period))];
    },
    filtered() {
      if (this.period === "all") return this.sales;
      return this.sales.filter((s) => s.period === this.period);
    },
    pages() {
      return Math.max(1, Math.ceil(this.filtered.length / PAGE_SIZE));
    },
    pageStart() {
      return (this.page - 1) * PAGE_SIZE;
    },
    pageSales() {
      return this.filtered.slice(this.pageStart, this.pageStart + PAGE_SIZE);
    },
    groups() {
      const groups = [];
      for (const sale of this.pageSales) {
        let group = groups[groups.length - 1];
        if (!group || group.period !== sale.period) {
          group = { period: sale.period, sales: [] };
          groups.push(group);
        }
        group.sales.push(sale);
      }
      return groups;
    },
  },
  watch: {
    period() {
      this.page = 1;
    },
  },
  filters: {
    date(val) {
      if (!val) return "—";
      return new Date(val).toLocaleDateString("es-PE", { timeZone: "America/Lima", day: "2-digit", month: "2-digit", year: "numeric" });
    },
    status(val) {
      if (val == "approved") return "Aprobada";
      if (val == "pending") return "Pendiente";
      if (val == "rejected") return "Rechazada";
      return val;
    },
  },
  methods: {
    money(val) {
      return Number(val || 0).toLocaleString("en-US", { minimumFractionDigits: 2, maximumFractionDigits: 2 });
    },
    periodLabel(p) {
      const [y, m] = String(p || "").split("-").map(Number);
      if (!y || !m) return "—";
      return `${MONTHS[m - 1]} ${y}`;
    },
    statusIcon(status) {
      if (status === "approved") return "fa-check";
      if (status === "rejected") return "fa-times";
      return "fa-clock";
    },
  },
  async created() {
    try {
      const { data } = await api.Sales.GET(this.session);

      if (data.error && data.msg == "invalid session") return this.$router.push("/login");
      if (data.error) {
        this.error = "No se pudo cargar el historial. Intenta nuevamente.";
        return;
      }

      this.sales = data.sales || [];
    } catch (e) {
      this.error = "No se pudo cargar el historial. Intenta nuevamente.";
    } finally {
      this.loading = false;
    }
  },
};
</script>

<style scoped>
.history {
  --navy: #10284a;
  --blue: #0d4fa1;
  --link: #1c6fd1;
  --green: #1f8a3b;
  --muted: #5d6b7c;
  --line: #e2e9f1;
  --card-shadow: 0 4px 16px rgba(16, 40, 74, 0.07);
  color: var(--navy);
}

p {
  margin: 0;
}

/* Encabezado */
.history-head {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 16px;
  flex-wrap: wrap;
}

.history-head h2 {
  margin: 0;
  font-size: 1.8rem;
  font-weight: 800;
  color: var(--navy);
  line-height: 1.2;
}

.history-head p {
  color: var(--muted);
  margin-top: 2px;
}

.sell-btn {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  background: var(--green);
  color: #fff;
  font-weight: 700;
  padding: 11px 22px;
  border-radius: 8px;
  text-decoration: none;
  white-space: nowrap;
  box-shadow: 0 2px 6px rgba(31, 138, 59, 0.25);
}

.sell-btn:hover {
  background: #197432;
}

/* Filtro */
.filter-bar {
  display: flex;
  align-items: center;
  gap: 16px;
  margin: 18px 0;
  padding: 12px 16px;
  background: #fff;
  border-radius: 12px;
  box-shadow: var(--card-shadow);
}

.filter-bar label {
  display: inline;
  margin: 0;
  font-weight: 700;
  color: var(--navy);
}

.filter-bar select {
  min-width: 320px;
  max-width: 100%;
  padding: 9px 12px;
  border: 1px solid #d3dce6;
  border-radius: 8px;
  background: #fff;
  color: var(--navy);
  font-size: 0.95rem;
}

.error {
  color: #c62828;
  margin: 10px 0;
}

.empty {
  text-align: center;
  color: var(--muted);
  padding: 40px 10px;
  background: #fff;
  border-radius: 12px;
  box-shadow: var(--card-shadow);
}

.empty i {
  font-size: 2.4rem;
  color: #c3ccd7;
  margin-bottom: 10px;
}

/* Tabla */
.table-card {
  background: #fff;
  border-radius: 12px;
  box-shadow: var(--card-shadow);
  overflow: hidden;
}

.table-wrap {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
  min-width: 900px;
  font-size: 0.9rem;
}

table td {
  font-size: 0.9rem;
}

thead th {
  background: var(--blue);
  color: #fff;
  font-weight: 700;
  text-align: left;
  padding: 14px 14px;
  white-space: nowrap;
}

td {
  padding: 12px 14px;
  border-bottom: 1px solid var(--line);
  border-right: 1px solid var(--line);
  vertical-align: middle;
  text-align: left;
  color: var(--navy);
  background: #fff;
}

td:last-child {
  border-right: none;
}

tr.alt td {
  background: #f6f9fd;
}

.group-row td {
  background: #e2eefb;
  font-weight: 800;
  font-size: 1.05rem;
  padding: 10px 16px;
  border-right: none;
  text-align: left;
}

.strong {
  font-weight: 600;
}

.num {
  text-align: right;
  font-weight: 700;
}

.center {
  text-align: center;
}

.nowrap {
  white-space: nowrap;
}

.muted {
  color: #8a97a8;
}

.buyer strong,
.buyer span {
  display: block;
}

.buyer span {
  color: var(--navy);
  font-size: 0.92rem;
}

.vouchers {
  display: inline-flex;
  gap: 8px;
}

.vouchers a {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  color: var(--link);
  font-size: 0.85rem;
  text-decoration: underline;
}

.vouchers img {
  width: 44px;
  height: 56px;
  object-fit: cover;
  border-radius: 3px;
  border: 1px solid #d3dce6;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12);
}

.status {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 7px 14px;
  border-radius: 999px;
  font-weight: 700;
  font-size: 0.82rem;
  text-transform: uppercase;
  white-space: nowrap;
}

.status i {
  font-size: 1rem;
}

.status.approved {
  background: #e3f5e8;
  color: #1f8a3b;
}

.status.pending {
  background: #fdefc8;
  color: #1a2b44;
}

.status.rejected {
  background: #fde4e4;
  color: #c62828;
}

.invoice-btn {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  background: var(--link);
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 9px 20px;
  font-weight: 700;
  font-size: 0.92rem;
  box-shadow: 0 2px 6px rgba(28, 111, 209, 0.25);
}

.invoice-btn:disabled {
  cursor: default;
}

.table-foot {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
  padding: 14px 16px;
  color: var(--navy);
  font-size: 0.92rem;
}

.pager {
  display: flex;
  gap: 8px;
}

.pager button {
  min-width: 32px;
  height: 32px;
  border: 1px solid #d3dce6;
  border-radius: 6px;
  background: #fff;
  color: var(--navy);
  font-weight: 700;
  cursor: pointer;
}

.pager button.active {
  background: var(--link);
  border-color: var(--link);
  color: #fff;
}

.pager button:disabled {
  color: #8a97a8;
  cursor: default;
}

@media (max-width: 600px) {
  .history-head h2 {
    font-size: 1.35rem;
  }

  .sell-btn {
    width: 100%;
    justify-content: center;
  }

  .filter-bar {
    flex-direction: column;
    align-items: stretch;
    gap: 8px;
  }

  .filter-bar select {
    min-width: 0;
    width: 100%;
  }
}
</style>
