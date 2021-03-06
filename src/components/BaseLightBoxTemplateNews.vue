<template>
  <div>
    <img class="baselightbox-post__leading-image" v-if="post.ogImage && isClientSide"
      :src="getFullUrl(post.ogImage)"
      @load="setLeadingImageOrientation(getFullUrl(post.ogImage), $event)">
    <div class="baselightbox-post__article-container">
      <article class="baselightbox-post__article">
        <section class="author-info">
          <router-link :to="`/profile/${authorId}`">
            <img class="author-info__thumbnail" :src="authorThumbnailImg" v-if="isClientSide">
          </router-link>
          <div class="author-info__meta">
            <p class="author-info__date" v-text="!isPostEmpty ? updatedAtYYYYMMDD(post.updatedAt) : ''"></p>
            <router-link class="author-info__author-nickname" :to="`/profile/${authorId}`" v-text="authorNickname"></router-link>
          </div>
        </section>
        <section class="article-content">
          <h1 v-text="!isPostEmpty ? post.title : ''"></h1>
          <template v-for="p in postContent">
            <div :class="{ 'yt-iframe-container': isElementContentYoutube(p) }" v-html="p"></div>
          </template>
        </section>
      </article>
      <div class="nav-container">
        <AppArticleNav
          :showFollow="false"
          :postId="post.id"
          :postRefId="assetRefId"
          :projectInfo="get(post, 'project')"
          :resource="get(postInstance, [ 'processed', 'resource' ])"
          :resourceType="get(postInstance, [ 'processed', 'resourceType', ], '')"
          :commentCount="commentCount"
          :shouldShowComment="true"
        >
          <PostShareNav slot="share" :post="post" class="baselightbox-post__share-nav"/>
          <TagNav
            v-if="post.tags && post.tags.length > 0"
            slot="tagNav"
            :tags="post.tags"
            class="baselightbox-post__tags"
          />
        </AppArticleNav>
      </div>
    </div>
  </div>
</template>
<script>
  import AppArticleNav from 'src/components/AppArticleNav.vue'
  import TagNav from 'src/components/tag/TagNav.vue'
  import PostShareNav from 'src/components/post/PostShareNav.vue'
  import { getFullUrl, isClientSide, updatedAtYYYYMMDD, onImageLoaded, getElementContentAttr, getElementContentSrc, isElementContentYoutube, isImg, isLink, clickImg, } from 'src/util/comm'
  import { getPostFullUrl, createPost, } from 'src/util/post/index'
  import { get, } from 'lodash'
  export default {
    name: 'BaseLightBoxTemplateNews',
    components: {
      AppArticleNav,
      TagNav,
      PostShareNav,
    },
    computed: {
      isClientSide,
      shareUrl () {
        return getPostFullUrl(this.post)
      },
      postInstance () {
        return createPost(this.post)
      },
    },
    methods: {
      get,
      getElementContentAttr,
      getImgSrc (content) { 
        return getFullUrl(getElementContentSrc(content))
      },       
      isImg,
      isLink,
      clickImg,
      isElementContentYoutube,       
      setLeadingImageOrientation (src, event) {
        onImageLoaded(src).then(({ width, height, }) => {
          width < height ? event.target.style.objectFit = 'contain' : event.target.style.objectFit = 'cover'
        }).catch(() => { event.target.style.objectFit = 'cover' })
      },
      setContentImageOrientation (src, event) {
        onImageLoaded(src).then(({ width, height, }) => {
          width < height ? event.target.classList.add('portrait') : event.target.classList.add('landscape')
        }).catch(() => { event.target.classList.add('landscape') })
      },    
      updatedAtYYYYMMDD,
      getFullUrl,
    },
    mounted () {},
    props: { 
      assetRefId: {}, 
      authorId: Number, 
      authorThumbnailImg: String, 
      authorNickname: String, 
      commentCount: Number, 
      isPostEmpty: Boolean,       
      post: Object, 
      postContent: Array, 
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
  word-break break-all
  & >>> div
    font-size 15px
    letter-spacing 0.5px
    line-height 1.6
    text-align initial
    margin 26px 0
  & >>> blockquote
    margin 0
    padding 0 0 0 16px
    border-left 4px solid #ccc
    line-height 1

.yt-iframe-container
  position relative
  padding-bottom 56.25% // 16:9
  padding-top 25px
  width 100%
  height 0
  & >>> iframe
    position absolute
    top 0
    left 0
    width 100%
    height 100%

.pointer
  cursor pointer
</style>