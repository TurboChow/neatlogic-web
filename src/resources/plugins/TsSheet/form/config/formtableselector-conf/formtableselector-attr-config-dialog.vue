<template>
  <TsDialog v-bind="dialogConfig" @on-close="close">
    <template v-slot>
      <div v-if="propertyLocal.isExtra">
        <TsForm v-model="propertyLocal" :item-list="formConfig">
          <template v-slot:isRequired>
            <TsFormSwitch v-model="propertyLocal.config.isRequired" :trueValue="true" :falseValue="false"></TsFormSwitch>
          </template>
          <template v-if="['formtext', 'formtextarea'].includes(propertyLocal.handler)" v-slot:config>
            <TsFormItem :label="$t('page.inputtip')">
              <TsFormInput v-model="propertyLocal.config.placeholder" :maxlength="50"></TsFormInput>
            </TsFormItem>
            <TsFormItem :label="$t('page.defaultvalue')">
              <div>
                <TsFormRadio
                  v-model="propertyLocal.config.defaultValueType"
                  :dataList="[
                    { value: 'custom', text: $t('page.custom') },
                    { value: 'matrix', text: $t('page.matrix') }
                  ]"
                  @on-change="propertyLocal.config.defaultValue = ''"
                ></TsFormRadio>
                <TsFormSelect
                  v-if="propertyLocal.config.defaultValueType === 'matrix'"
                  v-model="propertyLocal.config.defaultValue"
                  :dataList="tableMatrixColumnList"
                  transfer
                  border="border"
                ></TsFormSelect>
                <TsFormInput v-else-if="propertyLocal.config.defaultValueType === 'custom'" v-model="propertyLocal.config.defaultValue" :type="propertyLocal.handler.replace('form', '')"></TsFormInput>
              </div>
            </TsFormItem>
          </template>
          <template v-else-if="['formselect', 'formradio', 'formcheckbox'].includes(propertyLocal.handler)" v-slot:config>
            <TsFormItem :label="$t('page.multipleselection')">
              <TsFormSwitch v-model="propertyLocal.config.isMultiple" :trueValue="true" :falseValue="false"></TsFormSwitch>
            </TsFormItem>
            <TsFormItem :label="$t('page.datasource')">
              <TsFormSelect
                ref="formitem_datasource"
                v-model="propertyLocal.config.dataSource"
                :validateList="validateList"
                :dataList="dataSourceList"
                transfer
                border="border"
              ></TsFormSelect>
            </TsFormItem>
            <TsFormItem v-if="propertyLocal.config.dataSource === 'static'" :label="$t('page.dropdownoption')">
              <div class="padding-md radius-md" :class="validClass('dataList')">
                <StaticDataEditor v-model="propertyLocal.config.dataList"></StaticDataEditor>
              </div>
            </TsFormItem>
            <TsFormItem v-if="propertyLocal.config.dataSource === 'matrix'" :label="$t('page.matrix')">
              <TsFormSelect
                ref="formitem_matrixuuid"
                v-model="propertyLocal.config.matrixUuid"
                :validateList="validateList"
                dynamicUrl="/api/rest/matrix/search"
                rootName="tbodyList"
                textName="name"
                valueName="uuid"
                transfer
              ></TsFormSelect>
            </TsFormItem>
            <TsFormItem v-if="propertyLocal.config.dataSource === 'matrix' && propertyLocal.config.matrixUuid && mappingDataList.length > 0" :label="$t('page.fieldmapping')">
              <div class="bg-block padding-md radius-md">
                <Row :gutter="10">
                  <Col span="12">
                    <label class="text-grey require-label">{{ $t('page.value') }}</label>
                    <div class="formsetting-text">
                      <TsFormSelect
                        ref="formitem_value"
                        v-model="propertyLocal.config.mapping.value"
                        :validateList="validateList"
                        :dataList="mappingDataList"
                        valueName="uuid"
                        textName="name"
                        :transfer="true"
                      ></TsFormSelect>
                    </div>
                  </Col>
                  <Col span="12">
                    <label class="text-grey require-label">{{ $t('page.displaytext') }}</label>
                    <div class="formsetting-text">
                      <TsFormSelect
                        ref="formitem_text"
                        v-model="propertyLocal.config.mapping.text"
                        :validateList="validateList"
                        valueName="uuid"
                        :dataList="mappingDataList"
                        textName="name"
                        :transfer="true"
                      ></TsFormSelect>
                    </div>
                  </Col>
                </Row>
              </div>
            </TsFormItem>
            <TsFormItem v-if="propertyLocal.config.dataSource === 'matrix' && propertyLocal.config.matrixUuid" :label="$t('page.filtercondition')">
              <div class="bg-block padding-md radius-md">
                <div v-if="propertyLocal.config.sourceColumnList && propertyLocal.config.sourceColumnList.length > 0">
                  <Row
                    v-for="(sourceColumn, index) in propertyLocal.config.sourceColumnList"
                    :key="index"
                    :gutter="10"
                    class="mb-xs"
                  >
                    <Col span="10">
                      <TsFormSelect
                        v-model="sourceColumn.column"
                        :dataList="extraPropertyMatrixColumnList"
                        transfer
                        border="border"
                      ></TsFormSelect>
                    </Col>
                    <Col span="2" style="text-align:center" class="text-grey">{{ $t('term.expression.eq') }}</Col>
                    <Col span="10"><TsFormSelect
                      v-model="sourceColumn.valueColumn"
                      :dataList="tableMatrixColumnList"
                      transfer
                      border="border"
                    ></TsFormSelect></Col>
                    <Col span="2" style="text-align:center"><span class="tsfont-trash-o text-action" @click="removeSourceColumn(index)"></span></Col>
                  </Row>
                </div>
                <Button @click="addSourceColumn"><span class="tsfont-plus">{{ $t('page.filtercondition') }}</span></Button>
              </div>
            </TsFormItem>
            <TsFormItem v-if="propertyLocal.handler === 'formselect'" :label="$t('page.inputtip')">
              <TsFormInput v-model="propertyLocal.config.placeholder" :maxlength="50"></TsFormInput>
            </TsFormItem>
            <TsFormItem :label="$t('page.defaultvalue')">
              <div>
                <TsFormRadio
                  v-model="propertyLocal.config.defaultValueType"
                  :dataList="[
                    { value: 'custom', text: $t('page.custom') },
                    { value: 'matrix', text: $t('term.framework.tablematrix') }
                  ]"
                  @on-change="propertyLocal.config.defaultValue = ''"
                ></TsFormRadio>
                <div v-if="propertyLocal.config.defaultValueType === 'matrix'" class="bg-block padding-md radius-md">
                  <TsRow :gutter="10">
                    <Col :span="12">
                      <div class="text-grey">{{ $t('page.valuefield') }}</div>
                      <TsFormSelect
                        ref="formitem_defaultvalue"
                        v-model="propertyLocal.config.defaultValueField"
                        :validateList="validateList"
                        :dataList="tableMatrixColumnList"
                        transfer
                      ></TsFormSelect>
                    </Col>
                    <Col :span="12">
                      <div class="text-grey">{{ $t('term.framework.showtextfields') }}</div>
                      <TsFormSelect
                        ref="formitem_defaulttext"
                        v-model="propertyLocal.config.defaultTextField"
                        :validateList="validateList"
                        :dataList="tableMatrixColumnList"
                        transfer
                      ></TsFormSelect>
                    </Col>
                  </TsRow>
                </div>
                <TsFormSelect
                  v-else-if="propertyLocal.config.defaultValueType === 'custom'"
                  v-model="propertyLocal.config.defaultValue"
                  v-bind="defaultValueSetting"
                  search
                  transfer
                ></TsFormSelect>
              </div>
            </TsFormItem>
          </template>
          <template v-else-if="['formdate', 'formtime'].includes(propertyLocal.handler)" v-slot:config>
            <TsFormItem :label="$t('page.format')">
              <TsFormSelect
                v-if="propertyLocal.handler === 'formdate'"
                ref="formitem_format"
                v-model="propertyLocal.config.format"
                :validateList="validateList"
                :dataList="[
                  { value: 'yyyy-MM-dd', text: 'yyyy-MM-dd' },
                  { value: 'yyyy-MM', text: 'yyyy-MM' },
                  { value: 'yyyy', text: 'yyyy' }
                ]"
                transfer
              ></TsFormSelect>
              <TsFormSelect
                v-else-if="propertyLocal.handler === 'formtime'"
                ref="formitem_format"
                v-model="propertyLocal.config.format"
                :validateList="validateList"
                :dataList="[
                  { value: 'HH:mm:ss', text: 'HH:mm:ss' },
                  { value: 'HH:mm', text: 'HH:mm' },
                  { value: 'HH', text: 'HH' }
                ]"
                transfer
              ></TsFormSelect>
            </TsFormItem>
            <TsFormItem :label="$t('page.defaultvalue')">
              <TsFormDatePicker
                v-model="propertyLocal.config.defaultValue"
                transfer
                :type="propertyLocal.handler.replace('form', '')"
                :format="propertyLocal.config.format"
              ></TsFormDatePicker>
            </TsFormItem>
          </template>
          <template v-slot:reaction>
            <Tabs v-if="propertyLocal.reaction">
              <TabPane
                v-for="(r, key) in propertyLocal.reaction"
                :key="key"
                :label="getReactionLabel(key)"
                :name="key"
              >
                <ConditionGroup
                  :ref="'condition_' + key"
                  :value="r"
                  :formItemList="formItemList"
                  :formItem="propertyLocal"
                  @input="
                    rule => {
                      setReaction(key, rule);
                    }
                  "
                ></ConditionGroup>
              </TabPane>
            </Tabs>
          </template>
        </TsForm>
      </div>
      <div v-else>
        <TsFormItem :label="$t('dialog.title.createtarget',{'target':$t('page.hyperlink')})">
          <TsFormSwitch
            v-model="isActive"
            :trueValue="true"
            :falseValue="false"
            @on-change="changeActive"
          ></TsFormSwitch>
        </TsFormItem>
        <TsFormItem v-if="isActive" :label="'URL'+ $t('page.attribute')" :validateList="validateList">
          <TsFormSelect
            ref="formitem_urlAttributeValue"
            v-model="propertyLocal.config.urlAttributeValue"
            :dataList="basicAttrList"
            valueName="uuid"
            textName="label"
            :validateList="validateList"
            transfer
            :desc="$t('message.framework.matirxhttpurl')"
            border="border"
          ></TsFormSelect>
        </TsFormItem>
      </div>
    </template>
    <template v-slot:footer>
      <Button @click="close()">{{ $t('page.cancel') }}</Button>
      <Button type="primary" @click="save()">{{ $t('page.confirm') }}</Button>
    </template>
  </TsDialog>
