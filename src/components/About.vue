<template>
  <div class="about">
    <div class="about__thumbnail">
      <img v-show="thumbnail" :src="getFullUrl(thumbnail)">
    </div>
    <div class="about__name">
      <span class="name" v-text="name"></span>
      <span class="role" v-text="role"></span>
    </div>
    <div class="about__introduction" v-text="introduction"></div>
    <div class="about__edit" v-if="isCurrUser">
      <span class="about__edit__btn" v-text="editText" @click="goEdit"></span>
    </div>
    <BaseLightBox :showLightBox.sync="showLightBox" borderStyle="nonBorder" v-if="isCurrUser">
      <ProfileEdit :showLightBox="showLightBox" :profile="profile" @save="showLightBox = false"/>
    </BaseLightBox>
  </div>
</template>
<script>
  import { filter, get, } from 'lodash'
  import { ROLE_MAP, } from 'src/constants'
  import { getFullUrl, } from 'src/util/comm'
  import BaseLightBox from 'src/components/BaseLightBox.vue'
  import ProfileEdit from 'src/components/member/ProfileEdit.vue'

  const debug = require('debug')('CLIENT:About')
  export default {
    components: {
      BaseLightBox,
      ProfileEdit,
    },
    computed: {
      isCurrUser () {
        const currUser = get(this.$store, 'state.profile.id')
        debug('currUser', currUser)
        debug('profile', get(this.profile, 'id'))
        return currUser === get(this.profile, 'id')
      },
      introduction () {
        return get(this.profile, [ 'description', ], '')
      },
      name () {
        return get(this.profile, [ 'nickname', ])
      },
      role () {
        const text = get(filter(ROLE_MAP, { key: get(this.profile, [ 'role', ]), }), [ 0, 'value', ])
        return text && ` (${text})`
      },
      thumbnail () {
        return get(this.profile, [ 'profileImage', ]) || '/public/icons/exclamation.png'
      },
    },
    data () {
      return {
        editText: '(edit)',
        showLightBox: false,
      }
    },
    name: 'about',
    methods: {
      goEdit () {
        debug('isCurrUser', this.isCurrUser)
        this.isCurrUser && (this.showLightBox = true)
      },
      getFullUrl,
    },
    mounted () {},
    props: {
      profile: {
        type: Object,
        default: {},
      },
    },
  }
</script>
<style lang="stylus" scoped>
  .about
    position relative
    width 100%
    min-height 100px
    margin 0 auto
    padding 0 20px
    &__thumbnail
      width 75px
      height 75px
      border 1px solid #979797
      border-radius 50%
      overflow hidden
      > img
        width 100%
        height 100%
        object-fit cover
        object-position center center
    &__name
      margin-top 10px
      font-size 1.125rem
      font-weight 600
    &__introduction
      margin-top 5px
      font-size .9375rem
      font-weight 400
      line-height 1.5
      text-align justify
    &__edit
      text-align right
      &__btn
        color #4280a2
        font-size .75rem
        font-weight 600
        cursor pointer
  // .about
  //   width 100%
  //   min-height 100px
  //   margin 20px auto
  //   padding 0 20px
  //   position relative
  //   color #000
  //   &__thumbnail
  //     position absolute
  //     left 0
  //     top 0
  //     width 40px
  //     height 40px
  //     border 1px solid #979797
  //     border-radius 50%
  //     overflow hidden
  //     > img
  //       width 100%
  //       height 100%
  //       object-fit contain
  //       object-position center center
  //       // box-shadow inset 0 0 0 10px #fff
  //   &__name
  //     font-size 1.125rem
  //   &__introduction
  //     margin-top 10px
  //     font-size 0.9375rem
  //     font-weight 300
  //     line-height 1.375rem
  //   &__edit
  //     text-align right
  //     &__btn
  //       color #4280a2
  //       font-weight 600
  //       cursor pointer
  // @media (min-width 950px)
  //   .about
  //     max-width 950px
</style>