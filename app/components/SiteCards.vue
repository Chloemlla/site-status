<!-- Monitor cards -->
<template>
  <Transition name="fade" mode="out-in">
    <section v-if="!isEmpty(siteData)" class="site-cards">
      <div class="cards-toolbar">
        <div>
          <span class="section-kicker">{{ $t("card.overview") }}</span>
          <h2>{{ $t("card.monitors", { count: totalSites }) }}</h2>
        </div>
        <span class="updated-at">
          {{ $t("header.update") }}
          {{ lastUpdateText }}
        </span>
      </div>

      <article
        v-for="(site, index) in siteData"
        :key="site.id || index"
        :style="{
          '--status-color': `var(--${getSiteStatusType(site.status)}-color)`,
          animationDelay: `${index * 0.05}s`,
        }"
        class="site-item"
      >
        <div class="card-header">
          <div class="site-title-block">
            <span class="site-icon">
              <Icon name="icon:link" />
            </span>
            <div class="site-title">
              <h3>{{ site.name }}</h3>
              <div class="site-meta">
                <n-popover>
                  <template #trigger>
                    <button type="button" class="meta-chip">
                      {{ siteTypeMap[site.type]?.tag || "HTTP" }}
                    </button>
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
                <span class="meta-chip">{{ formatInterval(site?.interval) }}</span>
              </div>
            </div>
          </div>

          <div class="card-actions">
            <button
              :class="['status-pill', getSiteStatusType(site.status)]"
              type="button"
            >
              <span v-if="site.status !== 0" class="status-dot" />
              <Icon v-else name="icon:pause" />
              {{ siteStatusMap[site.status]?.text }}
            </button>
            <n-button
              v-if="site?.url"
              :focusable="false"
              circle
              secondary
              class="link-button"
              @click="jumpLink(site.url)"
            >
              <template #icon>
                <Icon name="icon:link" />
              </template>
            </n-button>
          </div>
        </div>

        <div v-if="site?.days?.length" class="timeline" role="list">
          <n-popover
            v-for="(day, dayIndex) in site.days"
            :key="day?.date || dayIndex"
          >
            <template #trigger>
              <button
                :style="{
                  backgroundColor: `var(--${getDayStatus(day.percent)}-color)`,
                }"
                :aria-label="day?.date ? formatTime(day.date) : $t('card.unknownDate')"
                class="day"
                type="button"
              />
            </template>
            <div class="day-data">
              <n-text class="date" depth="3">
                {{ day?.date ? formatTime(day.date) : $t("card.unknownDate") }}
              </n-text>
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
        </div>

        <div class="summary">
          <span class="date">{{ formatTime(site?.days?.[0]?.date || 0) }}</span>
          <strong>{{ site.percent }}%</strong>
          <span class="summary-copy">
            <template v-if="site?.down?.times">
              {{
                $t("card.summaryData", {
                  days: site?.days?.length,
                  times: site?.down?.times,
                  duration: formatDuration(site?.down?.duration),
                  percent: site?.percent,
                })
              }}
            </template>
            <template v-else>
              {{
                $t("card.summary", {
                  days: site?.days?.length,
                  percent: site?.percent,
                })
              }}
            </template>
          </span>
          <span class="date today">{{ $t("meta.today") }}</span>
        </div>
      </article>
    </section>

    <section
      v-else
      :style="{ '--color': `var(--${statusStore.siteStatus}-color)` }"
      class="site-cards loading"
    >
      <div class="empty-card">
        <Transition name="fade" mode="out-in">
          <div v-if="statusStore.siteStatus !== 'unknown'" class="loading-state">
            <n-spin />
            <span>{{ $t("header.loading") }}</span>
          </div>
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
      </div>
    </section>
  </Transition>
</template>

<script setup lang="ts">
import type { SiteStatusType, SiteType } from "~~/types/main";

const { t } = useI18n();
const statusStore = useStatusStore();

