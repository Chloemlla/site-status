<template>
  <footer id="footer">
    <n-flex class="link" align="center">
      <n-button
        v-for="(item, key, index) in linkData"
        :key="index"
        :focusable="false"
        quaternary
        circle
        class="footer-btn"
        :style="{ animationDelay: `${index * 0.1}s` }"
        @click="jumpLink(item)"
      >
        <template #icon>
          <Icon :name="`icon:${key}`" />
        </template>
      </n-button>
    </n-flex>
    <n-flex :size="4" class="text" align="center" vertical>
      <n-p depth="3">
        <n-text depth="3" @click="jumpLink(linkData.github)">
          SiteStatus
        </n-text>
        Version {{ version }}
      </n-p>
      <n-p depth="3">
        {{ $t("footer.basedOn") }}
        <n-text depth="3" @click="jumpLink('https://uptimerobot.com/')">
          {{ $t("uptimeRobot") }}
        </n-text>
        {{ $t("footer.interface") }} |
        {{ $t("footer.checkFrequency") }}
        {{ $t("footer.fiveMinutes") }}
      </n-p>
      <n-p depth="3">
        Copyright &copy; 2020 - {{ new Date().getFullYear() }}
        <n-text depth="3" @click="jumpLink(linkData.home)"> devhappys </n-text>
      </n-p>
    </n-flex>
  </footer>
</template>

<script setup lang="ts">
const { public: configPublic } = useRuntimeConfig();
const { version } = configPublic;

const linkData = {
  github: "https://github.com/devhappys/site-status",
  home: "https://github.com/devhappys",
  email: "mailto:noreply@hapxs.com",
};
</script>

<style lang="scss" scoped>
footer {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 60px 20px 90px;
  margin-top: auto;
  z-index: 100;
  opacity: 0;
  animation: fade-in-up 1s ease-out 0.5s forwards;
  
  .link {
    .footer-btn {
      opacity: 0;
      transform: translateY(20px) scale(0.8);
      animation: bounce-in-footer 0.6s cubic-bezier(0.68, -0.55, 0.265, 1.55) forwards;
      transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
      
      &:hover {
        transform: translateY(-5px) scale(1.15) rotate(10deg);
        box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
      }
      
      &:active {
        transform: translateY(-2px) scale(1.05);
      }
    }
  }
  
  .text {
    margin-top: 12px;
    .n-p,
    .n-text {
      margin: 0;
      font-size: 13px;
      line-height: 26px;
    }
    .n-text {
      font-weight: bold;
      cursor: pointer;
      transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
      
      &:hover {
        color: var(--normal-color);
        transform: scale(1.05);
      }
    }
  }
}

@keyframes fade-in-up {
  0% {
    opacity: 0;
    transform: translateY(30px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes bounce-in-footer {
  0% {
    opacity: 0;
    transform: translateY(20px) scale(0.8);
  }
  60% {
    opacity: 1;
    transform: translateY(-5px) scale(1.1);
  }
  80% {
    transform: translateY(2px) scale(0.95);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}
</style>
