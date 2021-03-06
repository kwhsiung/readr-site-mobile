<template>
  <div class="backstage editor">
    <control-bar
      @addNews="$_editor_showEditor({ postPanel: 'add', postType: config.type.NEWS })"
      @addReview="$_editor_showEditor({ postPanel: 'add', postType: config.type.REVIEW })"
      @addVideo="$_editor_showEditor({ postPanel: 'add', postType: config.type.VIDEO })"
      @closeControlBar="$_editor_closeControlBar"
      @editNews="$_editor_showDraftList(config.type.NEWS)"
      @editProfile="$_editor_showProfile"
      @editReview="$_editor_showDraftList(config.type.REVIEW)"
      @openPanel="$_editor_openPanel"
      :showControlBar="showControlBar"
    />
    <main class="backstage-container">
      <template v-if="activePanel === 'records'">
        <section class="backstage__records">
          <app-tab class="backstage__tab" :tabs="tabs" @changeTab="$_editor_tabHandler" :defaultTab="defaultTab">
            <post-list-tab
              slot="0"
              :posts="posts"
              @deletePost="$_editor_showAlert"
              @editPost="$_editor_showEditor"
              @filterChanged="$_editor_filterHandler">
            </post-list-tab>
            <post-list-tab
              slot="1"
              :posts="posts"
              @deletePost="$_editor_showAlert"
              @editPost="$_editor_showEditor"
              @filterChanged="$_editor_filterHandler">
            </post-list-tab>
            <following-list-tab slot="2"></following-list-tab>
            <PointManager slot="3" v-if="isDonationActive"></PointManager>
          </app-tab>
        </section>
      </template>
      <template v-else-if="activePanel === 'posts'">
        <section class="backstage__panel">
          <post-list
            :maxResult="20"
            :posts="posts"
            :sort="sort"
            @deletePosts="$_editor_showAlert"
            @editPost="$_editor_showEditor"
            @filterChanged="$_editor_filterHandler"
            @publishPosts="$_editor_showAlert">
          </post-list>
        </section>
      </template>
      <template v-else-if="activePanel === 'tags'">
        <section class="main-panel">
          <tag-list
            :maxResult="20"
            :sort="sort"
            :tags="tags"
            @addTag="$_editor_addTag"
            @deleteTags="$_editor_showAlert"
            @filterChanged="$_editor_filterHandler"
            @updateTagList="$_editor_updateTagList({})">
          </tag-list>
        </section>
      </template>
      <template v-else-if="activePanel === 'videos'">
        <section class="main-panel">
          <video-list
            :maxResult="20"
            :posts="posts"
            :sort="sort"
            @deletePosts="$_editor_showAlert"
            @editPost="$_editor_showEditor"
            @filterChanged="$_editor_filterHandler"
            @publishPosts="$_editor_showAlert">
          </video-list>
        </section>
      </template>
    </main>
    <base-light-box borderStyle="nonBorder" :showLightBox.sync="showProfile">
      <base-light-box-profile :profile="profile" :showLightBox="showProfile" @save="showProfile = false"></base-light-box-profile>
    </base-light-box>
    <base-light-box :showLightBox.sync="showDraftList">
      <post-list-detailed
        :posts="postsDraft"
        @deletePost="$_editor_showAlert"
        @editPost="$_editor_showEditor">
      </post-list-detailed>
    </base-light-box>
    <base-light-box class="text-editor" :showLightBox.sync="showEditor">
      <post-panel
        :action="postPanel"
        :editorType="postType"
        :initialPost="post"
        @closeEditor="showEditor = false"
        @updateList="$_editor_updatePostList">
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
        @deletePosts="$_editor_deletePosts"
        @deleteTags="$_editor_deleteTags"
        @closeAlert="$_editor_alertHandler(false)"
        @publishPosts="$_editor_publishPostHandler">
      </alert-panel>
    </base-light-box>
  </div>
