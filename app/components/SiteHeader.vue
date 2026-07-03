<!-- Site status hero -->
<template>
  <header id="header">
    <Transition name="fade" mode="in-out">
      <div
        :key="statusStore.siteStatus"
        :style="{ background: `var(--${statusStore.siteStatus}-cover)` }"
        :class="['status-cover', statusStore.siteStatus]"
      />
    </Transition>

    <div class="header-shell">
      <section class="status-panel">
        <div class="status-copy">
          <div class="eyebrow">
            <span
              :style="{ backgroundColor: `var(--${statusStore.siteStatus}-color)` }"
              class="status-dot"
            />
            <span>{{ config.public.siteTitle }}</span>
          </div>

          <Transition name="fade" mode="out-in">
            <div :key="statusStore.siteStatus" class="status-title-group">
              <h1>{{ siteGlobalText[statusStore.siteStatus] }}</h1>
              <p>{{ statusDescription }}</p>
            </div>
          </Transition>
        </div>

        <div class="refresh-panel">
          <span class="refresh-label">{{ $t("header.update") }}</span>
          <strong>{{ lastUpdateText }}</strong>
          <span>{{ $t("header.updateAt", { time: nextUpdateTime }) }}</span>
          <n-button
            :focusable="false"
            secondary
            circle
            class="refresh-button"
            @click="refresh"
          >
            <template #icon>
              <Icon name="icon:refresh" />
            </template>
          </n-button>
        </div>

        <div class="metric-grid">
          <div
            v-for="item in statusMetrics"
            :key="item.key"
            :class="['metric-card', item.key]"
          >
            <span>{{ item.label }}</span>
            <strong>{{ item.value }}</strong>
          </div>
        </div>
      </section>
    </div>

    <svg
      class="waves-area"
      xmlns="http://www.w3.org/2000/svg"
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
const config = useRuntimeConfig();
const statusStore = useStatusStore();

const updateTime = ref<number>(300);

const siteGlobalText = computed(() => ({
  loading: t("site.loading"),
  unknown: t("site.unknown"),
  normal: t("site.normal"),
  error: t("site.error"),
  warn: t("site.warn"),
}));

const statusDescription = computed(() => {
  if (statusStore.siteStatus === "loading") return t("header.loading");
  if (statusStore.siteStatus === "unknown") return t("header.unknown");
  return t("header.updateAt", { time: nextUpdateTime.value });
});

const lastUpdateText = computed(() =>
  statusStore.siteData?.timestamp
    ? formatTime(statusStore.siteData.timestamp, {
      showTime: true,
      showOnlyTimeIfToday: true,
    })
    : "--",
);

const statusMetrics = computed(() => {
  const status = statusStore.siteData?.status;
  return [
    { key: "total", label: t("header.metrics.total"), value: status?.count ?? "--" },
    { key: "normal", label: t("header.metrics.normal"), value: status?.ok ?? "--" },
    {
      key: "issue",
      label: t("header.metrics.issues"),
      value: status ? status.error + status.unknown : "--",
    },
  ];
});

const nextUpdateTime = computed(() => {
  const time = updateTime.value;
  const minutes = Math.floor(time / 60);
  const seconds = time % 60;
  return minutes > 0
    ? `${minutes} ${t("meta.minute")} ${seconds} ${t("meta.second")}`
    : `${seconds} ${t("meta.second")}`;
});

