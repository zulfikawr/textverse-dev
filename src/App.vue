<template>
  <div class="app-container">
    <LoadingSpinner v-if="authStore.isLoading" />
    <template v-else>
      <Navbar />
      <div class="scrollable-container">
        <router-view v-slot="{ Component, route }">
          <transition :name="transitionName" mode="out-in">
            <component :is="Component" :key="route.path"></component>
          </transition>
        </router-view>
        <Transition name="toast-slide">
          <Toast v-if="uiStore.showToast" :message="uiStore.toastMessage" />
        </Transition>
      </div>
    </template>
  </div>
</template>

<script setup lang="ts">
  import { ref, onMounted, onUnmounted, watch } from 'vue';
  import { useRouter } from 'vue-router';
  import { authStore, uiStore, notesStore, folderStore } from './store/stores';
  import Navbar from '@/components/navbar/navbar.vue';
  import Toast from '@/components/ui/toast.vue';
  import LoadingSpinner from '@/components/ui/loadingSpinner.vue';

  const router = useRouter();
  const routeOrder = ['Home', 'About', 'Settings', 'Trash', 'Sign In'];
  const transitionName = ref('slide-right');

  watch(
    () => router.currentRoute.value,
    (to, from) => {
      const toIndex = to ? routeOrder.indexOf(to.name as string) : -1;
      const fromIndex = from ? routeOrder.indexOf(from.name as string) : -1;
      transitionName.value = toIndex > fromIndex ? 'slide-left' : 'slide-right';
    },
    { immediate: true }
  );

  onMounted(async () => {
  try {
    await authStore.fetchCurrentUser();
    uiStore.applyTheme();
  } catch (error) {
    console.error('Error loading app:', error);
    uiStore.showToastMessage(
      'An error occurred while loading the app. Please try again.'
    );
  }
});

  onUnmounted(() => {
    folderStore.clearFolderListener();
    uiStore.clearSettingsListener();
  });
</script>

<style>
  html,
  body,
  #app {
    height: 100%;
    margin: 0;
  }

  .app-container {
    display: flex;
    flex-direction: column;
    height: 100%;
  }

  .scrollable-container {
    flex: 1;
    overflow-y: auto;
  }
</style>
