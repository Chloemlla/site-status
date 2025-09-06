<!-- 站点状态 -->
<template>
  <header id="header" :class="{ 'header-compact': !isAtTop }">
    <!-- 背景 -->
    <Transition name="slide-fade" mode="in-out">
      <div
        :key="statusStore.siteStatus"
        :style="{ background: `var(--${statusStore.siteStatus}-cover)` }"
        :class="['status-cover', statusStore.siteStatus]"
      />
    </Transition>
    <div class="status-content">
      <!-- 状态内容 -->
      <div class="site-status">
        <!-- 状态文本 -->
        <div class="status-text">
          <div class="point" />
          <Transition name="bounce-in" mode="out-in">
            <div :key="statusStore.siteStatus" class="text">
              <span class="title" :class="{ 'title-compact': !isAtTop }">
                {{ siteGlobalText[statusStore.siteStatus] }}
              </span>
              <div v-if="isAtTop" class="details">
                <span v-if="statusStore.siteStatus === 'loading'" class="tip">
                  {{ $t("header.loading") }}
                </span>
                <span
                  v-else-if="statusStore.siteStatus === 'unknown'"
                  class="tip"
                >
                  {{ $t("header.unknown") }}
                </span>
                <!-- 更新频率 -->
                <n-flex v-else :size="0" class="tip" align="center">
                  <span>
                    {{ $t("header.update") }}
                    {{
                      formatTime(statusStore.siteData?.timestamp || 0, {
                        showTime: true,
                        showOnlyTimeIfToday: true,
                      })
                    }}
                  </span>
                  <span>
                    {{ $t("header.updateAt", { time: nextUpdateTime }) }}
                  </span>
                  <n-button
                    :focusable="false"
                    color="#fff"
                    quaternary
                    circle
                    class="refresh-btn"
                    @click="refresh"
                  >
                    <template #icon>
                      <Icon name="icon:refresh" />
                    </template>
                  </n-button>
                </n-flex>
              </div>
            </div>
          </Transition>
        </div>
      </div>
    </div>
    <!-- 波纹 -->
    <svg
      class="waves-area"
      xmlns="http://www.w3.org/2000/svg"
      xmlns:xlink="http://www.w3.org/1999/xlink"
      viewBox="0 24 150 28"
      preserveAspectRatio="none"
      shape-rendering="auto"
    >
      <defs>
        <path
          id="gentle-wave"
          d="M -160 44 c 30 0 58 -18 88 -18 s 58 18 88 18 s 58 -18 88 -18 s 58 18 88 18 v 44 h -352 Z"
        />
      </defs>
      <g class="parallax">
        <use href="#gentle-wave" x="48" y="0" />
        <use href="#gentle-wave" x="48" y="3" />
        <use href="#gentle-wave" x="48" y="5" />
        <use href="#gentle-wave" x="48" y="7" />
      </g>
    </svg>
  </header>
</template>

<script setup lang="ts">
const { t } = useI18n();
const statusStore = useStatusStore();

// 滚动位置检测
const isAtTop = ref(true);

// 监听滚动事件
const handleScroll = () => {
  isAtTop.value = window.scrollY <= 50; // 当滚动距离小于等于50px时认为在顶部
};

// 组件挂载时添加滚动监听
onMounted(() => {
  window.addEventListener('scroll', handleScroll, { passive: true });
});

// 组件卸载时移除滚动监听
onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll);
});

// 倒计时
const updateTime = ref<number>(300);

// 站点状态文本
const siteGlobalText = computed(() => ({
  loading: t("site.loading"),
  unknown: t("site.unknown"),
  normal: t("site.normal"),
  error: t("site.error"),
  warn: t("site.warn"),
}));

// 更新倒计时
const nextUpdateTime = computed(() => {
  const time = updateTime.value;
  const minutes = Math.floor(time / 60);
  const seconds = time % 60;
  return minutes > 0
    ? `${minutes} ${t("meta.minute")} ${seconds} ${t("meta.second")}`
    : `${seconds} ${t("meta.second")}`;
});

// 更新数据
const refresh = async () => {
  const lastUpdate = statusStore.siteData?.timestamp || 0;
  if (!lastUpdate) return;
  // 小于 5 分钟
  if (Date.now() - lastUpdate < 5 * 60 * 1000) {
    window.$message.warning(t("meta.fastTip"));
    return;
  }
  updateTime.value = 300;
  await getSiteData();
};

// 执行倒计时
const { pause: pauseTime, resume: resumeTime } = useIntervalFn(
  async () => {
    if (updateTime.value > 0) updateTime.value--;
    if (updateTime.value === 0) {
      pauseTime();
      statusStore.siteStatus = "loading";
      await getSiteData();
      updateTime.value = 300;
      resumeTime();
    }
  },
  1000,
  { immediate: true },
);
</script>

