<template>
  <div class="root">
    <!-- Check box -->
    <input
      type="checkbox"
      class="checkbox"
      :checked="props.checked"
      @change="onCompletedChange(!props.checked)"
    >

    <!-- Title -->
    <span
      v-if="!isChangingTitle"
      :class="{ completed: props.checked }"
      @click="!props.checked && onStartChangingTitle()"
    >
      {{ props.title }}
    </span>
    <input
      v-else
      ref="newTitleInput"
      v-model="newTitle"
      type="text"
      class="input"
      @keyup.enter="onStopChangingTitle(true)"
      @blur="onStopChangingTitle(true)"
      @focusout="onStopChangingTitle(true)"
      @keyup.escape="onStopChangingTitle(false)"
    >

    <!-- Due date -->
    <span
      v-if="!isChangingDueDate"
      :class="{ overdue: isOverdue }"
      @click="!props.checked && onStartChangingDueDate()"
    >
      ({{ humanDate(props.date) }})
    </span>
    <input
      v-else
      ref="newDueDateInput"
      v-model="newDueDate"
      type="text"
      @keyup.enter="onStopChangingDueDate()"
    >
  </div>
</template>

<script setup lang="ts">
import { ref, watchPostEffect, computed } from 'vue'

export interface Props {
  title: string;
  checked: boolean;
  date?: Date;
}

const props = defineProps<Props>()
const emit = defineEmits(['titleChanged', 'dateChanged', 'checkChanged'])

/* -------------------------------------------------------------------------- */
/*                                    Title                                   */
/* -------------------------------------------------------------------------- */

const newTitle = ref<string>(props.title)
const newTitleInput = ref<HTMLInputElement | undefined>(undefined)
const isChangingTitle = ref<boolean>(false)

watchPostEffect(() => {
  newTitleInput.value?.focus()
})

function onStartChangingTitle() {
  newTitle.value = props.title
  isChangingTitle.value = true
}

function onStopChangingTitle(accept: boolean) {
  const titleChanged = newTitle.value !== props.title
  if (isChangingTitle.value && accept && titleChanged) {
    emit('titleChanged', newTitle.value)
  }
  isChangingTitle.value = false
}

/* -------------------------------------------------------------------------- */
/*                                  Due date                                  */
/* -------------------------------------------------------------------------- */

const newDueDate = ref<string>('')
const newDueDateInput = ref<HTMLInputElement | undefined>(undefined)
const isChangingDueDate = ref<boolean>(false)
const isOverdue = computed<boolean>(() => (props.date ? new Date() >= props.date : false))

watchPostEffect(() => {
  newDueDateInput.value?.focus()
})

function onStartChangingDueDate() {
  isChangingDueDate.value = true
  newDueDate.value = props.date?.toISOString() ?? ''
}

function onStopChangingDueDate() {
  if (isChangingDueDate.value) {
    isChangingDueDate.value = false
    emit('dateChanged', new Date(newDueDate.value))
  }
}

function humanDate(date: Date | undefined) {
  return date?.toString().split('T')[0]
}

/* -------------------------------------------------------------------------- */
/*                                  Completed                                 */
/* -------------------------------------------------------------------------- */

function onCompletedChange(value: boolean) {
  emit('checkChanged', value)
}
</script>

<style scoped>
.root {
  @apply bg-green-200 p-2 rounded;
}

.completed {
  @apply line-through text-gray-500;
}

.overdue {
  @apply text-red-800;
}

.input {
  @apply w-1/2;
}

.checkbox {
  @apply mx-2;
}
</style>
