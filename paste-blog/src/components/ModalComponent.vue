<template>
  <div
    v-if="isVisible"
    class="modal-overlay"
    @click.self="closeModal"
  >
    <div class="modal-content">
      <button @click="closeModal">Close</button>
      <slot></slot>
      <!-- Use slot for content flexibility -->
    </div>
  </div>
</template>

<script setup>
  import { ref } from 'vue';
  import { defineProps, defineEmits, watch } from 'vue';

  const props = defineProps({
    show: Boolean, // Use v-model for visibility
  });

  const isVisible = ref(props.show);

  watch(
    () => props.show,
    (newVal) => {
      isVisible.value = newVal;
    }
  );

  const emit = defineEmits(['update:show']);

  function closeModal() {
    emit('update:show', false); // Sync with parent component
  }
</script>

<style>
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .modal-content {
    background: white;
    padding: 20px;
    border-radius: 5px;
  }
</style>
