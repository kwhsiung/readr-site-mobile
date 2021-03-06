<template>
  <div>
    <article class="baselightbox-post__article">
      <img class="baselightbox-post__author-thumbnail" :src="authorThumbnailImg" v-if="isClientSide">
      <section class="article-content">
        <h2 class="article-content__date" v-text="!isPostEmpty ? updatedAtYYYYMMDD(post.updatedAt) : ''"></h2>
        <h2 class="article-content__author-nickname" v-text="authorNickname"></h2>
        <h1 class="article-content__title" v-text="!isPostEmpty ? post.title : ''"></h1>
        <div class="article-content__paragraph-container" v-html="!isPostEmpty ? post.content : ''"></div>
        <a class="article-content__source-link" :href="post.link" target="_blank" v-text="post.linkTitle ? post.linkTitle : post.link"></a>
        <AppArticleNav
          :showFollow="false"
          :postId="post.id"
          :postRefId="assetRefId" 
          :projectInfo="get(post, 'project')"
          :resource="get(postInstance, [ 'processed', 'resource' ])"
          :resourceType="get(postInstance, [ 'processed', 'resourceType', ], '')"
          :commentCount="commentCount"
          :inLightbox="true" @toogleComment="toogleComment"
        >
          <PostShareNav slot="share" :post="post" class="baselightbox-post__share-nav"/>
          <TagNav
            v-if="post.tags && post.tags.length > 0"
            slot="tagNav"
            :tags="post.tags"
            class="baselightbox-post__tags" />
        </AppArticleNav>
      </section>
    </article>
    <CommentContainer
      v-if="shouldRenderComment"
      v-show="showComment"
      class="baselightbox-post__comment"
      :asset="asset"
      :assetId="post.id"
      :assetRefId="assetRefId"
      :isPublic="!get(me, 'id')"
    />
  </div>
</template>
<script>
  import AppArticleNav from 'src/components/AppArticleNav.vue' 
  import CommentContainer from 'src/components/comment/CommentContainer.vue'
  import TagNav from 'src/components/tag/TagNav.vue'
  import PostShareNav from 'src/components/post/PostShareNav.vue'
  import { get, } from 'lodash'
  import { isClientSide, updatedAtYYYYMMDD, } from 'src/util/comm'
  import { getPostFullUrl, } from 'src/util/post/index'
  import { createPost, } from 'src/util/post'

  const debug = require('debug')('CLIENT:BaseLightBoxTemplatePost')
  export default {
    name: 'BaseLightBoxTemplatePost',
    components: {
      AppArticleNav,
      CommentContainer,
      TagNav,
      PostShareNav,
    },
    computed: {
      asset () { 
        debug('this.asset', `${get(this.$store, 'state.setting.HOST')}/${get(this.post, 'flag') || 'post'}/${this.post.id}`) 
        return `${get(this.$store, 'state.setting.HOST')}/${get(this.post, 'flag') === 'memo' ? `series/${get(this.$route, 'params.slug')}` : 'post'}/${this.post.id}` 
      },         
      isClientSide,
      me () {
        return get(this.$store, 'state.profile', {})
      },
      shouldRenderComment () { 
        return this.post.id ? true : false 
      },
      shareUrl () {
        return getPostFullUrl(this.post)
      },
      postInstance () {
        return createPost(this.post)
      },
    },
    data () {
      return {
        showComment: true,
      }
    },
    methods: {
      get,
      toogleComment () { 
        this.showComment = !this.showComment 
      },       
      updatedAtYYYYMMDD,
    },
    mounted () {},
    props: { 
      assetRefId: {}, 
      authorThumbnailImg: String, 
      authorNickname: String, 
      commentCount: Number, 
      isPostEmpty: Boolean, 
      post: Object, 
    },     
  }
</script>

<style lang="stylus" scoped>
.baselightbox-post
  &__article
    position relative
  &__share-nav
    margin 0 0 0 auto

.article-content
  width 100%
  &__paragraph-container, &__source-link
    word-break break-all
</style>