<style lang="scss" scoped>
header {
  position: relative;
  height: 44vh;
  width: 100%;
  color: white;
  transition: all 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94);
  
  &.header-compact {
    position: fixed;
    top: 0;
    left: 0;
    height: 70px;
    z-index: 99;
    background: var(--main-card-color);
    backdrop-filter: blur(10px);
    box-shadow: 0 2px 20px rgba(0, 0, 0, 0.1);
    border-bottom: 1px solid var(--main-border-color);
  }
  .status-cover {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-size: 400% !important;
    background: var(--loading-cover);
    z-index: -1;
    transition: filter 0.3s;
    filter: var(--cover-filter);
  }
  .status-content {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
    max-width: 900px;
    margin: 0 auto;
    padding: 30px 20px 80px;
    transition: all 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94);
    
    .site-status {
      display: flex;
      align-items: flex-end;
      justify-content: space-between;
      padding: 0 20px;
      height: 100%;
      transition: all 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94);
      
      .status-text {
        display: flex;
        align-items: center;
        margin-bottom: 12px;
        transition: all 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94);
        
        .point {
          position: relative;
          width: 40px;
          height: 40px;
          min-width: 40px;
          background-color: #fff;
          border-radius: 50%;
          margin-right: 30px;
          transition: all 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94);
          
          &::after {
            content: "";
            background-color: #ffffff80;
            position: absolute;
            width: 100%;
            height: 100%;
            top: 0;
            left: 0;
            border-radius: 50%;
            opacity: 1;
            z-index: -1;
            animation: breathing 1.5s ease infinite;
            transition: background-color 1s;
          }
        }
        
        .text {
          display: flex;
          flex-direction: column;
          
          .title {
            font-size: 40px;
            font-weight: bold;
            transition: all 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94);
            
            &.title-compact {
              font-size: 24px;
              line-height: 1.2;
            }
          }
          
          .details {
            transition: all 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94);
          }
          
          .tip {
            font-size: 14px;
            opacity: 0.8;
            .n-button {
              --n-height: 22px;
              margin-left: 10px;
            }
            span {
              &:first-child {
                &::after {
                  content: "|";
                  font-size: 12px;
                  margin: 0 8px;
                  opacity: 0.6;
                }
              }
            }
          }
        }
      }
    }
  }
  
  // 紧凑模式样式
  &.header-compact {
    color: var(--text-color-1);
    
    .status-cover {
      display: none;
    }
    
    .status-content {
      padding: 10px 20px;
      
      .site-status {
        align-items: center;
        height: auto;
        padding: 0;
        
        .status-text {
          margin-bottom: 0;
          
          .point {
            width: 20px;
            height: 20px;
            min-width: 20px;
            margin-right: 12px;
            background-color: var(--primary-color);
            
            &::after {
              background-color: var(--primary-color-hover);
            }
          }
          
          .text {
            .title {
              &.title-compact {
                font-size: 18px;
                line-height: 1.3;
                color: var(--text-color-1);
              }
            }
          }
        }
      }
    }
    
    .waves-area {
      display: none;
    }
  }
  .waves-area {
    width: 100%;
    height: 60px;
    position: absolute;
    bottom: 0;
    left: 0;
    z-index: -1;
    pointer-events: none;
    @media (max-width: 512px) {
      height: 40px;
    }
    .parallax {
      > use {
        animation: move-forever 25s cubic-bezier(0.55, 0.5, 0.45, 0.5) infinite;
        transition: fill 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        will-change: transform, fill;
      }

      > use:nth-child(1) {
        animation-delay: -2s;
        animation-duration: 7s;
        fill: rgba(var(--cover-fill-color), 0.741);
      }

      > use:nth-child(2) {
        animation-delay: -3s;
        animation-duration: 10s;
        fill: rgba(var(--cover-fill-color), 0.51);
      }

      > use:nth-child(3) {
        animation-delay: -4s;
        animation-duration: 13s;
        fill: rgba(var(--cover-fill-color), 0.212);
      }

      > use:nth-child(4) {
        animation-delay: -5s;
        animation-duration: 20s;
        fill: var(--main-bg-color);
      }
    }
  }
}

// 自定义过渡动画
.slide-fade-enter-active {
  transition: all 0.8s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}

.slide-fade-leave-active {
  transition: all 0.6s cubic-bezier(0.55, 0.085, 0.68, 0.53);
}

.slide-fade-enter-from {
  transform: translateX(-30px) scale(0.95);
  opacity: 0;
}

.slide-fade-leave-to {
  transform: translateX(30px) scale(1.05);
  opacity: 0;
}

.bounce-in-enter-active {
  animation: bounce-in 0.8s cubic-bezier(0.68, -0.55, 0.265, 1.55);
}

.bounce-in-leave-active {
  animation: bounce-out 0.5s cubic-bezier(0.55, 0.085, 0.68, 0.53);
}

@keyframes bounce-in {
  0% {
    transform: scale(0.3) translateY(20px);
    opacity: 0;
  }
  50% {
    transform: scale(1.05) translateY(-5px);
  }
  70% {
    transform: scale(0.95) translateY(2px);
  }
  100% {
    transform: scale(1) translateY(0);
    opacity: 1;
  }
}

@keyframes bounce-out {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  100% {
    transform: scale(0.3) translateY(-20px);
    opacity: 0;
  }
}

.refresh-btn {
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  
  &:hover {
    transform: rotate(180deg) scale(1.1);
  }
  
  &:active {
    transform: rotate(360deg) scale(0.95);
  }
}

@keyframes move-forever {
  0% {
    transform: translate3d(-90px, 0, 0);
  }
  100% {
    transform: translate3d(85px, 0, 0);
  }
}

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
</style>