</template>
<script>
  import { POST_PUBLISH_STATUS, POST_TYPE, TAG_ACTIVE, } from '../../api/config'
  import _ from 'lodash'
  import AlertPanelB from '../components/AlertPanel.vue'
  import BaseLightBox from '../components/BaseLightBox.vue'
  import FollowingListInTab from '../components/FollowingListInTab.vue'
  import PointManager from 'src/components/point/PointManager.vue'  
  import PostList from '../components/PostList.vue'
  import PostListDetailed from '../components/PostListDetailed.vue'
  import PostListInTab from '../components/PostListInTab.vue'
  import PostPanel from '../components/PostPanel.vue'
  import ProfileEdit from '../components/member/ProfileEdit.vue'
  import Tab from '../components/Tab.vue'
  import TagList from '../components/TagList.vue'
  import TheControlBar from '../components/TheControlBar.vue'
  import VideoList from '../components/VideoList.vue'

  const MAXRESULT = 20
  const DEFAULT_PAGE = 1
  const DEFAULT_SORT = '-updated_at'

  const addTags = (store, text = '') => {
    return store.dispatch('ADD_TAGS', {
      params: {
        text: text,
        updated_by: _.get(store, [ 'state', 'profile', 'id', ]),
      },
    })
  }

  const deletePosts = (store, { params, }) => {
    return store.dispatch('DELETE_POSTS', {
      params: params,
    })
  }

  const deleteTags = (store, ids = []) => {
    return store.dispatch('DELETE_TAGS', {
      params: {
        ids: ids,
        updated_by: _.get(store, [ 'state', 'profile', 'id', ]),
      },
    })
  }

  const getPosts = (store, {
    maxResult = MAXRESULT,
    page = DEFAULT_PAGE,
    sort = DEFAULT_SORT,
    where = {},
  } = {}) => {
    return store.dispatch('GET_POSTS', {
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

  const getTags = (store, {
    max_result = MAXRESULT,
    page = DEFAULT_PAGE,
    sort = DEFAULT_SORT,
    keyword = '',
    stats = false,
  } = {}) => {
    return store.dispatch('GET_TAGS', {
      params: {
        max_result: max_result,
        page: page,
        sort: sort,
        keyword: keyword,
        stats: stats,
      },
    })
  }

  const getTagsCount = (store, {
    keyword = '',
  } = {}) => {
    return store.dispatch('GET_TAGS_COUNT', {
      params: {
        keyword: keyword,
      },
    })
  }

  const publishPosts = (store, params) => {
    return store.dispatch('PUBLISH_POSTS', { params, })
  }

  export default {
    name: 'ManageEditor',
    metaInfo () {
      return {
        isTappayNeeded: this.isTappayRequired,
      }
    },       
    components: {
      'alert-panel': AlertPanelB,
      'app-tab': Tab,
      'base-light-box': BaseLightBox,
      'base-light-box-profile': ProfileEdit,
      'control-bar': TheControlBar,
      'following-list-tab': FollowingListInTab,
      'post-list': PostList,
      'post-list-detailed': PostListDetailed,
      'post-list-tab': PostListInTab,
      'post-panel': PostPanel,
      'tag-list': TagList,
      'video-list': VideoList,
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
        loading: true,
        needConfirm: false,
        page: DEFAULT_PAGE,
        post: {},
        postStatusChanged: false,
        postForPublishInEditor: {},
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
      $_editor_addTag (tagName) {
        this.itemsStatus = TAG_ACTIVE.ACTIVE
        this.needConfirm = false
        this.loading = true
        addTags(this.$store, tagName)
          .then(() => {
            this.$_editor_updateTagList({ needUpdateCount: true, })
            this.showAlert = true
            this.loading = false
          })
          .catch(() => {
            this.alertType = 'error'
            this.needConfirm = false
            this.showAlert = true
            this.loading = false
          })
      },
      $_editor_alertHandler (showAlert) {
        this.showAlert = showAlert
      },
      $_editor_closeControlBar () {
        this.$emit('closeControlBar')
      },
      $_editor_deletePosts () {
        deletePosts(this.$store, {
          params: {
            ids: this.itemsSelectedID,
            updated_by: _.get(this.$store.state, [ 'profile', 'id', ]),
          },
        }).then(() => {
          this.$_editor_updatePostList({ needUpdateCount: true, })
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
      $_editor_deleteTags () {
        deleteTags(this.$store, this.itemsSelectedID)
          .then(() => {
            this.$_editor_updateTagList({ needUpdateCount: true, })
            this.needConfirm = false
          })
          .catch(() => {
            this.alertType = 'error'
            this.needConfirm = false
            this.showAlert = true
          })
      },
      $_editor_filterHandler ({ keyword = '', sort = this.sort, page = this.page, }) {
        switch (this.activePanel) {
          case 'records':
          case 'posts':
          case 'videos':
            return this.$_editor_updatePostList({ sort: sort, page: page, })
          case 'tags':
            return this.$_editor_updateTagList({ keyword: keyword, sort: sort, page: page, needUpdateCount: true, })
        }
      },
      $_editor_openPanel (panel) {
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
          case 'posts':
            this.alertType = 'post'
            Promise.all([
              getPosts(this.$store, {
                where: { publish_status: [ POST_PUBLISH_STATUS.UNPUBLISHED, POST_PUBLISH_STATUS.PUBLISHED, POST_PUBLISH_STATUS.SCHEDULING, POST_PUBLISH_STATUS.PENDING, ], type: [ POST_TYPE.REVIEW, POST_TYPE.NEWS, POST_TYPE.REPORT, POST_TYPE.MEMO, ], },
              }),
              getPostsCount(this.$store, {
                where: { publish_status: [ POST_PUBLISH_STATUS.UNPUBLISHED, POST_PUBLISH_STATUS.PUBLISHED, POST_PUBLISH_STATUS.SCHEDULING, POST_PUBLISH_STATUS.PENDING, ], type: [ POST_TYPE.REVIEW, POST_TYPE.NEWS, POST_TYPE.REPORT, POST_TYPE.MEMO, ], },
              }),
              getTags(this.$store, { stats: true, }),
            ])
            .then(() => this.loading = false)
            .catch(() => this.loading = false)
            break
          case 'tags':
            this.alertType = 'tag'
            Promise.all([
              getTags(this.$store, { stats: true, }),
              getTagsCount(this.$store),
            ])
            .then(() => this.loading = false)
            .catch(() => this.loading = false)
            break
          case 'videos':
            this.alertType = 'video'
            Promise.all([
              getPosts(this.$store, {
                where: { type: POST_TYPE.VIDEO, },
              }),
              getPostsCount(this.$store, {
                where: { type: POST_TYPE.VIDEO, },
              }),
            ])
            .then(() => this.loading = false)
            .catch(() => this.loading = false)
            break
        }
      },
      $_editor_publishPostHandler () {
        this.alertType = 'post'
        this.loading = true
        const params = {}
        params.updated_by = _.get(this.$store.state, [ 'profile', 'id', ])
        params.ids = this.itemsSelectedID
        publishPosts(this.$store, params)
          .then(() => {
            this.$_editor_updatePostList({ needUpdateCount: true, })
            this.needConfirm = false
            this.loading = false
          })
          .catch(() => {
            this.alertType = 'error'
            this.needConfirm = false
            this.showAlert = true
            this.loading = false
          })
      },
      $_editor_showAlert (ids, itemsStatus) {
        this.itemsSelected = []
        const unionPosts = _.unionBy(this.posts, this.postsDraft, 'id')
        switch (this.alertType) {
          case 'post':
          case 'video':
            this.postStatusChanged = true
            this.isPublishPostInEditor = false
            this.itemsStatus = itemsStatus
            this.itemsSelected = _.filter(unionPosts, (o) => {
              return _.includes(ids, o.id)
            })
            break
          case 'tag':
            this.itemsStatus = TAG_ACTIVE.DEACTIVE
            this.itemsSelected = _.filter(this.tags, (o) => {
              return _.includes(ids, o.id)
            })
            break
        }
        this.needConfirm = true
        this.showAlert = true
      },
      $_editor_showDraftList (type) {
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
      $_editor_showEditor ({ postPanel, id, postType, }) {
        this.itemsSelected = []
        this.alertType = 'post'

        if (this.activePanel === 'videos') {
          this.alertType = 'video'
        }

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
      $_editor_showProfile () {
        this.showProfile = true
      },
      $_editor_tabHandler (tab) {
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
      $_editor_updatePostList ({ sort, page, needUpdateCount = false, } = {}) {
        this.sort = sort || this.sort
        this.page = page || this.page
        switch (this.activePanel) {
          case 'records':
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
            break
          case 'posts':
            if (needUpdateCount) {
              getPostsCount(this.$store, {
                where: { publish_status: [ POST_PUBLISH_STATUS.UNPUBLISHED, POST_PUBLISH_STATUS.PUBLISHED, POST_PUBLISH_STATUS.SCHEDULING, POST_PUBLISH_STATUS.PENDING, ], type: [ POST_TYPE.REVIEW, POST_TYPE.NEWS, POST_TYPE.REPORT, POST_TYPE.MEMO, ], },
              })
            }
            getPosts(this.$store, {
              page: this.page,
              sort: this.sort,
              where: { publish_status: [ POST_PUBLISH_STATUS.UNPUBLISHED, POST_PUBLISH_STATUS.PUBLISHED, POST_PUBLISH_STATUS.SCHEDULING, POST_PUBLISH_STATUS.PENDING, ], type: [ POST_TYPE.REVIEW, POST_TYPE.NEWS, POST_TYPE.REPORT, POST_TYPE.MEMO, ], },
            })
            .then(() => this.loading = false)
            .catch(() => this.loading = false)
            break
          case 'videos':
            if (needUpdateCount) {
              getPostsCount(this.$store, {
                where: { type: POST_TYPE.VIDEO, },
              })
            }
            getPosts(this.$store, {
              page: this.page,
              sort: this.sort,
              where: { type: POST_TYPE.VIDEO, },
            })
            .then(() => this.loading = false)
            .catch(() => this.loading = false)
            break
        }
      },
      $_editor_updateTagList ({ keyword = '', sort, page, needUpdateCount = false, }) {
        this.sort = sort || this.sort
        this.page = page || this.page
        if (needUpdateCount) {
          getTagsCount(this.$store, { keyword: keyword, })
        }
        getTags(this.$store, {
          keyword: keyword,
          page: this.page,
          sort: this.sort,
          stats: true,
        })
        .then(() => this.loading = false)
        .catch(() => this.loading = false)
      },
    },
  }
</script>
<style lang="stylus" scoped>
</style>