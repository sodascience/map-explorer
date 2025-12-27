<template>
    <section class="bg-white">
      <div class="p-4 space-y-6">
        <!-- Loading Skeleton -->
        <div v-if="loading || !config" class="space-y-6">
          <!-- Filter Options Skeleton -->
          <div>
            <div class="h-4 bg-gray-200 rounded animate-pulse mb-3 w-24"></div>
            <div class="space-y-4">
              <div v-for="i in 2" :key="i" class="space-y-2">
                <div class="h-3 bg-gray-200 rounded animate-pulse w-20"></div>
                <div class="h-10 bg-gray-200 rounded animate-pulse"></div>
              </div>
            </div>
          </div>

          <!-- Map Options Skeleton -->
          <div>
            <div class="h-4 bg-gray-200 rounded animate-pulse mb-3 w-24"></div>
            <div class="space-y-4">
              <div class="space-y-2">
                <div class="h-3 bg-gray-200 rounded animate-pulse w-24"></div>
                <div class="h-10 bg-gray-200 rounded animate-pulse"></div>
              </div>
              <div class="flex items-center gap-2">
                <div class="h-4 w-4 bg-gray-200 rounded animate-pulse"></div>
                <div class="h-3 bg-gray-200 rounded animate-pulse w-32"></div>
              </div>
              <div class="flex items-center gap-2">
                <div class="h-4 w-4 bg-gray-200 rounded animate-pulse"></div>
                <div class="h-3 bg-gray-200 rounded animate-pulse w-40"></div>
              </div>
              <div class="space-y-2">
                <div class="h-3 bg-gray-200 rounded animate-pulse w-28"></div>
                <div class="h-10 bg-gray-200 rounded animate-pulse"></div>
              </div>
              <div class="space-y-2">
                <div class="h-3 bg-gray-200 rounded animate-pulse w-28"></div>
                <div class="h-10 bg-gray-200 rounded animate-pulse"></div>
              </div>
            </div>
          </div>
        </div>

        <!-- Actual Content -->
        <div v-else>
          <!-- Filter Options -->
          <div>
            <h3 class="text-xs font-semibold uppercase tracking-wide text-gray-600 mb-3">
              Filter Options
            </h3>

            <div v-if="hasFilterOptions" class="space-y-4">
              <div
                v-for="(options, categoryName) in availableFilterOptions"
                :key="categoryName"
              >
                <Selection
                  :label="categoryName"
                  :options="options"
                  :defaultValue="getDefaultFilterValue(categoryName, options)"
                  :warningOptions="getInvalidOptions(categoryName)"
                  @selection-changed="(value) => handleFilterChanged(categoryName, value)"
                />
              </div>
            </div>

            <div v-else class="text-gray-500 text-sm italic">
              No filter options available.
            </div>
          </div>

          <!-- Map Options -->
          <h3 class="text-xs font-semibold uppercase tracking-wide text-gray-600 mb-3 mt-6">
            Map Options
          </h3>

            <div>
              <Selection
                :label="'Color Scheme'"
                :options="schemeNames"
                :defaultValue="config.mapColorConfig?.colorScheme"
                @selection-changed="handleColorSchemeChanged"
              />

              <Checkbox
                class="mt-3"
                label="Invert Color Scheme"
                :defaultValue="config.mapColorConfig?.colorSchemeInverted"
                @checkbox-changed="handleColorSchemeInvertedChanged"
              >
                Invert color scheme
              </Checkbox>

              <Checkbox
                class="mt-3"
                label="Dynamic Legend"
                :defaultValue="config.mapColorConfig?.dynamic"
                @checkbox-changed="handleDynamicLegendChanged"
              >
                Calculate the min and max from the data
              </Checkbox>

              <InputField
                class="mt-3"
                label="Legend Minimum"
                :defaultValue="config.mapColorConfig?.minValue"
                :disabled="config.mapColorConfig?.dynamic"
                placeholder="0.00"
                @input-changed="handleLegendMinimumChanged"
              />

              <InputField
                class="mt-3"
                label="Legend Maximum"
                :defaultValue="config.mapColorConfig?.maxValue"
                :disabled="config.mapColorConfig?.dynamic"
                placeholder="1.00"
                @input-changed="handleLegendMaximumChanged"
              />
            </div>
          </div>
        </div>
    </section>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue'
import Selection from './selection.vue'
import Checkbox from './checkbox.vue'
import InputField from './input-field.vue'
import { colorSchemes } from '../config/types.ts'
import type { MapConfig } from '../config/types.ts'
import type { ValidFilterLookup } from '../mapManager.ts'

const schemeNames: string[] = [...colorSchemes]

const props = defineProps<{
  availableFilterOptions?: Record<string, any>
  config?: MapConfig
  loading?: boolean
  validFilterLookup?: ValidFilterLookup
}>()

const emit = defineEmits([
  'filter-changed',
  'map-config-changed'
])

// Track the current selected filters
const selectedFilters = ref<Record<string, string>>({})

// Initialize selected filters when availableFilterOptions changes
watch(
  () => props.availableFilterOptions,
  (newOptions) => {
    if (!newOptions) return

    const newFilters: Record<string, string> = {}
    for (const categoryName in newOptions) {
      newFilters[categoryName] = getDefaultFilterValue(categoryName, newOptions[categoryName])
    }
    selectedFilters.value = newFilters
  },
  { immediate: true, deep: true }
)

const hasFilterOptions = computed(() =>
  props.availableFilterOptions && Object.keys(props.availableFilterOptions).length > 0
)

// Compute invalid options for each category based on current selected filters
const invalidOptionsByCategory = computed(() => {
  if (!props.validFilterLookup || !props.availableFilterOptions) {
    return {}
  }

  const result: Record<string, string[]> = {}

  for (const categoryName in props.availableFilterOptions) {

    const options = props.availableFilterOptions[categoryName]
    const validOptions = props.validFilterLookup.lookup(selectedFilters.value, categoryName)
    result[categoryName] = options.filter((option: string) => !validOptions.includes(option))
  }
  return result
})

function getDefaultFilterValue (categoryName: string, options: string[]) {
  if (
    props.config?.filter !== undefined &&
    Object.prototype.hasOwnProperty.call(props.config.filter, categoryName)
  ) {
    return props.config.filter[categoryName]
  }
  return options?.[0]
}

function getInvalidOptions(categoryName: string): string[] {
  return invalidOptionsByCategory.value[categoryName] || []
}

function handleFilterChanged (categoryName: string, value: string) {
  // Update local state
  selectedFilters.value = {
    ...selectedFilters.value,
    [categoryName]: value
  }

  // Emit to parent
  emit('filter-changed', categoryName, value)
}

function handleMapConfigChange (field: string, value: any) {
  if (!props.config) return

  emit('map-config-changed', {
    ...props.config.mapColorConfig,
    [field]: value
  })
}

function handleColorSchemeChanged (value: string) {
  handleMapConfigChange('colorScheme', value)
}

function handleColorSchemeInvertedChanged (value: boolean) {
  handleMapConfigChange('colorSchemeInverted', value)
}

function handleDynamicLegendChanged (value: boolean) {
  handleMapConfigChange('dynamic', value)
}

function handleLegendMinimumChanged (value: number) {
  handleMapConfigChange('minValue', value)
}

function handleLegendMaximumChanged (value: number) {
  handleMapConfigChange('maxValue', value)
}
</script>
