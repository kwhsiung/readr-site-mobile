<template>
  <div class="list-container">
    <h1 class="list-container__date" v-text="date"></h1>
    <ol class="list-container__list list">
      <FollowingListItem
        class="list__list-item"
        v-for="item in data"
        :key="`${item.resource}-${getItemContent(item).id}`"
        :resource="item.resource"
        :id="getItemContent(item).id"
      >
        <!-- slot template in FollowingListItem for tag -->
        <router-link
          v-if="item.resource === 'tag'"
          class="tag"
          v-text="getItemContent(item).text"
          :to="`/tag/${getItemContent(item).id}`"
        >
        </router-link>
        <!-- slot template in FollowingListItem for project -->
        <div
          v-else-if="item.resource === 'project'"
          class="report"
        >
          <router-link class="report__hero-img" :to="`/series/${getItemContent(item).slug}`">
            <img :src="getItemContent(item).heroImage" alt="">
          </router-link>
          <div class="report__info">
            <div>
              <p class="report__category" v-text="$t('FOLLOWING.PROJECT')"></p>
              <router-link class="report__title" :to="`/series/${getItemContent(item).slug}`" v-text="getItemContent(item).title"></router-link>
            </div>
            <p class="report__tags" v-text="getTags(item)"></p>
          </div>
        </div>
      </FollowingListItem>
    </ol>
  </div>
</template>

<script>
import { get, } from 'lodash'
import FollowingListItem from './FollowingListItem.vue'

export default {
  props: {
    date: {
      type: String,
      required: true,
    },
    data: {
      type: Array,
      required: true,
    },
  },
  components: {
    FollowingListItem,
  },
  methods: {
    getItemContent (item) {
      return get(item, 'item') || {}
    },
    getTags (item) {
      const tags = this.getItemContent(item).tags || []
      return tags.length === 0 ? '' : `# ${ tags.map(tag => tag.text).join('、') }`
    },
  },
}
</script>

<style lang="stylus" scoped>
.list-container
  &__date
    font-size 12px
    color black
    background-color #d3d3d3
    margin 0
    padding 6px 0 6px 20px
    width calc(100% + 20px + 20px)
    position relative
    left -20px
  &:last-child
    border-bottom 1px solid #d3d3d3

.list
  margin 0
  padding 0
  &__list-item
    border-bottom 1px solid #d3d3d3
    padding 9px 0 !important
    &:last-child
     border-bottom none

.tag
  font-size 12px
  font-weight 500
  color black
  margin 0

.report
  display flex
  &__hero-img
    d = 60px
    width d
    min-width d
    height d
    min-height d
    & > img
      width 100%
      height 100%
      object-fit cover
  &__info
    display flex
    flex-direction column
    justify-content space-between
    height 60px
    margin 0 0 0 10px
  &__category, &__title, &__tags
    margin 0
  &__category
    margin 0 0 4px 0
    font-size 10px
    color #808080
  &__title
    font-size 12px
    font-weight 600
    color black
  &__tags
    font-size 10px
</style>

