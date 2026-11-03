<template>
  <App :session="session" :title="title">

    <i class="load" v-if="loading"></i>

    <div class="rk" v-if="!loading && r">

      <!-- cabecera: rango y activación por separado -->
      <div class="rk-head">
        <div class="rk-chip">
          <span class="rk-label">Rango alcanzado</span>
          <strong><i class="fas fa-medal"></i> {{ r.rankName }}</strong>
        </div>
        <div class="rk-chip">
          <span class="rk-label">Activación</span>
          <strong :class="r.activated ? 'ok' : 'off'">
            <i :class="r.activated ? 'fas fa-check-circle' : 'fas fa-times-circle'"></i>
            {{ r.activated ? 'Activo' : 'Inactivo' }}
          </strong>
        </div>
      </div>
      <p class="rk-hint" v-if="!r.activated">Para comisionar necesitas una activación vigente: mensual S/ 50 o anual S/ 500.</p>

      <!-- período actual -->
      <h4 class="rk-title">Período actual de dos ciclos</h4>
      <p class="rk-sub">
        Desde {{ fmt(r.period.start) }}<span v-if="r.period.end"> hasta {{ fmt(r.period.end) }}</span>
        · Ciclo {{ r.period.cycle }} de {{ r.period.of }}
      </p>

      <div class="rk-cycles">
        <div class="rk-cycle" v-for="c in r.period.cycles" :key="c.number" :class="c.status">
          <span class="rk-label">Ciclo {{ c.number }} · {{ cycleStatus(c.status) }}</span>
          <span>{{ c.personal }} personales · {{ c.group }} grupales</span>
        </div>
      </div>

      <div class="rk-stats">
        <div class="rk-stat">
          <span class="rk-label">Ventas personales del período</span>
          <strong>{{ r.period.personal }}</strong>
        </div>
        <div class="rk-stat">
          <span class="rk-label">Ventas grupales del período</span>
          <strong>{{ r.period.group }}</strong>
        </div>
      </div>

      <!-- metas -->
      <div class="rk-rank" v-for="rank in r.ranks" :key="rank.key" :class="rank.status">
        <div class="rk-rank-top">
          <strong>{{ rank.name }}</strong>
          <span class="rk-badge" :class="rank.status">{{ rankStatus(rank.status) }}</span>
        </div>
        <p class="rk-sub">
          Meta: {{ rank.goal }} ventas {{ rank.type == 'personal' ? 'personales' : 'grupales' }} en el período ·
          Premio: {{ rank.prize }}
        </p>
        <div class="rk-bar"><div :style="{ width: rank.percent + '%' }"></div></div>
        <p class="rk-sub">
          {{ rank.progress }} de {{ rank.goal }} ({{ rank.percent }} %)
          <span v-if="rank.missing"> · faltan {{ rank.missing }}</span>
        </p>
        <p class="rk-sub ok" v-if="rank.achieved_at">Alcanzado el {{ fmt(rank.achieved_at) }} · período {{ rank.achieved_period }}</p>
        <a class="rk-link" @click="open = open == rank.key ? null : rank.key">Ver requisitos</a>
        <ul class="rk-req" v-if="open == rank.key">
          <li>{{ rank.goal }} ventas {{ rank.type == 'personal' ? 'personales' : 'grupales (niveles 1 al 7)' }} de membresías aprobadas, sumando los dos ciclos del período.</li>
          <li>Al terminar el período el contador vuelve a 0; el rango alcanzado se conserva.</li>
          <li v-if="rank.note">{{ rank.note }}</li>
        </ul>
      </div>

      <!-- histórico -->
      <h4 class="rk-title">Histórico general · Desde tu registro</h4>
      <div class="rk-stats">
        <div class="rk-stat">
          <span class="rk-label">Ventas personales históricas</span>
          <strong>{{ r.history.personal }}</strong>
        </div>
        <div class="rk-stat">
          <span class="rk-label">Ventas grupales históricas</span>
          <strong>{{ r.history.group }}</strong>
        </div>
      </div>
      <p class="rk-hint">Incluye el período actual. Las ventas de períodos anteriores no suman para los premios actuales.</p>

      <h4 class="rk-title" v-if="r.periods.length">Historial de períodos</h4>
      <div class="rk-cycle" v-for="p in r.periods" :key="p.number">
        <span class="rk-label">Período {{ p.number }} · {{ fmt(p.start) }} – {{ fmt(p.end) }}</span>
        <span v-for="c in p.cycles" :key="c.number">Ciclo {{ c.number }}: {{ c.personal }} personales · {{ c.group }} grupales</span>
        <span>Total: {{ p.personal }} personales · {{ p.group }} grupales</span>
        <span class="ok" v-if="p.achieved.length">Logros: {{ p.achieved.join(', ') }}</span>
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
      loading: true,
      r: null,
      open: null,
    }
  },
  computed: {
    session() { return this.$store.state.session },
    title() { return 'Bonificaciones' },
  },
  methods: {
    fmt(d) {
      if (!d) return '—'
      return new Date(d).toLocaleDateString('es-PE', { day: 'numeric', month: 'short', year: 'numeric', timeZone: 'America/Lima' })
    },
    cycleStatus(s) {
      return { closed: 'finalizado', current: 'en curso', next: 'próximo' }[s]
    },
    rankStatus(s) {
      return { achieved: 'Alcanzado', next: 'Próxima meta', pending: 'Pendiente' }[s]
    },
  },
  async created() {
    const { data } = await api.bonuses(this.session)
    this.r = data.ranks
    this.loading = false
  },
}
</script>

