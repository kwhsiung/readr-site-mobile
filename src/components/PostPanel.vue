<template>
  <section class="postPanelEdit">
    <input v-model="post.title" type="text" class="postPanelEdit__title" placeholder="輸入標題">
    <text-editor :contentEdit="post.content" :needReset="resetToggle" v-on:updateContent="$_postPanelEdit_updateContent"></text-editor>
    <div class="postPanelEdit__input postPanelEdit__link">
      <label for="">新聞連結：</label>
      <input v-model="post.link" type="url">
    </div>
    <div v-if="$can('editPostOg')" class="postPanelEdit__input postPanelEdit--publishDate">
      <label for="">發布日期：</label>
      <no-ssr>
        <datepicker v-model="post.date" :format="dateFormat" :input-class="'datepicker__input'" :language="'zh'"></datepicker>
      </no-ssr>
    </div>
    <div v-if="$can('editPostOg')" class="postPanelEdit__input">
      <label for="">分享標題：</label>
      <input v-model="post.ogTitle" type="text">
    </div>
    <div v-if="$can('editPostOg')" class="postPanelEdit__input">
      <label for="">分享說明：</label>
      <input v-model="post.ogDescription" type="text">
    </div>
    <div v-if="$can('editPostOg')" class="postPanelEdit__input">
      <label for="">分享縮圖：</label>
      <input v-model="post.ogImage" type="text" readonly>
      <button class="postPanelEdit__btn--img" @click="$_postPanelEdit_uploadImage"><img src="/public/icons/upload.png" alt="上傳"></button>
      <button class="postPanelEdit__btn--img" @click="$_postPanelEdit_cleanOgImage"><img src="/public/icons/delete.png" alt="刪除"></button>
    </div>
    <div v-show="post.ogImage && $can('editPostOg')" class="postPanelEdit__ogImg">
      <img :src="post.ogImage" :alt="post.ogTitle">
    </div>
    <div :class="{ advanced: $can('publishPost') }" class="postPanelEdit__submit">
      <button :disabled="(action === 'add') && isEmpty" class="postPanelEdit__btn draft" @click="$_postPanelEdit_submit(3)">存成草稿</button>
      <button :disabled="(action === 'add') && isEmpty" class="postPanelEdit__btn submit" @click="$_postPanelEdit_submit(2)">提交</button>
      <button v-if="$can('publishPost')" :disabled="(action === 'add') && isEmpty" class="postPanelEdit__btn" @click="$_postPanelEdit_submit(1)">發布</button>
    </div>
  </section>
