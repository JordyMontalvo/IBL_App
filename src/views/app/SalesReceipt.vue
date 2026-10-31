<template>
  <App :session="session" :title="title">
    <div class="receipt-page">
      <div class="receipt-actions">
        <router-link to="/sales-history" class="back">
          <i class="fas fa-arrow-left"></i> Volver al historial
        </router-link>
        <div class="buttons" v-if="receipt">
          <button type="button" class="btn pdf" @click="print">
            <i class="fas fa-file-pdf"></i> Descargar PDF
          </button>
          <a class="btn whatsapp" :href="whatsappUrl" target="_blank" rel="noopener">
            <i class="fab fa-whatsapp"></i> Compartir por WhatsApp
          </a>
        </div>
      </div>

      <i class="load" v-if="loading"></i>
      <p class="error" v-if="error">{{ error }}</p>

      <div class="receipt" v-if="receipt">
        <header class="receipt-head">
          <img class="logo" src="@/assets/img/logo/logoibl.png" alt="IBL Inmobiliaria" />
          <h1>COMPROBANTE DE COMPRA</h1>
          <p>No válido como comprobante fiscal</p>
        </header>

        <div class="grid">
          <div class="cell">
            <small>Fecha</small>
            <strong>{{ receipt.date | date }}</strong>
          </div>
          <div class="cell">
            <small>Hora</small>
            <strong>{{ receipt.date | time }}</strong>
          </div>
          <div class="cell">
            <small>Comprobante N.°</small>
            <strong>{{ voucherNumber }}</strong>
          </div>
          <div class="cell">
            <small>Orden de compra</small>
            <strong>{{ orderNumber }}</strong>
          </div>
        </div>

        <section>
          <h3><i class="fas fa-user"></i> DATOS DEL CLIENTE</h3>
          <dl>
            <dt>Nombre:</dt>
            <dd>{{ receipt.buyer.name || '—' }}</dd>
            <dt>DNI:</dt>
            <dd>{{ receipt.buyer.dni || '—' }}</dd>
          </dl>
        </section>

        <section>
          <h3><i class="fas fa-file-alt"></i> DETALLE DE LA MEMBRESÍA</h3>
          <table>
            <thead>
              <tr>
                <th>Membresía</th>
                <th class="center">Cantidad</th>
                <th class="center">Precio total</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td>{{ receipt.name }}</td>
                <td class="center">1</td>
                <td class="center">S/ {{ money(receipt.price) }}</td>
              </tr>
            </tbody>
          </table>

          <div class="totals">
            <div><span>Subtotal sin IGV</span><strong>S/ {{ money(subtotal) }}</strong></div>
            <div><span>IGV</span><strong>S/ {{ money(igv) }}</strong></div>
            <div class="total"><span>TOTAL</span><strong>S/ {{ money(receipt.price) }}</strong></div>
          </div>
        </section>

        <div class="payment">
          <div>
            <small>MÉTODO DE PAGO</small>
            <strong>{{ payMethod }}</strong>
          </div>
          <div>
            <small>TOTAL PAGADO</small>
            <strong class="paid">S/ {{ money(receipt.price) }}</strong>
          </div>
        </div>

        <p class="thanks"><i class="fas fa-check-circle"></i> ¡Gracias por su compra!</p>
      </div>
    </div>
  </App>
</template>

<script>
import App from '@/views/layouts/App'
import api from '@/api'

const IGV = 1.18
const TZ = 'America/Lima'

