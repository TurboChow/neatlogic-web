<template>
  <div class="form-li" :class="borderClass">
    <span v-if="readonly" :class="[readonlyClass, readonlyTextHighlightClass]" :title="renderFormat(labeList,selectedData)">{{ renderFormat(labeList,selectedData) }}</span>
    <div
      v-else
      :class="borderClass"
      :style="getStyle"
    >
      <!-- <Tooltip
        placement="top-start"
        max-width="300"
        transfer
        theme="light"
        style="width: 100%"
        :disabled="$utils.isEmpty(currentValue) || $utils.isEmpty(labeList)"
        :content="labels"
      > -->
      <Cascader
        v-model="currentValue"
        :data="actualDataList"
        :disabled="disabled"
        :render-format="renderFormat"
        :change-on-select="changeOnSelect"
        :trigger="trigger"
        :placeholder="placeholder"
        :clearable="clearable"
        :filterable="filterable"
        :transfer="transfer"
        :class="getClass"
        style="width:100%"
        @on-change="onChangeValue"
      ></Cascader>
      <!-- </Tooltip> -->
      <div v-if="desc && !descType" class="text-tip tips desc-text-mt">{{ desc }}</div>
      <Alert v-else-if="desc && descType" :type="descType">{{ desc }}</Alert>
      <transition name="fade">
        <slot name="validMessage">
          <span
            v-if="validMesage != ''"
            class="form-error-tip"
            :title="validMesage"
            v-html="validMesage"
          ></span>
        </slot>
      </transition>
    </div>
  </div>
</template>
<script>
import formMixins from '@/resources/mixins/formMixins.js';
import formScrollMixins from '@/resources/mixins/formScrollMixins.js';
export default {
  name: 'TsFormCascader',
  tagComponent: 'TsForm',
  mixins: [formMixins, formScrollMixins],
  model: {
    prop: 'value',
    event: 'change'
  },
  props: {
    value: {
      //默认值
      type: [Array, String],
      default: null
    },
    dataList: Array,
    format: {
      type: Function
    },
    changeOnSelect: { //当此项为 true 时，点选每级菜单选项值都会发生变化
      type: Boolean,
      default: false
    },
    trigger: {
      type: String,
      default: 'click'
    },
    clearable: {
      type: Boolean,
      default: true
    },
    placeholder: {
      type: String,
      default() {
        return this.$t('page.pleaseselect');
      }
    },
    filterable: {
      type: Boolean,
      default: false
    },
    transfer: {
      type: Boolean,
      default: false
    },
    isChangeWrite: {
      //赋值时，是否需要出发on-change事件
      type: Boolean,
      default: true
    },
    onChange: Function //改变时触发
  },
  data: function() {
    return {
      currentValue: [],
      validMesage: this.errorMessage || '',
      currentValidList: this.filterValid(this.validateList) || [],
      readonlyTitle: null,
      labels: null,
      labeList: [],
      selectedData: []
    };
  },
  mounted() {},
  beforeDestroy() {},

  methods: {
    onChangeValue(val, selectedData) {
      let value = this.$utils.deepClone(val);
      let isSame = this.$utils.isSame(this.value, value);
      this.$emit('change', value, selectedData);
      this.$emit('update:value', value);
      if (!(!this.isChangeWrite && isSame)) {
        //改变值时出发on-change事件
        this.$emit('on-change', value, selectedData);
        typeof this.onChange == 'function' && this.onChange(value);
      }

      if (!isSame) {
        if (this.currentValidList.length > 0) {
          this.valid(val);
        }
      } else {
        this.validMesage = '';
      }
    },
    renderFormat(labels, selectedData) {
      let data = labels.join(' / ');
      if (typeof this.format == 'function') {
        data = this.format(labels, selectedData);
      }
      this.labels = data;
      return data;
    },
    getLabel(i, value, list) {
      let _this = this;
      let finditem = list.find(l => l.value == value[i]);
      if (finditem) {
        _this.labeList.push(finditem.label);
        _this.selectedData.push(finditem);
        if (i < value.length && !_this.$utils.isEmpty(finditem.children)) {
          _this.getLabel(i + 1, value, finditem.children);
        }
      }
    },
    transferDataList(dataList) {
      if (dataList && dataList.length > 0) {
        dataList.forEach(data => {
          if (!data.hasOwnProperty('label') && data.hasOwnProperty('text')) {
            this.$set(data, 'label', data.text);
          }
          if (data.children && data.children.length > 0) {
            this.transferDataList(data.children);
          }
        });
      }
      return dataList;
    }
  },
  computed: {
    getClass: function() {
      let resultjson = [];
      this.disabled && resultjson.push('tsform-select-disabled');
      this.readonly && resultjson.push('tsform-select-readonly');
      (this.validMesage || this.$slots.validMessage) && resultjson.push('tsForm-formItem-error');
      return resultjson;
    },
    //将dataList格式转换成所需格式
    actualDataList() {
      const dataList = this.$utils.deepClone(this.dataList);
      return this.transferDataList(dataList);
    }
  },
  watch: {
    value: {
      handler(newValue) {
        let isSame = this.$utils.isSame(newValue, this.currentValue);
        if (!isSame) {
          this.validMesage = '';
          this.$nextTick(() => {
            this.currentValue = this.$utils.isEmpty(newValue) ? [] : [].concat(newValue);
            if (!this.$utils.isEmpty(this.currentValue) && !this.$utils.isEmpty(this.dataList)) {
              this.getLabel(0, this.currentValue, this.dataList);
            }
          });
        }
      },
      immediate: true,
      deep: true
    },
    currentValue: {
      handler(val) {
        if (val) {
          this.validMesage = '';
        }
      },
      immediate: true,
      deep: true
    }
  }
};
</script>
<style lang="less" scoped>
</style>
