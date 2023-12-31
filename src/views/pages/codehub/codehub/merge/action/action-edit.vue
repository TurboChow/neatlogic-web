<template>
  <div>
    <TsDialog
      v-bind="actionDialogConfig"
      @on-close="close"
      @on-ok="saveAction"
    >
      <template v-slot>
        <Loading
          v-if="loadingShow"
          :loadingShow="loadingShow"
          type="fix"
        ></Loading>
        <TsForm
          v-else
          ref="actionForm"
          v-model="actionData"
          :item-list="actionFormConfig"
        >
          <template v-slot:arguments>
            <Tabs v-model="activedTab">
              <TabPane :label="$t('term.pbc.paramconfig')" name="arguments">
                <ArgumentEdit ref="argumentEdit" :argumentData="actionData.arguments"></ArgumentEdit>
              </TabPane>
              <TabPane :label="$t('page.paramdesc')" name="param">
                <ParameterDescription></ParameterDescription>
              </TabPane>
            </Tabs>
          </template>
        </TsForm>
      </template>
    </TsDialog>
  </div>
</template>
<script>
export default {
  components: {
    TsForm: resolve => require(['@/resources/plugins/TsForm/TsForm'], resolve),
    ArgumentEdit: resolve => require(['./argument-edit.vue'], resolve), // 参数设置
    ParameterDescription: resolve => require(['./parameter-description.vue'], resolve) // 参数说明
  },
  props: {
    id: { type: Number }
  },
  data() {
    return {
      loadingShow: true,
      activedTab: 'arguments', // 默认参数设置
      actionData: { 
        id: null,
        name: '',
        triggerStatus: [],
        appSystemId: null,
        appModuleId: null,
        versionId: null,
        targetBranch: null,
        isActive: 1,
        arguments: {}
      },
      actionDialogConfig: {
        type: 'modal',
        maskClose: false,
        isShow: true,
        width: 'large',
        title: this.id ? this.$t('dialog.title.edittarget', { target: this.$t('page.config') }) : this.$t('dialog.title.addtarget', { target: this.$t('page.config') })
      },
      actionFormConfig: [
        {
          type: 'text',
          name: 'id',
          isHidden: true
        },
        {
          type: 'text',
          name: 'name',
          label: this.$t('term.codehub.actionname'),
          maxlength: 50,
          validateList: [
            'required', 
            'name-special',
            { name: 'searchUrl',
              url: '/api/rest/codehub/mergerequest/action/save', 
              key: 'name',
              message: this.$t('message.targetisexists', {target: this.$t('page.name')}),
              params: { id: this.id}
            }]
        },
        {
          type: 'select',
          name: 'triggerStatus',
          label: this.$t('term.deploy.triggerstate'),
          validateList: ['required'],
          search: true,
          multiple: true,
          dataList: [
            { 
              value: 'conflict', 
              text: this.$t('page.conflict') 
            },
            { 
              value: 'failed', 
              text: this.$t('page.fail') 
            },
            { 
              value: 'finish', 
              text: this.$t('page.finish') 
            },
            { 
              value: 'closed', 
              text: this.$t('term.rdm.isclosed') 
            }
          ]
        },
        {
          type: 'select',
          name: 'appSystemId',
          label: this.$t('term.codehub.triggersystem'),
          url: '/api/rest/codehub/appsystem/search',
          rootName: 'tbodyList',
          dealDataByUrl: this.$utils.getAppForselect,
          search: true,
          onChange: val => {
            this.actionData.appSystemId = val;
            this.actionData.appModuleId = null;
            this.actionData.versionId = null;
            this.actionFormConfig.forEach(item => {
              if (item.name == 'versionId') {
                item.params = {
                  appSystemId: val || null,
                  appModuleId: null
                };
              }
            });
            this.changeAppModule();
          }
        },
        {
          type: 'select',
          label: this.$t('term.codehub.triggerappmodule'),
          name: 'appModuleId',
          transfer: true,
          dealDataByUrl: this.$utils.getAppForselect,
          dynamicUrl: '',
          onChange: appModuleId => {
            this.actionFormConfig.forEach(item => {
              if (item.name == 'versionId') {
                item.params.appModuleId = appModuleId;
              }
            });
          }
        },
        {
          type: 'select',
          name: 'versionId',
          label: this.$t('term.codehub.triggerversion'),
          dynamicUrl: '/api/rest/codehub/version/search',
          rootName: 'tbodyList',
          valueName: 'id',
          textName: 'version',
          search: true,
          value: null,
          params: {
            appSystemId: null,
            appModuleId: null
          }
        },
        {
          type: 'text',
          name: 'targetBranch',
          label: this.$t('page.targetbranch'),
          desc: this.$t('term.codehub.targetbranchdesc'),
          clearable: true
        },
        {
          type: 'radio',
          name: 'isActive',
          label: this.$t('page.enable'),
          default: 1,
          dataList: [
            { text: this.$t('page.yes'), value: 1 },
            { text: this.$t('page.no'), value: 0 }
          ]
        },
        { 
          type: 'slot', 
          name: 'arguments',
          label: this.$t('term.inspect.sendanemail') 
        }
      ]
    };
  },
  beforeCreate() {},
  created() {
    this.getAction(this.id);
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
    getAction(id) {
      if (id) {
        this.loadingShow = true;
        this.$api.codehub.merge.getActionById({ id: id }).then(res => {
          if (res.Status == 'OK') {
            let dataInfo = res.Return;
            for (const key in dataInfo) {
              if (key == 'argumentJSON') {
                this.actionData['arguments'] = dataInfo[key] || {};
              } else if (key == 'statusList') { // 处理触发状态值
                if (!this.$utils.isEmpty(dataInfo[key])) {
                  dataInfo[key].forEach((item) => {
                    this.actionData.triggerStatus.push(item.value);
                  });
                }
              } else if (key != 'arguments' && key != 'triggerStatus' && this.actionData.hasOwnProperty(key)) {
                this.actionData[key] = dataInfo[key];
              }
            }
            this.changeAppModule();
          }
        }).finally(() => {
          this.loadingShow = false;
        });
      } else {
        this.loadingShow = false;
      }
    },
    changeAppModule() {
      if (this.actionData.appSystemId) {
        this.actionFormConfig.forEach(item => {
          if (item.name == 'appModuleId') {
            this.$set(item, 'params', { appSystemId: this.actionData.appSystemId });
            this.$set(item, 'dynamicUrl', '/api/rest/deploy/app/config/module/list');
          } else if (item.name == 'versionId') {
            this.$set(item, 'params', { appSystemId: this.actionData.appSystemId, appModuleId: this.actionData.appModuleId });
            this.$set(item, 'dynamicUrl', '/api/rest/codehub/version/search');
          }
        });
      } else {
        this.actionFormConfig.forEach(item => {
          if (item.name == 'appModuleId') {
            this.$set(item, 'params', {});
            this.$set(item, 'dynamicUrl', '');
          }
        });
      }
    },
    saveAction() {
      let form = this.$refs['actionForm'];
      let argumentEditForm = this.$refs.argumentEdit;
      let params = this.$utils.deepClone(this.actionData);
      params.arguments = argumentEditForm ? argumentEditForm.getFormValue() : {};
      if ((form && form.valid()) && (argumentEditForm && argumentEditForm.valid())) {
        this.$api.codehub.merge
          .saveAction(params)
          .then(res => {
            if (res.Status == 'OK') {
              this.$Message.success(this.$t('message.savesuccess'));
              this.close(true);
            }
          });
      }
    },
    close(needRefresh) {
      this.$emit('close', needRefresh);
    }
  },
  filter: {},
  computed: {},
  watch: {}
};
</script>
<style lang="less"></style>
