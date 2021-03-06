<template>
  <div class="tagList">
    <div class="tagList__control">
      <div class="tagList__add">
        <input v-model="addTag" type="text" :placeholder="$t('TAG_LIST.ADD_PLACEHOLDER')">
        <button :disabled="!tagNameValidated" @click="$_tagList_addTag" v-text="$t('TAG_LIST.ADD')"></button>
      </div>
      <pagination-nav :totalPages="totalPages" :currPage.sync="currPage"></pagination-nav>
    </div>
    <table>
      <thead>
        <tr>
          <th class="tagList__checkbox"><input type="checkbox" ref="checkboxSelectAll"  @click="$_tagList_toggleSelectAll"></th>
          <th class="tagList__text"><span @click="$_tagList_orderBy('text')" v-text="$t('TAG_LIST.TAG')"></span></th>
          <th class="tagList__count"><span @click="$_tagList_orderBy('related_reviews')" v-html="`${$t('TAG_LIST.RELATED')}<br>${$t('TAG_LIST.REVIEW_COUNT')}`"></span></th>
          <th class="tagList__count"><span @click="$_tagList_orderBy('related_news')" v-html="`${$t('TAG_LIST.RELATED')}<br>${$t('TAG_LIST.NEWS_COUNT')}`"></span></th>
          <!-- <th class="tagList__edit"></th> -->
          <th class="tagList__delete">
            <button class="tagList__btn tagList__btn--multiple" v-text="$t('TAG_LIST.DELETE')" @click="$_postList_deleteTags"></button>
          </th>
          <!-- <th class="tagList__sort">
            <select name="" id="">
              <option value="-updated_at" v-text="$t('tag_list.WORDING_TAGLIST_UPDATE_AT')"></option>
              <option value="-created_at" v-text="$t('tag_list.WORDING_TAGLIST_PUBLISH_AT')"></option>
            </select>
          </th> -->
        </tr>
      </thead>
      <tbody>
        <tr v-for="t in tags" :key="t.id">
          <td><input type="checkbox" ref="checkboxItems" @change="$_tagList_toggleHandler($event, t.id)"></td>
          <td ref="tagTextBlock" class="tagList__text">
            <span ref="originTagText" @click="$_tagList_editTag" v-text="t.text"></span>
            <input ref="inputTagText" type="text" :value="t.text">
            <button @click="$_tagList_confirmEdit($event, t.id)" v-text="$t('TAG_LIST.CONFIRM')"></button>
            <button @click="$_tagList_cancel" v-text="$t('TAG_LIST.CANCEL')"></button>
          </td>
          <td class="tagList__count" v-text="t.relatedReviews"></td>
          <td class="tagList__count" v-text="t.relatedNews"></td>
          <!-- <td class="tagList__edit"><button class="tagList__btn tagList__btn--single" @click="$_tagList_editTagBtn" v-text="$t('tag_list.WORDING_TAGLIST_EDIT')"></button></td> -->
          <!-- <td class="tagList__delete"><button class="tagList__btn tagList__btn--single" @click="$_tagList_deleteTag(t.id)" v-text="$t('tag_list.WORDING_TAGLIST_DELETE')"></button></td> -->
          <td></td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
