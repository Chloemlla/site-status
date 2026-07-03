<!-- 全局配置 -->
<template>
  <n-config-provider
    :locale="siteLang.locale"
    :date-locale="siteLang.date"
    :theme="theme"
    :theme-overrides="themeOverrides"
    abstract
    inline-theme-disabled
    preflight-style-disabled
  >
    <n-global-style />
    <n-loading-bar-provider>
      <n-dialog-provider>
        <n-notification-provider>
          <n-message-provider :max="1">
            <n-modal-provider>
              <slot />
              <NaiveProviderContent />
            </n-modal-provider>
          </n-message-provider>
        </n-notification-provider>
      </n-dialog-provider>
    </n-loading-bar-provider>
  </n-config-provider>
</template>

<script setup lang="ts">
import {
  zhCN,
  dateZhCN,
  darkTheme,
  useOsTheme,
  useLoadingBar,
  useModal,
  useDialog,
  useMessage,
  useNotification,
  type GlobalThemeOverrides,
} from "naive-ui";

const osTheme = useOsTheme();
const colorMode = useColorMode();
const statusStore = useStatusStore();

// 站点语言
const siteLang = computed(() =>
  statusStore.siteLang === "zh-CN"
    ? { locale: zhCN, date: dateZhCN }
    : { locale: undefined, date: undefined },
);

// 获取明暗模式
const theme = computed(() => {
  if (colorMode.preference === "system") {
    return osTheme.value === "dark" ? darkTheme : null;
  }
  return colorMode.preference === "dark" ? darkTheme : null;
});

// 调整主题
const themeOverrides = computed<GlobalThemeOverrides>(() => ({
  common: {
    bodyColor: "var(--main-bg-color)",
    cardColor: "var(--main-card-color)",
    textColorBase: "var(--main-text-color)",
    textColor1: "var(--main-text-color)",
    textColor2: "var(--main-muted-color)",
    textColor3: "var(--main-subtle-color)",
    borderColor: "var(--mian-border-color)",
    dividerColor: "var(--mian-border-color)",
    primaryColor: "var(--normal-color)",
    primaryColorHover: "var(--normal-color)",
    primaryColorPressed: "var(--normal-color)",
    primaryColorSuppl: "var(--normal-color)",
    borderRadius: "12px",
    borderRadiusSmall: "8px",
    fontFamily: "var(--font-sans)",
  },
  Card: {
    borderRadius: "24px",
    color: "var(--main-card-color)",
    colorEmbedded: "var(--main-card-color)",
    boxShadow: "var(--main-card-shadow)",
  },
  Button: {
    borderRadiusMedium: "12px",
    borderRadiusLarge: "14px",
    fontWeightStrong: "800",
  },
  Input: {
    borderRadius: "14px",
    color: "var(--main-elevated-color)",
    colorFocus: "var(--main-elevated-color)",
  },
  Tag: {
    borderRadius: "999px",
  },
}));

// 挂载 naive 组件
const setupNaiveTools = () => {
  // 进度条
  window.$loadingBar = useLoadingBar();
  // 通知
  window.$notification = useNotification();
  // 信息
  window.$message = useMessage();
  // 对话框
  window.$dialog = useDialog();
  // 模态框
  window.$modal = useModal();
};

// 挂载工具
const NaiveProviderContent = defineComponent({
  setup() {
    setupNaiveTools();
  },
  render() {
    return h("div", { className: "main-tools" });
  },
});
</script>