const siteStatusMap = computed<Record<SiteStatusType["status"], { text: string; type: SiteType }>>(() => ({
  0: { text: t("card.status.stop"), type: "unknown" },
  1: { text: t("card.status.unknown"), type: "unknown" },
  2: { text: t("card.status.normal"), type: "normal" },
  8: { text: t("card.status.error"), type: "error" },
  9: { text: t("card.status.down"), type: "error" },
}));

const siteTypeMap = computed(() => ({
  1: { tag: "HTTP", text: t("card.type.HTTP") },
  2: { tag: "KEYWORD", text: t("card.type.KEYWORD") },
  3: { tag: "PING", text: t("card.type.PING") },
  4: { tag: "PORT", text: t("card.type.PORT") },
  5: { tag: "HEARTBEAT", text: t("card.type.HEARTBEAT") },
}));

const siteData = computed<SiteStatusType[] | undefined>(
  () => statusStore.siteData?.data,
);

const totalSites = computed(() => siteData.value?.length || 0);

const lastUpdateText = computed(() =>
  statusStore.siteData?.timestamp
    ? formatTime(statusStore.siteData.timestamp, {
      showTime: true,
      showOnlyTimeIfToday: true,
    })
    : "--",
);

const getSiteStatusType = (status: SiteStatusType["status"]): SiteType =>
  siteStatusMap.value[status]?.type || "unknown";

const getDayStatus = (percent: number): SiteType => {
  if (percent >= 100) return "normal";
  if (percent >= 50 && percent < 100) return "warn";
  if (percent > 0 && percent < 50) return "error";
  return "unknown";
};

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
  gap: 16px;
  width: min(1120px, calc(100% - 40px));
  margin: -36px auto 34px;
  position: relative;
  z-index: 2;
}

.cards-toolbar {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 20px;
  padding: 0 4px 4px;

  .section-kicker {
    display: block;
    color: var(--main-muted-color);
    font-size: 12px;
    font-weight: 750;
    letter-spacing: 0.08em;
    text-transform: uppercase;
  }

  h2 {
    margin: 4px 0 0;
    color: var(--main-text-color);
    font-size: 26px;
    line-height: 1.18;
    letter-spacing: 0;
  }

  .updated-at {
    color: var(--main-muted-color);
    font-size: 13px;
    white-space: nowrap;
  }
}

.site-item {
  opacity: 0;
  display: flex;
  flex-direction: column;
  gap: 18px;
  padding: 22px;
  border: 1px solid var(--mian-border-color);
  border-radius: 26px;
  background: var(--main-card-color);
  box-shadow: var(--main-card-shadow);
  backdrop-filter: var(--main-backdrop-blur);
  animation: card-in 0.45s ease forwards;
  transition:
    border-color 0.2s ease,
    box-shadow 0.2s ease,
    transform 0.2s ease;

  &:hover {
    border-color: color-mix(in srgb, var(--status-color) 38%, var(--mian-border-color));
    box-shadow: var(--main-hover-shadow);
    transform: translateY(-3px);
  }
}

.card-header,
.site-title-block,
.card-actions,
.site-meta,
.status-pill,
.summary {
  display: flex;
  align-items: center;
}

.card-header {
  justify-content: space-between;
  gap: 18px;
}

.site-title-block {
  min-width: 0;
  gap: 14px;
}

.site-icon {
  display: grid;
  place-items: center;
  width: 44px;
  height: 44px;
  flex: 0 0 auto;
  border-radius: 16px;
  color: var(--status-color);
  background: color-mix(in srgb, var(--status-color) 13%, transparent);
  border: 1px solid color-mix(in srgb, var(--status-color) 20%, transparent);

  svg {
    font-size: 21px;
  }
}

.site-title {
  min-width: 0;

  h3 {
    margin: 0;
    color: var(--main-text-color);
    font-size: 18px;
    line-height: 1.25;
    font-weight: 780;
    overflow-wrap: anywhere;
  }
}

