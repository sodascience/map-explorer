<template>
  <div>
    <label v-if="label" class="block text-gray-700 text-sm font-bold mb-2">
      {{ label }}
    </label>
    <select
      class="w-full p-2 rounded border border-gray-300 bg-white"
      v-model="selectedValue"
      @change="emitSelection"
    >
      <option
        v-for="option in options"
        :key="option"
        :value="option"
      >
        {{ option }}{{ warningOptions.includes(option) ? ' (no data)' : '' }}
      </option>
    </select>
  </div>
</template>
<script setup lang="ts">

import { ref } from "vue"

const props = withDefaults(
  defineProps<{
    options: string[]
    label: string
    defaultValue?: string | null
    warningOptions?: string[]
  }>(),
  {
    warningOptions: () => []
  }
)

const emit = defineEmits<{
  (e: "selection-changed", value: string): void
}>()

const selectedValue = ref<string>(
  props.defaultValue ?? props.options[0]
)

function emitSelection() {
  emit("selection-changed", selectedValue.value)
}
</script>
