<template>
  <div class="navbar">
    <div class="left-side" :style="{ width: `${appStore.menuWidth}px` }">
      <a-space>
        <img
          alt="logo"
          src="//p3-armor.byteimg.com/tos-cn-i-49unhts6dw/dfdba5317c0c20ce20e64fac803d52bc.svg~tplv-49unhts6dw-image.image"
        />
        <a-typography-title
          :style="{ margin: 0, fontSize: '18px' }"
          :heading="5"
        >
          Arco Pro
        </a-typography-title>
      </a-space>
      <icon-menu-fold
        v-if="!appStore.menuCollapse"
        style="font-size: 22px; cursor: pointer"
        @click="toggle"
      />
      <icon-menu-unfold
        v-else
        style="font-size: 22px; cursor: pointer"
        @click="toggle"
      />
    </div>
    <div class="right-side">
      <Breadcrumb :items="['menu.list', 'menu.list.cardList']" />
      <ul :style="{ display: 'flex' }">
        <li>
          <a-tooltip :content="$t('settings.language')">
            <a-button
              class="nav-btn"
              type="outline"
              :shape="'circle'"
              @click="setDropDownVisible"
            >
              <template #icon>
                <icon-language />
              </template>
            </a-button>
          </a-tooltip>
          <a-dropdown trigger="click" @select="changeLocale as any">
            <div ref="triggerBtn" class="trigger-btn"></div>
            <template #content>
              <a-doption
                v-for="item in locales"
                :key="item.value"
                :value="item.value"
              >
                {{ item.label }}
              </a-doption>
            </template>
          </a-dropdown>
        </li>
        <li>
          <a-tooltip
            :content="
              theme === 'light'
                ? $t('settings.navbar.theme.toDark')
                : $t('settings.navbar.theme.toLight')
            "
          >
            <a-button
              class="nav-btn"
              type="outline"
              :shape="'circle'"
              @click="handleToggleTheme"
            >
              <template #icon>
                <icon-moon-fill v-if="theme === 'dark'" />
                <icon-sun-fill v-else />
              </template>
            </a-button>
          </a-tooltip>
        </li>
        <li>
          <a-tooltip
            :content="
              isFullscreen
                ? $t('settings.navbar.screen.toExit')
                : $t('settings.navbar.screen.toFull')
            "
          >
            <a-button
              class="nav-btn"
              type="outline"
              :shape="'circle'"
              @click="toggleFullScreen"
            >
              <template #icon>
                <icon-fullscreen-exit v-if="isFullscreen" />
                <icon-fullscreen v-else />
              </template>
            </a-button>
          </a-tooltip>
        </li>

        <li>
          <a-dropdown trigger="click">
            <a-avatar
              :size="32"
              :style="{ marginRight: '8px', cursor: 'pointer' }"
            >
              <img alt="avatar" :src="avatar" />
            </a-avatar>
            <template #content>
              <a-doption>
                <a-space @click="$router.push({ name: 'Info' })">
                  <icon-user />
                  <span>
                    {{ $t('messageBox.userCenter') }}
                  </span>
                </a-space>
              </a-doption>
              <a-doption>
                <a-space @click="$router.push({ name: 'Setting' })">
                  <icon-settings />
                  <span>
                    {{ $t('messageBox.userSettings') }}
                  </span>
                </a-space>
              </a-doption>
              <a-doption>
                <a-space @click="handleLogout">
                  <icon-export />
                  <span>
                    {{ $t('messageBox.logout') }}
                  </span>
                </a-space>
              </a-doption>
            </template>
          </a-dropdown>
        </li>
      </ul>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import { computed, ref, inject } from 'vue';
  import { Message } from '@arco-design/web-vue';
  import { useDark, useToggle, useFullscreen } from '@vueuse/core';
  import { useAppStore, useUserStore } from '@/store';
  import { LOCALE_OPTIONS } from '@/locale';
  import useLocale from '@/hooks/locale';
  import useUser from '@/hooks/user';
  import MessageBox from '../message-box/index.vue';

  const appStore = useAppStore();
  const userStore = useUserStore();
  const { logout } = useUser();
  const { changeLocale } = useLocale();
  const { isFullscreen, toggle: toggleFullScreen } = useFullscreen();
  const locales = [...LOCALE_OPTIONS];
  const avatar = computed(() => {
    return userStore.avatar;
  });
  const theme = computed(() => {
    return appStore.theme;
  });
  const isDark = useDark({
    selector: 'body',
    attribute: 'arco-theme',
    valueDark: 'dark',
    valueLight: 'light',
    storageKey: 'arco-theme',
    onChanged(dark: boolean) {
      // overridden default behavior
      appStore.toggleTheme(dark);
    },
  });
  const toggleTheme = useToggle(isDark);
  const handleToggleTheme = () => {
    toggleTheme();
  };
  const setVisible = () => {
    appStore.updateSettings({ globalSettings: true });
  };
  const refBtn = ref();
  const triggerBtn = ref();
  const setPopoverVisible = () => {
    const event = new MouseEvent('click', {
      view: window,
      bubbles: true,
      cancelable: true,
    });
    refBtn.value.dispatchEvent(event);
  };
  const handleLogout = () => {
    logout();
  };
  const setDropDownVisible = () => {
    const event = new MouseEvent('click', {
      view: window,
      bubbles: true,
      cancelable: true,
    });
    triggerBtn.value.dispatchEvent(event);
  };
  const switchRoles = async () => {
    const res = await userStore.switchRoles();
    Message.success(res as string);
  };
  const toggle = inject('toggle');
</script>

<style scoped lang="less">
  .navbar {
    display: flex;
    height: 100%;
    background-color: var(--color-bg-2);
    border-bottom: 1px solid var(--color-border);
  }

  .left-side {
    display: flex;
    align-items: center;
    padding-left: 20px;
    justify-content: space-between;
  }

  .right-side {
    flex: 1;
    display: flex;
    padding-right: 20px;
    list-style: none;
    justify-content: space-between;
    :deep(.locale-select) {
      border-radius: 20px;
    }
    li {
      display: flex;
      align-items: center;
      padding: 0 10px;
    }

    a {
      color: var(--color-text-1);
      text-decoration: none;
    }
    .nav-btn {
      border-color: rgb(var(--gray-2));
      color: rgb(var(--gray-8));
      font-size: 16px;
    }
    .trigger-btn,
    .ref-btn {
      position: absolute;
      bottom: 14px;
    }
    .trigger-btn {
      margin-left: 14px;
    }
  }
</style>

<style lang="less">
  .message-popover {
    .arco-popover-content {
      margin-top: 0;
    }
  }
</style>
