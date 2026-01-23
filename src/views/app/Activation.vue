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

    <div v-if="notification" class="custom-notification">
      {{ notification }}
    </div>

    <section v-if="!loading" class="catalog-container">
      <!-- Left Column: Product Catalog -->
      <div class="catalog-left">
        <h2 class="catalog-title">Catálogo de {{ activeCategory }}</h2>
        
        <!-- Filter Dropdown -->
        <div class="filter-dropdown-container">
          <select 
            class="filter-dropdown" 
            v-model="tab"
          >
            <option 
              v-for="(category, i) in categories" 
              :key="i"
              :value="category"
            >
              {{ category }}
            </option>
          </select>
        </div>

        <!-- Product Cards Section -->
        <div class="product-catalog-section">
          <h3 class="section-title">Datos del comprador</h3>
          
          <div class="product-cards">
            <div
              class="product-card"
              v-for="(prod, i) in products"
              v-if="prod.type == tab"
              :key="i"
              @click="selectProduct(prod)"
              :class="{ 'product-card-selected': product && product.id == prod.id }"
            >
              <div class="product-card-image">
                <img :src="prod.img || 'https://via.placeholder.com/150x100'" alt="Product" />
              </div>
              <div class="product-card-content">
                <h4 class="product-card-title">{{ prod.name }}</h4>
                <p class="product-card-subtitle">{{ prod.subtitle || prod.type }}</p>
                <span 
                  class="product-card-status"
                  :class="{
                    'status-disponible': prod.total == 0 && !isAlreadyActivated(prod),
                    'status-reservado': prod.total > 0,
                    'status-activado': isAlreadyActivated(prod)
                  }"
                  :style="isAlreadyActivated(prod) ? 'background: #27ae60; color: white;' : ''"
                >
                  {{ isAlreadyActivated(prod) ? 'Adquirido' : (prod.total > 0 ? 'Seleccionado' : 'Disponible') }}
                </span>
              </div>
              <div class="product-card-price">
                <div class="card-price-label">Precio</div>
                <div class="card-price-amount">S/ {{ formatNumber(prod.price) }}</div>
              </div>
            </div>
          </div>

          <!-- Product Info Section -->
          <div class="product-info-section" v-if="product">
            <h3 class="info-title">Información del Producto</h3>
            <div class="info-details">
              <!-- TERRENO specific info -->
              <template v-if="product.type == 'TERRENO'">
                <div class="info-row">
                  <span class="info-label">Área:</span>
                  <span class="info-value">{{ product.area }}</span>
                </div>
                <div class="info-row">
                  <span class="info-label">Ubicación:</span>
                  <span class="info-value">{{ product.location }}</span>
                </div>
              </template>

              <!-- ACTIVACIÓN specific info -->
              <template v-if="product.type == 'ACTIVACIÓN'">
                <div class="info-row">
                  <span class="info-label">Duración:</span>
                  <span class="info-value">{{ product.duration }}</span>
                </div>

              </template>

              <!-- MEMBRESÍA specific info -->
              <template v-if="product.type == 'MEMBRESÍA'">
                <div class="info-row">
                  <span class="info-label">Ubicación:</span>
                  <span class="info-value">{{ product.location }}</span>
                </div>
                <!-- <div class="info-row">
                  <span class="info-label">Beneficios:</span>
                  <span class="info-value">Acceso a Club y Eventos</span>
                </div> -->
              </template>

              <!-- Common info -->
              <div class="info-row">
                <span class="info-label">Tipo:</span>
                <span class="info-value">{{ product.type }}</span>
              </div>
              <div class="info-row">
                <span class="info-label">Puntos:</span>
                <span class="info-value">{{ formatNumber(product.points) }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Right Column: Purchase Form -->
      <div class="catalog-right">
        <h3 class="form-title">Venta de {{ tab || 'terreno' }}</h3>
        
        <!-- Buyer Data Section -->
        <div class="form-section" v-if="tab !== 'ACTIVACIÓN'">
          <label class="form-label">DNI</label>
          <input 
            type="text" 
            class="form-input" 
            placeholder="DNI"
            v-model="buyerData.dni"
          />

          <label class="form-label">Nombres y Apellidos</label>
          <input 
            type="text" 
            class="form-input" 
            placeholder="Nombres y Apellidos"
            v-model="buyerData.name"
          />
          
          <label class="form-label">Celular</label>
          <input 
            type="text" 
            class="form-input" 
            placeholder="Celular"
            v-model="buyerData.phone"
          />

          <label class="form-label">Correo</label>
          <input 
            type="text" 
            class="form-input" 
            placeholder="Correo"
            v-model="buyerData.email"
          />
          
          <label class="form-label">Dirección</label>
          <input 
            type="text" 
            class="form-input" 
            placeholder="Dirección"
            v-model="buyerData.address"
          />
        </div>

        <!-- Summary Section -->
        <div class="summary-section">
          <h3 class="summary-title">Resumen</h3>
          
          <div class="summary-row">
            <label class="summary-label">Precio de Venta</label>
            <span class="summary-price">S/ {{ formatNumber(price) }}</span>
          </div>


          <!-- Office Selection -->
          <div class="summary-row">
            <label class="summary-label">Oficina</label>
            <select class="form-select" v-model="office">
              <option value="null" disabled>Seleccione oficina</option>
              <option v-for="office in offices" :value="office">
                {{ office.name }}
              </option>
            </select>
          </div>

          <small v-if="office" class="office-address">{{ office.address }}</small>

          <!-- Bank Details -->
          <div v-if="office" class="bank-details">
            <textarea
              readonly
              class="form-textarea"
              rows="4"
              >{{ office.accounts }}</textarea>
          </div>

          <!-- Balance Checkbox -->
          <label class="checkbox-label">
            <input type="checkbox" v-model="check" />
            <span>Deseo usar mi saldo</span>
          </label>

          <div v-show="check" class="balance-info">
            <small>Saldo no disponible: {{ formatNumber(_balance) }}</small><br />
            <small>Saldo disponible: {{ formatNumber(balance) }}</small><br />
            <small v-if="remaining > 0">Restan: {{ formatNumber(remaining) }}</small>
          </div>

          <!-- Payment Method -->
          <div v-show="!(check && remaining == 0)" class="payment-section">
            <h4 class="payment-title">Medio de Pago</h4>
            
            <label class="radio-label">
              <input type="radio" :value="'bank'" v-model="pay_method" />
              <span>Banco</span>
            </label>
            
            <label class="radio-label">
              <input type="radio" :value="'cash'" v-model="pay_method" />
              <span>Efectivo</span>
            </label>

            <!-- Bank Payment Details -->
            <div v-if="pay_method == 'bank'" class="bank-payment">
              <small>Monto: {{ formatNumber(remaining) }}</small>
              <input class="form-input" v-model="bank" placeholder="Banco" />
              <input class="form-input" v-model="date" placeholder="Fecha" type="date" />
              <input 
                class="form-input" 
                v-model="voucher_number" 
                placeholder="Número de Voucher"
                oninput="this.value=this.value.replace(/(?![0-9])./gmi,'')"
              />
              
              <label class="file-label">
                <span v-show="!voucher">Comprobante de pago</span>
                <img class="voucher-preview" :src="voucher" v-if="voucher" />
                <input type="file" @change="onFileChange" />
              </label>
            </div>

            <div v-if="pay_method == 'cash'" class="cash-payment">
              <small>Monto: {{ formatNumber(remaining) }}</small>
            </div>
          </div>

          <!-- Error/Success Messages -->
          <small v-if="error" class="error-message">{{ error }}</small>
          <small v-if="success" class="success-message">Activación Enviada</small>

          <!-- Submit Button -->
          <button class="confirm-button" v-show="!sending" @click="POST">
            Confirmar venta
          </button>
          <button class="confirm-button" v-show="sending" disabled>
            Enviando orden ...
          </button>
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
      
      notification: null,
      
      // New fields for catalog design
      buyerData: {
        dni: '',
        name: '',
        email: '',
        phone: '',
        address: ''
      }
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
    
    selectProduct(product) {
      // Check for existing activation
      if (product.type === 'ACTIVACIÓN') {
         const name = (product.name || '').toUpperCase()
         const isMembership = name.includes('MEMBRESÍA') || name.includes('MEMBRESIA') || name.includes('CLUB')
         
         if (isMembership) {
            if (this.$store.state._activated) {
               this.showNotification('Ya posees una activación de Membresía activa.')
               return
            }
         } else {
            // Assume Lote/General
            if (this.$store.state.activated) {
               this.showNotification('Ya posees una activación de Lote activa.')
               return
            }
         }
      }

      // Si el producto ya está seleccionado, deseleccionarlo
      if (this.product && this.product.id === product.id && product.total > 0) {
        product.total = 0;
        this.product = null;
      } else {
        // Reset all products
        this.products.forEach(p => p.total = 0);
        // Select the clicked product
        product.total = 1;
        this.product = product;
      }
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

      // Find selected product to determine type
      const selectedProduct = products.find(p => p.total > 0)
      const isSale = selectedProduct && (
        selectedProduct.type === 'TERRENO' || 
        selectedProduct.type === 'MEMBRESÍA' || 
        selectedProduct.type === 'MEMBRESIA'
      )

      if (voucher)
        voucher = await lib.upload(this.file, this.file.name, "activations");

      let response;

      if (isSale) {
         response = await api.Sales.POST(this.session, {
            products, // Backend expects products array to find the selected one
            voucher,
            office: office.id,
            check,
            pay_method,
            bank,
            date,
            voucher_number,
            buyerData: this.buyerData // Send buyer data for sales
         });
      } else {
         response = await api.Activation.POST(this.session, {
            products,
            voucher,
            office: office.id,
            check,
            pay_method,
            bank,
            date,
            voucher_number,
          });
      }

      this.sending = false;

      // Check response error if any (assuming standard response format)
      if (response && response.data && response.data.error) {
          this.error = response.data.error;
          return;
      }

      this.success = true;

      this.reset();
    },
    formatNumber(value) {
      return value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ","); // Formatea el número con comas
    },
    isAlreadyActivated(prod) {
      if (prod.type !== 'ACTIVACIÓN') return false
      
      const name = (prod.name || '').toUpperCase()
      const isMembership = name.includes('MEMBRESÍA') || name.includes('MEMBRESIA') || name.includes('CLUB')

      if (isMembership && this.$store.state._activated) return true
      if (!isMembership && this.$store.state.activated) return true
      
      return false
    },
    
    showNotification(msg) {
      this.notification = msg
      setTimeout(() => {
        this.notification = null
      }, 3000)
    }
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

.custom-notification {
  position: fixed;
  top: 20px;
  right: 20px;
  background: #e74c3c;
  color: white;
  padding: 15px 25px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  z-index: 1000;
  animation: slideIn 0.3s ease;
  font-weight: 500;
}

@keyframes slideIn {
  from { transform: translateX(100%); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}
</style>

<style src="@/assets/style/activation-catalog.css"></style>
