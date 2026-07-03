<template>
  <footer id="footer">
    <div class="footer-shell">
      <div class="footer-brand">
        <button type="button" @click="jumpLink(linkData.github)">
          SiteStatus
        </button>
        <span>Version {{ version }}</span>
      </div>

      <div class="footer-copy">
        <span>
          {{ $t("footer.basedOn") }}
          <button type="button" @click="jumpLink('https://uptimerobot.com/')">
            {{ $t("uptimeRobot") }}
          </button>
          {{ $t("footer.interface") }}
        </span>
        <span>
          {{ $t("footer.checkFrequency") }}
          {{ $t("footer.fiveMinutes") }}
        </span>
        <span>
          Copyright &copy; 2020 - {{ new Date().getFullYear() }}
          <button type="button" @click="jumpLink(linkData.home)">Chloemlla</button>
        </span>
      </div>

      <n-flex class="link" align="center" justify="end">
        <n-button
          v-for="(item, key, index) in linkData"
          :key="index"
          :focusable="false"
          quaternary
          circle
          class="footer-btn"
          @click="jumpLink(item)"
        >
          <template #icon>
            <Icon :name="`icon:${key}`" />
          </template>
        </n-button>
      </n-flex>
    </div>
  </footer>
</template>

<script setup lang="ts">
const { public: configPublic } = useRuntimeConfig();
const { version } = configPublic;

const linkData = {
  github: "https://github.com/Chloemlla/site-status",
  home: "https://github.com/Chloemlla",
  email: "mailto:noreply@hapxs.com",
};
</script>

<style lang="scss" scoped>
footer {
  padding: 38px 20px 64px;
  margin-top: auto;
  z-index: 100;
}

.footer-shell {
  display: grid;
  grid-template-columns: minmax(160px, auto) minmax(0, 1fr) auto;
  gap: 22px;
  align-items: center;
  width: min(1120px, 100%);
  margin: 0 auto;
  padding: 18px 20px;
  border: 1px solid var(--mian-border-color);
  border-radius: 24px;
  background: var(--main-card-color);
  box-shadow: var(--main-card-shadow);
  backdrop-filter: var(--main-backdrop-blur);
}

.footer-brand {
  display: flex;
  flex-direction: column;
  gap: 4px;

  button {
    padding: 0;
    border: 0;
    color: var(--main-text-color);
    background: transparent;
    font-size: 15px;
    font-weight: 800;
    line-height: 1.2;
    text-align: left;
    cursor: pointer;
  }

  span {
    color: var(--main-muted-color);
    font-size: 12px;
  }
}

.footer-copy {
  display: flex;
  flex-wrap: wrap;
  gap: 6px 16px;
  color: var(--main-muted-color);
  font-size: 12px;
  line-height: 1.6;

  button {
    padding: 0;
    border: 0;
    color: var(--main-text-color);
    background: transparent;
    font-weight: 700;
    cursor: pointer;
  }
}

.link {
  .footer-btn {
    transition:
      background-color 0.2s ease,
      transform 0.2s ease;

    &:hover {
      background-color: color-mix(in srgb, var(--normal-color) 12%, transparent);
      transform: translateY(-1px);
    }
  }
}

@media (max-width: 820px) {
  .footer-shell {
    grid-template-columns: 1fr;
    gap: 16px;
  }

  .link {
    justify-content: flex-start !important;
  }
}
</style>
