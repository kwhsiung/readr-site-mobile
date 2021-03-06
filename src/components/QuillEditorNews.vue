<template>
  <section class="editor news" :class="{ disabled: disabled }">
    <div class="editor__heading">
      <div class="editor__heading-text" v-text="$t('POST_EDITOR.EDITOR')"></div>
      <div class="editor__heading-switch" @click="$_quillEditor_toggleHtml">&lt; / &gt;</div>
    </div>
    <div id="toolbar">
      <span class="ql-formats">
        <button class="ql-bold"></button>
        <button class="ql-header" value="1"></button>
        <button class="ql-header" value="2"></button>
        <button class="ql-italic" value="2"></button>
        <button class="ql-blockquote"></button>
        <button class="ql-code-block"></button>
      </span>
      <span class="ql-formats">
        <button class="ql-image"></button>
      </span>
      <span class="ql-formats">
        <button class="ql-link"></button>
        <button class="ql-video"></button>
        <button v-show="$can('editPostOg')" class="ql-hr"></button>
      </span>
    </div>
    <div
      ref="quillEditorNews"
      :content="content"
      :disabled="disabled"
      class="editor__quill"
      v-quill:quillEditorNews="editorOption"
      @change="$_quillEditor_onEditorChange($event)">
    </div>
    <div class="editor__html" v-text="content"></div>
    <input ref="uploadImg" type="file" accept="image/*" @change="$_quillEditor_uploadImg">
  </section>
</template>

<script>
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'

const uploadImage = (store, file) => {
  return store.dispatch('UPLOAD_IMAGE', { file, type: 'post', })
}

export default {
  name: 'QuillEditorNews',
  props: {
    content: {
      default: '',
    },
    disabled: {
      type: Boolean,
    },
  },
  data () {
    return {
      editorOption: {
        formats: [ 'bold', 'header', 'italic', 'blockquote', 'code-block', 'link', 'image', 'video', 'hr', 'figcaption', ],
        modules: {
          toolbar: {
            container: '#toolbar',
            handlers: {
              'image': this.$_quillEditor_imageHandler,
              'hr': this.$_quillEditor_customHrHandler,
            },
          },
          clipboard: {
            matchVisual: false,
          },
        },
        bounds: '.post-panel-container',
      },
    }
  },
  methods: {
    $_quillEditor_customHrHandler () {
      if (this.$can('editPostOg')) {
        const range = this.quillEditorNews.getSelection()
        this.quillEditorNews.insertEmbed(range.index, 'hr', 'null')
      }
    },
    $_quillEditor_imageHandler () {
      this.$refs.uploadImg.click()
    },
    $_quillEditor_insertToEditor (url) {
      const range = this.quillEditorNews.getSelection()
      this.quillEditorNews.insertEmbed(range.index, 'image', url)
      this.quillEditorNews.insertEmbed(range.index + 1, 'figcaption', 'null')
    },
    $_quillEditor_onEditorChange (event) {
      if (event.html) {
        this.$emit('updateContent', event.html)
      }
    },
    $_quillEditor_toggleHtml (event) {
      event.target.parentNode.parentNode.classList.toggle('showHtml')
    },
    $_quillEditor_uploadImg () {
      const file = this.$refs.uploadImg.files[0]
      if (/^image\//.test(file.type)) {
        const fd = new FormData()
        fd.append('image', file)
        uploadImage(this.$store, fd)
          .then((res) => {
            this.$_quillEditor_insertToEditor(res.body.url)
          })
          .catch((err) => {
            console.error(err)
          })
      }
    },
  },
}
</script>

<style lang="stylus" scoped>
  .editor
    position relative
    margin-top 15px
    transition background-color .5s linear
    >>> .ql-hr
      width 54px
      &:after
        content 'More'
    >>> hr
      height 0px
      margin-top 5px
      margin-bottom 5px
    >>> figcaption
      color #b3b3b1
      font-size .75rem
      font-weight 400
    > input
      display none
    &.showHtml
      .editor__heading-switch
        background-color #d8ca21
      .editor__quill
        width 50%
      .editor__html
        display block
    &.input--error
      background-color #ffe5e5
    &.disabled
      background-color rgba(0, 0, 0, .3)
    &__heading
      display flex
      height 20px
      padding-left 10px
      color #fff
      font-size .875rem
      line-height 20px
      background-color #000
      &-text
        flex 1
        user-select none
      &-switch
        width 50px
        text-align center
        background-color #808080
        cursor pointer
        user-select none
        
    &__quill
      height 380px
      font-size 1rem
      overflow-y auto
      >>> img
        max-width 50%
      p:has(img)
        color red
    &__html
      display none
      position absolute
      right 0
      bottom 0
      width 50%
      height 380px
      padding 10px
      border-right 1px solid #d3d3d3
      border-bottom 1px solid #d3d3d3

</style>
