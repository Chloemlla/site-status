<!-- 站点数据卡片 -->
<template>
  <Transition name="fade" mode="out-in">
    <div v-if="!isEmpty(siteData)" class="site-cards">
      <n-card
        v-for="(site, index) in siteData"
        :key="index"
        :style="{ animationDelay: `${index * 0.1}s` }"
        class="site-item"
        hoverable
      >
        <!-- 信息 -->
        <n-flex class="meta" justify="space-between">
          <n-flex :size="8" class="title" align="center">
            <n-text class="site-name">{{ site.name }}</n-text>
            <n-popover>
              <template #trigger>
                <n-tag :bordered="false" size="small" round>
                  {{ siteTypeMap[site.type]?.tag || "HTTP" }} /
                  {{ formatInterval(site?.interval) }}
                </n-tag>
              </template>
              <n-text>
                {{
                  $t("card.type.tip", {
                    interval: formatInterval(site?.interval) || "30s",
                    type: siteTypeMap[site.type]?.text,
                  })
                }}
              </n-text>
            </n-popover>
            <!-- 跳转 -->
            <n-button
              v-if="site?.url"
              :focusable="false"
              size="tiny"
              tertiary
              round
              @click="jumpLink(site.url)"
            >
              <template #icon>
                <Icon name="icon:link" />
              </template>
            </n-button>
          </n-flex>
          <n-flex
            :style="{
              '--bg-color': `var(--${siteStatusMap[site.status]?.type || 'unknown'}-color)`,
            }"
            class="status"
            align="center"
          >
            <div v-if="site.status !== 0" class="point" />
            <Icon v-else name="icon:pause" />
            <n-text>{{ siteStatusMap[site.status]?.text }}</n-text>
          </n-flex>
        </n-flex>
        <!-- 每日数据 -->
        <n-flex
          v-if="site?.days?.length"
          :size="2"
          class="timeline"
          justify="space-between"
        >
          <n-popover
            v-for="(day, dayIndex) in site.days"
            :key="day?.date || dayIndex"
          >
            <template #trigger>
              <div
                :style="{
                  backgroundColor: `var(--${getDayStatus(day.percent)}-color)`,
                }"
                class="day"
              />
            </template>
            <div class="day-data">
              <n-text class="date" depth="3">
                {{ day?.date ? formatTime(day.date) : $t("card.unknownDate") }}
              </n-text>
              <!-- 详细 -->
              <n-text v-if="day?.percent >= 100">
                {{ $t("card.percent", { percent: day?.percent }) }}
              </n-text>
              <n-text v-else-if="day?.percent > 0 && day?.percent < 100">
                {{
                  $t("card.percentData", {
                    times: day?.down?.times,
                    duration: formatDuration(day?.down?.duration),
                    percent: day?.percent,
                  })
                }}
              </n-text>
              <n-text v-else>{{ $t("card.unknownData") }}</n-text>
            </div>
          </n-popover>
        </n-flex>
        <!-- 总结 -->
        <n-flex class="summary" justify="space-between">
          <n-text class="date" depth="3">
            {{ formatTime(site?.days?.[0]?.date || 0) }}
          </n-text>
          <n-text v-if="site?.down?.times" depth="3">
            {{
              $t("card.summaryData", {
                days: site?.days?.length,
                times: site?.down?.times,
                duration: formatDuration(site?.down?.duration),
                percent: site?.percent,
              })
            }}
          </n-text>
          <n-text v-else depth="3">
            {{
              $t("card.summary", {
                days: site?.days?.length,
                percent: site?.percent,
              })
            }}
          </n-text>
          <n-text class="date" depth="3">{{ $t("meta.today") }}</n-text>
        </n-flex>
      </n-card>
    </div>
    <div
      v-else
      :style="{ '--color': `var(--${statusStore.siteStatus}-color)` }"
      class="site-cards loading"
    >
      <n-card class="site-item" hoverable>
        <Transition name="fade" mode="out-in">
          <n-spin v-if="statusStore.siteStatus !== 'unknown'" />
          <n-result
            v-else
            status="error"
            :title="$t('card.error')"
            :description="$t('card.errorText')"
          >
            <template #footer>
              <n-button tertiary round @click="refresh">
                {{ $t("meta.refresh") }}
              </n-button>
            </template>
          </n-result>
        </Transition>
      </n-card>
    </div>
  </Transition>
