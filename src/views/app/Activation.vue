<template>
  <App :session="session" :office_id="office_id" :title="title">
    <h4 class="tabs" style="margin-bottom: 25px">
      <router-link class="tab" to="/activation"> Comprar </router-link>
      &nbsp;&nbsp;
      <router-link class="tab" to="/activations" v-if="!office_id">
        Historial
      </router-link>
    </h4>

    <i class="load" v-if="loading"></i>

    <section v-if="!loading">
      <div class="flex">
        <div class="flex-activacion">
          <div style="width: 100%">
            <div style="display: flex; gap: 10px">
              <button
                class="_tabs"
                v-for="(category, i) in categories"
                @click="tab = category"
                :class="{
                  selected: tab == categories[i],
                  'not-selected': tab != categories[i],
                }"
              >
                {{ category }}
              </button>
            </div>
            <div
              class="_tab_content"
              style="color: rgba(8, 56, 92, 1)"
              v-for="(category, i) in categories"
              v-show="tab == categories[i]"
            >
              <article
                class="product"
                v-for="(product, i) in products"
                @click="touch(i)"
                v-if="product.type == category"
              >
                <small>
                  <p class="_name">{{ product.name }}</p>
                  <span
                    style="
                      color: rgba(8, 56, 92, 1);
                      font-size: 14px;
                      font-weight: bold;
                    "
                    >S/. {{ formatNumber(product.price) }}</span
                  >&nbsp; &nbsp;
                  <span style="color: #aaa3a3; font-weight: bold"
                    >PTS {{ formatNumber(product.points) }}</span
                  >
                  <!-- <span>Val. a comisionar
                  <i v-if="product.val">{{ product.val   }}</i>
                  <i v-else>            {{ product.price }}</i>
                  &nbsp; &nbsp; <i class="_price">$ {{ product.price }}</i></span> -->
                </small>

                <div class="control" style="color: green">
                  <i class="fas fa-minus-square" @click="less(product)"></i>
                  <input v-model="product.total" readonly />
                  <i class="fas fa-plus-square" @click="more(product)"></i>
                </div>
              </article>
            </div>
            <br />

            <div>
              <p style="color: rgba(8, 56, 92, 1); font-size: 18px">
                {{ activeCategory }}
              </p>
              <img
                :src="product.img"
                style="margin: 12px 0; width: 370px; height: 170px"
              />

              <!-- <div class="flex"> -->
              <div>
                <textarea
                  readonly
                  style="
                    width: 390px;
                    height: 100px;
                    border-radius: 20px;
                    padding: 10px;
                  "
                  v-model="product.description"
                ></textarea>
              </div>
            </div>
          </div>

          <div>
            <p style="color: rgba(74, 176, 46, 1); font-size: 24px">
              Datos Bancarios
            </p>

            <small style="color: rgba(8, 56, 92, 1)">Impuesto: {{ IGV }}</small>
            <br />

            <div class="input-wrapper">
              <i class="fa-solid fa-money-bill-wave icon"></i>
              <input class="input" readonly v-model="_price" /> <br />
            </div>
            <div class="input-wrapper">
              <i class="fa-solid fa-hand-holding-usd icon"></i>
              <input class="input" readonly v-model="_points" /> <br />
            </div>
            <div class="input-wrapper">
              <select class="input" v-model="office" v-if="!pending">
                <i class="fas fa-building icon"></i>
                <option value="null" disabled>Oficina</option>
                <option v-for="office in offices" :value="office">
                  {{ office.name }}
                </option>
              </select>
              <br />
            </div>
            <small v-if="office">{{ office.address }}</small> <br />
            <div class="input-wrapper">
              <div v-if="office">
                <textarea
                  readonly
                  class="input"
                  style="width: 390px; height: 200px; border-radius: 20px"
                  rows="5"
                  >{{ office.accounts }}</textarea
                >
                <br /><br />
              </div>
            </div>
            <label>
              <input type="checkbox" v-model="check" />
              <small style="color: rgba(8, 56, 92, 1)"
                >Deseo usar mi saldo</small
              >
            </label>
            <br />

            <div v-show="check">
              <small style="color: rgba(8, 56, 92, 1)"
                >Saldo no disponible: {{ formatNumber(_balance) }}</small
              >
              <br />
              <small style="color: rgba(8, 56, 92, 1)"
                >Saldo disponible: {{ formatNumber(balance) }}</small
              >
              <br />
              <small v-if="remaining > 0"
                >restan: {{ formatNumber(remaining) }}</small
              >
            </div>

            <br />

            <div v-show="!(check && remaining == 0)">
              <small style="color: rgba(74, 176, 46, 1); font-size: 24px"
                >Medio de Pago</small
              >
              <br />

              <div v-if="check">
                <small v-if="plan == 'default'"
                  >Cash: {{ formatNumber(balance + selec_plan.pay) }}</small
                >
                <small v-else>
                  Cash: {{ formatNumber(_balance + balance) }}</small
                >

                <br />
              </div>

              <label>
                <input type="radio" :value="'bank'" v-model="pay_method" />
                <small style="color: rgba(8, 56, 92, 1)">Banco</small>
              </label>
              <br />

              <label>
                <input type="radio" :value="'cash'" v-model="pay_method" />
                <small style="color: rgba(8, 56, 92, 1)">Efectivo</small> <br />
              </label>

              <div v-if="pay_method == 'bank'">
                <br />
                <small>Monto: {{ formatNumber(remaining) }}</small> <br />

                <input class="input" v-model="bank" placeholder="Banco" />
                <br />
                <input
                  class="input"
                  v-model="date"
                  placeholder="Fecha"
                  type="date"
                />
                <br />
                <input
                  class="input"
                  v-model="voucher_number"
                  placeholder="Número de Voucher"
                  oninput="this.value=this.value.replace(/(?![0-9])./gmi,'')"
                />
                <br />

                <label>
                  <span class="input" v-show="!voucher"
                    >Comprobante de pago</span
                  >

                  <img class="voucher" :src="voucher" />

                  <input type="file" @change="onFileChange" />
                </label>
              </div>

              <div v-if="pay_method == 'cash'">
                <br />
                <small>Monto: {{ formatNumber(remaining) }}</small> <br />
              </div>
            </div>

            <small v-if="error" style="color: red"
              ><br />
              {{ error }}</small
            >
            <small v-if="success" class="success">Activación Enviada</small>
            <br />

            <button class="button" v-show="!sending" @click="POST">
              Ordenar productos
            </button>
            <button class="button" v-show="sending" disabled>
              Enviando orden ...
            </button>
          </div>

          <br /><br /><br />
        </div>
      </div>
    </section>
  </App>
