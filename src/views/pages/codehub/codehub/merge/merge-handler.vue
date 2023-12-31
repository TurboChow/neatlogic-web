<template>
  <div>
    <TsContain v-if="mrActionType =='standard'" class="bg-block">
      <template slot="top">
        <div class="clearfix">
          <div class="ts-angle-left d_f cursor-pointer" @click="gotoPrev()">{{ prevPath.name }}</div>
          <div v-if="mrData" class="d_f top-title">
            <h3 class="title">
              <span 
                class="overflow" 
                :style="setTitlewidth(mrData.status)"
                :title="mrData.description"
              >{{ mrData.description || $t('term.codehub.handlemergerequest') }}</span>
              <Tag 
                :color="setStatus('color',mrData.status)" 
                class="ml-sm status-tag"
              >
                {{ setStatus('text',mrData.status) }}
              </Tag>
              <Tooltip
                v-if="errorMessage && mrData.status == 'failed'"
                theme="light"
                max-width="400"
                transfer
              >
                <div class="text-error cursor-pointer ts-info"></div>
                <div slot="content">
                  <div v-html="errorMessage"></div>
                </div>
              </Tooltip>
              <div v-else-if="mrData.status == 'failed'" class="ivu-tooltip"><div class="ivu-tooltip-rel"><div class="text-error cursor-pointer ts-info" @click="showError()"></div></div></div>
            </h3>
            <div v-if="mrData" class="desc">
              <Tooltip v-if="showtips(mrData)" theme="light" max-width="300">
                <div>{{ setTxt(mrData, 'text') }}</div>
                <div slot="content">
                  <div>{{ setTxt(mrData, 'tips') }}</div>
                </div>
              </Tooltip>
              <span v-else>{{ setTxt(mrData, 'text') }}</span>
              <Tag class="mr-sm ml-nm status-tag" color="success">{{ mrData.versionVo.name }}</Tag>
              <span v-if="mrData && mrData.versionTypeStrategyRelationVo" class="text-tip ml-sm">{{ $t('page.sourcebranch') }}:</span>
              <span v-if="mrData && mrData.versionTypeStrategyRelationVo" class="ml-sm">{{ mrData.srcBranch }}</span>
              <span v-if="mrData && mrData.versionTypeStrategyRelationVo" class="text-tip ml-sm">{{ $t('page.targetbranch') }}:</span>
              <span v-if="mrData && mrData.versionTypeStrategyRelationVo" class="ml-sm">{{ mrData.targetBranch }}</span>
            </div>
          </div>
        </div>
      </template>
      <div slot="content">
        <div class="padding-md">
          <MergeDetail 
            :id="id" 
            :mrData="mrData" 
            :mrstatusList="statusList" 
            :statusList="issuestatusList" 
            @reload="getDetail" 
            @updateStatus="updateStatus"
            @revert="revertIssue"
          ></MergeDetail>
        </div>
      </div>
    </TsContain>
    <MergeRevert
      v-else-if="mrActionType =='revert'"
      :mrData="mrData"
      v-bind="revertConfig"
      @resetRevert="resetRevert"
    ></MergeRevert>
  </div>
