<template>
  <div>
    <TsForm :item-list="formConfig">
      <template v-slot:members>
        <div>
          <div><a class="tsfont-plus" href="javascript:void(0)" @click="addMember()">{{ $t('dialog.title.addtarget', {target: $t('page.member')}) }}</a></div>
          <div v-for="(item,index) in myConfig.members" :key="index" class="grid">
            <div>
              <a class="tsfont-minus" href="javascript:void(0)" @click="removeMember(index)"></a>
            </div>
            <div class="mb-xs">
              <TsFormInput
                :width="300"
                :value="item"
                maxlength="30"
                @change="(val)=>{ setMember(val,index)}"
              ></TsFormInput>
            </div>
          </div>
          <div v-if="error" class="text-error">{{ error }}</div>
        </div>
      </template>
      <template v-slot:isMultiple>
        <i-switch v-model="myConfig.isMultiple" :true-value="1" :false-value="0"></i-switch>
        <span v-html="myConfig.isMultiple == 1 ? $t('page.yes') : $t('page.no')"></span>
      </template>
    </TsForm>
  </div>
</template>
<script>
export default {
  name: '',
  components: {
    TsForm: resolve =>
      require(['@/resources/plugins/TsForm/TsForm'], resolve),
    TsFormInput: resolve => require(['@/resources/plugins/TsForm/TsFormInput'], resolve)
  },
  props: {
    config: {type: Object}
  },
  data() {
    return {
      error: null,
      dataList: [],
      myConfig: {},
      formConfig: {
        members: {
          type: 'slot',
          label: this.$t('page.member')
        },
        isMultiple: {
          type: 'slot',
          label: this.$t('page.multipleselection')
        }
      }
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
    valid() {
      this.error = '';
      if (!this.myConfig.members || this.myConfig.members.length == 0) {
        this.error = this.$t('form.placeholder.pleaseadd', {target: this.$t('page.member')});
        return false;
      } else {
        if (this.myConfig.members.filter(v => !v || !v.trim()).length > 0) {
          this.error = this.$t('form.validate.required', {target: this.$t('page.member')});
          return false;
        } else if (new Set(this.myConfig.members).size != this.myConfig.members.length) {
          this.error = this.$t('message.cannotrepeat', {target: this.$t('page.member')});
          return false;
        }
      }
      return true;
    },
    addMember() {
      this.error = '';
      if (!this.myConfig.members) {
        this.$set(this.myConfig, 'members', []);
      }
      this.myConfig.members.push('');
    },
    removeMember(index) {
      this.error = '';
      this.myConfig.members.splice(index, 1);
    },
    setMember(val, index) {
      this.error = '';
      this.$set(this.myConfig.members, index, val && val.trim());
    }
  },
  filter: {},
  computed: {},
  watch: {
    myConfig: {
      handler: function(val) {
        this.$emit('setConfig', val);
      },
      deep: true
    },
    config: {
      handler: function(val) {
        this.myConfig = val || {};
        if (this.myConfig) {
          for (const k in this.myConfig) {
            if (this.formConfig[k]) {
              this.formConfig[k].value = this.myConfig[k];
            }
          }
        }
      },
      deep: true,
      immediate: true
    }
  }
};
</script>
<style lang="less" scoped>
.grid{
  display: grid;
  grid-template-columns: 16px auto;
}
</style>
