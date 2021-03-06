<template>
  <section class="projects main">
    <h1 v-text="$t('SECTIONS.SERIES_LIST')"></h1>
    <ProjectsFigure v-for="project in projects" :key="project.id" class="project" :project="project" ></ProjectsFigure>
  </section>
</template>

<script>
import { PROJECT_STATUS, PROJECT_PUBLISH_STATUS, } from '../../api/config'
import { get, uniqBy, } from 'lodash'
import { isScrollBarReachBottom, } from '../util/comm'
import ProjectsFigure from '../components/projects/ProjectsFigure.vue'


const DEFAULT_PAGE = 1
const DEFAULT_SORT = 'project_order,-updated_at'
const MAX_RESULT = 3

const fetchFollowing = (store, params) => {
  return store.dispatch('GET_FOLLOWING_BY_RESOURCE', {
    mode: params.mode,
    ids: params.ids,
    resource: params.resource,
  })
}

const fetchProjectsList = (store, {
  max_result = MAX_RESULT,
  page = DEFAULT_PAGE,
  sort = DEFAULT_SORT,
} = {}) => {
  return store.dispatch('GET_PUBLIC_PROJECTS', {
    params: {
      max_result: max_result,
      page: page,
      sort: sort,
      where: {
        status: [ PROJECT_STATUS.DONE, PROJECT_STATUS.WIP, ],
        publish_status: PROJECT_PUBLISH_STATUS.PUBLISHED,
      },
    },
  })
}

const getUserFollowing = (store, { id = get(store, 'state.profile.id'), resource, resourceType = '', } = {}) => {
  return store.dispatch('GET_FOLLOWING_BY_USER', {
    id: id,
    resource: resource,
    resource_type: resourceType,
  })
}

export default {
  name: 'PublicProjects',
  asyncData ({ store, }) {
    return fetchProjectsList(store)
  },
  metaInfo () {
    return {
      description: this.$i18n ? this.$t('OG.DESCRIPTION') : 'Readr',
      ogTitle: this.$i18n ? this.$t('OG.PROJECT_LIST') : 'Readr',
      title: this.$i18n ? this.$t('OG.PROJECT_LIST') : 'Readr',
      metaUrl: this.$route.path,
    }
  },    
  components: {
    ProjectsFigure,
  },
  data () {
    return {
      currentPage: DEFAULT_PAGE,
      hasMore: true,
      loading: false,
    }
  },
  computed: {
    projects () {
      return get(this.$store, 'state.publicProjects.normal', []) || []
    },
    projectsTagIds () {
      return uniqBy(this.projects.map(project => project.tags).filter(tags => tags).reduce((all, tags) => all.concat(tags), []), 'id').map(tag => tag.id)
    },
  },
  watch: {
    projectsTagIds (ids) {
      fetchFollowing(this.$store, { ids: ids, resource: 'tag', })
    },
  },
  beforeMount () {    
    getUserFollowing(this.$store, { resource: 'project', })
    getUserFollowing(this.$store, { resource: 'tag', })
    getUserFollowing(this.$store, { resource: 'project', })
  },  
  mounted () {
    window.addEventListener('scroll', this.$_projects_loadmoreHandler)
  },
  beforeDestroy () {
    window.removeEventListener('scroll', this.$_projects_loadmoreHandler)
  },
  methods: {
    $_projects_loadmoreHandler () {
      if (this.hasMore && !this.loading && isScrollBarReachBottom(1/3)) {
        const origCount = get(this.projects, [ 'length', ], 0)
        this.loading = true
        fetchProjectsList(this.$store, { page: this.currentPage + 1, })
        .then((res) => {
          this.currentPage += 1
          get(this.projects, [ 'length', ], 0) <= origCount ? this.hasMore = false : true
          this.loading = false
          if (this.hasMore && this.$store.state.isLoggedIn) {
            const projectIds = res.map(project => project.id)
            fetchFollowing(this.$store, {
              mode: 'update',
              ids: projectIds,
              resource: 'project',
            })
          }
        })
      }
    },
  },
}
</script>

<style lang="stylus" scoped>
.projects
  padding 40px 20px 20px
  h1
    margin 20px 0 0
    font-size 1.125rem
  .project
    margin-top 10px
    & + .project
      margin-top 20px
</style>

