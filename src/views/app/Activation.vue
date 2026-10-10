<template>
  <App :session="session" :office_id="office_id" :title="title" :crumbs="crumbs">
    <i class="load" v-if="loading"></i>

    <div v-if="notification" class="custom-notification">
      {{ notification }}
    </div>

    <!-- Pop-up Modal de Éxito -->
    <div class="custom-modal-overlay" v-if="showSuccessModal">
      <div class="custom-modal-content">
        <i class="fas fa-check-circle success-icon"></i>
        <h3>¡Gracias por su compra!</h3>
        <p>La operación se ha procesado correctamente.</p>
        <div class="custom-modal-actions">
          <button @click="goToHome" class="btn-primary">Ir al inicio</button>
          <button @click="closeModal" class="btn-secondary">Nueva operación</button>
        </div>
      </div>
    </div>

    <section v-if="!loading" class="ibl-shop">
      <div class="mobile-page-head">
        <h1>{{ title }}</h1>
        <p>Productos › {{ title }}</p>
      </div>

      <div class="shop-layout">
        <div class="shop-main">
          <div class="hero" :class="{ 'hero--membership': mode === 'membership' }">
            <div class="hero-copy">
              <h2>{{ mode === 'membership' ? 'Vende una membresía' : 'Activa tu participación' }}</h2>
              <p v-if="mode === 'activation'">
                Para comercializar las membresías de IBL y acceder al plan de compensación, debes mantener una activación vigente. Esta activación te da acceso al modelo comercial.
              </p>
              <p v-else>
                Brinda acceso a un club exclusivo y genera comisiones por cada venta.
              </p>
            </div>
            <div class="hero-points" v-if="mode === 'activation'">
              <div class="hero-point"><i class="fas fa-chart-bar"></i><span>Comercializa membresías</span></div>
              <div class="hero-point"><i class="fas fa-users"></i><span>Construye tu equipo</span></div>
              <div class="hero-point"><i class="fas fa-trophy"></i><span>Accede a comisiones y beneficios</span></div>
            </div>
            <div class="hero-points" v-else>
              <div class="hero-point"><i class="fas fa-star"></i><span>Experiencias exclusivas</span></div>
              <div class="hero-point"><i class="fas fa-users"></i><span>Más bienestar y calidad de vida</span></div>
              <div class="hero-point"><i class="fas fa-chart-line"></i><span>Oportunidad de crecimiento</span></div>
            </div>
          </div>

          <div class="panel" id="planes" v-if="mode === 'activation'">
            <div class="panel-head">
              <h3><i class="fas fa-bolt"></i> Elige tu activación</h3>
              <router-link class="history-link" to="/activations" v-if="!office_id">Historial</router-link>
            </div>

            <div class="plan-grid" v-if="visibleProducts.length">
              <article
                class="plan-card"
                v-for="prod in visibleProducts"
                :key="prod.id"
                :class="{ selected: isSelected(prod) }"
                @click="selectProduct(prod)"
              >
                <span class="best-badge" v-if="isConvenient(prod)">Más conveniente</span>
                <div class="plan-icon"><i class="fas fa-calendar-alt"></i></div>
                <p class="plan-kicker">Activación</p>
                <h4>{{ prod.name }}</h4>
                <p class="plan-blurb">{{ planBlurb(prod) }}</p>
                <p class="plan-price">S/ {{ money(prod.price) }}</p>
                <ul class="feature-list">
                  <li v-for="(item, idx) in featuresFor(prod)" :key="idx">
                    <i class="fas fa-check-circle"></i>
                    <span>{{ item }}</span>
                  </li>
                </ul>
                <button type="button" class="pick-btn">
                  <span class="pick-dot"></span>
                  Seleccionar
                </button>
              </article>
            </div>
            <p v-else class="empty-plans">No hay activaciones disponibles.</p>

            <div class="note">
              <i class="fas fa-info-circle"></i>
              <div>
                <strong>Importante:</strong>
                <ul>
                  <li>La activación te habilita para comercializar membresías y participar en el plan.</li>
                  <li>El pago de activación no genera comisión para el patrocinador.</li>
                </ul>
              </div>
            </div>
          </div>

          <template v-else>
            <div class="panel" id="planes">
              <div class="panel-head">
                <h3><span class="step-no">1</span> Selecciona la membresía</h3>
              </div>
              <div class="member-grid" v-if="visibleProducts.length">
                <article
                  class="member-card"
                  v-for="(prod, i) in visibleProducts"
                  :key="prod.id"
                  :class="{ selected: isSelected(prod) }"
                  @click="selectProduct(prod)"
                >
                  <div class="member-photo" :style="{ backgroundImage: 'url(' + cardImage(prod, i) + ')' }">
                    <span class="photo-radio"></span>
                  </div>
                  <p class="member-kicker">Membresía</p>
                  <h4>{{ shortName(prod) }}</h4>
                  <p class="member-blurb">{{ tagline(prod) }}</p>
                  <p class="member-price">S/ {{ money(prod.price) }}</p>
                  <ul class="feature-list">
                    <li v-for="(item, idx) in featuresFor(prod)" :key="idx">
                      <i class="fas fa-check-circle"></i>
                      <span>{{ item }}</span>
                    </li>
                  </ul>
                </article>
              </div>
              <p v-else class="empty-plans">No hay membresías disponibles.</p>
            </div>

            <div class="panel" id="comprador">
              <div class="panel-head">
                <h3><span class="step-no">2</span> Datos del comprador</h3>
              </div>
              <div class="buyer-grid">
                <div class="field">
                  <label>DNI <span class="req">*</span></label>
                  <input v-model="buyerData.dni" placeholder="DNI" inputmode="numeric" @input="onlyDigits('dni')" />
                </div>
                <div class="field">
                  <label>Correo</label>
                  <input v-model="buyerData.email" placeholder="Correo" />
                </div>
                <div class="field">
                  <label>Nombres y Apellidos <span class="req">*</span></label>
                  <input v-model="buyerData.name" placeholder="Nombres y Apellidos" @input="onlyLetters" />
                </div>
                <div class="field">
                  <label>Dirección</label>
                  <input v-model="buyerData.address" placeholder="Dirección" />
                </div>
                <div class="field">
                  <label>Celular <span class="req">*</span></label>
                  <input v-model="buyerData.phone" placeholder="Celular" inputmode="numeric" @input="onlyDigits('phone')" />
                </div>
              </div>
            </div>
          </template>
        </div>

        <aside class="shop-side">
          <div class="summary-card">
            <h3>{{ mode === 'membership' ? 'Resumen de venta' : 'Resumen de compra' }}</h3>

            <div class="summary-product" v-if="product">
              <div v-if="mode === 'activation'" class="cal"><i class="fas fa-calendar-alt"></i></div>
              <div
                v-else
                class="summary-thumb"
                :style="{ backgroundImage: 'url(' + cardImage(product, selectedIndex) + ')' }"
              ></div>
              <div>
                <strong>{{ mode === 'membership' ? 'MEMBRESÍA ' + shortName(product) : product.name }}</strong>
                <small v-if="mode === 'activation'">Vigencia: {{ vigenciaText(product) }}</small>
                <small v-else>Valor comercial</small>
                <div v-if="mode === 'membership'">
                  <button type="button" class="linkish" @click="scrollTo('planes')">Cambiar</button>
                </div>
              </div>
              <div class="summary-price">S/ {{ money(price) }}</div>
            </div>
            <p v-else class="empty-plans">Selecciona un producto.</p>

            <div class="total-line" v-if="mode === 'activation'">
              <span>Total a pagar</span>
              <span>S/ {{ money(price) }}</span>
            </div>

            <div class="buyer-preview" v-if="mode === 'membership'">
              <header>
                <h4>Datos del comprador</h4>
                <button type="button" class="linkish" @click="scrollTo('comprador')">Editar</button>
              </header>
              <p><i class="fas fa-user"></i> {{ buyerData.name || 'Nombres y Apellidos' }}</p>
              <p><i class="fas fa-id-card"></i> DNI: {{ buyerData.dni || '—' }}</p>
              <p><i class="fas fa-phone"></i> Celular: {{ buyerData.phone || '—' }}</p>
              <p><i class="fas fa-envelope"></i> Correo: {{ buyerData.email || '—' }}</p>
              <p><i class="fas fa-map-marker-alt"></i> Dirección: {{ buyerData.address || '—' }}</p>
            </div>

            <div class="pay-block">
              <div class="pay-head">
                <h4>Método de pago</h4>
              </div>

              <button type="button" class="pay-option" :class="{ active: payChoice === 'balance' }" @click="setPay('balance')">
                <i class="fas fa-wallet method"></i>
                <span class="grow">
                  <strong>Deseo usar mi saldo</strong>
                  <small>Saldo disponible: S/ {{ money(balance) }}</small>
                </span>
                <span class="dot"></span>
              </button>
              <button type="button" class="pay-option" :class="{ active: payChoice === 'bank' }" @click="setPay('bank')">
                <i class="fas fa-university method"></i>
                <span class="grow"><strong>Transferencia / Depósito bancario</strong></span>
                <span class="dot"></span>
              </button>
              <button type="button" class="pay-option" :class="{ active: payChoice === 'cash' }" @click="setPay('cash')">
                <i class="fas fa-money-bill-wave method"></i>
                <span class="grow"><strong>Efectivo</strong></span>
                <span class="dot"></span>
              </button>

              <small v-if="payChoice === 'balance' && remaining > 0" class="pay-hint">
                El saldo no cubre el total. Restan S/ {{ money(remaining) }}. Elige transferencia o efectivo.
              </small>

              <div v-if="payChoice === 'bank'" class="pay-extra">
                <input v-model="bank" placeholder="Banco" />
                <input v-model="date" type="date" />
                <input v-model="voucher_number" placeholder="Número de operación / voucher" @input="onlyVoucher($event, 'voucher_number')" />
                <div class="file-label">
                  <img class="voucher-preview" :src="voucher" v-if="voucher" />
                  <span>{{ voucher ? 'Cambiar comprobante' : 'Comprobante de pago' }}</span>
                  <small>JPG o PNG</small>
                  <input type="file" accept=".jpg,.jpeg,.png,image/jpeg,image/png" @change="onFileChange($event, 1)" />
                </div>
                <input
                  v-if="voucher2"
                  v-model="voucher_number2"
                  placeholder="Número de operación del segundo comprobante"
                  @input="onlyVoucher($event, 'voucher_number2')"
                />
                <div class="file-label">
                  <img class="voucher-preview" :src="voucher2" v-if="voucher2" />
                  <span>{{ voucher2 ? 'Cambiar segundo comprobante' : 'Segundo comprobante de pago (opcional)' }}</span>
                  <small>JPG o PNG</small>
                  <input type="file" accept=".jpg,.jpeg,.png,image/jpeg,image/png" @change="onFileChange($event, 2)" />
                </div>
                <small v-if="office && office.accounts">{{ office.accounts }}</small>
              </div>

              <div class="pay-box">
                <h4>Resumen de pago</h4>
                <div class="row">
                  <span>{{ mode === 'membership' ? 'Precio de venta' : 'Subtotal' }}</span>
                  <span>S/ {{ money(price) }}</span>
                </div>
                <div class="row">
                  <span>Usar saldo</span>
                  <span>- S/ {{ money(appliedBalance) }}</span>
                </div>
                <div class="row strong">
                  <span>Total a pagar</span>
                  <span>S/ {{ money(remaining) }}</span>
                </div>
              </div>

              <small v-if="error" class="error-message">{{ error }}</small>
              <small v-if="success" class="success-message">
                {{ mode === 'membership' ? 'Venta enviada' : 'Activación enviada' }}
              </small>

              <button class="confirm-btn" v-show="!sending" @click="POST">
                <i class="fas fa-lock"></i>
                {{ mode === 'membership' ? 'Confirmar venta de membresía' : 'Confirmar compra' }}
              </button>
              <button class="confirm-btn" v-show="sending" disabled>Enviando orden ...</button>
            </div>
          </div>
        </aside>
      </div>
    </section>
  </App>
