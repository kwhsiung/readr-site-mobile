<template>
  <li class="tag-item">
    <span class="tag-item__hashtag">#</span>
    <div class="tag-item__tag tag">
      <div class="tag__header">
        <span class="tag__text" v-text="tag.text"></span>
        <span v-if="isLoggedIn" class="tag__action tag-action">
          <img :src="isFollow(tag.id) ? '/public/icons/star-blue.png' : '/public/icons/star-line-blue.png'" @click="toogleFollow(tag.id)">
          <span
            v-if="shouldShowActionTooltip"
            :class="[ 'tag-action__tooltip', { 'tag-action__tooltip--toogled': showActionTooltip } ]"
            v-text="isFollow(tag.id) ? $t('FOLLOWING.FOLLOW_TAG') : $t('FOLLOWING.UNFOLLOW_TAG') "
          >
          </span>
        </span>
      </div>
      <!-- TODO: add related projects while data available -->
      <ul v-if="shouldShowRelatedsList && isTagRelatedProjectsExist" class="tag__relateds-list">
        <TagItemRelatedsListItem
          v-for="(projects, i) in tag.relatedProjects"
          :key="i"
          class="tag__relateds-list-item"
        />
      </ul>
    </div>
    <!-- TODO: add trending-rank -->
    <p v-if="shouldShowTrendingRank" class="tag-item__trending-rank"></p>
  </li>
</template>

<script>
import { get, } from 'lodash'
import TagItemRelatedsListItem from './TagItemRelatedsListItem.vue'

const publishAction = (store, data) => store.dispatch('FOLLOW', { params: data, })

const updateStoreFollowingByResource = (store, { action, resource, resourceId, userId, }) => {
  return store.dispatch('UPDATE_FOLLOWING_BY_RESOURCE', {
    params: {
      action: action,
      resource: resource,
      resourceId: resourceId,
      userId: userId,
    },
  })
}

export default {
  name: 'TagItem',
  props: {
    tag: {
      type: Object,
      required: true,
    },
    shouldShowTrendingRank: {
      type: Boolean,
      default: false,
    },
    shouldShowRelatedsList: {
      type: Boolean,
      default: false,
    },
    shouldShowActionTooltip: {
      type: Boolean,
      default: false,
    },
  },
  components: {
    TagItemRelatedsListItem,
  },
  data () {
    return {
      showActionTooltip: false,
    }
  },
  computed: {
    isLoggedIn () {
      return this.$store.state.isLoggedIn
    },
    isTagRelatedProjectsExist () {
      return get(this.tag, 'relatedProjects') !== null
    },
    isMouseover () {
      return get(this.$store.state, [ 'tagsIsMouseover', this.tag.id, ], this.isMouseoverLocal)
    },
  },
  methods: {
    isFollow (id) {
      const data = this.$store.state.followingByResource.tag.find(tag => tag.resourceID === id)
      const followers = data ? data.followers : []
      return followers.find(memberId => memberId === this.$store.state.profile.id )
    },
    toogleFollow (id) {
      if (this.isFollow(id)) {
        publishAction(this.$store, {
          action: 'unfollow',
          resource: 'tag',
          subject: this.$store.state.profile.id,
          object: id,
        })
        updateStoreFollowingByResource(this.$store, {
          action: 'unfollow',
          resource: 'tag',
          resourceId: id,
          userId: this.$store.state.profile.id,
        })
      } else {
        publishAction(this.$store, {
          action: 'follow',
          resource: 'tag',
          subject: this.$store.state.profile.id,
          object: id,
        })
        updateStoreFollowingByResource(this.$store, {
          action: 'follow',
          resource: 'tag',
          resourceId: id,
          userId: this.$store.state.profile.id,
        })
      }

      this.toogleFollowTooltip()
    },
    toogleFollowTooltip () {
      this.showActionTooltip = true
      if (this._timer) {
        clearTimeout(this._timer)
      }
      this._timer = setTimeout(() => {
        this.showActionTooltip = false
      }, 1000)
    },
  },
}
</script>

<style lang="stylus" scoped>
.tag-item
  display inline-flex
  align-items flex-start
  &__hashtag
    font-size 24px
    font-weight 600
    margin 0 15.5px 0 0
    color #11b8c9
  &__tag
    // font-size .9375rem
    line-height 1.5rem
    // font-weight 700
    border 1px solid #11b8c9
    border-radius 12px
    user-select none
    background-color white
    flex 1 1 auto
  &__trending-rank
    width 22.5px
    min-width 22.5px
    height 24px
    line-height 24px
    font-size 9px
    text-align center
    margin 0 0 0 6px

.tag
  &__header
    padding 0 8px 0 13.5px
    display flex
    justify-content space-between
    align-items center
    border-radius 12px
    background-color white
    color black
  &__text
    font-size 12px
    font-weight 400
    white-space nowrap
    overflow hidden
    text-overflow ellipsis
  &__action
    margin-left 5px
    > img
      position relative
      top 2px
      width 20px
      height 15px
      padding-left 5px
      border-left 1px solid #d3d3d3
      cursor pointer
  &__relateds-list
    list-style none
    margin 0
    padding 9px 8px 9px 13.5px
    border-top 1px solid #11b8c9
    position relative
    &:before
      content ''
      position absolute
      top 0
      left 30px
      width 0
      height 0
      border-style solid
      border-width 10px 4px 0 4px
      border-color #11b8c9 transparent transparent transparent
    &:after
      content ''
      position absolute
      top -1px
      left 30.5px
      width 0
      height 0
      border-style solid
      border-width 9px 3px 0 3px
      border-color white transparent transparent transparent
  &__relateds-list-item
    & + &
      border-top 1px solid #d3d3d3

.tag-action
  position relative
  &__tooltip
    pointer-events none
    padding 1px 2px
    position absolute
    top 5%
    left 30px
    width max-content
    height 90%
    font-size 10px
    color #444746
    background-color white
    border 1px solid #d3d3d3
    z-index 100
    display flex
    align-items center
    opacity 0
    transition opacity .25s
    &:before
      position absolute
      top 2.5px
      left -10px
      content ''
      width 0
      height 0
      border-style solid
      border-width 7px 10px 7px 0
      border-color transparent #d3d3d3 transparent transparent
    &:after
      position absolute
      top 3.5px
      left -9px
      content ''
      width 0
      height 0
      border-style solid
      border-width 6px 9px 6px 0
      border-color transparent white transparent transparent
    &--toogled
      opacity 1
</style>

