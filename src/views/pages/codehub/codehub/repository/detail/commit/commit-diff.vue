<template>
  <div class="pl-nm">
    <div v-if="diffList && diffList.length">
      <DiffDetail
        :showType="showType"
        :diffList="diffList"
        :supportTypeList="supportTypeList"
        :leftCommitId="leftCommitId"
        :rightCommitId="rightCommitId"
        :queryName="queryName"
        :queryType="queryType"
        :repositoryId="id"
        :appModuleId="appModuleId"
        readOnly
        @endScroll="endScoll"
        @hasFixtop="showFixtop"
        @getMore="getMore"
      ></DiffDetail>
    </div>
    <div v-else class="bg-grey border-color" style="line-height: 2;padding: 4px;border: 1px solid;">{{ $t('page.nodata') }}</div>
  </div>
</template>
<script>
export default {
  name: '',
  components: {
    DiffDetail: resolve => require(['@/views/pages/codehub/codehub/merge/review/tab/diff/diff-detail.vue'], resolve)
  },
  filters: {},
  props: {
    diffInfo: Object,
    leftCommitId: String,
    rightCommitId: String,
    id: Number,
    appModuleId: Number,
    queryName: String,
    queryType: String
  },
  provide() {
    return {
      appModuleId: this.appModuleId || null,
      repositoryId: this.id || null,
      branchname: null
    };
  },
  data() {
    return {
      diffList: [],
      showType: 'combine', //代码对比显示的模式
      supportTypeList: [
        //支持哪些文件类型的展示
        'css',
        'bat',
        'cls',
        'cnf',
        'bmp',
        'cmd',
        'gzip',
        'doc',
        'exe',
        'gif',
        'mov',
        'html',
        'mp4',
        'java',
        'jpg',
        'png',
        'pdf',
        'py',
        'pptx',
        'ppt',
        'js',
        'rar',
        'rtf',
        'docx',
        'rpm',
        'txt',
        'jpeg',
        'sql',
        'sh',
        'zip',
        'tar',
        'vbs',
        'xls',
        'xlsm',
        'xml',
        'xlsx',
        'svg'
      ]
    };
  },
  beforeCreate() {},
  created() {},
  beforeMount() {},
  mounted() {},
  beforeUpdate() {},
  updated() {},
  activated() {},
  deactivated() {},
  beforeDestroy() {},
  destroyed() {},
  methods: {
    endScoll() {},
    showFixtop() {},
    getMore(path, index) {
      let param = {
        repositoryId: this.id,
        filePath: path,
        rightCommitId: this.rightCommitId
      };

      if (this.diffList && this.diffList[index]) {
        Object.assign(this.diffList[index], {
          loadingMore: true
        });
      }
      this.$api.codehub.repositorydetail
        .getCommitDiff(param)
        .then(res => {
          Object.assign(this.diffList[index], {
            loadingMore: false
          });
          if (res.Status == 'OK') {
            if (this.diffList && this.diffList[index]) {
              let hunks = res.Return.fileDiffList[0];
              Object.assign(this.diffList[index], hunks);
            }
          }
        })
        .catch(res => {
          Object.assign(this.diffList[index], {
            loadingMore: false
          });
        });
    }
  },
  computed: {},
  watch: {
    diffInfo: {
      handler: function(val) {
        if (val && val.fileDiffList) {
          this.diffList = val.fileDiffList;
        } else {
          this.diffList = [];
        }
      },
      immediate: true,
      deep: true
    }
  }
};
</script>
<style lang="less"></style>