</template>

<script>
import App from "@/views/layouts/App";
import api from "@/api";
import lib from "@/lib";

const FALLBACK_PHOTOS = [
  "https://images.unsplash.com/photo-1507525428034-b723cf961d3e?auto=format&fit=crop&w=900&q=70",
  "https://images.unsplash.com/photo-1613490493576-7fde63acd811?auto=format&fit=crop&w=900&q=70",
];

export default {
  components: {
    App,
  },
  data() {
    return {
      current_points: null,
      current_profit: null,
      products: null,
      product: null,
      balance: null,
      _balance: null,
      check: true,
      voucher: null,
      voucher2: null,
      error: null,
      file: null,
      file2: null,
      office: null,
      offices: null,
      loading: true,
      sending: false,
      success: false,
      pending: false,
      tab: null,
      pay_method: null,
      payChoice: "balance",
      bank: null,
      date: null,
      voucher_number: null,
      voucher_number2: null,
      notification: null,
      showSuccessModal: false,
      buyerData: {
        dni: "",
        name: "",
        email: "",
        phone: "",
        address: "",
      },
      adjustmentReason: "Promoción",
    };
  },
  computed: {
    session() {
      return this.$store.state.session;
    },
    office_id() {
      return this.$store.state.office_id;
    },
    mode() {
      return this.$route.path === "/membership" ? "membership" : "activation";
    },
    title() {
      return this.mode === "membership" ? "Venta de Membresías" : "Activaciones";
    },
    crumbs() {
      return ["Productos", this.title];
    },
    visibleProducts() {
      if (!this.products) return [];
      return this.products.filter((prod) =>
        this.mode === "activation" ? this.isActivationProduct(prod) : this.isMembershipProduct(prod)
      );
    },
    selectedIndex() {
      const idx = this.visibleProducts.findIndex((prod) => this.product && prod.id === this.product.id);
      return idx < 0 ? 0 : idx;
    },
    price() {
      if (!this.products) return 0;
      return this.products.reduce((sum, item) => sum + item.price * item.total, 0);
    },
    points() {
      if (!this.products) return 0;
      return this.products.reduce((sum, item) => sum + item.points * item.total, 0);
    },
    total() {
      if (!this.products) return 0;
      return this.products.reduce((sum, item) => sum + item.total, 0);
    },
    remaining() {
      const price = Number(this.price) || 0;
      if (!this.check) return price;
      const pool = Number(this.balance || 0) + Number(this._balance || 0);
      const left = price - pool;
      return left > 0 ? left : 0;
    },
    appliedBalance() {
      if (!this.check) return 0;
      const price = Number(this.price) || 0;
      const pool = Number(this.balance || 0) + Number(this._balance || 0);
      return Math.min(Math.max(pool, 0), price);
    },
  },
  watch: {
    "$route.path": function () {
      this.error = null;
      this.success = false;
      this.showSuccessModal = false;
      this.buyerData = { dni: "", name: "", email: "", phone: "", address: "" };
      if (this.products) this.selectDefault();
    },
  },
  async created() {
    const { data } = await api.Activation.GET(this.session);

    this.loading = false;

    if (data.error && data.msg == "invalid session") this.$router.push("/login");

    this.$store.commit("SET_NAME", data.name);
    this.$store.commit("SET_LAST_NAME", data.lastName);
    this.$store.commit("SET_AFFILIATED", data.affiliated);
    this.$store.commit("SET_ACTIVATED", data.activated);
    this.$store.commit("SET__ACTIVATED", data._activated);
    this.$store.commit("SET_PLAN", data.plan);
    this.$store.commit("SET_COUNTRY", data.country);
    this.$store.commit("SET_PHOTO", data.photo);
    this.$store.commit("SET_TREE", data.tree);

    this.current_points = data.points;
    this.current_profit = data.profit;
    this.products = data.products.map((item) => ({ ...item, total: 0 }));
    this.balance = data.balance;
    this._balance = data._balance;
    this.offices = data.offices;

    if (this.office_id && this.offices) {
      const match = this.offices.find((item) => item.id == this.office_id);
      this.office = match || null;
    }

    this.selectDefault();
  },
  methods: {
    norm(value) {
      return String(value || "")
        .toUpperCase()
        .normalize("NFD")
        .replace(/[\u0300-\u036f]/g, "");
    },
    isActivationProduct(prod) {
      return this.norm(prod.type).includes("ACTIV");
    },
    isMembershipProduct(prod) {
      return this.norm(prod.type).includes("MEMBRE");
    },
    isSelected(prod) {
      return !!(this.product && this.product.id === prod.id && prod.total > 0);
    },
    isConvenient(prod) {
      return this.mode === "activation" && this.norm(prod.name).includes("ANUAL");
    },
    vigenciaText(prod) {
      if (prod.duration) return prod.duration;
      const name = this.norm(prod.name);
      if (name.includes("ANUAL")) return "12 cierres";
      if (name.includes("MENSUAL")) return "1 cierre";
      return "vigente";
    },
    planBlurb(prod) {
      const vigencia = this.vigenciaText(prod);
      if (vigencia === "vigente") return "Mantén tu acceso vigente.";
      return "Mantén tu acceso vigente por " + vigencia + ".";
    },
    shortName(prod) {
      return String(prod.name || "")
        .replace(/membres[ií]a/gi, "")
        .replace(/activaci[oó]n/gi, "")
        .trim();
    },
    tagline(prod) {
      const name = this.norm(prod.name);
      if (name.includes("VIP")) return "Experiencia premium del club";
      return "Acceso a los beneficios del club";
    },
    featuresFor(prod) {
      if (this.isActivationProduct(prod)) {
        const items = [
          "Vigencia: " + this.vigenciaText(prod),
          "Habilita la venta de membresías",
          "Acceso al plan de compensación",
          "Soporte en la plataforma",
        ];
        if (this.isConvenient(prod)) items.push("Mejor relación costo – beneficio");
        return items;
      }

      if (this.norm(prod.name).includes("VIP")) {
        return [
          "Todos los beneficios Estándar",
          "Beneficios premium",
          "Experiencias exclusivas",
          "Atención preferencial",
          "Válida a nivel nacional",
        ];
      }

      return [
        "Acceso al club",
        "Programas y actividades",
        "Beneficios exclusivos",
        "Válida a nivel nacional",
      ];
    },
    cardImage(prod, index) {
      if (prod && prod.img) return prod.img;
      return FALLBACK_PHOTOS[index % FALLBACK_PHOTOS.length];
    },
    selectDefault() {
      const list = this.visibleProducts;
      if (!list.length) {
        this.product = null;
        this.tab = null;
        return;
      }

      let chosen = list[0];
      if (this.mode === "activation") {
        chosen = list.find((prod) => this.norm(prod.name).includes("ANUAL")) || list[0];
      } else {
        chosen =
          list.find((prod) => {
            const name = this.norm(prod.name);
            return name.includes("ESTANDAR") || name.includes("STANDARD");
          }) || list.slice().sort((a, b) => Number(a.price) - Number(b.price))[0];
      }

      this.products.forEach((prod) => {
        prod.total = 0;
      });
      if (!(this.mode === "activation" && this.isAlreadyActivated(chosen))) chosen.total = 1;
      this.product = chosen;
      this.tab = chosen.type;
    },
    selectProduct(product) {
      if (this.mode === "activation" && this.isAlreadyActivated(product)) {
        this.showNotification("Ya posees una activación vigente.");
        return;
      }

      this.products.forEach((prod) => {
        prod.total = 0;
      });
      product.total = 1;
      this.product = product;
      this.tab = product.type;
      this.success = false;
      this.error = null;
    },
    setPay(choice) {
      this.payChoice = choice;
      this.error = null;
      if (choice === "balance") {
        this.check = true;
        this.pay_method = null;
        return;
      }
      this.check = false;
      this.pay_method = choice === "bank" ? "bank" : "cash";
    },
    onlyDigits(field) {
      this.buyerData[field] = String(this.buyerData[field] || "").replace(/[^0-9]/g, "");
    },
    onlyLetters(event) {
      this.buyerData.name = event.target.value.replace(/[0-9]/g, "");
    },
    onlyVoucher(event, field) {
      this[field] = event.target.value.replace(/(?![0-9])./gmi, "");
    },
    scrollTo(id) {
      const el = document.getElementById(id);
      if (el) el.scrollIntoView({ behavior: "smooth", block: "start" });
    },
    money(value) {
      const amount = Number(value) || 0;
      const digits = Number.isInteger(amount) ? 0 : 2;
      return amount.toLocaleString("en-US", {
        minimumFractionDigits: digits,
        maximumFractionDigits: digits,
      });
    },
    onFileChange(e, slot) {
      const selected = e.target.files[0];
      if (!selected) return;

      const name = (selected.name || "").toLowerCase();
      const type = (selected.type || "").toLowerCase();
      const extOk = [".jpg", ".jpeg", ".png"].some((ext) => name.endsWith(ext));
      const typeOk = ["image/jpeg", "image/jpg", "image/png", "image/pjpeg"].includes(type);
      if (!extOk && !typeOk) {
        this.error = "El comprobante debe ser JPG o PNG";
        e.target.value = "";
        return;
      }
      this.error = null;

      const reader = new FileReader();
      reader.onload = (event) => {
        if (slot === 2) this.voucher2 = event.target.result;
        else this.voucher = event.target.result;
      };
      reader.readAsDataURL(selected);

      if (slot === 2) this.file2 = selected;
      else this.file = selected;
    },
    reset() {
      if (this.products) {
        this.products.forEach((product) => {
          product.total = 0;
        });
      }
      this.buyerData = {
        dni: "",
        name: "",
        email: "",
        phone: "",
        address: "",
      };
      this.bank = null;
      this.date = null;
      this.voucher_number = null;
      this.voucher_number2 = null;
      this.voucher = null;
      this.voucher2 = null;
      this.file = null;
      this.file2 = null;
      this.office = null;
      this.setPay("balance");
      this.error = null;
      this.success = false;
      this.selectDefault();
    },
    goToHome() {
      this.showSuccessModal = false;
      this.$router.push('/dashboard');
    },
    closeModal() {
      this.showSuccessModal = false;
    },
    async POST() {
      let { products, office, check, voucher, voucher2, pay_method, bank, date, voucher_number, voucher_number2 } = this;

      if (this.mode === "membership") {
        if (!this.buyerData.dni) return (this.error = "Ingrese DNI del cliente");
        if (!this.buyerData.name) return (this.error = "Ingrese Nombres y Apellidos del cliente");
        if (!this.buyerData.phone) return (this.error = "Ingrese Celular del cliente");
        if (!this.buyerData.email) return (this.error = "Ingrese Correo del cliente");
        if (!this.buyerData.address) return (this.error = "Ingrese Dirección del cliente");
      }

      if (pay_method == "bank") {
        if (!bank) return (this.error = "Nombre de banco");
        if (!date) return (this.error = "Fecha de voucher");
        if (!voucher_number) return (this.error = "Número de voucher");
        if (!voucher) return (this.error = "Voucher de pago");
        if (voucher2 && !voucher_number2) {
          return (this.error = "Ingresa el número de operación del segundo comprobante");
        }
        if (voucher2 && String(voucher_number) === String(voucher_number2)) {
          return (this.error = "Los dos comprobantes no pueden tener el mismo número de operación");
        }
      }

      if (!this.total) return (this.error = "Seleccione productos");

      if (this.payChoice === "balance" && this.remaining > 0) {
        this.error = "El saldo no cubre el total. Elige transferencia o efectivo.";
        return;
      }

      if (!check && !pay_method) return (this.error = "Seleccione Medio de Pago");

      this.error = null;
      this.sending = true;

      const selectedProduct = products.find((item) => item.total > 0);
      const isSale =
        selectedProduct &&
        (selectedProduct.type === "TERRENO" ||
          this.norm(selectedProduct.type).includes("MEMBRE"));

      if (voucher) voucher = await lib.upload(this.file, this.file.name, "activations");
      if (voucher2) voucher2 = await lib.upload(this.file2, this.file2.name, "activations");

      let response;
      const payload = {
        products,
        voucher,
        voucher2,
        office: office ? office.id : null,
        check,
        pay_method,
        bank,
        date,
        voucher_number,
        voucher_number2: voucher2 ? voucher_number2 : null,
        buyerData: this.buyerData,
      };

      if (isSale) response = await api.Sales.POST(this.session, payload);
      else response = await api.Activation.POST(this.session, payload);

      this.sending = false;

      if (response && response.data && response.data.error) {
        this.error = response.data.msg || response.data.error;
        return;
      }

      this.reset();
      this.showSuccessModal = true;
    },
    isAlreadyActivated(prod) {
      if (!this.isActivationProduct(prod)) return false;
      const name = this.norm(prod.name);
      const isMembership = name.includes("MEMBRESIA") || name.includes("CLUB");
      if (isMembership && this.$store.state._activated) return true;
      if (!isMembership && this.$store.state.activated) return true;
      return false;
    },
    showNotification(msg) {
      this.notification = msg;
      setTimeout(() => {
        this.notification = null;
      }, 3000);
    },
  },
};
</script>