<script>
  import _ from 'lodash'
  import PaginationNav from './PaginationNav.vue'

  const updateTags = (store, id, text) => {
    return store.dispatch('UPDATE_TAGS', {
      params: {
        id: id,
        text: text,
        updated_by: _.get(store, [ 'state', 'profile', 'id', ]),
      },
    })
  }

  export default {
    name: 'TagList',
    components: {
      'pagination-nav': PaginationNav,
    },
    props: {
      maxResult: {
        type: Number,
        required: true,
      },
      sort: {
        type: String,
        required: true,
      },
      tags: {
        type: Array,
        required: true,
      },
    },
    data () {
      return {
        addTag: '',
        checkedIems: [],
        currPage: 1,
      }
    },
    computed: {
      tagNameValidated () {
        return this.addTag.trim()
      },
      totalPages () {
        return Math.ceil(_.get(this.$store, [ 'state', 'tagsCount', ], 0) / this.maxResult)
      },
    },
    watch: {
      tags () {
        this.addTag = ''
      },
      currPage () {
        this.$emit('filterChanged', { page: this.currPage, })
      },
    },
    methods: {
      $_tagList_addTag () {
        if (this.tagNameValidated) {
          this.$emit('addTag', this.addTag.trim())
        }
      },
      $_tagList_cancel (event) {
        event.target.parentNode.classList.toggle('modify')
      },
      $_tagList_confirmEdit (event, id) {
        const newText = event.target.parentNode.querySelector('input').value
        const originText = event.target.parentNode.querySelector('span').innerText
        if (newText === originText) {
          event.target.parentNode.classList.toggle('modify')
        } else {
          updateTags(this.$store, id, newText)
          .then(() => {
            event.target.parentNode.classList.toggle('modify')
            this.$emit('updateTagList')
          })
        }
      },
      $_tagList_deleteTag (id) {
        this.$emit('deleteTags', [ id, ])
      },
      $_postList_deleteTags () {
        this.$emit('deleteTags', this.checkedIems)
      },
      $_tagList_editTag (event) {
        event.target.parentNode.classList.toggle('modify')
      },
      $_tagList_editTagBtn (event) {
        event.target.parentNode.parentNode.querySelector('.tagList__text').classList.toggle('modify')
      },
      $_tagList_orderBy (field) {
        let order
        if (this.sort === field || this.sort === `-${field}`) {
          order = this.sort.indexOf('-') === 0 ? field : `-${field}`
        } else {
          order = field
        }
        this.$emit('filterChanged', { sort: order, })
      },
      $_tagList_toggleHandler (event, id) {
        if (event.target.checked) {
          this.checkedIems.push(id)
        } else {
          const index = this.checkedIems.indexOf(id)
          if (index > -1) {
            this.checkedIems.splice(index, 1)
          }
        }
      },
      $_tagList_toggleSelectAll () {
        this.checkedIems = []
        _.map(this.$refs.checkboxItems, (item) => {
          item.checked = this.$refs.checkboxSelectAll.checked
        })
        if (this.$refs.checkboxSelectAll.checked) {
          _.map(this.tags, (item) => {
            this.checkedIems.push(item.id)
          })
          this.checkedIems = _.uniq(this.checkedIems)
        }
      },
    },
  }
</script>
<style lang="stylus" scoped>
.tagList
  table
    width 100%
    margin-top 10px
    text-align left
    border-collapse collapse
    thead
      color #808080
      font-size .9375rem
      input
        padding 15px 0
    tbody
      font-size .875rem
    th
      padding-bottom 5px
      border-bottom 2px solid #000
      span
        position relative
        cursor pointer
        &::before
          content ''
          position absolute
          top 0
          left 100%
          width 15px
          height 100%
          background-position center center
          background-repeat no-repeat
          background-size 7px auto
          background-image url(/public/icons/double-triangle.png)
    tr
      &:first-of-type
        td
          padding-top 10px
      &:last-of-type
        td
          border-bottom none
    td
      padding-top 5px
      padding-bottom 5px
      border-bottom 1px solid #d3d3d3
    input[type="checkbox"]
      width 15px
      height 15px
  &__control
    display flex
    flex-direction column
    justify-content space-between
  &__add
    display flex
    margin-bottom 10px
    input
      flex 1
      height 25px
      padding 0 0 0 10px
      color #808080
      font-size 15px
      font-weight 300
      border 1px solid #d3d3d3
    button 
      width 60px
      height 25px
      margin 0 0 0 10px
      color #fff
      background-color #4280a2
      border none
      border-radius 4px
    button:disabled
      background-color #ccc
  &__checkbox
    width 15px
    padding-right 10px
    line-height 18px
    vertical-align baseline
  &__text
    position relative
    // width 250px
    > span
      cursor pointer
    > input, > button
      display none
    &.modify
      > span
        display none
      > input
        display inline-block
        width 120px
        height 22px
        padding-left 5px
        margin-right 5px
      > button
        display inline-block
        height 22px
        margin 0 2px
        background transparent
        border-radius 4px
        outline none
  &__delete
    width 50px
    text-align center
  &__count
    width 65px
    padding-right 15px
    text-align center
  &__sort
    width 105px
    select
      height 25px
      line-height 25px
  &__btn
    background-color transparent
    border none
    outline none
    &--single
      color #4280a2
    &--multiple
      height 25px
      color #fff
      line-height 20px
      background-color #808080
      border-radius 4px
</style>