.site-meta {
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 8px;
}

.meta-chip {
  display: inline-flex;
  align-items: center;
  min-height: 24px;
  padding: 0 9px;
  border: 1px solid var(--mian-border-color);
  border-radius: 999px;
  color: var(--main-muted-color);
  background: transparent;
  font-size: 12px;
  line-height: 1;
}

button.meta-chip {
  cursor: pointer;
}

.card-actions {
  flex: 0 0 auto;
  justify-content: flex-end;
  gap: 10px;
}

.status-pill {
  gap: 8px;
  min-height: 36px;
  padding: 0 13px;
  border: 1px solid color-mix(in srgb, var(--status-color) 24%, transparent);
  border-radius: 999px;
  color: var(--status-color);
  background: color-mix(in srgb, var(--status-color) 11%, transparent);
  font-size: 13px;
  font-weight: 700;
  white-space: nowrap;
  cursor: default;

  svg {
    font-size: 18px;
  }
}

.status-dot {
  width: 8px;
  height: 8px;
  border-radius: 999px;
  background: currentColor;
  box-shadow: 0 0 0 4px color-mix(in srgb, currentColor 16%, transparent);
}

.link-button {
  flex: 0 0 auto;
}

.timeline {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(4px, 1fr));
  gap: 3px;
  min-height: 34px;
  padding: 7px;
  border-radius: 16px;
  background: color-mix(in srgb, var(--main-muted-color) 8%, transparent);
}

.day {
  min-width: 4px;
  height: 20px;
  border: 0;
  border-radius: 999px;
  cursor: pointer;
  opacity: 0.9;
  transition:
    opacity 0.15s ease,
    transform 0.15s ease;

  &:hover {
    opacity: 1;
    transform: translateY(-2px) scaleY(1.2);
  }
}

.summary {
  display: grid;
  grid-template-columns: minmax(70px, auto) auto minmax(0, 1fr) minmax(52px, auto);
  gap: 14px;
  padding-top: 2px;
  color: var(--main-muted-color);
  font-size: 13px;
  line-height: 1.5;

  strong {
    color: var(--status-color);
    font-size: 22px;
    line-height: 1;
  }

  .summary-copy {
    min-width: 0;
  }

  .today {
    text-align: right;
  }
}

.loading {
  margin-top: -36px;
}

.empty-card {
  display: grid;
  min-height: 260px;
  place-items: center;
  border: 1px solid var(--mian-border-color);
  border-radius: 26px;
  background: var(--main-card-color);
  box-shadow: var(--main-card-shadow);
  backdrop-filter: var(--main-backdrop-blur);
}

.loading-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 14px;
  color: var(--main-muted-color);

  :deep(.n-spin-body) {
    --n-size: 42px;
    --n-color: var(--color);
  }
}

.day-data {
  display: flex;
  flex-direction: column;
  gap: 4px;
  max-width: 260px;

  .date {
    font-size: 12px;
  }
}

@keyframes card-in {
  from {
    opacity: 0;
    transform: translateY(14px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (max-width: 720px) {
  .site-cards {
    width: min(100% - 28px, 1120px);
    margin-top: -28px;
  }

  .cards-toolbar,
  .card-header {
    align-items: stretch;
    flex-direction: column;
  }

  .cards-toolbar .updated-at {
    white-space: normal;
  }

  .card-actions {
    justify-content: space-between;
  }

  .status-pill {
    flex: 1 1 auto;
    justify-content: center;
  }

  .summary {
    grid-template-columns: 1fr auto;

    .summary-copy {
      grid-column: 1 / -1;
      order: 3;
    }
  }
}

@media (max-width: 480px) {
  .site-item {
    padding: 18px;
    border-radius: 22px;
  }

  .site-icon {
    width: 40px;
    height: 40px;
    border-radius: 14px;
  }

  .timeline {
    gap: 2px;
    padding: 6px;
  }
}
</style>
