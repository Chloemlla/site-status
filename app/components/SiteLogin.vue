<!-- Password gate -->
<template>
  <section class="site-login">
    <div class="login-shell">
      <div class="brand-panel">
        <span class="brand-icon">
          <Icon name="icon:home" />
        </span>
        <span class="brand-kicker">{{ config.public.siteDescription }}</span>
        <h1>{{ config.public.siteTitle }}</h1>
        <p>{{ $t("login.tip") }}</p>
      </div>

      <n-card class="login-content" :bordered="false">
        <n-form ref="formRef" :model="formData" :rules="formRules">
          <n-form-item :label="$t('login.password')" path="password">
            <n-input
              v-model:value="formData.password"
              :placeholder="$t('login.placeholder')"
              type="password"
              size="large"
              show-password-on="mousedown"
              @keyup.enter="toLogin"
            />
          </n-form-item>
        </n-form>
        <n-button
          :loading="loading"
          :disabled="loading"
          type="primary"
          size="large"
          block
          @click="toLogin"
        >
          {{ $t("login.submit") }}
        </n-button>
      </n-card>
    </div>
  </section>
</template>

<script setup lang="ts">
import type { FormInst, FormRules } from "naive-ui";
import SHA256 from "crypto-js/sha256";

const { t } = useI18n();
const config = useRuntimeConfig();
const statusStore = useStatusStore();

const formRef = ref<FormInst>();
const formData = ref<{ password: string }>({ password: "" });
const formRules: FormRules = {
  password: {
    required: true,
    message: t("login.placeholder"),
    trigger: ["input", "blur"],
  },
};
const loading = ref<boolean>(false);

const toLogin = useDebounce(
  async () => {
    try {
      await formRef.value?.validate();
      loading.value = true;
      const delay = Math.floor(Math.random() * 1000) + 500;
      await sleep(delay);
      const password = SHA256(formData.value.password).toString();
      await $fetch("/api/verify", { method: "POST", body: { password } });
      statusStore.loginStatus = true;
      window.$message.success(t("login.success"));
    } catch (error) {
      console.error("error in login", error);
      window.$message.error(t("login.error"));
    } finally {
      loading.value = false;
    }
  },
  300,
  { leading: true, trailing: false },
);
</script>

<style lang="scss" scoped>
.site-login {
  display: grid;
  min-height: 100vh;
  place-items: center;
  padding: 104px 20px 56px;
}

.login-shell {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(320px, 420px);
  gap: 22px;
  width: min(960px, 100%);
}

.brand-panel,
.login-content {
  border: 1px solid var(--mian-border-color);
  border-radius: 28px;
  background: var(--main-card-color);
  box-shadow: var(--main-soft-shadow);
  backdrop-filter: var(--main-backdrop-blur);
}

.brand-panel {
  position: relative;
  overflow: hidden;
  min-height: 340px;
  padding: 34px;

  &::before {
    content: "";
    position: absolute;
    inset: 0;
    background:
      linear-gradient(135deg, rgba(34, 197, 94, 0.18), transparent 46%),
      linear-gradient(315deg, rgba(14, 165, 233, 0.16), transparent 42%);
    pointer-events: none;
  }

  > * {
    position: relative;
  }

  .brand-icon {
    display: grid;
    place-items: center;
    width: 52px;
    height: 52px;
    border-radius: 18px;
    color: #fff;
    background: var(--normal-cover);
    box-shadow: 0 18px 38px rgba(22, 163, 74, 0.24);

    svg {
      font-size: 24px;
    }
  }

  .brand-kicker {
    display: block;
    margin-top: 42px;
    color: var(--main-muted-color);
    font-size: 13px;
    font-weight: 750;
    letter-spacing: 0.08em;
    text-transform: uppercase;
  }

  h1 {
    max-width: 520px;
    margin: 12px 0 0;
    color: var(--main-text-color);
    font-size: clamp(34px, 5vw, 56px);
    line-height: 1;
    font-weight: 850;
    letter-spacing: 0;
    text-wrap: balance;
  }

  p {
    max-width: 520px;
    margin: 18px 0 0;
    color: var(--main-muted-color);
    font-size: 15px;
    line-height: 1.8;
  }
}

.login-content {
  align-self: end;
  padding: 28px;

  :deep(.n-card__content) {
    padding: 0;
  }

  :deep(.n-form-item-label) {
    color: var(--main-text-color);
    font-weight: 700;
  }

  :deep(.n-input) {
    border-radius: 14px;
  }

  :deep(.n-button) {
    margin-top: 10px;
    border-radius: 14px;
    font-weight: 800;
  }
}

@media (max-width: 780px) {
  .login-shell {
    grid-template-columns: 1fr;
  }

  .brand-panel {
    min-height: auto;
  }

  .login-content {
    align-self: stretch;
  }
}

@media (max-width: 480px) {
  .site-login {
    padding: 88px 14px 36px;
  }

  .brand-panel,
  .login-content {
    border-radius: 22px;
  }

  .brand-panel,
  .login-content {
    padding: 22px;
  }
}
</style>
