<template>
  <div class="backstage member">
    <control-bar
      @closeControlBar="$_member_closeControlBar"
      @editProfile="$_member_showProfile"
      :showControlBar="showControlBar"
    />
    <main class="backstage-container">
      <section class="backstage__records">
        <app-tab class="backstage__tab" :tabs="tabs" :defaultTab="defaultTab">
          <following-list-tab slot="0"></following-list-tab>
          <PointManager slot="1" v-if="isDonationActive"></PointManager>
        </app-tab>
      </section>
    </main>
    <base-light-box borderStyle="nonBorder" :showLightBox.sync="showProfile">
      <base-light-box-profile :profile="profile" :showLightBox="showProfile" @save="showProfile = false"></base-light-box-profile>
    </base-light-box>
  </div>
</template>
<script>
  import BaseLightBox from '../components/BaseLightBox.vue'
  import FollowingListInTab from '../components/FollowingListInTab.vue'
  import PointManager from 'src/components/point/PointManager.vue'  
  import ProfileEdit from '../components/member/ProfileEdit.vue'
  import Tab from '../components/Tab.vue'
  import TheControlBar from '../components/TheControlBar.vue'
  import { get, } from 'lodash'

  // const MAXRESULT = 20
  const DEFAULT_PAGE = 1
  // const DEFAULT_SORT = '-updated_at'
  
  export default {
    name: 'ManageMember',
    metaInfo () {
      return {
        isTappayNeeded: this.isTappayRequired,
      }
    },     
    components: {
      'app-tab': Tab,
      'base-light-box': BaseLightBox,
      'base-light-box-profile': ProfileEdit,
      'control-bar': TheControlBar,
      'following-list-tab': FollowingListInTab,
      PointManager,
    },
    props: {
      showControlBar: {
        type: Boolean,
      },
    },
    data () {
      return {
        defaultTab: 0,
        page: DEFAULT_PAGE,
        showProfile: false,
      }
    },
    computed: {
      isDonationActive () { 
        return get(this.$store, 'state.setting.DONATION_IS_DEPOSIT_ACTIVE', false) 
      },        
      isTappayRequired () {
        return get(this.$store, 'state.isTappayRequired', false)
      },         
      profile () {
        return get(this.$store, [ 'state', 'profile', ], {})
      },
      tabs () {
        const defaultTabs = [
          this.$t('TAB.FOLLOW_RECORD'),
        ]
        this.isDonationActive && defaultTabs.push(this.$t('TAB.POINTS_RECORD')) 
        return defaultTabs
      },        
    },
    watch: {
      isTappayRequired () {
        this.$forceUpdate()
      },
    },
    methods: {
      $_member_closeControlBar () {
        this.$emit('closeControlBar')
      },
      $_member_showProfile () {
        this.showProfile = true
      },
    },
    beforeMount () {
      if (get(this.$route, 'params.panel')) { 
        switch (get(this.$route, 'params.panel')) {
          case 'profile-edit':
            this.showProfile = true
            break
          default:
            this.activePanel = get(this.$route, 'params.panel')
        }
        switch (get(this.$route, 'params.tool')) {
          case 'following':
            this.defaultTab = 0
            break
          case 'point-manager':
            this.isDonationActive && (this.defaultTab = 1)
            break
        } 
      }        
    },
  }
</script>
<style lang="stylus" scoped>
</style>
