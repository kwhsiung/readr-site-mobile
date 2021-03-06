<template>
  <div class="backstage guestEditor">
    <control-bar
      @addNews="$_guestEditor_showEditor({ postPanel: 'add', postType: config.type.NEWS })"
      @addReview="$_guestEditor_showEditor({ postPanel: 'add', postType: config.type.REVIEW })"
      @closeControlBar="$_guestEditor_closeControlBar"
      @editNews="$_guestEditor_showDraftList(config.type.NEWS)"
      @editProfile="$_guestEditor_showProfile"
      @editReview="$_guestEditor_showDraftList(config.type.REVIEW)"
      @openPanel="$_guestEditor_openPanel"
      :showControlBar="showControlBar"
    />
    <main class="backstage-container">
      <template v-if="activePanel === 'records'">
        <section class="backstage__records">
          <app-tab class="backstage__tab" :tabs="tabs" @changeTab="$_guestEditor_tabHandler" :defaultTab="defaultTab">
            <post-list-tab
              slot="0"
              :posts="posts"
              @deletePost="$_guestEditor_showAlert"
              @editPost="$_guestEditor_showEditor"
              @filterChanged="$_guestEditor_filterHandler">
            </post-list-tab>
            <post-list-tab
              slot="1"
              :posts="posts"
              @deletePost="$_guestEditor_showAlert"
              @editPost="$_guestEditor_showEditor"
              @filterChanged="$_guestEditor_filterHandler">
            </post-list-tab>
            <following-list-tab slot="2"></following-list-tab>
            <PointManager slot="3" v-if="isDonationActive"></PointManager>
          </app-tab>
        </section>
      </template>
    </main>
    <base-light-box borderStyle="nonBorder" :showLightBox.sync="showProfile">
      <base-light-box-profile :profile="profile" :showLightBox="showProfile" @save="showProfile = false"></base-light-box-profile>
    </base-light-box>
    <base-light-box :showLightBox.sync="showDraftList">
      <post-list-detailed
        :posts="postsDraft"
        @deletePost="$_guestEditor_showAlert"
        @editPost="$_guestEditor_showEditor">
      </post-list-detailed>
    </base-light-box>
    <base-light-box class="text-editor" :showLightBox.sync="showEditor">
      <post-panel
        :action="postPanel"
        :editorType="postType"
        :initialPost="post"
        @closeEditor="showEditor = false"
        @updateList="$_guestEditor_updatePostList">
      </post-panel>
    </base-light-box>
    <base-light-box :isAlert="true" :showLightBox.sync="showAlert">
      <alert-panel
        :status="itemsStatus" 
        :statusChanged="postStatusChanged"
        :items="itemsSelected"
        :needConfirm="needConfirm"
        :showLightBox="showAlert"
        :type="alertType"
        @closeAlert="$_guestEditor_alertHandler(false)"
        @deletePosts="$_guestEditor_deletePosts">
      </alert-panel>
    </base-light-box>
  </div>
