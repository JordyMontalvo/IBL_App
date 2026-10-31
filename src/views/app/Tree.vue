<template>
  <App :session="session" :office_id="office_id" :title="title">

    <h4>EQUIPO</h4>
    <p class="tree-hint" v-if="!loading">Explora tu red y despliega más niveles con el botón +.</p>

    <i class="load" v-if="loading"></i>

    <div v-if="!loading">

      <div id="body">

        <div class="tree-container">
          <i v-if="node.parent && node.id != id" class="fas fa-arrow-left" @click="GET(node.parent)" style="position: absolute; right: 0; margin-right: 80px; z-index: 1;"></i>
          <ul class="tree">
            <tree-node :key="node.id" :node="node" :depth="0" @select="click" />
          </ul>
        </div>
      </div>

      <p class="tree-hint">Pulsa + para mostrar el siguiente nivel de esa rama.</p>

      <div class="modal" :class="{ open }" @click="closed">
        <div class="inner" @click.stop="">
          <img class="photo" :src="selec_node.photo">
          <p style="text-align: center;">{{ selec_node.country }}</p>
          <br>
          <p>ID: {{ selec_node.dni }}</p>
          <p>Nombre: {{ selec_node.name }}</p>
          <p>Apellido: {{ selec_node.lastName }}</p>
          <p>Teléfono: {{ selec_node.phone }}</p>
          <p>Correo: {{ selec_node.email }}</p>
          <p>Rango Cerrado: {{ selec_node._rank }}</p>
          <!-- <p>Rango Actual:  {{ selec_node.rank | _rank }}</p> -->
          <!-- <p>Siguiente Rango: {{ selec_node.next_rank.name | _rank }}</p> -->
          <!-- <p>Rango Próximo: Hans Evanglelista</p> -->
          <p>Puntos Personales: {{ selec_node.points }}</p>
          <p>Puntos Grupales: {{ selec_node.closed_points_arr }}</p>
          <br>
          <p>Últimas compras:</p>
          <br>
          <table>
            <thead>
              <tr>
                <th>Fecha</th><th>Monto</th><th>Puntos</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="activation in selec_node.last_activations.slice().reverse()">
                <td>{{ activation.date | date }}</td>
                <td>{{ activation.price }}</td>
                <td>{{ activation.points }}</td>
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
import TreeNode from '@/views/app/TreeNode'
import api from '@/api'

export default {
  components: {
    App,
    TreeNode,
  },
  data() {
    return {
      loading: true,
      id:   null,
      node: null,
      count: 0,
      selec_node: {},
      open: false,
    }
  },
  computed: {
    session()   { return this.$store.state.session },
    office_id() { return this.$store.state.office_id },
    name()      { return this.$store.state.name },
    activated() { return this.$store.state.activated },
    title() { return 'Equipo comercial' },
  },

  filters: {
    date(val) {
      return new Date(val).toLocaleDateString()
      // return new Date(val).toLocaleString()
    },
    _rank(val) {
      if(val == 'none')              return 'Ninguno'
      if(val == 'active')            return 'ACTIVO'
      if(val == 'star')              return 'MASTER'
      if(val == 'master')            return 'PLATA'
      if(val == 'silver')            return 'PLATINO'
      if(val == 'gold')              return 'ORO'
      if(val == 'sapphire')          return 'ZAFIRO'
      if(val == 'RUBI')              return 'DIAMANTE RUBI'
      if(val == 'DIAMANTE')          return 'DIAMANTE ESTRELLA'
      if(val == 'DOBLE DIAMANTE')    return 'DIAMANTE DOS ESTRELLAS'
      if(val == 'TRIPLE DIAMANTE')   return 'DIAMANTE TRES ESTRELLAS'
      if(val == 'DIAMANTE ESTRELLA') return 'DIAMANTE CBM'
    },
  },
  async created() {
    // GET data
    const { data } = await api.tree(this.session, null); console.log({ data })

    this.loading = false

    // error
    if(data.error && data.msg == 'invalid session') this.$router.push('/login')
    if(data.error && data.msg == 'unverified user') this.$router.push('/verify')

    // success
    this.$store.commit('SET_NAME',       data.name)
    this.$store.commit('SET_LAST_NAME',  data.lastName)
    this.$store.commit('SET_AFFILIATED', data.affiliated)
    this.$store.commit('SET__ACTIVATED',  data._activated)
    this.$store.commit('SET_ACTIVATED',  data.activated)
    this.$store.commit('SET_PLAN',       data.plan)
    this.$store.commit('SET_COUNTRY',    data.country)
    this.$store.commit('SET_PHOTO',      data.photo)
    this.$store.commit('SET_TREE',       data.tree)

    this.id   = data.id
    // this.nodes = data.nodes
    this.node = data.node
    this.selec_node = data.node
  },
  methods: {
    click(node) {
      this.count += 1
      setTimeout(() => {
        if(this.count == 1) {
          console.log('click ...')
          this.selec_node = node
          this.open = true
        } else {
          console.log('double click ...')
          this.GET(node.id)
        }
        this.count = 0
      }, 300)
    },
    async GET(id) {
      console.log('GET ... ', id)
      this.loading = true

      // GET data by id
      const { data } = await api.tree(this.session, id); console.log({ data })

      this.loading = false

      // success
      // this.nodes = data.nodes
      this.node = data.node
    },

    closed() {
      console.log('closed ...')
      this.open = false
    },
  }
};
</script>

