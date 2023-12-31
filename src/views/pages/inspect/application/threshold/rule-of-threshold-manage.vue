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
  <div>
    <Loading :loadingShow="loadingShow" type="fix"></Loading>
    <TsContain>
      <template v-slot:navigation>
        <span class="text-action ts-angle-left" @click="$back()">{{ $getFromPage() }}</span>
      </template>
      <template v-slot:topRight>
        <div>
          <InputSearcher
            v-model="formInputSetting.value"
            v-bind="formInputSetting"
            @change="getTableList()"
          ></InputSearcher>
        </div>
      </template>
      <div slot="content">
        <div v-if="!loadingShow && inspectDefinitionList.length > 0" class="content-main">
          <div v-for="(item, index) in inspectDefinitionList" :key="index">
            <div v-if="item && item.cardList && item.cardList.length > 0" class="title text-title ci-title-text">
              <span class="text-grey ts-catalogue">{{ item.collection }}</span>
            </div>
            <div>
              <TsCard
                v-if="item && item.cardList && item.cardList.length > 0"
                v-bind="item"
                :sm="8"
                :lg="6"
                :xl="6"
                :xxl="4"
                :boxShadow="false"
              >
                <template slot-scope="{ row }">
                  <div class="ci-main" @click="toRuleOfThresholdDetail(row)">
                    <div>
                      <div class="ci-icon">
                        <i :class="`tsfont-${row.name}`"></i>
                      </div>
                    </div>
                    <div class="overflow">
                      <div class="ci-label overflow" :title="row.label">{{ row.label }}</div>
                      <div class="ci-name overflow text-grey" :title="row.name">{{ row.name }}</div>
                    </div>
                  </div>
                </template>
              </TsCard>
            </div>
          </div>
        </div>
        <NoData v-else />
      </div>
    </TsContain>
  </div>
</template>
<script>
export default {
  name: 'RuleOfThresholdManage', // 巡检规则
  components: {
    InputSearcher: resolve => require(['@/resources/components/InputSearcher/InputSearcher.vue'], resolve),
    TsCard: resolve => require(['@/resources/components/TsCard/TsCard.vue'], resolve)
  },
  filters: {},
  props: {},
  data() {
    return {
      loadingShow: true,
      inspectDefinitionList: [],
      appSystemId: null,
      formInputSetting: {
        placeholder: this.$t('page.name'),
        search: true,
        clearable: true,
        value: '',
        border: 'border'
      }
     
    };
  },
  beforeCreate() {},
  created() {
    let query = this.$route.query;
    if (query && query.appSystemId) {
      this.appSystemId = parseInt(query.appSystemId);
      this.getTableList();
    }
  },
  beforeMount() {},
  mounted() {},
  beforeUpdate() {},
  updated() {},
  activated() {},
  deactivated() {},
  beforeDestroy() {},
  methods: {
    getTableList() {
      this.loadingShow = true;
      this.$api.inspect.definition.getThresholdsCollectionList({appSystemId: this.appSystemId, name: this.formInputSetting.value}).then(res => {
        if (res.Status == 'OK') {
          let tempObj = {};
          let tbodyList = res.Return ? res.Return : [];
          tbodyList.forEach((item) => {
            let {collection} = item;
            if (!tempObj[collection]) {
              tempObj[collection] = {
                collection,
                cardList: []
              };
            }
            tempObj[collection].cardList.push(item);
          });
          this.inspectDefinitionList = Object.values(tempObj);
        }
      }).finally(() => {
        this.loadingShow = false;
      });
    },
    toRuleOfThresholdDetail(row) {
      if (row && !this.$utils.isEmptyObj(row)) {
        this.$router.push({
          path: '/rule-of-threshold-detail',
          query: { appSystemId: this.appSystemId, name: row.name, label: row.label }
        });
      }
    }
  },
  computed: {},
  watch: {}
};
</script>
<style lang="less" scoped>
.content-main {
  .type-main{
    margin-bottom:10px;
  }
  .ci-title-text{
    padding-left: 12px;
  }
}
.ci-main {
  cursor: pointer;
  display: grid;
  width: 100%;
  grid-template-columns: 40px auto;
  margin: 10px;
  .ci-icon {
    font-size: 24px;
  }
  .ci-label {
    font-size: 14px;
    margin-bottom: 2px;
  }
  .ci-name {
    font-size: 12px;
  }
}
</style>
