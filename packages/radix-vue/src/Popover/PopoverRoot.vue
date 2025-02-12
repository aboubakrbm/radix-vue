<script lang="ts">
import type { InjectionKey, Ref } from 'vue'
import { useId } from '@/shared'

export interface PopoverRootProps {
  /**
   * The open state of the popover when it is initially rendered. Use when you do not need to control its open state.
   */
  defaultOpen?: boolean
  /**
   * The controlled open state of the popover.
   */
  open?: boolean
  /**
   * The modality of the popover. When set to true, interaction with outside elements will be disabled and only popover content will be visible to screen readers.
   *
   * @default false
   */
  modal?: boolean
}
export interface PopoverRootEmits {
  (e: 'update:open', value: boolean): void
}

export const POPOVER_INJECTION_KEY
  = Symbol() as InjectionKey<PopoverProvideValue>

export interface PopoverProvideValue {
  triggerElement: Ref<HTMLElement | undefined>
  contentId: string
  open: Ref<boolean>
  modal: Ref<boolean>
  onOpenChange(value: boolean): void
  onOpenToggle(): void
  hasCustomAnchor: Ref<boolean>
}
</script>

<script setup lang="ts">
import { provide, ref, toRefs } from 'vue'
import { useVModel } from '@vueuse/core'
import { PopperRoot } from '@/Popper'

const props = withDefaults(defineProps<PopoverRootProps>(), {
  defaultOpen: false,
  open: undefined,
  modal: false,
})
const emit = defineEmits<PopoverRootEmits>()
const { modal } = toRefs(props)
const open = useVModel(props, 'open', emit, {
  defaultValue: props.defaultOpen,
  passive: true,
})

const triggerElement = ref<HTMLElement>()
const hasCustomAnchor = ref(false)

provide<PopoverProvideValue>(POPOVER_INJECTION_KEY, {
  contentId: useId(),
  modal,
  open,
  onOpenChange: (value) => {
    open.value = value
  },
  onOpenToggle: () => {
    open.value = !open.value
  },
  triggerElement,
  hasCustomAnchor,
})
</script>

<template>
  <PopperRoot>
    <slot />
  </PopperRoot>
</template>
