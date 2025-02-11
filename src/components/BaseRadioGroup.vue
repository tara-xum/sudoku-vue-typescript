<script setup lang="ts">
import BaseRadio from './BaseRadio.vue';
import type { DifficultyLevel } from '../App.vue'


defineProps<{
  options: DifficultyLevel[]
  name: string
  modelValue: string | number
  vertical: boolean
}>()
/*
defineProps({
  options: {
    type: Array<DifficultyLevel>,
    required: true
  },
  name: {
    type: String,
    required: true
  },
  modelValue: {
    type: [String, Number],
    required: true
  },
  vertical: {
    type: Boolean,
    default: false
  }
})
*/
</script>

<template>
  <div >
    <component
      v-for="option in options"
      :key="option.value"
      :is="vertical ? 'div' : 'span'"
      :class="{ horizontal: !vertical }"
    >
      <BaseRadio 
        :label="option.label"
        :value="option.value"
        :modelValue="modelValue"
        :name="name"
        @update:modelValue="$emit('update:modelValue', $event)"
      />
    </component>
  </div>
</template>
<style scoped>
.horizontal {
  margin-right: 20px
}
</style>