<template>
  <div class="leading">
    <template v-if="route === 'series'">
      <slot name="tagNav"></slot>
      <ProjectIntroSingle :projId="Number(get(this.$route, 'params.id'))"></ProjectIntroSingle>
    </template>
  </div>
</template>
<script>
import ProjectIntroSingle from 'src/components/projects/ProjectIntroSingle.vue'
import { get, } from 'lodash'

export default {
  name: 'Leading',
  components: {
    ProjectIntroSingle,
  },
  computed: {
    route () {
      return this.$route.fullPath.split('/')[ 1 ]
    },
  },
  methods: {
    get,
  },
  mounted () {},
}
</script>
<style lang="stylus" scoped>
.leading
  width 100%
  margin-bottom 15px
  margin-top 20px
</style>