const refresh = async () => {
  const lastUpdate = statusStore.siteData?.timestamp || 0;
  if (!lastUpdate) return;
  if (Date.now() - lastUpdate < 5 * 60 * 1000) {
    window.$message.warning(t("meta.fastTip"));
    return;
  }
  updateTime.value = 300;
  await getSiteData();
};

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
  min-height: 430px;
  width: 100%;
  color: #fff;
  overflow: hidden;

  .status-cover {
    position: absolute;
    inset: 0;
    z-index: -2;
    background-size: 180% 180% !important;
    filter: var(--cover-filter, saturate(1.06));
  }

  .status-cover::after {
    content: "";
    position: absolute;
    inset: 0;
    background:
      linear-gradient(180deg, rgba(2, 6, 23, 0.14), rgba(2, 6, 23, 0.32)),
      radial-gradient(circle at 78% 18%, rgba(255, 255, 255, 0.22), transparent 28%);
  }

  .header-shell {
    width: min(1120px, calc(100% - 40px));
    min-height: 430px;
    margin: 0 auto;
    padding: 108px 0 96px;
  }

  .status-panel {
    display: grid;
    grid-template-columns: minmax(0, 1fr) minmax(220px, 280px);
    gap: 22px;
    align-items: end;
  }

  .status-copy {
    min-width: 0;
  }

  .eyebrow {
    display: inline-flex;
    align-items: center;
    max-width: 100%;
    gap: 10px;
    padding: 8px 12px;
    border: 1px solid rgba(255, 255, 255, 0.24);
    border-radius: 999px;
    background: rgba(255, 255, 255, 0.12);
    backdrop-filter: blur(16px);
    color: rgba(255, 255, 255, 0.88);
    font-size: 13px;
    font-weight: 650;
  }

  .status-dot {
    width: 10px;
    height: 10px;
    border-radius: 999px;
    box-shadow: 0 0 0 6px rgba(255, 255, 255, 0.15);
  }

  .status-title-group {
    margin-top: 24px;

    h1 {
      max-width: 760px;
      margin: 0;
      font-size: clamp(34px, 7vw, 76px);
      line-height: 0.98;
      font-weight: 850;
      letter-spacing: 0;
      text-wrap: balance;
    }

    p {
      max-width: 600px;
      margin: 18px 0 0;
      color: rgba(255, 255, 255, 0.8);
      font-size: 16px;
      line-height: 1.7;
    }
  }

  .refresh-panel {
    display: flex;
    flex-direction: column;
    gap: 6px;
    align-self: stretch;
    min-height: 168px;
    padding: 20px;
    border: 1px solid rgba(255, 255, 255, 0.24);
    border-radius: 24px;
    background: rgba(255, 255, 255, 0.14);
    box-shadow: 0 24px 60px rgba(2, 6, 23, 0.18);
    backdrop-filter: blur(18px) saturate(140%);

    .refresh-label,
    span:last-of-type {
      color: rgba(255, 255, 255, 0.72);
      font-size: 13px;
      line-height: 1.5;
    }

    strong {
      color: #fff;
      font-size: 26px;
      line-height: 1.2;
    }

    .refresh-button {
      margin-top: auto;
      align-self: flex-end;
      color: #fff;
      --n-color: rgba(255, 255, 255, 0.16);
      --n-color-hover: rgba(255, 255, 255, 0.24);
      --n-color-pressed: rgba(255, 255, 255, 0.2);
      --n-text-color: #fff;
      --n-text-color-hover: #fff;
      --n-text-color-pressed: #fff;
      --n-border: 1px solid rgba(255, 255, 255, 0.22);
      --n-border-hover: 1px solid rgba(255, 255, 255, 0.36);
      --n-border-pressed: 1px solid rgba(255, 255, 255, 0.3);
    }
  }

  .metric-grid {
    grid-column: 1 / -1;
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 12px;
    margin-top: 8px;
  }

  .metric-card {
    min-width: 0;
    padding: 18px;
    border: 1px solid rgba(255, 255, 255, 0.2);
    border-radius: 20px;
    background: rgba(255, 255, 255, 0.12);
    backdrop-filter: blur(14px);

    span {
      display: block;
      color: rgba(255, 255, 255, 0.7);
      font-size: 13px;
      line-height: 1.4;
    }

    strong {
      display: block;
      margin-top: 8px;
      color: #fff;
      font-size: 28px;
      line-height: 1;
    }
  }

  .waves-area {
    position: absolute;
    left: 0;
    bottom: -1px;
    z-index: -1;
    width: 100%;
    height: 74px;
    pointer-events: none;
  }

  .parallax > use {
    animation: move-forever 22s cubic-bezier(0.55, 0.5, 0.45, 0.5) infinite;
  }

  .parallax > use:nth-child(1) {
    animation-delay: -2s;
    animation-duration: 7s;
    fill: rgba(var(--cover-fill-color), 0.26);
  }

  .parallax > use:nth-child(2) {
    animation-delay: -3s;
    animation-duration: 10s;
    fill: rgba(var(--cover-fill-color), 0.42);
  }

  .parallax > use:nth-child(3) {
    animation-delay: -4s;
    animation-duration: 13s;
    fill: rgba(var(--cover-fill-color), 0.62);
  }

  .parallax > use:nth-child(4) {
    animation-delay: -5s;
    animation-duration: 20s;
    fill: var(--main-bg-color);
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

@media (max-width: 820px) {
  header {
    min-height: 560px;

    .header-shell {
      width: min(100% - 28px, 1120px);
      min-height: 560px;
      padding: 92px 0 82px;
    }

    .status-panel {
      grid-template-columns: 1fr;
    }

    .refresh-panel {
      min-height: auto;
    }
  }
}

@media (max-width: 560px) {
  header {
    min-height: 590px;

    .header-shell {
      min-height: 590px;
    }

    .metric-grid {
      grid-template-columns: 1fr;
    }

    .metric-card {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 12px;
      padding: 14px 16px;

      strong {
        margin-top: 0;
        font-size: 22px;
      }
    }
  }
}
</style>
