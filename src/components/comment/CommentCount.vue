<template>
  <span v-text="commentAmount"></span>
</template>
<script>
  // import { getAssetUrl, } from 'src/util/talk'
  // import { get, } from 'lodash'
  
  const debug = require('debug')('CLIENT:commentCount')
  // const getCommentCount = (store, { assetUrl, postId, type, }) => {
  //   return store.dispatch('FETCH_COMMENT_COUNT', {
  //     params: {
  //       assetUrl,
  //       postId,
  //     },
  //     type,
  //   })
  // }
  export default {
    name: 'CommentCount',
    computed: {
      // getAssetUrl () {
        // return getAssetUrl(this.assetUrl || `/post/${this.postId}`)
      // },
    },
    methods: {
      // seUpfetchCommentCountInterval () {
      //   return new Promise((resolve) => {
      //     setInterval(() => {
      //       getCommentCount(this.$store, {
      //         assetUrl: this.getAssetUrl,
      //         postId: this.postId,
      //         type: this.type,
      //       })
      //     }, 60000 * 2)
      //     resolve()
      //   })
      // },
    },
    mounted () {
      // getCommentCount(this.$store, {
      //   assetUrl: this.getAssetUrl,
      //   postId: this.postId,
      //   type: this.type,
      // })
      // this.seUpfetchCommentCountInterval()
    },
    props: {
      assetUrl: {
        type: String,
      },
      commentAmount: {
        default: () => 0,
      },
      postId: {
         default: () => '',
      },
      type: {
        default: () => 'publicPosts',
      },
    },
    watch: {
      commentAmount: function () {
        debug('Comment count for post', this.postId, 'change to', this.commentAmount)
      },
    },
  }
</script>
<style lang="stylus" scoped></style>