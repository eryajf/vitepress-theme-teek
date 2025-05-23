<script setup lang="ts" name="TeekLayout">
import type { TeekConfig } from "@teek/config";
import type { Language } from "@teek/locale";
import DefaultTheme from "vitepress/theme";
import { useData } from "vitepress";
import { computed, provide, watch } from "vue";
import { useNamespace, localeContextKey } from "@teek/hooks";
import { isBoolean, isClient } from "@teek/helper";
import { useTeekConfig, usePage } from "@teek/components/theme/ConfigProvider";
import { TkArchivesPage } from "@teek/components/theme/ArchivesPage";
import { TkDocAfterAppreciation, TkAsideBottomAppreciation } from "@teek/components/theme/ArticleAppreciation";
import { TkCataloguePage } from "@teek/components/theme/CataloguePage";
import { TkArticleShare } from "@teek/components/theme/ArticleShare";
import { TkArticleAnalyze } from "@teek/components/theme/ArticleAnalyze";
import { TkArticleImagePreview } from "@teek/components/theme/ArticleImagePreview";
import { TkBodyBgImage } from "@teek/components/theme/BodyBgImage";
import { TkVpContainer } from "@teek/components/common/VpContainer";
import { TkThemeEnhance } from "@teek/components/theme/ThemeEnhance";
import { TkFooterGroup } from "@teek/components/theme/FooterGroup";
import { TkFooterInfo } from "@teek/components/theme/FooterInfo";
import { TkRightBottomButton } from "@teek/components/theme/RightBottomButton";
import { TkCommentTwikoo } from "@teek/components/theme/CommentTwikoo";
import { TkCommentArtalk } from "@teek/components/theme/CommentArtalk";
import { TkCommentGiscus } from "@teek/components/theme/CommentGiscus";
import { TkCommentWaline } from "@teek/components/theme/CommentWaline";
import { TkCodeBlockToggle } from "@teek/components/theme/CodeBlockToggle";
import { TkArticlePageStyle } from "@teek/components/theme/ArticlePageStyle";
import { TkNotice } from "@teek/components/theme/Notice";
import { TkHome } from "@teek/components/theme/Home";
import { TkArticleHeadingHighlight } from "@teek/components/theme/ArticleHeadingHighlight";
import { TkArticleUpdate } from "@teek/components/theme/ArticleUpdate";
import { TkArticleOverviewPage } from "@teek/components/theme/ArticleOverviewPage";

defineOptions({ name: "TeekLayout" });

const props = defineProps<{ locale?: Language }>();

provide(
  localeContextKey,
  computed(() => props.locale)
);

const { Layout } = DefaultTheme;

const ns = useNamespace("layout");
const { getTeekConfigRef } = useTeekConfig();
const { isHomePage, isArchivesPage, isCataloguePage, isArticleOverviewPage } = usePage();
const { frontmatter, localeIndex, page } = useData();

// 支持 theme 或 frontmatter 配置
const teekConfig = getTeekConfigRef<Required<TeekConfig>>(null, {
  teekTheme: true,
  teekHome: true,
  vpHome: true,
  codeBlock: { disabled: false },
  themeSize: "",
  bodyBgImg: {},
  notice: {},
  comment: { provider: "" },
  article: { articleUpdate: true },
  articleShare: {},
  appreciation: {},
});

const commentConfig = computed(() => {
  const comment = frontmatter.value.comment ?? teekConfig.value.comment;
  if (isBoolean(comment)) return { enabled: comment };

  return {
    enabled: true,
    components: {
      twikoo: TkCommentTwikoo,
      waline: TkCommentWaline,
      giscus: TkCommentGiscus,
      artalk: TkCommentArtalk,
    },
    provider: comment.provider,
    options: comment.options,
  };
});

const topTipConfig = computed(() => {
  return teekConfig.value.article.topTip?.(frontmatter.value, localeIndex.value, page.value);
});

const themeSizeAttribute = ns.joinNamespace("theme-size");
watch(
  () => teekConfig.value.themeSize,
  newValue => {
    if (!isClient) return;
    // 设置或删除主题尺寸
    if (newValue) document.documentElement.setAttribute(themeSizeAttribute, newValue);
    else document.documentElement.removeAttribute(themeSizeAttribute);
  },
  { immediate: true, flush: "post" }
);

// 维护已使用的插槽，防止外界传来的插槽覆盖已使用的插槽
const usedSlots = [
  "home-hero-before",
  "nav-bar-content-after",
  "layout-bottom",
  "doc-before",
  "doc-after",
  "aside-bottom",
  "page-top",
  "aside-outline-before",
];
</script>

