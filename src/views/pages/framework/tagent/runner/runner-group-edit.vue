/*
 * Copyright(c) 2023 NeatLogic Co., Ltd. All Rights Reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
<template>
  <div class="runner-group-add-wrap">
    <TsDialog
      v-bind="editTsDialog"
      :isShow="isShow"
      @on-close="close"
      @on-cancel="close"
      @on-ok="saveRunnerGroup"
    >
      <template v-slot>
        <div>
          <Tabs 
            v-model="current"
            class="block-tabs"
            :animated="false"
          >
            <TabPane
              :label="$t('page.basicinfo')"
              name="basicInfo"
            >
              <div class="padding bg-op radius-lg">
                <div v-show="current == 'basicInfo'">
                  <TsForm
                    ref="form"
                    v-model="formValue"
                    :item-list="formConfig"
                    labelPosition="left"
                    :labelWidth="60"
                  >
                    <template slot="groupNetworkList">
                      <TsTable
                        ref="editFormTable"
                        :theadList="theadList"
                        :tbodyList="netWorkList"
                      >
                        <template v-slot:networkIp="{ row }">
                          <TsFormInput
                            ref="formInput"
                            v-model="row.networkIp"
                            clearable
                            class="groupwork"
                            :validateList="validConfig.validIp"
                          />
                        </template>
                        <template v-slot:mask="{ row }">
                          <TsFormInput
                            ref="formInput"
                            v-model="row.mask"
                            class="groupwork"
                            clearable
                            :validateList="validConfig.validMask"
                          />
                        </template>
                        <template v-slot:act="{ index }">
                          <div class="actions flex-start">
                            <span v-if="netWorkList.length>1" class="tsfont-close-o text-action" @click="operationRow('del', index)"></span>
                            <span class="tsfont-plus-o text-action" @click="operationRow('add', index)"></span>
                          </div>
                        </template>
                      </TsTable>
                    </template>
                  </TsForm>
                 
                </div>
              </div>
            </TabPane>
            <TabPane
              :label="$t('term.deploy.associatedrunner')"
              name="toolParam"
            >
              <div class="padding bg-op radius-lg">
                <div v-show="current == 'toolParam'">
                  <RunnerRelate
                    v-model="runnerList"
                    class="runner-manage-group-wrap"
                    :groupId="runnerGroupData.id"
                    :isShowHeader="false"
                    @close="close"
                  ></RunnerRelate>
                </div>
              </div>
            </TabPane>
          </Tabs>
        </div>
      </template>
    </TsDialog>
  </div>
</template>
<script>
import RunnerRelate from './runner-relate.vue'; // 关联runner
import {runnerGroupMixin} from './src/runner-group-mixin.js';
export default {
  name: 'RunnerGroupEdit', // 添加分组
  components: {
    RunnerRelate,
    TsForm: resolve => require(['@/resources/plugins/TsForm/TsForm.vue'], resolve),
    TsFormInput: resolve => require(['@/resources/plugins/TsForm/TsFormInput'], resolve),
    TsTable: resolve => require(['@/resources/components/TsTable/TsTable.vue'], resolve)
  },
  filters: {}, 
  mixins: [runnerGroupMixin],
  props: {
    isShow: {
      // 是否显示dialog 弹窗
      type: Boolean,
      default: false
    },
    runnerGroupData: {
      type: Object,
      default: function() {
        return {};
      }
    }
  },
  data() {
    return {
      runnerList: [],
      current: 'basicInfo', // 默认显示第一个
      formValue: {}, 
      editTsDialog: {
        // 弹框配置信息
        type: 'modal',
        isShow: false,
        title: this.$t('dialog.title.edittarget', {target: this.$t('page.group')}),
        okText: this.$t('page.save')
      }    
    };
  },
  beforeCreate() {},
  created() {
    this.formValue = this.runnerGroupData;
    let groupNetworkList = this.runnerGroupData.groupNetworkList;
    this.netWorkList = this.$utils.isEmpty(groupNetworkList) ? this.netWorkList : groupNetworkList;
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
    saveRunnerGroup() {
      // 保存runner
      let form = this.$refs.form;
      if (!form.valid()) {
        return;
      }
      if (!this.validNetwork()) {
        return;
      }
      let params = {
        ...form.getFormValue(),
        groupNetworkList: this.filterEmptyNetworkList(),
        runnerList: this.runnerList // 关联runner列表
      };
      this.$api.framework.runner.saveRunnerGroup(params).then(res => {
        if (res.Status == 'OK') {
          this.$Message.success(this.$t('message.savesuccess'));
          this.$emit('refreshList');
          this.close();
        }
      });
    }
  },
  computed: {},
  watch: {}
};
</script>

<style lang="less" scoped>
@import './src/runner-group.less';
@import './src/common.less';
</style>