</template>
<script>
  import _ from 'lodash'
  import AlertPanel from './AlertPanel.vue'
  import BaseLightBox from './BaseLightBox.vue'
  import Datepicker from 'vuejs-datepicker'
  import TextEditor from './TextEditor.vue'
  import NoSSR from 'vue-no-ssr'
  
  const addPost = (store, params) => {
    return store.dispatch('ADD_POST', { params })
  }

  const updatePost = (store, params) => {
    return store.dispatch('UPDATE_POST', { params })
  }

  const uploadImage = (store, file) => {
    return store.dispatch('UPLOAD_IMAGE', { file })
  }

  export default {
    name: 'PostPanel',
    components: {
      'alert-panel': AlertPanel,
      'base-light-box': BaseLightBox,
      'datepicker': Datepicker,
      'text-editor': TextEditor,
      'no-ssr': NoSSR
    },
    props: {
      action: {
        default: undefined
      },
      isCompleted: {
        default: false
      },
      post: {
        type: Object,
        default: {}
      },
      showLightBox: {
        default: false
      }
    },
    data () {
      return {
        active: undefined,
        dateFormat: 'yyyy/MM/d',
        resetToggle: true,
        showAlert: false,
      }
    },
    computed: {
      isEmpty () {
        return _.isEmpty(_.trim(_.get(this.post, [ 'link' ], '')))
          && _.isEmpty(_.trim(_.get(this.post, [ 'title' ], '')))
          && _.isEmpty(_.trim(_.replace(_.get(this.post, [ 'content' ], ''), /<[^>]*>/g, '')))
      },
      postActive () {
        return _.get(this.post, [ 'active' ])
      }
    },
    watch: {
      showAlert (val) {
        if (!val) {
          this.$emit('closeLightBox')
        }
      },
      showLightBox (val) {
        if (!val) {
          if (!this.isEmpty && !this.isCompleted) {
            this.$_postPanelEdit_submit(this.postActive)
          }
        }
      }
    },
    mounted () {},
    methods: {
      $_postPanelEdit_cleanOgImage () {
        this.post.ogImage = ''
      },
      $_postPanelEdit_resetContent () {
        this.resetToggle = !this.resetToggle
      },
      $_postPanelEdit_submit (active = 3) {
        const params = {}
        params.active = active
        params.author = _.get(this.$store.state, [ 'profile', 'id' ])
        params.content = _.get(this.post, [ 'content' ])
        params.link = _.get(this.post, [ 'link' ])
        params.og_description = _.get(this.post, [ 'ogDescription' ])
        params.og_image = _.get(this.post, [ 'ogImage' ])
        params.og_title = _.get(this.post, [ 'ogTitle' ]) || _.get(this.post, [ 'title' ])
        params.title = _.get(this.post, [ 'title' ])
        params.updated_by = _.get(this.$store.state, [ 'profile', 'id' ])
        
        this.active = active

        if (Date.parse(_.get(this.post, [ 'date' ]))) {
          params.published_at = _.get(this.post, [ 'date' ])
        }

        if (this.action === 'add') {
          addPost(this.$store, params)
            .then(() => {
              this.$emit('showAlert', true, active, true)
            })
        }
        if (this.action === 'edit') {
          params.id = _.get(this.post, [ 'id' ])
          

          if (this.active === 2 || this.active === 3) {
            updatePost(this.$store, params)
              .then(() => {
                this.$emit('showAlert', true, active, true)
              })
          }
        }
      },
      $_postPanelEdit_updateContent (content) {
        this.post.content = content
      },
      $_postPanelEdit_uploadImage () {
        const input = document.createElement('input')
        input.setAttribute('type', 'file')
        input.setAttribute('accept', 'image/*')
        input.click()
        input.onchange = () => {
          const file = input.files[0]
          if (/^image\//.test(file.type)) {
            const fd = new FormData()
            fd.append('image', file)
            uploadImage(this.$store, fd)
              .then((res) => {
                this.ogImage = res.body.url
              })
              .catch((err) => {
                console.log(err)
              })
          }
        }
      }
    }
  }
</script>
<style lang="stylus" scoped>

.postPanelEdit
  width 900px
  padding 30px 100px
  > input 
    width 100%
    height 25px
  &__title
    padding-left 10px
    color #000
    font-size 18px
    font-weight 600
  &__input
    display flex
    width 100%
    margin-top 10px
    label
      line-height 25px
    input
      flex 1
      padding-left 10px
      width 440px
      height 25px
      color #808080
    button
      width 25px
      height 25px
      img
        width 100%
        height auto
  &__ogImg
    padding-left 80px
    margin-top 10px
    img
      width 180px
      height auto
  &--publishDate
    margin-top 26px
    .vdp-datepicker
      flex 1
      
  &__submit
    display flex
    justify-content space-between
    width 100%
    margin-top 20px
    &.advanced
      .postPanelEdit__btn
        width 210px
  &__btn
    display inline-flex
    justify-content center
    align-items center
    width 340px
    height 30px
    margin 0
    font-size 14px
    font-weight 300
    border 1px solid #d3d3d3
    user-select none
    &.submit
      color #fff
      background-color #808080
    &--img
      padding 0
      margin 0 5px
      background none
      border none
      cursor pointer
      outline none
      user-select none
      &:first-of-type
        margin-left 10px

.alert
  width 325px
  color #000
  font-size 15px
  background #fff
  box-shadow: 1px 1px 2.5px 0 rgba(0, 0, 0, 0.5)
  > p
    margin .5em 25px
  &__control
    display flex
    justify-content space-between
    margin-top 30px
  &__title
    display block
    margin 15px 25px 0
    color #4280a2
  &__btn
    flex 1
    height 30px
    background transparent
    border .5px solid #d3d3d3
    border-collapse collapse
    outline none
    &:first-of-type
      border-right none

</style>