<template>
  <div class="search">
    <div class="search__container">
      <main class="search__container__main">
        <SearchFilter :filter.sync="currFilter" />
        <div v-if="currFilter === 'post'">
          <HomeArticleMain v-for="(post, k) in items" :articleData="post" :key="`post-${k}`" />
        </div>
        <div v-else-if="currFilter === 'project'">
          <ProjectsFigure v-for="project in items" :project="project" :key="project.id" />
        </div>
      </main>
    </div>
  </div>
</template>
<script>
  import { get, } from 'lodash'
  import HomeArticleMain from 'src/components/home/HomeArticleMain.vue'
  import ProjectsFigure from 'src/components/projects/ProjectsFigure.vue'
  import SearchFilter from 'src/components/search/SearchFilter.vue'

  const MAXRESULT = 12
  const PAGE = 1
  const debug = require('debug')('CLIENT:views:Search')

  const fetchSearch = (store, { keyword, params, }) => {
    return store.dispatch('SEARCH', {
      keyword,
      params,
    })
  }
  const fetchData = (store, route, objectType) => {
    debug('store.state.route.params', route.params)
    debug('store.state.route.params.keyword', route.params.keyword)
    return Promise.all([
      fetchSearch(store, {
        keyword: route.params.keyword,
        params: {
          page: PAGE,
          max_results: MAXRESULT,
          object_type: objectType || 'post',
        },
      }),
    ])
  }

  export default {
    name: 'Search',
    components: {
      HomeArticleMain,
      ProjectsFigure,
      SearchFilter,
    },
    computed: {
      items () {
        return get(this.$store, 'getters.searchResultNormalized')
      },
    },
    data () {
      return {
        currFilter: 'post',
        isProcessing: false,
      }
    },
    methods: {
      fetchItems () {
        if (!this.isProcessing) {
          this.isProcessing = true
          fetchData(this.$store, this.$route, this.currFilter).then(() => {
            this.isProcessing = false
          }).catch(() => {
            this.isProcessing = false
          })
        }
      },
      searchChange (key) {
        debug('Filter changes to:', key)
        this.currFilter = key
      },
    },
    beforeMount () {
      fetchData(this.$store, this.$route)
    },    
    watch: {
      currFilter () {
        this.fetchItems()
      },
      '$route.fullPath': function () {
        /**
         * Revise this.currFilter to trigger data fetching.
         */
        this.currFilter = 'post'
        this.fetchItems()
      },
    },
  }
</script>
<style lang="stylus" scoped>
  .search
    min-height 100vh
    &__container
      max-width 1200px
      margin auto
      padding 25px 0
      display flex
      &__aside
        width 75px
        height 100%
        position sticky
        top 60px
        z-index 999
      &__main
        display flex
        flex-direction column
        justify-content flex-start
        align-items flex-start
</style>