<template>
  <div>
    <article class="baselightbox-post__article">
      <img class="baselightbox-post__author-thumbnail" :src="authorThumbnailImg" v-if="isClientSide">
      <section class="article-content">
        <h2 class="article-content__date" v-text="!isPostEmpty ? updatedAtYYYYMMDD(post.updatedAt) : ''"></h2>
        <h2 class="article-content__author-nickname" v-text="authorNickname"></h2>
        <h1 class="article-content__title" v-text="!isPostEmpty ? post.title : ''"></h1>
        <div class="article-content__paragraph-container" v-html="!isPostEmpty ? post.content : ''"></div>
        <a class="article-content__source-link" :href="post.link" target="_blank" v-text="post.linkTitle"></a>
        <AppArticleNav
          :postId="post.id"
          :postRefId="assetRefId" 
          :articleType="this.post.flag"
          :commentCount="commentCount"
          :inLightbox="true" @toogleComment="toogleComment"/>
      </section>
    </article>
    <CommentContainer class="baselightbox-post__comment" v-show="showComment" v-if="shouldRenderComment"
      :asset="asset" :assetId="post.id" :assetRefId="assetRefId"></CommentContainer>  
  </div>
</template>
<script>
  import AppArticleNav from 'src/components/AppArticleNav.vue' 
  import CommentContainer from 'src/components/comment/CommentContainer.vue' 
  import { get, } from 'lodash'
  import { isClientSide, updatedAtYYYYMMDD, } from 'src/util/comm'
  const debug = require('debug')('CLIENT:BaseLightBoxTemplatePost')
  export default {
    name: 'BaseLightBoxTemplatePost',
    components: {
      AppArticleNav,
      CommentContainer,
    },
    computed: {
      asset () { 
        debug('this.asset', `${get(this.$store, 'state.setting.HOST')}/${get(this.post, 'flag') || 'post'}/${this.post.id}`) 
        return `${get(this.$store, 'state.setting.HOST')}/${get(this.post, 'flag') === 'memo' ? `series/${get(this.$route, 'params.slug')}` : 'post'}/${this.post.id}` 
      },         
      isClientSide,
      shouldRenderComment () { 
        return this.post.id ? true : false 
      },
    },
    data () {
      return {
        showComment: false,
      }
    },
    methods: {
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