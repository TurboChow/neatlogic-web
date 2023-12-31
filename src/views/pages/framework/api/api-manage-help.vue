<template>
  <TsDialog v-bind="dialogConfig" @on-close="close">
    <template v-slot>
      <div>
        <div v-if="Object.keys(helpData).length">
          <TsFormItem :label="$t('page.interface')" labelPosition="left" labelWidth="80">
            {{ rowData.url }}
          </TsFormItem>
          <TsFormItem
            v-if="helpData.description"
            :label="$t('page.description')"
            labelPosition="left"
            labelWidth="80"
          >
            {{ helpData.description }}
          </TsFormItem>
          <TsFormItem
            v-if="helpData.input"
            :label="$t('page.inputparam')"
            labelPosition="left"
            labelWidth="80"
          >
            <Table size="small" :columns="inputColumns" :data="helpData.input"></Table>
          </TsFormItem>
          <TsFormItem
            v-if="helpData.output"
            :label="$t('page.outputparam')"
            labelPosition="left"
            labelWidth="80"
          >
            <Table
              size="small"
              row-key="id"
              :columns="outputColumns"
              :data="helpData.output"
            ></Table>
          </TsFormItem>
          <TsFormItem
            v-if="helpData.example"
            :label="$t('term.report.example')"
            labelPosition="left"
            labelWidth="80"
          >
            <JsonViewer boxed copyable :value="helpData.example"></JsonViewer>
          </TsFormItem>
        </div>
        <div v-else>{{ helpMessage }}</div>
      </div>
    </template>
  </TsDialog>
</template>

<script>
export default {
  name: 'ApiHelp',
  components: {
    TsFormItem: resolve => require(['@/resources/plugins/TsForm/TsFormItem'], resolve),
    JsonViewer: resolve => require(['vue-json-viewer'], resolve)
  },
  props: {
    rowData: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      dialogConfig: {
        title: this.$t('term.framework.apihelp'),
        type: 'slider',
        isShow: true,
        maskClose: true,
        width: 'large',
        hasFooter: false
      },
      helpData: {},
      helpMessage: '',
      inputColumns: Object.freeze([
        { title: this.$t('page.name'), key: 'name' },
        { title: this.$t('page.description'), key: 'description' },
        { title: this.$t('page.type'), key: 'type' },
        { title: this.$t('page.rule'), key: 'rule' },
        {
          title: this.$t('page.isrequired'),
          key: 'isRequired',
          width: 100,
          render: (h, params) => {
            if (params.row.isRequired) {
              return h('div', { class: 'text-success' }, this.$t('page.yes'));
            } else {
              return h('div', { class: 'text-grey' }, this.$t('page.no'));
            }
          }
        },
        { title: this.$t('page.maxlength'), key: 'maxLength' },
        { title: this.$t('page.explain'), key: 'help' }
      ]),
      outputColumns: Object.freeze([
        { title: this.$t('page.name'), key: 'name', tree: true },
        { title: this.$t('page.description'), key: 'description' },
        { title: this.$t('page.type'), key: 'type' },
        { title: this.$t('page.explain'), key: 'help' }
      ])
    };
  },
  created() {
    // eslint-disable-next-line generator-star-spacing
    this.counter = (function* () {
      let i = 1;
      while (true) yield i++;
    })();
    this.getHelpData(this.rowData);
  },
  methods: {
    close() {
      this.$emit('close');
    },
    getHelpData({ type, token, helpUrl }) {
      this.helpMessage = this.$t('page.loadingtip');
      const param = `${type === 'object' ? 'rest' : type}/${token}`;
      //help接口返回无return层和status层
      return this.$api.framework.apiManage
        .help(helpUrl)
        .then(res => {
          this.helpData = this.addId(res);
          if (!Object.values(this.helpData).length) {
            this.helpMessage = this.$t('message.framework.notapihelp', { target: this.rowData.token });
          }
        })
        .catch(error => {
          this.$Notice.error({ title: this.$t('message.framework.apihelperror'), desc: error });
          this.helpMessage = this.$t('message.framework.apihelperror');
        });
    },
    addId(res) {
      // https://www.iviewui.com/components/table#SXSJ
      // table组件使用树形数据时，必须指定 row-key，比如 id
      // id必须唯一，不能重复，不然控制台会提示key重复的报错，而且鼠标悬浮高亮行会出错
      // 这里用生成器counter给有children的output表格数据添加唯一的id
      if (!res.output) return res;
      res.output.forEach(item => {
        item.id = this.counter.next().value;
        if (item.children) {
          item.children.forEach(child => {
            child.id = this.counter.next().value;
          });
        }
      });
      return res;
    }
  }
};
</script>

<style lang="less" scoped></style>
