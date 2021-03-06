<template>
  <div class="login-light" v-if="active" @click="turnOffThis" :class="{ window: TYPE.WINDOW === type, }">
    <div class="login-light__panel" @click.stop="() => {}">
      <div class="login-light__close" @click.stop="turnOffThis"></div>
      <div class="login-light__logo"><img src="/public/icons/readr-logo.png"></div>
      <div class="login-light__container">
        <template v-if="!isRegistering">        
          <div class="login-by-social-media">
            <FacebookLogin type="mix" :isDoingLogin.sync="isProcessing" theme="light" :panelType="type"></FacebookLogin>
            <GooglePlusLogin type="mix" :isDoingLogin.sync="isProcessing" theme="light" :panelType="type"></GooglePlusLogin>
          </div>
          <div class="login-msg"><span v-text="message"></span></div>
          <div class="login-by-email">
            <div class="hint-text"><span v-text="$t('login.USE_EMAIL_INSTEAD')"></span></div>
            <Login :isDoingLogin.sync="isProcessing" theme="dark" :panelType="type"></Login>
          </div>
        </template>
        <template v-else>
          <div class="login-msg"><span v-text="message"></span></div>
          <div class="register">
            <Register></Register>
          </div>
        </template>
        <div class="registration-switcher">
          <span class="question" 
            v-text="isRegistering ? $t('login.MEMBER_ALREAY') : $t('login.NOT_MEMBER_YET')"></span>
          <span class="switcher" @click="switchAction"
            v-text="isRegistering ? $t('login.WORDING_LOGIN') : $t('login.WORDING_REGISTER')"></span>
        </div>
      </div>
      <div class="login-light__processing-hint" v-if="isProcessing"><Spinner :show="true"></Spinner></div>
    </div>
  </div>
</template>
<script>
  import Cookie from 'vue-cookie'
  import FacebookLogin from './FacebookLogin.vue'
  import GooglePlusLogin from './GooglePlusLogin.vue'
  import Login from './Login.vue'
  import Register from 'src/components/register/Register.vue'
  import Spinner from 'src/components/Spinner.vue'
  import preventScroll from 'prevent-scroll'
  import { get, } from 'lodash'
  import { loadRecaptcha, loadGapiSDK, loadFbSDK, } from 'src/util/comm'
  const switchOff = store => store.dispatch('LOGIN_ASK_TOGGLE', { active: false, message: '', })
  const getDisposableToken = store => store.dispatch('DISPOSABLE_TOKEN', { type: 'register', })
  // const debug = require('debug')('CLIENT:LoginLight')
  const TYPE = {
    WINDOW: 'WINDOW',
  }
  export default {
    name: 'LoginLight',
    components: {
      FacebookLogin,
      GooglePlusLogin,
      Login,
      Register,
      Spinner,
    },
    computed: {
      active () {
        return get(this.$store, 'state.loginAskFlag.active', false)
      },   
      message () {
        return get(this.$store, 'state.loginAskFlag.message', this.$t('login.REGISTER_BONUS'))
      },   
      type () {
        return get(this.$store, 'state.loginAskFlag.type', 'confirm')
      }, 
    },
    data () {
      return {
        TYPE,
        isProcessing: false,
        isRegistering: false,
      }
    },
    methods: {
      switchAction () {
        this.isRegistering = !this.isRegistering
      },
      turnOffThis () {
        /** if isProcessing is true, user is not allowed to close the this comp */
        if (this.isProcessing) { return }
        switchOff(this.$store)
      },
    },
    mounted () {},
    watch: {
      active () {
        if (this.active) {
          preventScroll.on()
          Cookie.set('location-replace-from', this.$route.fullPath, { expires: '60s', })
          Promise.all([
            getDisposableToken(this.$store),
            loadRecaptcha(this.$store),
            loadGapiSDK(this.$store),
            loadFbSDK(this.$store),         
          ])
        } else {
          preventScroll.off()
          Cookie.delete('location-replace-from')
          this.$forceUpdate()          
        }
      },
    },
  }
</script>
<style lang="stylus" scoped>
  .login-light
    position fixed
    left 0
    top 0
    background-color rgba(255,255,255,0.9)
    z-index 999999
    width 100vw
    height 100vh
    display flex
    justify-content center
    align-items center
    &.window
      background-color #444746
      .login-light__close
        display none    
    &__panel
      width 100%
      height 100%
      background-color #444746
      padding 25px 10px 90px
      position relative
      overflow auto
    &__logo
      margin 0 auto
      width 50px
      img
        width 100%
    &__container
      margin-top 37px
      .login-msg
        margin-top 10px
        display flex
        justify-content center
        font-size 0.9375rem
        font-weight normal
        font-style normal
        font-stretch normal
        line-height normal
        letter-spacing normal
        color #ddcf21      
      .login-by-email
        margin-top 38px
        .hint-text
          display flex
          justify-content center
          color #fff
          font-size 0.9375rem
          margin-bottom 12px
      .registration-switcher
        margin-top 30px
        display flex
        justify-content center
        font-size 0.9375rem
        font-weight 600
        font-style normal
        font-stretch normal
        line-height normal
        letter-spacing normal
        text-align center
        .question
          color #fff
        .switcher
          color #ddcf21
          cursor pointer
    &__processing-hint
      display flex
      justify-content center
      align-items center
      position absolute
      top 0
      left 0
      width 100%
      height 100%
      background-color rgba(0,0,0,0.8)
      z-index 1

    &__close
      width 20px
      height 20px
      position fixed
      right 0
      top 0
      background-size contain
      background-position center center
      background-repeat no-repeat
      background-image url(/public/icons/shutdown.jpg)
      cursor pointer
</style>