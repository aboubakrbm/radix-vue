<script lang="ts">
import type { InjectionKey, Ref } from 'vue'
import type { Direction } from '../shared/types'
import { useNewCollection } from '@/shared'

export interface MenubarRootProps {
  modelValue?: string
  defaultValue?: string
  dir?: Direction
  loop?: boolean
}
export interface MenubarRootEmits {
  (e: 'update:modelValue', value: boolean): void
}

export const MENUBAR_INJECTION_KEY
  = Symbol() as InjectionKey<MenubarContextValue>

export interface MenubarContextValue {
  modelValue: Ref<string>
  dir: Ref<Direction>
  loop: Ref<boolean>
  onMenuOpen(value: string): void
  onMenuClose(): void
  onMenuToggle(value: string): void
}
</script>

<script setup lang="ts">
import { provide, ref, toRefs } from 'vue'
import { Primitive, usePrimitiveElement } from '@/Primitive'
import { useVModel } from '@vueuse/core'

const props = withDefaults(defineProps<MenubarRootProps>(), {
  dir: 'ltr',
  loop: false,
})
const emit = defineEmits<MenubarRootEmits>()

const { primitiveElement, currentElement } = usePrimitiveElement()
const { createCollection } = useNewCollection('menubar')
createCollection(currentElement)

const modelValue = useVModel(props, 'modelValue', emit, {
  passive: true,
  defaultValue: props.defaultValue ?? '',
}) as Ref<string>

const currentTabStopId = ref<string | null>(null)

const { dir, loop } = toRefs(props)
provide(MENUBAR_INJECTION_KEY, {
  modelValue,
  dir,
  loop,
  onMenuOpen: (value) => {
    modelValue.value = value
    currentTabStopId.value = value
  },
  onMenuClose: () => {
    modelValue.value = ''
  },
  onMenuToggle: (value) => {
    modelValue.value = modelValue.value ? '' : value
    // `openMenuOpen` and `onMenuToggle` are called exclusively so we
    // need to update the id in either case.
    currentTabStopId.value = value
  },
})
</script>

<template>
  <Primitive ref="primitiveElement" role="menubar">
    <slot />
  </Primitive>
</template>