</template>
<script>
  import { POST_PUBLISH_STATUS, POST_TYPE, } from '../../api/config'
  import _ from 'lodash'
  import AlertPanel from '../components/AlertPanel.vue'
  import BaseLightBox from '../components/BaseLightBox.vue'
  import FollowingListInTab from '../components/FollowingListInTab.vue'
  import PointManager from 'src/components/point/PointManager.vue'    
  import PaginationNav from '../components/PaginationNav.vue'
  import PostList from '../components/PostList.vue'
  import PostListDetailed from '../components/PostListDetailed.vue'
  import PostListInTab from '../components/PostListInTab.vue'
  import PostPanel from '../components/PostPanel.vue'
  import ProfileEdit from '../components/member/ProfileEdit.vue'
  import Tab from '../components/Tab.vue'
  import TheControlBar from '../components/TheControlBar.vue'

  const MAXRESULT = 5
  const DEFAULT_PAGE = 1
  const DEFAULT_SORT = '-updated_at'

  const deletePost = (store, id) => {
    return store.dispatch('DELETE_POST', { id: id, })
  }

  const getPostsByUser = (store, {
    maxResult = MAXRESULT,
    page = DEFAULT_PAGE,
    sort = DEFAULT_SORT,
    where = {},
  }) => {
    return store.dispatch('GET_POSTS_BY_USER', {
      params: {
        max_result: maxResult,
        page: page,
        show_author: true,
        show_updater: true,
        sort: sort,
        where: where,
      },
    })
  }

  const getPostsCount = (store, params = {}) => {
    return store.dispatch('GET_POSTS_COUNT', {
      params: params,
    })
  }

  export default {
    name: 'ManageGuestEditor',
    metaInfo () {
      return {
        isTappayNeeded: this.isTappayRequired,
      }
    },       
    components: {
      'alert-panel': AlertPanel,
      'app-tab': Tab,
      'base-light-box': BaseLightBox,
      'base-light-box-profile': ProfileEdit,
      'control-bar': TheControlBar,
      'following-list-tab': FollowingListInTab,
      'pagination-nav': PaginationNav,
      'post-list': PostList,
      'post-list-detailed': PostListDetailed,
      'post-list-tab': PostListInTab,
      'post-panel': PostPanel,
      PointManager,
    },
    props: {
      showControlBar: {
        type: Boolean,
      },
    },
    data () {
      return {
        activePanel: 'records',
        activeTab: 'reviews',
        alertType: 'post',
        config: {
          type: POST_TYPE,
        },
        defaultTab: 0,
        isPublishPostInEditor: false,
        itemsStatus: undefined,
        itemsSelected: [],
        loading: false,
        needConfirm: false,
        page: DEFAULT_PAGE,
        post: {},
        postStatusChanged: false,
        postPanel: 'add',
        postType: POST_TYPE.REVIEW,
        sort: DEFAULT_SORT,
        showAlert: false,
        showDraftList: false,
        showEditor: false,
        showProfile: false,
      }
    },
    computed: {
      isDonationActive () { 
        return _.get(this.$store, 'state.setting.DONATION_IS_DEPOSIT_ACTIVE', false) 
      },   
      isTappayRequired () {
        return _.get(this.$store, 'state.isTappayRequired', false)
      },                
      itemsSelectedID () {
        const items = []
        _.forEach(this.itemsSelected, (item) => {
          items.push(item.id)
        })
        return items
      },
      posts () {
        return _.get(this.$store, [ 'state', 'posts', ], [])
      },
      postsDraft () {
        return _.get(this.$store, [ 'state', 'postsDraft', ], [])
      },
      profile () {
        return _.get(this.$store, [ 'state', 'profile', ], {})
      },
      tabs () {
        const defaultTabs = [
          this.$t('TAB.REVIEW_RECORD'),
          this.$t('TAB.NEWS_RECORD'),
          this.$t('TAB.FOLLOW_RECORD'),
        ]
        this.isDonationActive && defaultTabs.push(this.$t('TAB.POINTS_RECORD')) 
        return defaultTabs
      },          
      tags () {
        return _.get(this.$store, [ 'state', 'tags', ], [])
      },
    },
    watch: {
      isTappayRequired () {
        this.$forceUpdate()
      },      
    },
    beforeMount () {
      this.loading = true
      Promise.all([
        getPostsByUser(this.$store, {
          where: {
            author: _.get(this.profile, [ 'id', ]),
            type: POST_TYPE.REVIEW,
          },
        }),
        getPostsCount(this.$store, {
          where: {
            author: _.get(this.profile, [ 'id', ]),
            type: POST_TYPE.REVIEW,
          },
        }),
      ])
      .then(() => this.loading = false)
      .catch(() => this.loading = false)
      
      if (_.get(this.$route, 'params.panel')) { 
        switch (_.get(this.$route, 'params.panel')) {
          case 'profile-edit':
            this.showProfile = true
            break
          default:
            this.activePanel = _.get(this.$route, 'params.panel')
        }
        switch (_.get(this.$route, 'params.tool')) {
          case 'following':
            this.defaultTab = 2
            break
          case 'point-manager':
            this.isDonationActive && (this.defaultTab = 3)
            break
        }
      }        
    },
    methods: {
      $_guestEditor_alertHandler (showAlert) {
        this.showAlert = showAlert
      },
      $_guestEditor_closeControlBar () {
        this.$emit('closeControlBar')
      },
      $_guestEditor_deletePosts () {
        deletePost(this.$store, this.itemsSelectedID)
          .then(() => {
            this.$_guestEditor_updatePostList({ needUpdateCount: true, })
            this.showEditor = false
            this.showDraftList = false
            this.needConfirm = false
          })
          .catch(() => {
            this.alertType = 'error'
            this.needConfirm = false
            this.showAlert = true
          })
      },
      $_guestEditor_filterHandler ({ sort = this.sort, page = this.page, }) {
        switch (this.activePanel) {
          case 'records':
            return this.$_guestEditor_updatePostList({ sort: sort, page: page, })
        }
      },
      $_guestEditor_openPanel (panel) {
        this.loading = true
        this.activePanel = panel
        this.sort = DEFAULT_SORT
        this.page = DEFAULT_PAGE
        switch (this.activePanel) {
          case 'records':
            this.alertType = 'post'
            Promise.all([
              getPostsByUser(this.$store, {
                where: { author: _.get(this.profile, [ 'id', ]), type: POST_TYPE.REVIEW, },
              }),
              getPostsCount(this.$store, {
                where: { author: _.get(this.profile, [ 'id', ]), type: POST_TYPE.REVIEW, },
              }),
            ])
            .then(() => this.loading = false)
            .catch(() => this.loading = false)
            break
        }
      },
      $_guestEditor_showAlert (ids, itemsStatus) {
        this.itemsSelected = []

        const unionPosts = _.unionBy(this.posts, this.postsDraft, 'id')
        this.postStatusChanged = true 
        this.isPublishPostInEditor = false
        this.itemsStatus = itemsStatus
        this.itemsSelected = _.filter(unionPosts, (o) => {
          return _.includes(ids, o.id)
        })

        this.needConfirm = true
        this.showAlert = true
      },
      $_guestEditor_showDraftList (type) {
        this.loading = true
        switch (type) {
          case POST_TYPE.REVIEW:
            Promise.all([
              getPostsByUser(this.$store, {
                where: {
                  author: _.get(this.profile, [ 'id', ]),
                  publish_status: POST_PUBLISH_STATUS.DRAFT,
                  type: POST_TYPE.REVIEW,
                },
              }),
              getPostsCount(this.$store, {
                where: {
                  author: _.get(this.profile, [ 'id', ]),
                  publish_status: POST_PUBLISH_STATUS.DRAFT,
                  type: POST_TYPE.REVIEW,
                },
              }),
            ])
            .then(() => {
              this.loading = false
              this.showDraftList = true
            })
            .catch(() => this.loading = false)
            break
          case POST_TYPE.NEWS:
            Promise.all([
              getPostsByUser(this.$store, {
                where: {
                  author: _.get(this.profile, [ 'id', ]),
                  publish_status: POST_PUBLISH_STATUS.DRAFT,
                  type: POST_TYPE.NEWS,
                },
              }),
              getPostsCount(this.$store, {
                where: {
                  author: _.get(this.profile, [ 'id', ]),
                  publish_status: POST_PUBLISH_STATUS.DRAFT,
                  type: POST_TYPE.NEWS,
                },
              }),
            ])
            .then(() => {
              this.loading = false
              this.showDraftList = true
            })
            .catch(() => this.loading = false)
            break
        }
      },
      $_guestEditor_showEditor ({ postPanel, id, postType, }) {
        this.itemsSelected = []
        this.alertType = 'post'
        if (postPanel === 'add') {
          this.post = {}
          this.postType = postType
        } else {
          this.post = _.find(this.posts, { 'id': id, }) || _.find(this.postsDraft, { 'id': id, })
          this.postType = _.get(this.post, [ 'type', ])
          this.itemsSelected.push(this.post)
        }
        this.postPanel = postPanel
        this.showEditor = true
      },
      $_guestEditor_showProfile () {
        this.showProfile = true
      },
      $_guestEditor_tabHandler (tab) {
        this.loading = true
        switch (tab) {
          case 0:
            this.activeTab = 'reviews'
            Promise.all([
              getPostsByUser(this.$store, {
                where: { author: _.get(this.profile, [ 'id', ]), type: POST_TYPE.REVIEW, },
              }),
              getPostsCount(this.$store, {
                where: { author: _.get(this.profile, [ 'id', ]), type: POST_TYPE.REVIEW, },
              }),
            ])
            .then(() => this.loading = false)
            .catch(() => this.loading = false)
            break
          case 1:
            this.activeTab = 'news'
            Promise.all([
              getPostsByUser(this.$store, {
                where: { author: _.get(this.profile, [ 'id', ]), type: POST_TYPE.NEWS, },
              }),
              getPostsCount(this.$store, {
                where: { author: _.get(this.profile, [ 'id', ]), type: POST_TYPE.NEWS, },
              }),
            ])
            .then(() => this.loading = false)
            .catch(() => this.loading = false)
            break
          case 2:
            this.activeTab = 'followings'
            break
        }
      },
      $_guestEditor_updatePostList ({ sort, page, needUpdateCount = false, } ={}) {
        this.sort = sort || this.sort
        this.page = page || this.page
        
        switch (this.activeTab) {
          case 'reviews':
            if (needUpdateCount) {
              getPostsCount(this.$store, {
                where: { author: _.get(this.profile, [ 'id', ]), type: POST_TYPE.REVIEW, },
              })
            }
            getPostsByUser(this.$store, {
              page: this.page,
              where: { author: _.get(this.profile, [ 'id', ]), type: POST_TYPE.REVIEW, },
            })
            break
          case 'news':
            if (needUpdateCount) {
              getPostsCount(this.$store, {
                where: { author: _.get(this.profile, [ 'id', ]), type: POST_TYPE.NEWS, },
              })
            }
            getPostsByUser(this.$store, {
              page: this.page,
              where: { author: _.get(this.profile, [ 'id', ]), type: POST_TYPE.NEWS, },
            })
            break
        }
      },
    },
  }
</script>
<style lang="stylus" scoped>
</style>