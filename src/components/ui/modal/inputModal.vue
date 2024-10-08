<!-- inputModal.vue -->

<template>
  <ModalBackdrop v-model="props.isOpen" />
  <transition name="zoom">
    <div
      v-if="props.isOpen"
      class="fixed inset-0 z-40 flex items-center justify-center"
    >
      <div @click="closeModal" class="absolute inset-0"></div>
      <form
        @submit.prevent="handleSubmit"
        @click.stop
        class="z-50 font-serif p-5 relative flex flex-col w-11/12 md:w-3/4 lg:w-1/2 xl:w-1/3"
        :class="[uiStore.blurEnabled ? 'custom-card-blur' : 'custom-card']"
      >
        <h1 class="text-xl font-bold mb-4">{{ modalTitle }}</h1>
        <input
          v-model="inputValue"
          @focus="handleFocus"
          @blur="handleBlur"
          :placeholder="placeholder"
          class="w-full p-1 bg-transparent border-0 border-b-[1px] border-black dark:border-white outline-none placeholder-black dark:placeholder-white placeholder-opacity-50"
        />
        <span
          v-if="showCharCount"
          :class="[
            'flex justify-end font-normal text-gray-500 text-sm mt-1',
            { 'text-red-500': inputValue.length > maxLength },
          ]"
        >
          {{ inputValue.length }} / {{ maxLength }}
        </span>
        <div class="flex justify-end mt-6">
          <button
            @click.prevent="closeModal"
            class="flex items-center px-2 py-1 custom-card hover:bg-[#d9c698] dark:hover:bg-gray-700 mr-4 cursor-pointer"
          >
            <PhProhibit :size="20" class="mr-2" />
            <span class="text-sm">Cancel</span>
          </button>
          <button
            :disabled="!isValid"
            type="submit"
            :class="[
              'text-sm flex items-center px-2 py-1 custom-card hover:bg-[#d9c698] dark:hover:bg-gray-700',
              {
                'text-blue-500 hover:text-blue-600 hover:bg-blue-700': isValid,
                'text-gray-400 cursor-default': !isValid,
              },
            ]"
          >
            <PhCheckCircle :size="20" class="size-5 mr-2" />
            <span>Save</span>
          </button>
        </div>
      </form>
    </div>
  </transition>
</template>

<script setup lang="ts">
  import { ref, computed, watch } from 'vue';
  import { PhProhibit, PhCheckCircle } from '@phosphor-icons/vue';
  import { uiStore } from '@/store/stores';
  import ModalBackdrop from '@/components/ui/modal/backdropModal.vue';

  const props = defineProps<{
    isOpen: boolean;
    mode: 'username' | 'folder' | 'title';
    currentValue?: string;
    maxLength?: number;
  }>();

  const emit = defineEmits<{
    (e: 'close'): void;
    (e: 'update', value: string): void;
  }>();

  const inputValue = ref(props.currentValue || '');
  const placeholder = ref('Enter name');

  const modalTitle = computed(() => {
    if (props.mode === 'username') return 'Rename';
    if (props.mode === 'folder')
      return props.currentValue ? 'Rename Folder' : 'Create New Folder';
    if (props.mode === 'title')
      return props.currentValue ? 'Edit Title' : 'Enter Title';
  });

  const showCharCount = computed(() => props.mode === 'folder');
  const maxLength = computed(() => props.maxLength || 30);

  const isValid = computed(() => {
    const trimmedLength = inputValue.value.trim().length;
    return trimmedLength > 0 && trimmedLength <= maxLength.value;
  });

  const handleSubmit = () => {
    if (isValid.value) {
      emit('update', inputValue.value);
      closeModal();
    }
  };

  const closeModal = () => {
    emit('close');
  };

  const handleFocus = () => {
    placeholder.value = '';
  };

  const handleBlur = () => {
    if (inputValue.value.trim() === '') {
      placeholder.value =
        props.mode === 'username'
          ? 'Enter your username'
          : props.mode === 'folder'
            ? 'Enter folder name'
            : 'Enter title';
    }
  };

  watch(
    () => props.currentValue,
    (newValue) => {
      inputValue.value = newValue || '';
    }
  );
</script>
