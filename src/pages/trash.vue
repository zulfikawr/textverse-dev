<template>
  <div class="trashContainer">
    <div
      class="mb-[28px] md:mb-[46px] my-6 md:my-8 font-serif"
      @click="handleOutsideClick"
    >
      <div
        class="flex flex-col md:flex-row md:justify-center items-center mb-6 md:mb-8 mx-2"
      >
        <p
          class="custom-card px-4 py-2 md:px-2 md:py-1.5 w-full md:w-auto text-sm text-gray-800 dark:text-gray-400 text-center"
        >
          Notes in trash will be permanently deleted after 30 days.
        </p>
        <Button
          v-if="deletedNotes.length > 0"
          variant="danger"
          @click="emptyTrash"
          class="mt-3 md:mt-0 md:ml-4 text-sm"
        >
          <PhTrash :size="20" class="mr-2" />
          Empty Trash
        </Button>
      </div>
      <div
        v-if="deletedNotes.length > 0"
        class="md:w-11/12 mx-auto px-2 md:px-0"
      >
        <transition-group
          name="list"
          tag="ul"
          :class="[
            'relative min-w-[300px] md:mx-auto',
            {
              'columns-1 md:max-w-xl': uiStore.columns === 1,
              'columns-2 gap-2 md:gap-7 md:max-w-4xl': uiStore.columns === 2,
              'columns-3 sm:columns-2 md:columns-3 gap-8':
                uiStore.columns === 3,
              'columns-4 sm:columns-2 md:columns-3 lg:columns-4 gap-4':
                uiStore.columns === 4,
            },
          ]"
        >
          <li
            v-for="note in deletedNotes"
            :key="note.id"
            class="notes bg-cream dark:bg-gray-750 border-[1px] border-black dark:border-gray-400 rounded-lg shadow-lg hover:shadow-2xl transition-all duration-300 break-inside-avoid h-min mb-[9px] p-2 cursor-pointer relative group select-none"
            :class="{
              'z-50': showMenu && notesStore.selectedNote?.id === note.id,
              shadow: notesStore.selectedNotes.includes(note.id),
              'border-[4px] dark:border-white':
                notesStore.selectedNotes.includes(note.id),
              [computedMb]: true,
            }"
            @click.stop="toggleNoteSelection(note.id)"
          >
            <div
              class="absolute top-0 -left-3 custom-card-rounded hover:bg-[#ebdfc0] dark:hover:bg-gray-700 transition-opacity duration-200"
              :class="{
                'opacity-100': notesStore.selectedNotes.includes(note.id),
                'opacity-0 group-hover:opacity-100':
                  !notesStore.selectedNotes.includes(note.id),
              }"
              @click.stop="toggleNoteSelection(note.id)"
              style="transform: translateY(-50%)"
            >
              <div class="p-1 rounded-full flex items-center justify-center">
                <PhCheck :size="16" />
              </div>
            </div>
            <div class="flex justify-between items-start">
              <h1
                class="font-bold text-sl font-serif cursor-pointer dark:text-white"
              >
                {{ note.title }}
              </h1>
            </div>
            <div>
              <div
                class="font-serif text-sm mt-2 dark:text-white truncate-text"
                v-html="sanitizeHtml(truncatedContent(note.content))"
              ></div>
              <div
                class="flex justify-between items-center pt-3 mt-auto font-serif text-gray-700 dark:text-gray-400 text-xs"
              >
                <div class="flex items-center text-left text-[10px] md:text-xs">
                  <div
                    @click.stop="permanentlyDeleteNote(note.id)"
                    class="flex items-center px-2 py-1 mr-2 cursor-pointer truncate custom-card text-red-500 hover:text-red-100 hover:bg-red-700/50 dark:hover:bg-red-800/60"
                  >
                    <PhTrash :size="16" />
                  </div>
                  <div
                    @click.stop="restoreNote(note.id)"
                    class="flex items-center px-2 py-1 cursor-pointer truncate custom-card hover:text-black dark:hover:text-white hover:bg-[#d9c698] dark:hover:bg-gray-700"
                  >
                    <PhClockClockwise :size="16" />
                  </div>
                </div>
                <div class="text-left text-[10px] md:text-xs">
                  <p
                    class="flex items-center px-2 py-1 cursor-pointer truncate custom-card"
                  >
                    <PhFolder :size="16" class="mr-2" />
                    {{ note.folder }}
                  </p>
                </div>
              </div>
            </div>
          </li>
        </transition-group>
      </div>

      <div v-else class="flex flex-col items-center justify-center h-[60vh]">
        <PhTrash :size="100" class="text-gray-400 dark:text-gray-600 mb-4" />
        <p class="text-gray-600 dark:text-gray-400 text-lg">Trash empty</p>
      </div>

      <AlertModal
        :is-open="isDeleteAlertOpen"
        :message="deleteAlertMessage"
        @confirm="confirmPermanentlyDeleteNote"
        @cancel="isDeleteAlertOpen = false"
      />

      <AlertModal
        :is-open="isEmptyTrashAlertOpen"
        :message="emptyTrashAlertMessage"
        @confirm="confirmEmptyTrash"
        @cancel="isEmptyTrashAlertOpen = false"
      />
    </div>
  </div>