</template>

<script setup lang="ts">
import type { SiteStatusType, SiteType } from "~~/types/main";

const { t } = useI18n();
const statusStore = useStatusStore();

// 站点类型
const siteStatusMap = computed(() => ({
  0: { text: t("card.status.stop"), type: "unknown" },
  1: { text: t("card.status.unknown"), type: "unknown" },
  2: { text: t("card.status.normal"), type: "normal" },
  8: { text: t("card.status.error"), type: "error" },
  9: { text: t("card.status.down"), type: "error" },
}));

// 请求类型
const siteTypeMap = computed(() => ({
  1: { tag: "HTTP", text: t("card.type.HTTP") },
  2: { tag: "KEYWORD", text: t("card.type.KEYWORD") },
  3: { tag: "PING", text: t("card.type.PING") },
  4: { tag: "PORT", text: t("card.type.PORT") },
  5: { tag: "HEARTBEAT", text: t("card.type.HEARTBEAT") },
}));

// 全部站点数据
const siteData = computed<SiteStatusType[] | undefined>(
  () => statusStore.siteData?.data,
);

// 当天站点状态
const getDayStatus = (percent: number): SiteType => {
  if (percent >= 100) return "normal";
  else if (percent >= 50 && percent < 100) return "warn";
  else if (percent > 0 && percent < 50) return "error";
  else return "unknown";
};

// 重试
const refresh = async () => {
  statusStore.$patch({
    siteStatus: "loading",
    siteData: undefined,
  });
  await getSiteData();
};

onMounted(getSiteData);
</script>

<style lang="scss" scoped>
.site-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
  max-width: 900px;
  margin: 30px auto 20px;
  padding: 0 20px;
  .site-item {
    opacity: 0;
    border-radius: 12px;
    animation: float-up 0.8s cubic-bezier(0.25, 0.46, 0.45, 0.94) forwards;
    overflow: hidden;
    transform: translateY(30px) scale(0.95);
    transition: all 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94);
    
    &:hover {
      transform: translateY(-8px) scale(1.02);
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.15);
    }
    .meta {
      .site-name {
        font-weight: bold;
      }
      .n-tag {
        --n-height: 20px;
        cursor: pointer;
      }
      .status {
        .n-text {
          color: var(--bg-color);
        }
        svg {
          font-size: 22px;
          margin-right: -4px;
          color: var(--bg-color);
        }
      }
      .point {
        position: relative;
        width: 14px;
        height: 14px;
        min-width: 14px;
        background-color: var(--bg-color);
        border-radius: 50%;
        &::after {
          content: "";
          background-color: var(--bg-color);
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
    }
    .timeline {
      margin: 15px 0 10px;
      .day {
        height: 26px;
        flex: 1;
        border-radius: 25px;
        background-color: var(--normal-color);
        transition: all 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
        transform-origin: bottom;
        cursor: pointer;
        position: relative;
        overflow: hidden;
        
        &::before {
          content: '';
          position: absolute;
          top: 0;
          left: -100%;
          width: 100%;
          height: 100%;
          background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
          transition: left 0.6s;
        }
        
        &:hover {
          transform: scale(1.15) translateY(-2px);
          box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
          
          &::before {
            left: 100%;
          }
        }
      }
    }
    .summary {
      .date {
        width: 100px;
        &:last-child {
          text-align: right;
        }
      }
      .n-text {
        font-size: 13px;
      }
    }
  }
  &.loading {
    .site-item {
      min-height: 200px;
      :deep(.n-card__content) {
        padding: 24px;
        display: flex;
        align-items: center;
        justify-content: center;
      }
    }
    .n-spin-body {
      --n-size: 40px;
      --n-color: var(--color);
    }
  }
}

// 动画关键帧
@keyframes float-up {
  0% {
    opacity: 0;
    transform: translateY(30px) scale(0.95);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

@keyframes breathing {
  0%, 100% {
    transform: scale(1);
    opacity: 0.8;
  }
  50% {
    transform: scale(1.3);
    opacity: 0.4;
  }
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
}

.day-data {
  display: flex;
  flex-direction: column;
  .date {
    font-size: 12px;
  }
}
</style>