</template>

<script>
import App from "@/views/layouts/App";
import api from "@/api";
import lib from "@/lib";

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
      //  office:   null,
      check: false,
      voucher: null,

      error: null,

      file: null,
      office: null,
      offices: null,

      loading: true,
      sending: false,
      success: false,

      pending: false,

      tab: null,

      pay_method: null,

      bank: null,
      date: null,
      voucher_number: null,
    };
  },
  computed: {
    session() {
      return this.$store.state.session;
    },
    office_id() {
      return this.$store.state.office_id;
    },
    title() {
      return "Productos";
    },

    price() {
      console.log("price");
      let price = this.products.reduce((a, b) => a + b.price * b.total, 0);
      return price;
    },
    points() {
      return this.products.reduce((a, b) => a + b.points * b.total, 0);
    },
    // commission() { return this.products.reduce((a, b) => a + (b.val ? b.val : b.price) * b.total, 0) },
    total() {
      return this.products.reduce((a, b) => a + b.total, 0);
    },

    _price() {
      return `Total: S/. ${this.price}`;
    },
    // _points() { return `A comisionar: ${this.commission}` },
    _points() {
      return `Puntos: ${this.points}`;
    },

    IGV() {
      return (this.price - this.price / 1.18).toFixed(2);
    },

    remaining() {
      if (this.check) {
        let ret = this.price - (this.balance + this._balance);

        return ret > 0 ? ret : 0;
      } else {
        return this.price;
      }
    },

    categories() {
      const arr = this.products.map(function (x) {
        return x.type;
      });

      let ret = arr.filter(function (v, i, self) {
        return i == self.indexOf(v);
      });

      return ret;
    },

    activeCategory() {
      return this.tab
        ? this.tab.charAt(0).toUpperCase() + this.tab.slice(1).toLowerCase()
        : "Terreno";
    },
  },
  async created() {
    // GET data
    const { data } = await api.Activation.GET(this.session);
    console.log({ data });

    this.loading = false;

    // error
    if (data.error && data.msg == "invalid session")
      this.$router.push("/login");

    // success
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
    this.products = data.products.map((a) => ({ ...a, total: 0 }));
    this.product = this.products[0];

    this.balance = data.balance;
    this._balance = data._balance;

    if (this.office_id) this.office = this.office_id;

    this.offices = data.offices;

    this.tab = this.categories[0];
  },
  methods: {
    touch(i) {
      this.product = this.products[i];
    },

    more(product) {
      const isActivated =
        this.$store.state.activated || this.$store.state._activated;

      // Restablecer la cantidad del producto previamente seleccionado
      if (this.product && this.product !== product) {
        this.product.total = 0; // Reiniciar el total del producto anterior
      }

      // Verificar si el usuario está activado
      /*if (isActivated) {
      // Para usuarios activados, permitir agregar productos de cualquier tipo
      if (product.total >= 1) return;
      if(product.type === "ACTIVACIÓN" && product.total >= 1) return; 
    } else {
      // Solo permitir agregar productos de tipo 'activación'
      if (product.total >= 1) return; 
    }*/

      if (this.total >= 1) return;
      product.total += 1;
      this.product = product; // Actualizar el producto seleccionado
    },

    less(product) {
      if (product.total == 0) return;
      product.total -= 1;
    },
    onFileChange(e) {
      this.file = e.target.files[0];

      const reader = new FileReader();
      reader.onload = (e) => {
        this.voucher = e.target.result;
      };

      reader.readAsDataURL(this.file);
    },
    reset() {
      console.log("reset ...");

      this.products.forEach((product) => {
        product.total = 0;
      });
    },
    async POST() {
      let {
        products,
        office,
        check,
        voucher,
        pay_method,
        bank,
        date,
        voucher_number,
      } = this;

      if (pay_method == "bank") {
        if (!bank) {
          this.error = "Nombre de banco";
          return;
        }
        if (!date) {
          this.error = "Fecha de voucher";
          return;
        }
        if (!voucher_number) {
          this.error = "Número de voucher";
          return;
        }
        if (!voucher) {
          this.error = "Voucher de pago";
          return;
        }
      }

      if (!this.total) {
        this.error = "Seleccione productos";
        return;
      }

      if (!office) {
        this.error = "Seleccione oficina";
        return;
      }

      if (!check && !pay_method) {
        this.error = "Seleccione Medio de Pago";
        return;
      }

      if (check && this.remaining && !pay_method) {
        this.error = "Seleccione Medio de Pago";
        return;
      }

      this.error = null;

      // POST Affiliation
      this.sending = true;

      if (voucher)
        voucher = await lib.upload(this.file, this.file.name, "activations");

      const { data } = await api.Activation.POST(this.session, {
        products,
        voucher,
        office: office.id,
        check,
        pay_method,
        bank,
        date,
        voucher_number,
      });

      this.sending = false;

      this.success = true;

      this.reset();
    },
    formatNumber(value) {
      return value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ","); // Formatea el número con comas
    },
  },
};
</script>

<style lang="stylus">
.product
  small
    width 240px
    font-weight 300
  ._name
    font-weight 600
    font-size 12.5px

._light
  font-weight 300
  font-size 12.5px

._price
  font-weight 600

.not-selected {
  opacity: 0.5;
}
</style>