<style lang="stylus">
.custom-notification
  position fixed
  top 20px
  right 20px
  background #e74c3c
  color white
  padding 15px 25px
  border-radius 8px
  box-shadow 0 4px 6px rgba(0,0,0,0.1)
  z-index 1000
  font-weight 500

.custom-modal-overlay
  position fixed
  top 0
  left 0
  right 0
  bottom 0
  background rgba(0, 0, 0, 0.6)
  display flex
  align-items center
  justify-content center
  z-index 2000

.custom-modal-content
  background white
  padding 30px 40px
  border-radius 12px
  text-align center
  max-width 400px
  width 90%
  box-shadow 0 10px 25px rgba(0,0,0,0.2)

  h3
    margin 15px 0 10px
    font-size 22px
    color #2c3e50

  p
    color #7f8c8d
    margin-bottom 25px

  .success-icon
    font-size 60px
    color #2ecc71

  .custom-modal-actions
    display flex
    gap 15px
    justify-content center

    button
      padding 10px 20px
      border none
      border-radius 6px
      cursor pointer
      font-weight 600
      transition all 0.2s

    .btn-primary
      background #08385c
      color white
      &:hover
        background #062b47

    .btn-secondary
      background #ecf0f1
      color #2c3e50
      &:hover
        background #bdc3c7
</style>

<style src="@/assets/style/activation-catalog.css"></style>
