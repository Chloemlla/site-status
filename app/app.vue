<template>
  <GlobalProvider>
    <n-scrollbar
      :content-style="{
        minHeight: '100%',
        display: 'flex',
        flexDirection: 'column',
      }"
      style="height: 100vh"
      @scroll="siteScroll"
    >
      <SiteNav />
      <Transition name="fade">
        <SiteHeader v-if="statusStore.loginStatus" />
      </Transition>
      <!-- 主内容 -->
      <main v-if="siteLoaded" id="main">
        <Transition name="content-fade" mode="out-in">
          <!-- 密码验证 -->
          <SiteLogin v-if="!statusStore.loginStatus" />
          <!-- 站点卡片 -->
          <SiteCards v-else />
        </Transition>
      </main>
      <SiteFooter />
      <!-- 回到顶部 -->
      <n-back-top :visibility-height="10" />
    </n-scrollbar>
  </GlobalProvider>
</template>

<script setup lang="ts">
const config = useRuntimeConfig();
const statusStore = useStatusStore();

const { setLocale } = useI18n();

// 加载状态
const siteLoaded = ref<boolean>(false);

// 验证状态
const checkSite = async () => {
  try {
    const result = await $fetch("/api/check", { method: "POST" });
    // 更改登录状态
    statusStore.loginStatus = result.code === 200;
  } catch (error) {
    console.error("error in checkSite", error);
  } finally {
    siteLoaded.value = true;
  }
};

// 页面滚动
const siteScroll = (e: Event) => {
  // 滚动高度
  const scrollTop = (e.target as HTMLElement).scrollTop;
  statusStore.scrollTop = scrollTop;
  
  // 动态调整body padding避免文字重叠
  const isCompact = scrollTop > 50;
  document.body.style.paddingTop = isCompact ? '80px' : '0px';
};

// 更改站点语言
const setSiteLang = (lang: string) => {
  setLocale(lang);
  useHead({ htmlAttrs: { lang } });
};

// 监听站点状态
watch(
  () => statusStore.siteStatus,
  (status) => {
    const { siteTitle } = config.public;
    // 错误数据
    const isError = status === "error" || status === "warn";
    const error = statusStore.siteData?.status?.error || 0;
    const unknown = statusStore.siteData?.status?.unknown || 0;
    // 更改信息
    useHead({
      // 更改标题
      title: isError ? `( ${error + unknown} ) ` + siteTitle : siteTitle,
    });
    // 更改图标
    useFavicon(isError ? "/favicon-error.ico" : "/favicon.ico");
  },
);

// 语言更改
watch(() => statusStore.siteLang, setSiteLang);

onBeforeMount(checkSite);

onMounted(() => {
  setSiteLang(statusStore.siteLang);
});
</script>

<style lang="scss">
/* 自定义字体定义 */
@font-face {
  font-family: '1666963922';
  src: url('https://jsd.cdn.zzko.cn/gh/54ayao/ACG@main/static/fonts/1666963922.woff') format('woff');
  font-weight: normal;
  font-style: normal;
}

/* 全局样式 */
body {
  font-family: '1666963922', sans-serif; /* 使用自定义字体 */
  color: #333;
  margin: 0;
  padding: 0;
  background-image: url('https://www.loliapi.com/acg/'); /* 自适应背景图片API */
  background-attachment: fixed;
  background-size: cover;
  background-position: center center;
  background-repeat: no-repeat;
}

/* 导航栏链接悬停样式 */
.navbar a:hover {
  text-decoration: underline;
}

/* 卡片样式 */
.card {
  background-color: #fff;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
  padding: 20px;
}

/* 卡片标题样式 */
.card-title {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 10px;
}

/* 在明亮主题下，应用不同的背景色及75%透明度 */
.shadow-box {
  background-color: rgba(255, 255, 255, 0.75);
  padding: 10px;
  margin: 5px;
  border-radius: 10px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  text-decoration: none;
}

