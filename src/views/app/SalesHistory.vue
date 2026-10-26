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

        <div class="table-wrap" v-else>
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
              <tr v-for="sale in group.sales" :key="sale.id">
                <td class="nowrap">{{ sale.date | date }}</td>
                <td>{{ sale.name }}</td>
                <td class="nowrap">{{ periodLabel(sale.period) }}</td>
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

        <div class="table-foot" v-if="filtered.length">
          <span>Mostrando {{ pageSales.length }} de {{ filtered.length }} {{ filtered.length === 1 ? 'venta' : 'ventas' }}</span>
          <div class="pager" v-if="pages > 1">
            <button type="button" :disabled="page === 1" @click="page--" aria-label="Página anterior"><i class="fas fa-chevron-left"></i></button>
            <button type="button" v-for="n in pages" :key="n" :class="{ active: n === page }" @click="page = n">{{ n }}</button>
            <button type="button" :disabled="page === pages" @click="page++" aria-label="Página siguiente"><i class="fas fa-chevron-right"></i></button>
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
    pageSales() {
      const start = (this.page - 1) * PAGE_SIZE;
      return this.filtered.slice(start, start + PAGE_SIZE);
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
  --green: #1f8a3b;
  --muted: #5d6b7c;
  color: var(--navy);
  background: #fff;
  border-radius: 14px;
  box-shadow: 0 4px 16px rgba(16, 40, 74, 0.07);
  padding: 20px 22px;
  max-width: 1300px;
}

p {
  margin: 0;
}

.history-head {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 16px;
  flex-wrap: wrap;
}

.history-head h2 {
  margin: 0;
  font-size: 1.7rem;
  font-weight: 800;
  color: var(--navy);
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
  padding: 10px 20px;
  border-radius: 8px;
  text-decoration: none;
  white-space: nowrap;
}

.sell-btn:hover {
  background: #197432;
}

.filter-bar {
  display: flex;
  align-items: center;
  gap: 14px;
  margin: 18px 0;
  padding: 14px 16px;
  border: 1px solid #e3e9f0;
  border-radius: 10px;
}

.filter-bar label {
  font-weight: 700;
}

.filter-bar select {
  min-width: 280px;
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
}

.empty i {
  font-size: 2.4rem;
  color: #c3ccd7;
  margin-bottom: 10px;
}

.table-wrap {
  overflow-x: auto;
  border-radius: 10px;
  border: 1px solid #e3e9f0;
}

table {
  width: 100%;
  border-collapse: collapse;
  min-width: 960px;
}

thead th {
  background: var(--blue);
  color: #fff;
  font-weight: 700;
  text-align: left;
  padding: 14px 16px;
  white-space: nowrap;
}

td {
  padding: 14px 16px;
  border-bottom: 1px solid #edf1f5;
  vertical-align: middle;
  text-align: left;
  color: var(--navy);
}

thead th,
.group-row td {
  text-align: left;
}

.group-row td {
  background: #e3effc;
  font-weight: 800;
  font-size: 1.05rem;
  padding: 10px 16px;
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

.buyer {
  display: table-cell;
}

.buyer strong,
.buyer span {
  display: block;
}

.buyer span {
  color: var(--muted);
  font-size: 0.9rem;
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
  color: #1c6fd1;
  font-size: 0.85rem;
  text-decoration: underline;
}

.vouchers img {
  width: 46px;
  height: 46px;
  object-fit: cover;
  border-radius: 4px;
  border: 1px solid #d3dce6;
}

.status {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 6px 14px;
  border-radius: 999px;
  font-weight: 700;
  font-size: 0.85rem;
  text-transform: uppercase;
  white-space: nowrap;
}

.status.approved {
  background: #e3f5e8;
  color: #1f8a3b;
}

.status.pending {
  background: #fdf1d3;
  color: #8a5a00;
}

.status.rejected {
  background: #fde4e4;
  color: #c62828;
}

.invoice-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: #1c6fd1;
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 9px 18px;
  font-weight: 700;
  font-size: 0.9rem;
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
  margin-top: 14px;
  color: var(--muted);
}

.pager {
  display: flex;
  gap: 6px;
}

.pager button {
  min-width: 34px;
  height: 34px;
  border: 1px solid #d3dce6;
  border-radius: 6px;
  background: #fff;
  color: var(--navy);
  cursor: pointer;
}

.pager button.active {
  background: #1c6fd1;
  border-color: #1c6fd1;
  color: #fff;
}

.pager button:disabled {
  opacity: 0.4;
  cursor: default;
}

@media (max-width: 600px) {
  .history {
    padding: 16px;
  }

  .history-head h2 {
    font-size: 1.3rem;
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
