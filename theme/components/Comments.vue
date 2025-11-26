<script setup lang="ts">
import { useData } from "vitepress";
import Giscus from "@giscus/vue";

// const { frontmatter, title } = useData();
const { site, frontmatter, page } = useData();

const PREFIX = "comments+";

const p2k = (str: string) => {
  const type = typeof frontmatter.value.comments;
  if (type === "string" || type === "number") {
    return PREFIX + frontmatter.value.comments.toString().replace(/\s/g, "-");
  }
  return PREFIX + str.replace(/\.\w+$/, "");
};
</script>

<template>
  <div
    class="giscus comments-box"
    :key="p2k(page.filePath)"
    :data-key="p2k(page.filePath)"
    v-if="!!site.themeConfig.comments && frontmatter.comments !== false"
  >
    <Giscus
      repo="pj-newstar/wiki-docs"
      repoId="R_kgDOMqjJaA"
      category="Comments"
      categoryId="DIC_kwDOMqjJaM4CnZZo"
      mapping="specific"
      :term="p2k(page.filePath)"
      strict="1"
      reactionsEnabled="1"
      emitMetadata="0"
      inputPosition="top"
      theme="preferred_color_scheme"
      :lang="site.lang"
      loading="lazy"
      crossorigin="anonymous"
    />
  </div>
</template>

<style lang="scss">
.comments-box {
  border-top: 1px solid var(--vp-c-divider);
  margin-top: 24px;
  padding-top: 2rem;
}
</style>