/* 在暗色主题下，应用不同的背景色及65%透明度 */
.dark .shadow-box:not(.alert) {
  background-color: rgba(0, 0, 0, 0.65);
  padding: 20px;
  margin: 10px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

/* 侧边栏样式 */
.sidebar {
  background-color: rgba(255, 255, 255, 0.9);
  padding: 20px;
  margin: 10px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

/* 特定元素和类的样式 */
span[data-v-7d4a7f28],
.item-name,
div.description[data-v-7d4a7f28][data-v-b8247e57][contenteditable="true"],
div[data-v-7d4a7f28][data-v-b8247e57].alert-heading.p-2,
.refresh-info > div,
.alert-heading.p-2 > div,
.alert-heading.p-2 > p,
.alert-heading.p-2 > h1 {
  background-image: linear-gradient(90deg, #07c160, #fb6bea 25%, #3aedff 50%, #fb6bea 75%, #28d079);
  font-family: '1666963922', sans-serif;
  -webkit-text-fill-color: transparent;
  -webkit-background-clip: text;
  background-clip: text;
  background-size: 400% 100%;
  animation: breathing 10s linear infinite;
}

/* 动画关键帧定义 */
@keyframes breathing {
  0%, 100% {
    transform: scale(1);
    opacity: 0.8;
  }
  50% {
    transform: scale(1.2);
    opacity: 0.4;
  }
}

// 页面过渡动画
.slide-fade-header-enter-active {
  transition: all 0.8s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}

.slide-fade-header-leave-active {
  transition: all 0.6s cubic-bezier(0.55, 0.085, 0.68, 0.53);
}

.slide-fade-header-enter-from {
  transform: translateY(-50px);
  opacity: 0;
}

.content-fade-enter-active {
  transition: all 0.6s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}

.content-fade-leave-active {
  transition: all 0.4s cubic-bezier(0.55, 0.085, 0.68, 0.53);
}

.content-fade-enter-from {
  transform: translateY(20px) scale(0.98);
  opacity: 0;
}

.content-fade-leave-to {
  transform: translateY(-20px) scale(1.02);
  opacity: 0;
}

// 全局页面加载动画
.fade-enter-active {
  transition: all 0.5s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}

.fade-leave-active {
  transition: all 0.3s cubic-bezier(0.55, 0.085, 0.68, 0.53);
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(10px);
}

// 回到顶部按钮增强
:global(.n-back-top) {
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  
  &:hover {
    transform: scale(1.1) translateY(-2px);
  }
  
  &:active {
    transform: scale(0.95);
  }
}

#app {
  font-family: '1666963922', BlinkMacSystemFont, 'segoe ui', Roboto, 'helvetica neue', Arial, 'noto sans', sans-serif, 'apple color emoji', 'segoe ui emoji', 'segoe ui symbol', 'noto color emoji' !important;
}

main {
  width: 100%;
  height: 100%;
  flex: 1;
  
  // 响应式主容器
  @media screen and (max-width: 1200px) {
    padding: 0 1rem;
  }
  
  @media screen and (max-width: 768px) {
    padding: 0 0.75rem;
  }
  
  @media screen and (max-width: 480px) {
    padding: 0 0.5rem;
  }
}

// 全局响应式工具类
:global(.responsive-container) {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
  
  @media screen and (max-width: 1200px) {
    max-width: 100%;
    padding: 0 16px;
  }
  
  @media screen and (max-width: 768px) {
    padding: 0 12px;
  }
  
  @media screen and (max-width: 480px) {
    padding: 0 8px;
  }
}

// 响应式文字大小
:global(.text-responsive) {
  font-size: 1rem;
  
  @media screen and (max-width: 768px) {
    font-size: 0.9rem;
  }
  
  @media screen and (max-width: 480px) {
    font-size: 0.85rem;
  }
}

// 响应式间距
:global(.spacing-responsive) {
  padding: 1rem;
  
  @media screen and (max-width: 768px) {
    padding: 0.75rem;
  }
  
  @media screen and (max-width: 480px) {
    padding: 0.5rem;
  }
}
</style>