</template>
<script>
export default {
  name: '',
  components: {
    TsForm: resolve => require(['@/resources/plugins/TsForm/TsForm'], resolve),
    TsFormSwitch: resolve => require(['@/resources/plugins/TsForm/TsFormSwitch'], resolve),
    TsFormItem: resolve => require(['@/resources/plugins/TsForm/TsFormItem'], resolve),
    TsFormInput: resolve => require(['@/resources/plugins/TsForm/TsFormInput'], resolve),
    TsFormRadio: resolve => require(['@/resources/plugins/TsForm/TsFormRadio'], resolve),
    TsFormSelect: resolve => require(['@/resources/plugins/TsForm/TsFormSelect'], resolve),
    TsFormDatePicker: resolve => require(['@/resources/plugins/TsForm/TsFormDatePicker'], resolve),
    StaticDataEditor: resolve => require(['../common/static-data-editor.vue'], resolve),
    ConditionGroup: resolve => require(['@/resources/plugins/TsSheet/form/config/common/condition-group.vue'], resolve)
  },
  props: {
    formItemConfig: { type: Object }, //表单组件配置
    property: { type: Object } //属性配置
  },
  data() {
    const _this = this;
    return {
      propertyLocal: null,
      reactionName: {
        mask: this.$t('page.invisible'),
        hide: this.$t('page.hide'),
        display: this.$t('page.display'),
        readonly: this.$t('page.readonly'),
        disable: this.$t('page.disable'),
        required: this.$t('page.require')
      },
      reactionError: {}, //交互异常信息
      errorMap: {},
      validateList: [{ name: 'required', message: ' ' }],
      mappingDataList: [],
      dialogConfig: {
        type: 'modal',
        title: this.$t('page.edit'),
        maskClose: false,
        isShow: true,
        width: 'medium'
      },
      dataSourceList: [
        { value: 'static', text: this.$t('page.staticdatasource') },
        { value: 'matrix', text: this.$t('page.matrix') }
      ],
      formConfig: [
        {
          name: 'label',
          label: this.$t('page.name'),
          type: 'text',
          maxlength: 50,
          validateList: ['required']
        },
        {
          name: 'handler',
          label: this.$t('term.framework.inputtype'),
          type: 'select',
          transfer: true,
          dataList: [
            { text: this.$t('page.input'), value: 'formtext' },
            { text: this.$t('page.textfield'), value: 'formtextarea' },
            { text: this.$t('page.select'), value: 'formselect' },
            { text: this.$t('page.radio'), value: 'formradio' },
            { text: this.$t('page.checkbox'), value: 'formcheckbox' },
            { text: this.$t('page.date'), value: 'formdate' },
            { text: this.$t('page.time'), value: 'formtime' }
          ],
          validateList: ['required']
        },
        {
          name: 'isRequired',
          label: this.$t('page.require'),
          type: 'slot'
        },
        {
          name: 'config',
          hideLabel: true,
          type: 'slot'
        },
        {
          name: 'reaction',
          label: this.$t('page.eachother'),
          type: 'slot'
        }
      ],
      isActive: false
    };
  },
  beforeCreate() {},
  created() {
    this.propertyLocal = this.property;
    if (!this.propertyLocal.config) {
      this.$set(this.propertyLocal, 'config', {
        isRequired: false,
        isMask: false,
        isHide: false
      });
    } else {
      if (this.propertyLocal.config.urlAttributeValue) {
        this.isActive = true;
      }
    }
    if (!this.propertyLocal.reaction) {
      this.$set(this.propertyLocal, 'reaction', { mask: {}, hide: {}, display: {}, readonly: {}, disable: {}, required: {} });
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
    close() {
      this.$emit('close');
    },
    save() {
      let isValid = true;
      if (this.$refs) {
        for (let key in this.$refs) {
          if (key.startsWith('formitem_')) {
            if (this.$refs[key] && !this.$refs[key].valid()) {
              isValid = false;
            }
          }
        }
      }
      const config = this.propertyLocal.config;
      this.errorMap = {};
      if (config.dataSource === 'static' && (!config.dataList || config.dataList.filter(d => d.value).length === 0)) {
        this.$set(this.errorMap, 'dataList', true);
        isValid = false;
      }
      if (isValid) {
        this.$emit('close', this.propertyLocal);
      }
    },
    getReactionLabel(key) {
      return h => {
        const returnList = [h('span', this.reactionName[key])];
        if (this.reactionError[key]) {
          returnList.push(
            h('span', {
              class: ['tsfont-info-s', 'text-error']
            })
          );
        }
        if (this.propertyLocal.reaction[key] && !this.$utils.isEmpty(this.propertyLocal.reaction[key])) {
          returnList.push(
            h('span', {
              class: ['tsfont-lightning', 'text-warning']
            })
          );
        }
        return h('div', returnList);
      };
    },
    setReaction(key, rule) {
      this.$set(this.propertyLocal.reaction, key, rule);
    },
    //验证后的样式
    validClass(attrName) {
      const c = { 'bg-error-grey': !!this.errorMap[attrName], 'bg-block': !this.errorMap[attrName] };
      return c;
    },
    addSourceColumn() {
      if (!this.propertyLocal.config.sourceColumnList) {
        this.$set(this.propertyLocal.config, 'sourceColumnList', []);
      }
      this.propertyLocal.config.sourceColumnList.push({ column: '', valueColumn: '' });
    },
    removeSourceColumn(index) {
      this.propertyLocal.config.sourceColumnList.splice(index, 1);
    },
    changeActive() {
      this.propertyLocal.config.urlAttributeValue = '';
    }
  },
  filter: {},
  computed: {
    formItemList() {
      //所有表单组件列表
      return this.formItemConfig.dataConfig;
    },
    //表格选择组件矩阵的字段
    tableMatrixColumnList() {
      const columnList = [];
      this.formItemConfig.dataConfig
        .filter(d => !d.isExtra)
        .forEach(d => {
          columnList.push({ value: d.uuid, text: d.label });
        });
      return columnList;
    },
    //扩展属性矩阵的字段
    extraPropertyMatrixColumnList() {
      const columnList = [];
      this.mappingDataList.forEach(d => {
        columnList.push({ value: d.uuid, text: d.name });
      });
      return columnList;
    },
    defaultValueSetting() {
      const setting = {};
      const config = this.propertyLocal.config;
      if (config.isMultiple) {
        setting.multiple = config.isMultiple;
      }
      if (config.dataSource === 'matrix') {
        setting.dynamicUrl = '/api/rest/matrix/column/data/search/forselect';
        setting.rootName = 'dataList';
        const params = { matrixUuid: config.matrixUuid };
        if (config.mapping) {
          params.keywordColumn = config.mapping.text;
          params.valueField = config.mapping.value;
          params.textField = config.mapping.text;
        }
        setting.params = params;
      } else {
        setting.dataList = config.dataList;
      }
      return setting;
    },
    basicAttrList() {
      let dataList = [];
      if (this.formItemConfig.dataConfig && this.formItemConfig.dataConfig.length > 0) {
        dataList = this.formItemConfig.dataConfig.filter(item => {
          return !item.isExtra;
        });
      }
      return dataList;
    }
  },
  watch: {
    'propertyLocal.config.matrixUuid': {
      handler: function(val) {
        if (val) {
          this.$api.framework.matrix.getMatrixAttributeByUuid({ matrixUuid: val }).then(res => {
            if (res.Status == 'OK') {
              this.mappingDataList = res.Return.tbodyList;
            }
          });
        }
      },
      deep: true,
      immediate: true
    },
    'propertyLocal.config.dataSource': {
      handler: function(val) {
        if (val === 'matrix') {
          if (!this.propertyLocal.config.mapping) {
            this.$set(this.propertyLocal.config, 'mapping', {});
          }
        }
      },
      deep: true,
      immediate: true
    },
    'propertyLocal.handler': {
      handler: function(newVal, oldVal) {
        if (newVal && oldVal && oldVal != newVal) {
          this.$set(this.propertyLocal, 'config', {});
        }
      },
      deep: true,
      immediate: true
    }
  }
};
</script>
<style lang="less" scoped></style>
