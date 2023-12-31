<template>
  <div>
    <TsContain class="bg-block">
      <template slot="topLeft">
        <span class="tsfont-plus text-action" @click="addMerge()">MR</span>
      </template>
      <template slot="topRight">
        <CombineSearcher
          v-model="searchVal"
          v-bind="searchConfig"
          class="mr-xs"
          @change="() => changeCurrent(1)"
        ></CombineSearcher>
      </template>
      <div slot="content">
        <div v-if="mergeTypeList.length>0">
          <Loading v-if="isLoad" loadingShow type="fix"></Loading>
          <Tabs v-model="activetab" class="block-tabs" :animated="false">
            <TabPane
              v-for="(tab,tabindex) in mergeTypeList"
              :key="tabindex"
              :label="renderTabLabel(tab)"
              :name="tab.status"
            >
              <MergeTable
                v-if="activetab == tab.status && !isLoad"
                :mergeData="mergeData"
                :mergeTypeList="mergeTypeList"
                @changePageSize="changePageSize"
                @changeCurrent="changeCurrent"
              ></MergeTable>
            </TabPane>
          </Tabs>
        </div>
      </div>
    </TsContain>
    <MergeAddDialog
      v-if="isShowMergeAddDialog"
      @close="close"
    ></MergeAddDialog>
  </div>
</template>
<script>
export default {
  name: 'MergeOverview',
  components: {
    CombineSearcher: resolve => require(['@/resources/components/CombineSearcher/CombineSearcher.vue'], resolve),
    MergeAddDialog: resolve => require(['./overview/merge-add-dialog.vue'], resolve),
    MergeTable: resolve => require(['./overview/merge-table.vue'], resolve)
  },
  filters: {},
  props: [''],
  data() {
    return {
      mergeData: {
        currentPage: 1,
        pageSize: 20
      },
      mergeTypeList: [],
      activetab: 'open',
      isLoad: true,
      isShowMergeAddDialog: false,
      searchVal: {},
      searchConfig: {
        search: true,
        searchList: [
          {
            name: 'appSystemId',
            type: 'select',
            label: this.$t('page.system'),
            transfer: true,
            dynamicUrl: '/api/rest/codehub/appsystem/search',
            rootName: 'tbodyList',
            dealDataByUrl: this.$utils.getAppForselect,
            onChange: (val) => {
              this.changeAppModule(val);
            }
          },
          {
            name: 'appModuleId',
            type: 'select',
            label: this.$t('page.subsystem'),
            transfer: true,
            rootName: 'tbodyList',
            dealDataByUrl: this.$utils.getAppForselect
          }
        ]
      }
    };
  },
  beforeCreate() {},
  created() {
    this.getMergeList();
  },
  beforeMount() {},
  mounted() {},
  beforeUpdate() {},
  updated() {},
  activated() {},
  deactivated() {},
  beforeDestroy() {},
  destroyed() {},
  methods: {
    changePageSize(pageSize) {
      this.mergeData.currentPage = 1;
      this.mergeData.pageSize = pageSize;
      this.getMergeList();
    },
    changeCurrent(currentPage) {
      this.mergeData.currentPage = currentPage;
      this.getMergeList();
    },
    changeAppModule(appSystemId) {
      if (appSystemId) {
        this.searchConfig.searchList.forEach((item) => {
          if (item && (item.name == 'appModuleId')) {
            this.$set(item, 'params', {appSystemId: appSystemId});
            this.$set(item, 'dynamicUrl', '/api/rest/codehub/appmodule/search');
          } 
        });
      } else {
        this.searchConfig.searchList.forEach((item) => {
          if (item && (item.name == 'appModuleId')) {
            this.$set(item, 'params', {});
            this.$set(item, 'dynamicUrl', '');
          } 
        });
      }
    },
    restoreHistory(historyData) {
      this.activetab = historyData['activetab'];
      this.searchVal = historyData['searchVal'];
      if (historyData['searchPage']) {
        this.mergeData.currentPage = historyData['searchPage']['currentPage'];
        this.mergeData.pageSize = historyData['searchPage']['pageSize'];
      }
    },
    async getMergeList() {
      await this.getStatuslist();
      let param = {
        currentPage: this.mergeData.currentPage,
        pageSize: this.mergeData.pageSize,
        ...this.searchVal
      };
      if (this.activetab) {
        this.$set(param, 'status', this.activetab == 'all' ? '' : this.activetab);
      }
      this.isLoad = true;
      this.$addHistoryData('activetab', this.activetab);
      this.$addHistoryData('searchVal', this.searchVal);
      this.$addHistoryData('searchPage', {currentPage: this.mergeData.currentPage, pageSize: this.mergeData.pageSize});
      this.$api.codehub.merge.getList(param).then(res => {
        if (res && res.Status == 'OK') {
          this.mergeData.currentPage = res.Return.currentPage;
          this.mergeData.pageCount = res.Return.pageCount;
          this.mergeData.pageSize = res.Return.pageSize;
          this.mergeData.rowNum = res.Return.rowNum;
          this.mergeData.tbodyList = res.Return.tbodyList || [];
        } else {
          this.mergeData.tbodyList = [];
        }
      }).finally(() => {
        this.isLoad = false;
      });
    },
    addMerge() {
      this.isShowMergeAddDialog = true;
    },
    getStatuslist() {
      let param = {
        ...this.searchVal
      };
      return this.$api.codehub.merge.getStatusCount(param).then(res => {
        if (res && res.Status == 'OK') {
          this.mergeTypeList = res.Return.list;
        } else {
          this.mergeTypeList = [];
        }
      });
    },
    close() {
      this.isShowMergeAddDialog = false;
    }
  },
  computed: {
    renderTabLabel(h) {
      return function(tab) {
        return (h) => {
          return h('div', [
            h('span', tab.text),
            h('span', {
              style: {
                'font-size': '12px',
                'padding-left': '4px'
              }
            }, tab.count ? (tab.count > 99 ? '99+' : tab.count.toString()) : '0')
          ]);
        };
      };
    }
  },
  watch: {
    activetab() {
      this.changeCurrent(1);
    },
    $route: {
      handler: function(val) {
        if (val && val.name == 'merge-overview') {
          this.getMergeList();
        }
      }
    }
  }
};
</script>
<style lang="less" scoped>
</style>