</template>

<script lang="ts" setup>
  import { ref, computed, onMounted, onUnmounted } from 'vue';
  import { notesStore, uiStore } from '@/store/stores';
  import AlertModal from '@/components/ui/modal/alertModal.vue';
  import {
    PhCheck,
    PhFolder,
    PhTrash,
    PhClockClockwise,
  } from '@phosphor-icons/vue';
  import DOMPurify from 'dompurify';
  import Button from '@/components/ui/button.vue';

  const showMenu = ref(false);
  const selectedNotes = ref<string[]>([]);

  const deletedNotes = computed(() => notesStore.deletedNotes);

  const computedMb = computed(() => {
    if (uiStore.columns === 4) {
      return 'md:mb-4';
    } else {
      return 'md:mb-8';
    }
  });

  const sanitizeHtml = (content: string) => {
    return DOMPurify.sanitize(content);
  };

  const truncatedContent = (content: string) => {
    const div = document.createElement('div');
    div.innerHTML = content;
    return div.innerHTML;
  };

  const isSelectMode = ref(false);

  const toggleNoteSelection = (noteId: string) => {
    const index = selectedNotes.value.indexOf(noteId);
    if (index === -1) {
      selectedNotes.value.push(noteId);
      notesStore.addSelectedNote(noteId);
    } else {
      selectedNotes.value.splice(index, 1);
      notesStore.removeSelectedNote(noteId);
    }

    if (!isSelectMode.value) {
      isSelectMode.value = true;
    }
  };

  const isDeleteAlertOpen = ref(false);
  const isEmptyTrashAlertOpen = ref(false);
  const deleteAlertMessage = ref('');
  const emptyTrashAlertMessage = ref('');
  const noteToDelete = ref<string | null>(null);

  const permanentlyDeleteNote = (noteId: string) => {
    noteToDelete.value = noteId;
    isDeleteAlertOpen.value = true;
    deleteAlertMessage.value =
      'Are you sure you want to permanently delete this note? This action cannot be undone.';
  };

  const confirmPermanentlyDeleteNote = async () => {
    if (noteToDelete.value) {
      try {
        await notesStore.permanentlyDeleteNote(noteToDelete.value);
        uiStore.showToastMessage('Note permanently deleted');
      } catch (error) {
        console.error('Error deleting note:', error);
        uiStore.showToastMessage('Failed to delete note. Please try again.');
      }
    }
    isDeleteAlertOpen.value = false;
    noteToDelete.value = null;
  };

  const restoreNote = async (noteId: string) => {
    try {
      await notesStore.restoreNote(noteId);
      uiStore.showToastMessage('Note restored');
    } catch (error) {
      console.error('Error restoring note:', error);
      uiStore.showToastMessage('Failed to restore note. Please try again.');
    }
  };

  const emptyTrash = () => {
    isEmptyTrashAlertOpen.value = true;
    emptyTrashAlertMessage.value =
      'Are you sure you want to empty the trash? This action cannot be undone.';
  };

  const confirmEmptyTrash = async () => {
    try {
      await notesStore.emptyTrash();
      uiStore.showToastMessage('Trash emptied successfully');
    } catch (error) {
      console.error('Error emptying trash:', error);
      uiStore.showToastMessage('Failed to empty trash. Please try again.');
    }
    isEmptyTrashAlertOpen.value = false;
  };

  const handleOutsideClick = (event: MouseEvent) => {
    if (!(event.target as HTMLElement).closest('li')) {
      isSelectMode.value = false;
      selectedNotes.value = [];
      notesStore.clearSelectedNotes();
    }
  };

  onMounted(async () => {
    await notesStore.loadDeletedNotes();
  });

  const isMobile = ref(window.innerWidth < 640);

  const handleResize = () => {
    const newIsMobile = window.innerWidth < 640;
    if (newIsMobile !== isMobile.value) {
      isMobile.value = newIsMobile;
      if (isMobile.value && uiStore.columns > 2) {
        uiStore.setColumns(2);
      } else if (!isMobile.value && uiStore.columns < 3) {
        uiStore.setColumns(4);
      }
    }
  };

  onMounted(() => {
    window.addEventListener('resize', handleResize);
    handleResize();
  });

  onUnmounted(() => {
    window.removeEventListener('resize', handleResize);
  });
</script>

<style scoped>
  .truncate-text {
    display: -webkit-box;
    -webkit-line-clamp: 8;
    line-clamp: 8;
    -webkit-box-orient: vertical;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: pre-wrap;
  }

  @media (min-width: 640px) {
    .truncate-text {
      -webkit-line-clamp: 20;
      line-clamp: 20;
    }
  }

  .shadow {
    box-shadow:
      0 30px 60px -15px rgba(0, 0, 0, 0.3),
      0 12px 16px -8px rgba(0, 0, 0, 0.2);
  }

  .shadow:hover {
    box-shadow:
      0 30px 60px -15px rgba(0, 0, 0, 0.3),
      0 12px 16px -8px rgba(0, 0, 0, 0.2);
  }

  .notes:active {
    transform: scale(0.98);
  }
</style>
