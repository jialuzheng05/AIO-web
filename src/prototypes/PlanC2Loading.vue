<script setup lang="ts">
defineProps<{
  steps: string[]
  currentStep: number
}>()
</script>

<template>
  <div
    v-for="(step, index) in steps"
    :key="step"
    class="plan-c-generating-step"
    :class="{
      complete: index < currentStep,
      current: index === currentStep,
      pending: index > currentStep,
    }"
  >
    <span class="plan-c-step-icon">
      <svg v-if="index < currentStep" viewBox="0 0 20 20" aria-hidden="true">
        <path d="m5.5 10.2 3 3 6-6.4" />
      </svg>
      <svg v-else viewBox="0 0 20 20" aria-hidden="true">
        <path d="M10 3a7 7 0 1 1-7 7" />
      </svg>
    </span>
    <span>{{ step }}</span>
  </div>
</template>
