<template>
  <section v-show="showLightBox" class="baseLightBox">
    <div class="baseLightBox__container" :class="containerClass">
      <slot></slot>
      <button
        class="baseLightBox__btn--close"
        :class="closeButtonClass"
        @click="$_baseLightBox_close">
      </button>
    </div>
    <div class="baseLightBox__curtain" @click="$_baseLightBox_close"></div>
  </section>
</template>
<script>
  import pathToRegexp from 'path-to-regexp'
  import preventScroll from 'prevent-scroll'
  
  export default {
    name: 'BaseLightBox',
    components: {
    },
    computed: {
      closeButtonClass () {
        return {
          'hide': this.hideCloseButton,
        }
      },
      containerClass () {
        return {
          'alert': this.isAlert,
          'non-border': this.borderStyle === 'nonBorder',
        }
      },
    },
    props: {
      borderStyle: {
        default: 'normal',
      },
      isAlert: {
        type: Boolean,
        default: false,
      },
      isConversation: {
        default: false,
      },
      hideCloseButton: {
        default: false,
      },
      showLightBox: {
        type: Boolean,
        default: false,
      },
    },
    watch: {
      showLightBox (val) {
        if (val) {
          preventScroll.on()
        } else {
          preventScroll.off()
        }
      },
    },
    mounted () {
      this.showLightBox && preventScroll.on()
    },
    methods: {
      $_baseLightBox_close () {
        pathToRegexp('/post/:postId').test(this.$route.path) ? this.$emit('closeLightBox') : this.$emit('update:showLightBox', false)
      },
    },
  }
</script>
<style lang="stylus" scoped>

.baseLightBox
  position fixed
  top 0
  left 0
  right 0
  bottom 0
  z-index 1000
  display flex
  justify-content center
  align-items center
  width 100%
  // height 100vh
  background-color rgba(0,0,0,.6)
  &.text-editor
    .baseLightBox__container
      padding 55px 0 0px
      &.alert
        padding 0
  &__btn
    &--close
      cursor pointer
      position absolute
      top -5px
      right -5px
      width 45px
      height 45px
      background-position center center
      background-repeat no-repeat
      background-size cover
      background-image url(/public/icons/shutdown.jpg)
      outline none
      border none
      &.hide
        display none
  &__container
    position relative
    z-index 10
    display flex
    flex-direction column
    justify-content center
    align-items center
    position relative
    width 100%
    height 100%
    background-color #fff
    border 5px solid #d8ca21
    &.non-border
      border none
    &.non-border 
      > .baseLightBox__btn--close
        top 0
        right 0
    &.alert
      width auto
      height auto
      border none
      > .baseLightBox__btn--close
        display none
  &__curtain
    position absolute
    top 0
    left 0
    right 0
    bottom 0
    z-index 1

@media (min-width 950px)
  .baseLightBox
    &__container
      width auto
      height auto
    &__btn
      &--close
        top -5px
        right -55px
        width 50px
        height 50px

</style>
