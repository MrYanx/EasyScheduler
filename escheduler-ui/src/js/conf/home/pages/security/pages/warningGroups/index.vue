<template>
  <div class="main-layout-box">
    <m-secondary-menu :type="'security'"></m-secondary-menu>
    <template>
      <m-list-construction :title="'Warning group manage'">
        <template slot="conditions">
          <m-conditions @on-conditions="_onConditions">
            <template slot="button-group">
              <x-button type="ghost" size="small" @click="_create('')">{{$t('Create alarm group')}}</x-button>
            </template>
          </m-conditions>
        </template>
        <template slot="content">
          <template v-if="alertgroupList.length">
            <m-list :alertgroup-list="alertgroupList" :page-no="searchParams.pageNo" :page-size="searchParams.pageSize"></m-list>
            <div class="page-box">
              <x-page :current="parseInt(searchParams.pageNo)" :total="total" :page-size="searchParams.pageSize" show-elevator @on-change="_page"></x-page>
            </div>
          </template>
          <template v-if="!alertgroupList.length">
            <m-no-data></m-no-data>
          </template>
          <m-spin :is-spin="isLoading"></m-spin>
        </template>
      </m-list-construction>
    </template>
  </div>
</template>
<script>
  import _ from 'lodash'
  import { mapActions } from 'vuex'
  import mList from './_source/list'
  import mSpin from '@/module/components/spin/spin'
  import mCreateWarning from './_source/createWarning'
  import mNoData from '@/module/components/noData/noData'
  import listUrlParamHandle from '@/module/mixin/listUrlParamHandle'
  import mConditions from '@/module/components/conditions/conditions'
  import mSecondaryMenu from '@/module/components/secondaryMenu/secondaryMenu'
  import mListConstruction from '@/module/components/listConstruction/listConstruction'

  export default {
    name: 'warning-groups-index',
    data () {
      return {
        total: null,
        isLoading: false,
        alertgroupList: [],
        searchParams: {
          pageSize: 10,
          pageNo: 1,
          searchVal: ''
        }
      }
    },
    mixins: [listUrlParamHandle],
    props: {},
    methods: {
      ...mapActions('security', ['getAlertgroupP']),
      /**
       * Inquire
       */
      _onConditions (o) {
        this.searchParams = _.assign(this.searchParams, o)
        this.searchParams.pageNo = 1
      },
      _page (val) {
        this.searchParams.pageNo = val
      },
      _create (item) {
        let self = this
        let modal = this.$modal.dialog({
          closable: false,
          showMask: true,
          escClose: true,
          className: 'v-modal-custom',
          transitionName: 'opacityp',
          render (h) {
            return h(mCreateWarning, {
              on: {
                onUpdate () {
                  self._debounceGET('false')
                  modal.remove()
                },
                close () {
                  modal.remove()
                }
              },
              props: {
                item: item
              }
            })
          }
        })
      },
      _getList (flag) {
        this.isLoading = !flag
        this.getAlertgroupP(this.searchParams).then(res => {
          this.alertgroupList = []
          this.alertgroupList = res.totalList
          this.total = res.total
          this.isLoading = false
        }).catch(e => {
          this.isLoading = false
        })
      }
    },
    watch: {
      // router
      '$route' (a) {
        // url no params get instance list
        this.searchParams.pageNo = _.isEmpty(a.query) ? 1 : a.query.pageNo
      }
    },
    created () {
    },
    mounted () {
    },
    components: { mSecondaryMenu, mList, mListConstruction, mConditions, mSpin, mNoData }
  }
</script>
