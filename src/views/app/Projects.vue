<template>
  <App :session="session" :office_id="office_id" :title="title">

    <i class="load" v-if="loading"></i>

    <div v-if="!loading">

      <!-- Visor de mapa -->
      <div v-if="selected" class="project-viewer">
        <a class="back" @click="selected = null">
          <i class="fas fa-arrow-left"></i> Volver a proyectos
        </a>
        <h2 class="project-title">{{ selected.name }}</h2>
        <p class="project-location"><i class="fas fa-map-marker-alt"></i> {{ selected.location }}</p>
        <div class="viewer-box">
          <iframe class="pdf-frame" :src="pdfUrl(selected)" :title="selected.name"></iframe>
        </div>
      </div>

      <!-- Listado -->
      <div v-else>
        <h2 class="project-title">Proyectos</h2>
        <p class="project-subtitle">Consulta los proyectos disponibles y sus mapas.</p>

        <div class="project-list">
          <div class="project-item" v-for="project in projects" :key="project.id">
            <div>
              <h3 class="project-name">{{ project.name }}</h3>
              <p class="project-location"><i class="fas fa-map-marker-alt"></i> {{ project.location }}</p>
            </div>
            <button class="load-map" @click="selected = project">
              <i class="fas fa-file-alt"></i> Cargar mapa
            </button>
          </div>
          <p class="empty" v-if="!projects.length">Aún no hay proyectos disponibles.</p>
        </div>
      </div>

    </div>
  </App>
</template>

<script>
import App from '@/views/layouts/App'
import api from '@/api'

const SERVER = process.env.VUE_APP_SERVER

export default {
  components: {
    App,
  },
  data() {
    return {
      loading: true,
      projects: [],
      selected: null,
    }
  },
  computed: {
    session()   { return this.$store.state.session },
    office_id() { return this.$store.state.office_id },
    title()     { return 'Proyectos' },
  },
  methods: {
    pdfUrl(project) {
      return `${SERVER}/api/app/projects?session=${this.session}&id=${project.id}`
    },
  },
  async created() {
    const { data } = await api.projects(this.session)

    this.loading = false

    // error
    if (data.error && data.msg == 'invalid session') return this.$router.push('/login')

    // success
    this.projects = data.projects || []
  },
}
</script>

<style lang="stylus">
  .project-title
    font-size 32px
    font-weight 700
    color #0d1f4d
    margin-bottom 6px

  .project-subtitle
    color #5b6b82
    margin-bottom 24px

  .project-location
    color #6b7a90
    margin 4px 0 0

    i
      margin-right 8px

  .project-list
    background white
    border-radius 12px
    padding 8px 28px
    box-shadow 0 2px 10px rgba(13, 31, 77, 0.06)

  .project-item
    display flex
    align-items center
    justify-content space-between
    gap 16px
    padding 20px 0
    border-bottom 1px solid #e8eef5

    &:last-child
      border-bottom 0

  .project-name
    font-size 20px
    font-weight 700
    color #0d1f4d
    margin 0

  .load-map
    background #1976f2
    color white
    border 0
    border-radius 8px
    padding 12px 22px
    font-weight 700
    cursor pointer
    white-space nowrap

    i
      margin-right 8px

    &:hover
      background #1263cc

  .back
    display inline-block
    color #1976f2
    cursor pointer
    margin-bottom 12px

  .viewer-box
    background white
    border-radius 12px
    padding 12px
    margin-top 20px
    box-shadow 0 2px 10px rgba(13, 31, 77, 0.06)

  .pdf-frame
    border 0
    width 100%
    height 75vh
    border-radius 6px

  .empty
    padding 24px 0
    text-align center
    color #6b7a90
</style>
