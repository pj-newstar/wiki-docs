<script setup lang="ts">
// import Link from '@/components/docs/Link.vue'
import { ElCollapse, ElCollapseItem, ElTable, ElTableColumn } from "element-plus";
import "element-plus/es/components/collapse/style/css";
import "element-plus/es/components/collapse-item/style/css";

import authors from "./author-list.json";

const vprops = withDefaults(
  defineProps<{
    keys?: string[];
    title_tmpl?: (key: string) => string;
    lables?: [string, string, string];
    props?: [string, string, string];
  }>(),
  {
    keys: () => Object.keys(authors),
    title_tmpl: (key: string) => {
      return `Week ${key.replace(/[^\d]/g, "")}`;
    },
    lables: () => ["题目名称", "方向", "出题人"],
    props: () => ["name", "category", "author"],
  }
);

// const author_keys = Object.keys(authors);
</script>

<template>
  <ElCollapse class="author-list" v-for="i in keys.length">
    <ElCollapseItem :name="`author-list-${i}`">
      <template #title>{{ title_tmpl(keys[i - 1]) }}</template>

      <ElTable :data="authors[keys[i - 1] as keyof typeof authors]" stripe style="width: 100%">
        <ElTableColumn :prop="props[0]" :label="lables[0]" />
        <ElTableColumn :prop="props[1]" :label="lables[1]" width="180" />
        <ElTableColumn :prop="props[2]" :label="lables[2]" width="180" />
      </ElTable>
    </ElCollapseItem>
  </ElCollapse>
</template>

<style lang="scss">
.author-list table {
  margin: initial;
  tr {
    border-top: initial;
  }
  th,
  td {
    border: initial;
  }
}

.author-list th {
  --el-table-header-bg-color: var(--vp-c-bg-elv);
}

.author-list tr {
  --el-table-tr-bg-color: var(--vp-c-bg);
  --el-fill-color-lighter: var(--vp-c-bg-soft);
}

.author-list td {
  --el-table-border-color: var(--vp-c-divider);
}

.author-list {
  .el-collapse-item__header {
    padding-left: 8px;
  }
  .el-collapse-item__content {
    padding-bottom: 0;
  }
}
</style>
