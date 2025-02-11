<script setup lang="ts">

defineProps({
    label: {
      type: String,
      default: ''
    },
    modelValue: {
      type: [String, Number],
      default: ""
    },
    value: {
      type: [String, Number],
      required:true
    }
})

const emit = defineEmits([
    "update:modelValue"
])
</script>

<template>
  <input 
    type="radio" 
    :checked="modelValue === value" 
    :value="value" 
    v-bind="$attrs"
    name="chosenDifficultyLevel"
    @change="$emit('update:modelValue', value)" 
  />
  <label v-if="label">{{ label }}</label>
</template>
