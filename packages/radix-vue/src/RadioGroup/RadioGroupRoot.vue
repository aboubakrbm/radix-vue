<script lang="ts">
import {
  Primitive,
  type PrimitiveProps,
} from '@/Primitive'
import type { DataOrientation, Direction } from '@/shared/types'
import { useVModel } from '@vueuse/core'
import type { InjectionKey, Ref } from 'vue'

export interface RadioGroupRootProps extends PrimitiveProps {
  modelValue?: string
  defaultValue?: string
  disabled?: boolean
  name?: string
  required?: boolean
  orientation?: DataOrientation
  dir?: Direction
  loop?: boolean
}
export interface RadioGroupRootEmits {
  (e: 'update:modelValue', payload: string): void
}

interface RadioGroupProvideValue {
  modelValue?: Readonly<Ref<string | undefined>>
  changeModelValue: (value?: string) => void
  disabled: Ref<boolean>
  loop: Ref<boolean>
  orientation: Ref<DataOrientation | undefined>
  name?: string
  required: Ref<boolean>
}

export const RADIO_GROUP_INJECTION_KEY
  = Symbol() as InjectionKey<RadioGroupProvideValue>
</script>

<script setup lang="ts">
import { provide, toRefs } from 'vue'
import { RovingFocusGroup } from '@/RovingFocus'

const props = withDefaults(defineProps<RadioGroupRootProps>(), {
  disabled: false,
  required: false,
  orientation: undefined,
  dir: 'ltr',
  loop: true,
})

const emits = defineEmits<RadioGroupRootEmits>()

const modelValue = useVModel(props, 'modelValue', emits, {
  defaultValue: props.defaultValue,
  passive: true,
})

const { disabled, loop, orientation, name, required } = toRefs(props)
provide<RadioGroupProvideValue>(RADIO_GROUP_INJECTION_KEY, {
  modelValue,
  changeModelValue: (value?: string) => {
    modelValue.value = value
  },
  disabled,
  loop,
  orientation,
  name: name?.value,
  required,
})
</script>

<template>
  <RovingFocusGroup as-child :orientation="orientation" :dir="dir" :loop="loop">
    <Primitive
      role="radiogroup"
      :data-disabled="disabled ? '' : undefined"
      :as-child="asChild"
      :as="as"
      :required="required"
      :aria-orientation="orientation"
      :aria-required="required"
      :dir="dir"
      :name="name"
    >
      <slot />
    </Primitive>
  </RovingFocusGroup>
</template>