<template>
  <template v-if="teekConfig.teekTheme">
    <TkBodyBgImage v-if="teekConfig.bodyBgImg?.imgSrc" />
    <TkArticleHeadingHighlight />
    <TkRightBottomButton>
      <!-- 通用插槽 -->
      <template v-for="(_, name) in $slots" :key="name" #[name]><slot :name="name" /></template>
    </TkRightBottomButton>

    <TkNotice v-if="teekConfig.notice?.enabled">
      <template v-for="(_, name) in $slots" :key="name" #[name]><slot :name="name" /></template>
    </TkNotice>

    <Layout :class="[ns.b(), { [ns.m('hide-vp-home')]: !teekConfig.vpHome }]">
      <template #home-hero-before>
        <slot name="home-hero-before" />
        <slot name="teek-home-before" />

        <!-- 自定义首页 -->
        <TkHome v-if="teekConfig.teekHome">
          <template v-for="(_, name) in $slots" :key="name" #[name]><slot :name="name" /></template>
        </TkHome>

        <slot name="teek-home-after" />
      </template>

      <template #nav-bar-content-after>
        <slot name="nav-bar-content-after" />

        <TkThemeEnhance position="top">
          <template v-for="(_, name) in $slots" :key="name" #[name]><slot :name="name" /></template>
        </TkThemeEnhance>
      </template>

      <template #layout-bottom>
        <TkFooterGroup v-if="isHomePage" />
        <slot name="teek-footer-info-before" />

        <TkFooterInfo v-if="isHomePage" />

        <slot name="teek-footer-info-after" />
        <slot name="layout-bottom" />
      </template>

      <template #doc-before>
        <slot name="doc-before" />
        <slot name="teek-article-analyze-before" />

        <TkArticleAnalyze />
        <TkArticleImagePreview />
        <TkArticlePageStyle />
        <TkCodeBlockToggle v-if="!teekConfig.codeBlock.disabled" />
        <TkVpContainer v-if="topTipConfig" v-bind="topTipConfig" />

        <slot name="teek-article-analyze-after" />
      </template>

      <template #doc-after>
        <slot name="doc-after" />

        <slot name="teek-doc-update-before" />
        <TkArticleUpdate v-if="teekConfig.article.articleUpdate && frontmatter.articleUpdate !== false" />
        <slot name="teek-doc-update-after" />

        <slot name="teek-doc-after-appreciation-before" />
        <TkDocAfterAppreciation v-if="teekConfig.appreciation.position === 'doc-after'" />
        <slot name="teek-doc-after-appreciation-after" />

        <slot name="teek-comment-before" />

        <!-- 评论区 -->
        <template v-if="commentConfig.enabled && commentConfig.provider">
          <template v-if="commentConfig.provider === 'render'"><slot name="teek-comment" /></template>
          <component
            v-else
            :is="commentConfig.components?.[commentConfig.provider]"
            :id="`${ns.namespace}-comment`"
            :class="ns.e('comment')"
          />
        </template>

        <slot name="teek-comment-after" />
      </template>

      <template #aside-bottom>
        <slot name="aside-bottom" />

        <slot name="teek-aside-bottom-appreciation-before" />
        <TkAsideBottomAppreciation v-if="teekConfig.appreciation.position === 'aside-bottom'" />
        <slot name="teek-aside-bottom-appreciation-after" />
      </template>

      <template #page-top>
        <slot name="page-top" />
        <slot name="teek-page-top-before" />

        <TkArchivesPage v-if="isArchivesPage">
          <template v-for="(_, name) in $slots" :key="name" #[name]><slot :name="name" /></template>
        </TkArchivesPage>
        <TkCataloguePage v-if="isCataloguePage">
          <template v-for="(_, name) in $slots" :key="name" #[name]><slot :name="name" /></template>
        </TkCataloguePage>
        <TkArticleOverviewPage v-if="isArticleOverviewPage" />

        <slot name="teek-page-top-after" />
      </template>

      <template #aside-outline-before>
        <slot name="teek-article-share-before" />

        <TkArticleShare v-if="teekConfig.articleShare.enabled" />

        <slot name="teek-article-share-after" />
        <slot name="aside-outline-before" />
      </template>

      <!-- 其他 VP 插槽 -->
      <template
        v-for="name in Object.keys($slots).filter(name => !usedSlots.includes(name))"
        :key="name"
        #[name]="slotData"
      >
        <slot :name="name" v-bind="slotData"></slot>
      </template>
    </Layout>
  </template>

  <template v-else>
    <Layout>
      <template v-for="(_, name) in $slots" :key="name" #[name]="slotData">
        <slot :name="name" v-bind="slotData"></slot>
      </template>
    </Layout>
  </template>
</template>