export default {
  components: {
    App,
  },
  data() {
    return {
      receipt: null,
      loading: true,
      error: null,
    }
  },
  computed: {
    session() { return this.$store.state.session },
    title()   { return 'Comprobante de compra' },
    subtotal() { return Math.round((Number(this.receipt.price) / IGV) * 100) / 100 },
    igv()      { return Math.round((Number(this.receipt.price) - this.subtotal) * 100) / 100 },
    voucherNumber() { return 'IBL-' + String(this.receipt.number).padStart(6, '0') },
    orderNumber()   { return 'ORD-' + String(this.receipt.number).padStart(6, '0') },
    payMethod() {
      if (this.receipt.pay_method === 'bank') return 'Transferencia bancaria'
      if (this.receipt.pay_method === 'cash') return 'Efectivo'
      return 'Saldo del monedero'
    },
    whatsappUrl() {
      const r = this.receipt
      const text = [
        '*IBL Inmobiliaria — Comprobante de compra*',
        `Comprobante: ${this.voucherNumber}`,
        `Fecha: ${this.$options.filters.date(r.date)} ${this.$options.filters.time(r.date)}`,
        `Cliente: ${r.buyer.name || '—'} (DNI ${r.buyer.dni || '—'})`,
        `Membresía: ${r.name}`,
        `Total pagado: S/ ${this.money(r.price)}`,
        '¡Gracias por su compra!',
      ].join('\n')
      return 'https://wa.me/?text=' + encodeURIComponent(text)
    },
  },
  filters: {
    date(val) {
      return new Date(val).toLocaleDateString('es-PE', { timeZone: TZ, day: '2-digit', month: '2-digit', year: 'numeric' })
    },
    time(val) {
      return new Date(val).toLocaleTimeString('es-PE', { timeZone: TZ, hour: '2-digit', minute: '2-digit' })
    },
  },
  methods: {
    money(val) {
      return Number(val || 0).toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 })
    },
    print() {
      window.print()
    },
  },
  async created() {
    try {
      const { data } = await api.Sales.RECEIPT(this.session, this.$route.params.id)

      if (data.error && data.msg == 'invalid session') return this.$router.push('/login')
      if (data.error) return this.error = data.msg

      this.receipt = data.receipt
    } catch (e) {
      this.error = 'No se pudo cargar el comprobante. Intenta nuevamente.'
    } finally {
      this.loading = false
    }
  },
}
</script>

<style lang="stylus">
  .receipt-page
    --navy #10284a
    --blue #0d4fa1
    --green #1f8a3b
    color var(--navy)

    .receipt-actions
      display flex
      align-items center
      justify-content center
      flex-wrap wrap
      gap 16px
      position relative
      margin-bottom 20px

      .back
        position absolute
        left 0
        color #1c6fd1
        font-weight 600

      .buttons
        display flex
        gap 12px
        flex-wrap wrap
        justify-content center

      .btn
        display inline-flex
        align-items center
        gap 10px
        border 0
        border-radius 8px
        padding 12px 26px
        color white
        font-weight 700
        font-size 16px
        cursor pointer
        text-decoration none

        &.pdf
          background #1c6fd1
        &.whatsapp
          background var(--green)

    .error
      color #c62828
      text-align center

    .receipt
      background white
      max-width 620px
      margin 0 auto
      padding 24px 26px 28px
      border-radius 14px
      box-shadow 0 4px 16px rgba(16, 40, 74, 0.08)

      .receipt-head
        text-align center
        border-bottom 2px solid var(--green)
        padding-bottom 10px

        .logo
          height 80px

        h1
          margin 4px 0 0
          font-size 26px
          font-weight 800
          color var(--navy)

        p
          margin 2px 0 0
          color #5d6b7c

      .grid
        display grid
        grid-template-columns 1fr 1fr
        border-bottom 1px solid #e2e9f1

        .cell
          padding 12px 6px
          small
            display block
            color #5d6b7c
          strong
            display block
            font-size 17px

          &:nth-child(even)
            border-left 1px solid #e2e9f1
            padding-left 22px

      section
        border-bottom 1px solid #e2e9f1
        padding 12px 0

        h3
          margin 0 0 8px
          font-size 17px
          font-weight 800
          color var(--navy)

          i
            margin-right 10px

        dl
          display grid
          grid-template-columns 110px 1fr
          row-gap 4px
          margin 0

          dt
            color #5d6b7c
            font-weight 400
          dd
            margin 0
            font-weight 700

        table
          width 100%
          border-collapse collapse
          border 1px solid #e2e9f1

          th
            background var(--blue)
            color white
            padding 10px
            text-align left

          td
            padding 10px
            text-align left
            border 1px solid #e2e9f1

          .center
            text-align center

        .totals
          width 60%
          margin 10px 0 0 auto

          div
            display flex
            justify-content space-between
            padding 3px 4px

          .total
            margin-top 6px
            padding-top 8px
            border-top 1px solid #e2e9f1
            font-size 20px
            font-weight 800

            strong
              color var(--green)

      .payment
        display grid
        grid-template-columns 1fr 1fr
        background #eaf2fb
        border-radius 10px
        padding 14px 18px
        margin-top 14px

        small
          display block
          color #5d6b7c
          font-size 12px
        strong
          font-size 17px

        div + div
          border-left 1px solid #c9d6e6
          padding-left 22px

        .paid
          font-size 19px

      .thanks
        text-align center
        font-size 20px
        font-weight 800
        margin 18px 0 0

        i
          color var(--green)
          margin-right 10px

  @media print
    body *
      visibility hidden
    .receipt, .receipt *
      visibility visible
    .receipt
      position absolute
      left 0
      top 0
      width 100%
      box-shadow none
</style>
