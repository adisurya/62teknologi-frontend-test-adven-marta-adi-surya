<template>
    <q-dialog v-model="value">
      <q-card>
        <q-card-section>
          <div class="text-h6">{{ title }}</div>
        </q-card-section>
        <q-card-section class="q-pt-none">
          {{ message }}
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="OK" color="primary" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  modelValue: {
    type: Boolean,
    required: true,
    default: false,
  },
  message: {
    type: String,
    default: ''
  },
  title: {
    type: String,
    default: 'Alert'
  },

});

const emit = defineEmits(['update:modelValue']);

const value = computed({
  get() {
    return props.modelValue;
  },
  set(value) {
    emit('update:modelValue', value);
  }
});
</script>