</template>
<script>
export default {
  name: '',
  components: {
    TsContain: resolve => require(['@/resources/components/TsContain/TsContain.vue'], resolve),
    MergeRevert: resolve => require(['./merge-revert.vue'], resolve),
    MergeDetail: resolve => require(['./handler/detail.vue'], resolve)
  },
  props: [''],
  data() {
    return {
      id: null, //mrid
      mrData: null, //接口返回的mr数据
      prevPath: {
        //上一步
        path: '/merge-overview',
        name: 'MR列表'
      },
      statusList: [], //需单独通过接口获取的状态下拉，用于mr状态中文回显
      issuestatusList: [], //需单独通过接口获取的需求状态下拉，用于需求状态中文回显，最外层获取减少调用次数
      duringAction: false, //是否是调用接口的操作标志，操作中的禁用改操作按钮防止重复点击
      errorMessage: null, //错误信息
      mrActionType: 'standard', //mr的操作类型（标准的是正常的处理，如果是revert则是从当前mr处理正文切换为mr创建的正文但是数据一致）
      revertConfig: {//需撤回的issue数据
        issueList: [],
        srcBranch: null,
        targetBranch: null      
      }
    };
  },
  beforeCreate() {},
  created() {
    this.getStatuslist();
    this.getissueStatuslist();
    if (sessionStorage.getItem('mergehandlerPrev')) {
      let prevsetting = JSON.parse(sessionStorage.getItem('mergehandlerPrev'));
      if (prevsetting.router != '/') {
        this.prevPath = prevsetting;
      }
    }
    if (this.$route.query.id) {
      this.id = this.$route.query.id;
      this.getDetail();
    }
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
    gotoPrev() {
      this.$router.push({
        path: this.prevPath.path
      });
    },
    getDetail(status) {
      let param = { id: this.id };
      this.$api.codehub.merge.getDetail(param).then(res => {
        if (res && res.Status == 'OK') {
          this.mrData = res.Return;
        } else {
          this.mrData = null;
        }
      }).catch((e) => {
        this.mrData = null;
      });
    },
    getStatuslist() {
      this.$api.codehub.merge.getStatusList({ type: 'mr' }).then(res => {
        if (res && res.Status == 'OK') {
          this.statusList = res.Return.list;
        }
      });
    },
    getissueStatuslist() {
      this.$api.codehub.merge.getStatusList({ type: 'issue' }).then(res => {
        if (res && res.Status == 'OK') {
          this.issuestatusList = res.Return.list;
        }
      });
    },
    updateStatus(val) {
      this.$set(this.mrData, 'status', val);
    },
    revertMr() {
      if (this.mrData.versionTypeStrategyRelationVo && this.mrData.versionTypeStrategyRelationVo.versionStrategyType && this.mrData.versionTypeStrategyRelationVo.versionStrategyType == 'branch' && this.mrData.status == 'finish') {
        //分支类型且状态为已结束的才可以revert
        let param = { mrId: this.id };
        this.duringAction = true;
        this.$api.codehub.merge
          .revertMerge(param)
          .then(res => {
            this.duringAction = false;
            if (res.Status == 'OK') {
              this.$Message.success(this.$t('message.executesuccess'));
              this.getDetail();
            }
          })
          .catch(error => {
            this.duringAction = false;
          });
      }
    },
    showError() {
      //查看失败的mr的错误消息
      let param = {
        mrId: this.id 
      };
      this.$api.codehub.merge.getError(param).then((res) => {
        if (res.Status == 'OK' && res.Return) {
          this.errorMessage = res.Return.errorMessage || this.$t('term.codehub.thereiscurrentlynoerrormessage');
        } else {
          this.$Message.info(this.$t('term.codehub.thereiscurrentlynoerrormessage'));
        }
      });
    },
    resetRevert() {
      //从撤销需求那里返回
      this.mrActionType = 'standard';
      Object.assign(this.revertConfig, {
        issueList: [],
        srcBranch: null,
        targetBranch: null
      });
    },
    revertIssue(config) {
      Object.assign(this.revertConfig, {
        issueList: config.issueList,
        srcBranch: config.srcBranch,
        targetBranch: config.targetBranch
      });
      this.mrActionType = 'revert';
    }
  },
  filter: {},
  computed: {
    showtips() {
      //如果系统、子系统的描述都没有，不显示提示tooltip
      return function(config) {
        let isshow = false;
        if ((config.systemVo && config.systemVo.description) || (config.subSystemVo && config.subSystemVo.description)) {
          isshow = true;
        }
        return isshow;
      };
    },
    setTxt() {
      //回显对应的系统、子系统的描述
      return function(config, type) {
        let text = '';
        let prev = config.systemVo || '';
        let next = config.subSystemVo || '';
        if (type == 'text') {
          text = (prev ? prev.name : '') + (next ? '/' + next.name : '');
        } else if (type == 'tips') {
          text = (prev && prev.description ? prev.description : '') + (next && next.description ? '/' + next.description : '');
        }
        return text;
      };
    },
    setStatus() {
      //状态的颜色
      return function(type, val) {
        let text = '';
        this.statusList.length > 0 &&
          this.statusList.forEach(status => {
            if (status.value == val) {
              text = status[type];
            }
          });
        if (text == 'info') {
          text = '#00c1de';
        }
        return text;
      };
    },
    setTitlewidth() {
      return function(status) {
        let styles = {
          'display': 'inline-block',
          'max-width': status == 'failed' ? 'calc(100vw - 240px)' : 'calc(100vw - 220px)'
        };
        return styles;
      };
    }
  },
  watch: {},
  beforeRouteEnter(to, from, next) {
    if (from.fullPath && from.fullPath != '/') {
      let prevsetting = {
        path: from.fullPath,
        name: from.meta.title
      };
      sessionStorage.setItem('mergehandlerPrev', JSON.stringify(prevsetting));
    }
    next();
  },
  beforeRouteLeave(to, from, next) {
    sessionStorage.removeItem('mergehandlerPrev');
    next();
  }
};
</script>
<style lang="less" scoped>
@import (reference) '~@/resources/assets/css/variable.less';
.action-item {
  width: 24px;
}
.top-title {
  .title {
    line-height: 26px;
  }
  .desc {
    line-height: 20px;
  }
  padding-left: 20px;
  margin-left: 20px;
  border-left: 1px solid @default-border;
}
/deep/ .TsFormSelect .select-top {
  line-height: 24px;
  min-height: 24px;
}
.status-tag {
  transform: scale(0.9);
}
/deep/.tag-item{
  position: relative;
  .tag-new{
    position: absolute;
    line-height: 18px;
    right: -20px;
    top: -10px;
    color: @default-primary-color;
    transform: scale(0.9);
    border-radius: 3px;
    animation: fade 2s infinite;
  }
}
</style>