<style lang="stylus">
  .tree-hint
    color #5b6b82
    font-size 14px
    margin 6px 0 12px

  .modal
    background rgba(#000, 0.72)
    position absolute
    top 0
    bottom 0
    left 0
    right 0
    padding 80px 20px
    display none
    z-index 2
    overflow auto
    &.open
      display block
    .inner
      background #eaebec
      border-radius 20px
      padding 20px 20px 32px 20px
      max-width 480px
      margin auto

</style>

<style>
/* https://codepen.io/ea-ger/pen/rNJjxYr
https://codepen.io/team/amcharts/pen/poPxojR */

/*  @import url('https://fonts.googleapis.com/css2?family=Oxygen:wght@300&display=swap');*/

  #body {
    margin: 0;
    padding: 0;
/*    background: #fafafa;*/
/*    font-family: 'Oxygen', sans-serif;*/
    letter-spacing: 0.2px;
/*    height: 100vh;*/
/*    width: 100vw;*/
/*    background-color: var(--bg-1);*/
    position: relative;
  }

  :root {
    --col-1: #c8ddef;
    --col-2: #c8ddef;
    --bg-1: #0e182d;
    --highlighted: #ff5722;
  }

  .tree-container {
    /*display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;*/

    overflow: auto;

    width: 100%;
    /*padding-top: 5em;
    padding-bottom: 5em;*/
  }

  .tree-container>h1 {
    color: var(--col-1);
    font-weight: 400;
  }

  .tree,
  .tree ul,
  .tree li {
    list-style: none;
    margin: 0;
    padding: 0;
    position: relative;
  }

  .tree {
/*    margin: 0 0 1em;*/
    text-align: center;
  }

  .tree,
  .tree ul {
    display: table;
    margin: auto;
    width: 100%;
  }

  .tree ul {
    width: 100%;
  }

  .tree li {
    display: table-cell;
    padding: .5em 0;
    vertical-align: top;
  }

  .tree li:before {
    outline: solid 1px var(--col-2);
    content: "";
    left: 0;
    position: absolute;
    right: 0;
    top: 0;
    direction: rtl;
  }

  /*.tree li:hover:before {
    outline: solid 1px var(--col-1);
  }*/

  .tree li:first-child:before {
    left: 50%;
  }

  .tree li:last-child:before {
    right: 50%;
  }

  .tree code,
  .tree span {
/*    border: solid .1em var(--col-1);*/
    border-radius: .2em;
    display: inline-block;
    margin: 0 .2em .5em;
    padding: .2em .5em;
    position: relative;
/*    background-color: var(--bg-1);*/
    transition: all 0.2s ease;
/*    color: var(--col-1);*/
/*    font-size: 14px;*/
    font-size: 12px;
  }

  .tree span i {
    font-size: 32px;
    color: #fff;
  }
  .tree span i.aff{
    color: #ffe400;
  }
  .tree span i.act{
    color: #14ec42;
  }

  .tree span i.fa-gem {
    font-size: 18px;
    position: absolute;
    transform: translateY(10px);
    display: none;
  }

  .tree span i.fa-gem.star {
    display: inline;
    color: #ffe400; /*yellow*/
  }

  .tree span i.fa-gem.master {
    display: inline;
    color: #14ec42; /*green*/
  }

  .tree span i.fa-gem.silver {
    display: inline;
    color: #D3D3D3; /*silver*/
  }

  .tree span i.fa-gem.gold {
    display: inline;
    color: #D4AF37; /*gold*/
  }

  /*.tree span:hover {
    background-color: var(--col-1);
    color: var(--bg-1);
  }

  .tree li:hover>span {
    background-color: var(--col-1);
    color: var(--bg-1);
  }

  .tree span:hover::after,
  .tree li:hover>span::after {
    box-shadow: 0 0 5px 8px var(--col-1) inset;
  }*/

  .tree ul:before,
  .tree code:before,
  .tree span:before {
    outline: solid 1px var(--col-2);
    content: "";
    height: .5em;
    left: 50%;
    position: absolute;
  }

/*  .tree ul:hover:before,
  .tree code:hover:before,
  .tree li:hover>span:before {
    outline: solid 1px var(--col-1);
  }*/

  .tree span::after {
    outline: solid 1px var(--col-1);
    content: "";
    top: -8px;
    left: calc(50% - 5px);
    width: 8px;
    height: 8px;
/*    background-color: var(--bg-1);*/
    background-color: #888;
    border: 1px solid var(--col-1);
    position: absolute;
    opacity: 1;
    border-radius: 100%;
    transition: all 0.2s ease;
  }

  .tree ul:before {
    top: -.5em;
  }

  .tree code:before,
  .tree span:before {
    top: -.55em;
  }

  .tree>li {
    margin-top: 0;
  }

  .tree>li:before,
  .tree>li:after,
  .tree>li>code:before,
  .tree>li>span:before,
  .tree>li>span:after {
    outline: none;
    display: none;

  }

  .highlighted {
    border: 2px solid var(--highlighted) !important;
  }

  /*.highlighted:hover {
    background-color: var(--highlighted) !important;
  }*/
</style>