<style scoped>
  .rk { max-width: 720px; padding-bottom: 24px; }
  .rk-head, .rk-stats { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
  .rk-chip, .rk-stat, .rk-cycle, .rk-rank {
    background: #fff; border-radius: 12px; padding: 14px 16px;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.08);
    display: flex; flex-direction: column; gap: 4px;
  }
  .rk-stat strong { font-size: 26px; color: #0b3558; }
  .rk-label { font-size: 12px; color: #6b7a8c; text-transform: uppercase; letter-spacing: .03em; }
  .rk-title { margin: 24px 0 4px; }
  .rk-sub { margin: 2px 0; font-size: 13px; color: #44515f; }
  .rk-hint { font-size: 12px; color: #6b7a8c; margin: 8px 0 0; }
  .rk-cycles { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin: 10px 0 12px; }
  .rk-cycle { margin-bottom: 10px; font-size: 14px; }
  .rk-cycle.current { border-left: 4px solid #2e7d32; }
  .rk-cycle.next { opacity: .6; }
  .rk-rank { margin-top: 12px; }
  .rk-rank.next { border: 2px solid #2e7d32; }
  .rk-rank-top { display: flex; justify-content: space-between; align-items: center; gap: 8px; }
  .rk-badge { font-size: 12px; padding: 2px 10px; border-radius: 999px; background: #eef1f5; color: #44515f; white-space: nowrap; }
  .rk-badge.achieved { background: #e3f4e5; color: #2e7d32; }
  .rk-badge.next { background: #2e7d32; color: #fff; }
  .rk-bar { height: 8px; background: #eef1f5; border-radius: 999px; overflow: hidden; margin: 6px 0 2px; }
  .rk-bar div { height: 100%; background: #2e7d32; }
  .rk-link { font-size: 13px; color: #0b3558; text-decoration: underline; cursor: pointer; align-self: flex-start; }
  .rk-req { margin: 4px 0 0 18px; font-size: 13px; color: #44515f; }
  .ok { color: #2e7d32; }
  .off { color: #c0392b; }
  @media (max-width: 520px) {
    .rk-head, .rk-stats, .rk-cycles { grid-template-columns: 1fr; }
  }
</style>
