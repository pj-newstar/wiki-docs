<script setup lang="ts">
// import Link from '@/components/docs/Link.vue'
import { ElCollapse, ElCollapseItem, ElTable, ElTableColumn } from "element-plus";
import "element-plus/es/components/collapse/style/css";
import "element-plus/es/components/collapse-item/style/css";

import authors from "./author-list.json";

interface Contribution {
  author: string;
  normalCnt: number;
  extraCnt: number;
  inspectorCnt: number;
  realRate: number;
  normalizedRate: number;
}

const contributions: Record<string, Contribution> = {}; // author -> Contribution

let challengeNum = 0;

for (const [week, items] of Object.entries(authors)) {
  for (const item of items) {
    challengeNum += 1;
    const { author, inspectors } = item;
    if (!contributions[author]) {
      contributions[author] = {
        author,
        normalCnt: 0,
        extraCnt: 0,
        inspectorCnt: 0,
        realRate: 0,
        normalizedRate: 0,
      };
    }
    if (week === "extras") {
      contributions[author].extraCnt += 1;
    } else {
      contributions[author].normalCnt += 1;
    }
    for (const inspector of inspectors) {
      if (!contributions[inspector]) {
        contributions[inspector] = {
          author: inspector,
          normalCnt: 0,
          extraCnt: 0,
          inspectorCnt: 0,
          realRate: 0,
          normalizedRate: 0,
        };
      }
      contributions[inspector].inspectorCnt += 1;
    }
  }
}

let realRateSum = 0;

for (const [_, contribution] of Object.entries(contributions)) {
  const { normalCnt, extraCnt, inspectorCnt } = contribution;
  const score = normalCnt * 0.5 + extraCnt + (inspectorCnt * 0.5) / 3;
  contribution.realRate = score / challengeNum;
  realRateSum += contribution.realRate;
}

for (const [_, contribution] of Object.entries(contributions)) {
  contribution.normalizedRate = contribution.realRate / realRateSum;
}

const constributionArray = Object.values(contributions)
  .map((contribution) => ({
    ...contribution,
    totalCnt: contribution.normalCnt + contribution.extraCnt + contribution.inspectorCnt,
    realRateStr: (contribution.realRate * 100).toFixed(2) + "%",
    normalizedRateStr: (contribution.normalizedRate * 100).toFixed(2) + "%",
  }))
  .sort((a, b) => b.normalizedRate - a.normalizedRate);

const vprops = withDefaults(
  defineProps<{
    title_tmpl?: (key: string) => string;
    lables?: [string, string, string, string, string, string, string];
    props?: [string, string, string, string, string, string, string];
  }>(),
  {
    lables: () => ["出/验题人", "常规题出题数", "挑战题出题数", "总出题数量", "测题数量", "实际贡献度", "贡献度"],
    props: () => ["author", "normalCnt", "extraCnt", "totalCnt", "inspectorCnt", "realRateStr", "normalizedRateStr"],
  }
);

// const author_keys = Object.keys(authors);
</script>

<template>
  <ElCollapse class="contribution-list">
    <ElCollapseItem :name="`contribution-list`">
      <template #title>贡献度名单</template>

      <ElTable :data="constributionArray" stripe style="width: 100%">
        <ElTableColumn :prop="props[0]" :label="lables[0]" />
        <ElTableColumn :prop="props[1]" :label="lables[1]" width="130" />
        <ElTableColumn :prop="props[2]" :label="lables[2]" width="130" />
        <ElTableColumn :prop="props[3]" :label="lables[3]" width="120" />
        <ElTableColumn :prop="props[4]" :label="lables[4]" width="120" />
        <ElTableColumn :prop="props[5]" :label="lables[5]" width="130" />
        <ElTableColumn :prop="props[6]" :label="lables[6]" width="120" />
      </ElTable>
    </ElCollapseItem>
  </ElCollapse>
</template>

<style lang="scss">
.contribution-list table {
  margin: initial;
  tr {
    border-top: initial;
  }
  th,
  td {
    border: initial;
  }
}

.contribution-list th {
  --el-table-header-bg-color: var(--vp-c-bg-elv);
}

.contribution-list tr {
  --el-table-tr-bg-color: var(--vp-c-bg);
  --el-fill-color-lighter: var(--vp-c-bg-soft);
}

.contribution-list td {
  --el-table-border-color: var(--vp-c-divider);
}

.contribution-list {
  .el-collapse-item__header {
    padding-left: 8px;
  }
  .el-collapse-item__content {
    padding-bottom: 0;
  }
}
</style>
