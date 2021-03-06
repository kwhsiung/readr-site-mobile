<template>
  <div class="login" @keyup="keyupHandler">
    <TextItem class="login__input-email" type="email"
      :placeHolder="$t('login.WORDING_EMAIL_PLACEHOLDER')"
      :alert.sync="alert.mail"
      :backgroundColor="'transparent'"
      :border="'solid 1px #9b9b9b'"
      :color="'#fff'"
      :value.sync="formData.mail"></TextItem>
    <TextItem class="login__input-pwd" type="password"
      :placeHolder="$t('login.WORDING_PASSWORD')"
      :alert.sync="alert.pwd"
      :backgroundColor="'transparent'"
      :border="'solid 1px #9b9b9b'"
      :color="'#fff'"
      :value.sync="formData.pwd"></TextItem>
    <div class="login__wrapper">
      <div class="keep-login-alive">
        <input type="checkbox" id="keep-alive" ref="keep-alive">
        <label for="keep-alive" v-text="' ' + $t('login.WORDING_KEEP_ALIVE')"></label>
      </div>
      <div class="forget-pwd">
        <span v-text="$t('login.WORDING_FORGET_PASSWORD')" @click="goRecoverPwd"></span>
      </div>
    </div>
    <div v-if="resMsg" class="login__msg">
      <div class='content' v-text="resMsg"></div>
    </div>
    <div class="login__btn" :test-group="abIndicator" test-name="20181112-verticalswitch" @click="login">
      <span v-text="$t('login.WORDING_LOGIN')"></span>
    </div>
  </div>
</template>
<script>
  import { get, } from 'lodash'
  import { getIndicator, } from 'src/util/abTest'
  import TextItem from 'src/components/form/TextItem.vue'
  import validator from 'validator'
  import VueCookie from 'vue-cookie'

  const debug = require('debug')('CLIENT:Login')
  const switchOffLoginAsk = store => store.dispatch('LOGIN_ASK_TOGGLE', { active: false, message: '', })
  const login = (store, profile, token) => {
    return store.dispatch('LOGIN', {
      params: {
        email: profile.email,
        password: profile.password,
        keepAlive: profile.keepAlive,
      },
      token,
    })
  }

  export default {
    components: {
      TextItem,
    },
    data () {
      return {
        abIndicator: '',
        alert: {},
        formData: {},
        resMsg: null,
      }
    },
    name: 'Login',
    methods: {
      getIndicator,
      goRecoverPwd () {
        this.$emit('goRecoverPwd')
      },
      keyupHandler (e) {
        if (e.keyCode === 13) {
          this.login()
        }
      },
      login () {
        if (this.validatInput()) {
          this.$emit('update:isDoingLogin', true)
          login(this.$store, {
            email: this.formData.mail,
            password: this.formData.pwd,
            keepAlive: this.$refs[ 'keep-alive' ].checked,
          }, get(this.$store, [ 'state', 'register-token', ])).then((res) => {
            this.$emit('update:isDoingLogin', false)
            if (res.status === 200) {
              const isPublicComment = this.$route.path === '/comment'
              const from = VueCookie.get('location-replace-from')
              const isFromPathExist = from !== null

              if (this.panelType === 'WINDOW') {
                window.opener.location.reload()
                window.close()
              } else if (isPublicComment) {
                this.$router.push(this.$route.fullPath)
              } else {
                if (isFromPathExist) {
                  VueCookie.delete('location-replace-from')
                  this.$router.push(from)
                } else {
                  this.$router.push('/')
                }
              }
              // revolke switchOffLoginAsk for LoginLight
              switchOffLoginAsk(this.$store)
            } else {
              this.resMsg = this.$t('login.WORDING_LOGIN_INFAIL_VALIDATION_ISSUE')
            }
          }).catch((err) => {
            this.$emit('update:isDoingLogin', false)
            if (err.status === 401) {
              this.resMsg = this.$t('login.WORDING_LOGIN_UNAUTHORIZED')
            }
          })
        }
      },
      validatInput () {
        let pass = true
        if (!this.formData.mail || !validator.isEmail(this.formData.mail)) {
          pass = false
          this.alert.mail = {
            flag: true,
            msg: this.$t('login.WORDING_REGISTER_EMAIL_VALIDATE_IN_FAIL'),
          }
          debug('Mail wrong', this.formData.mail)
        }
        if (!this.formData.pwd || validator.isEmpty(this.formData.pwd)) {
          pass = false
          this.alert.pwd = {
            flag: true,
            msg: this.$t('login.WORDING_REGISTER_PWD_EMPTY'),
          }
          debug('Empty pwd', this.formData.pwd)
        }
        this.$forceUpdate()
        return pass
      },
    },
    beforeMount () {
      this.abIndicator = this.getIndicator()
    },
    mounted () {},
    props: {
      isDoingLogin: {
        type: Boolean,
        default: false,
      },
      panelType: {},
    },
  }
</script>
<style lang="stylus" scoped>
  .login
    width 100%
    height 100%
    position relative
    color #fff
    &__input-email, &__input-pwd
      margin 15px 0
      &:first-of-type
        margin-top 0
    &__wrapper
      display flex
      justify-content space-between
      font-size 0.875rem
      .keep-login-alive
        > input
          vertical-align top
          width 15px
          height 15px
    &__msg
      margin 15px 0
      width 100%
      text-align right
      color red
    &__btn
      // position absolute
      // bottom 0
      // left 0
      width 100%
      height 35px
      margin-top 30px
      background-color #ddcf21
      color #444746
      display flex
      justify-content center
      align-items center
      cursor pointer
      &:hover
        background-color #737373
  .forget-pwd
    cursor pointer
  
  .closed-beta
    .login
      padding-bottom 25px
    .login__wrapper
      color #fff
    .login__btn
      height 25px
      color #444746
      background-color #ddcf21
  
  @media (min-height 667px)
    .closed-beta
      .login
        margin-top 25px
      .login__msg
        margin 5px 0
        font-size .9375rem
  
</style>
