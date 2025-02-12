<script lang="ts">
export interface RovingFocusGroupProps extends PrimitiveProps {
  /**
   * The orientation of the group.
   * Mainly so arrow navigation is done accordingly (left & right vs. up & down)
   */
  orientation?: Orientation
  /**
   * The direction of navigation between items.
   */
  dir?: Direction
  /**
   * Whether keyboard navigation should loop around
   * @defaultValue false
   */
  loop?: boolean
  currentTabStopId?: string
  defaultCurrentTabStopId?: string
}

export interface RovingFocusGroupEmits {
  (e: 'entryFocus', event: Event): void
}

interface RovingContextValue {
  orientation: Ref<Orientation>
  dir: Ref<Direction>
  loop: Ref<boolean>
  currentTabStopId: Ref<string | undefined>
  onItemFocus(tabStopId: string): void
  onItemShiftTab(): void
  onFocusableItemAdd(): void
  onFocusableItemRemove(): void
}

export const ROVING_FOCUS_INJECTION_KEY
  = Symbol() as InjectionKey<RovingContextValue>
</script>

<script setup lang="ts">
import { type InjectionKey, type Ref, provide, ref, toRefs } from 'vue'
import { useActiveElement } from '@vueuse/core'
import {
  type Direction,
  ENTRY_FOCUS,
  EVENT_OPTIONS,
  type Orientation,
} from './utils'
import { focusFirst } from './utils'
import { useNewCollection } from '@/shared'
import {
  Primitive,
  type PrimitiveProps,
  usePrimitiveElement,
} from '@/Primitive'

const props = withDefaults(defineProps<RovingFocusGroupProps>(), {
  loop: false,
  dir: 'ltr',
  orientation: 'horizontal',
})
const emits = defineEmits<RovingFocusGroupEmits>()
const { loop, orientation, dir } = toRefs(props)
const currentTabStopId = ref(props.defaultCurrentTabStopId)
const isTabbingBackOut = ref(false)
const isClickFocus = ref(false)
const focusableItemsCount = ref(0)

const activeElement = useActiveElement()
const { primitiveElement, currentElement } = usePrimitiveElement()
const { createCollection } = useNewCollection('rovingFocus')
const collections = createCollection(currentElement)

function handleFocus(event: FocusEvent) {
  // We normally wouldn't need this check, because we already check
  // that the focus is on the current target and not bubbling to it.
  // We do this because Safari doesn't focus buttons when clicked, and
  // instead, the wrapper will get focused and not through a bubbling event.
  const isKeyboardFocus = !isClickFocus.value

  if (
    event.currentTarget
    && event.target === event.currentTarget
    && isKeyboardFocus
    && !isTabbingBackOut.value
  ) {
    const entryFocusEvent = new CustomEvent(ENTRY_FOCUS, EVENT_OPTIONS)
    event.currentTarget.dispatchEvent(entryFocusEvent)

    emits('entryFocus', event)

    if (!entryFocusEvent.defaultPrevented) {
      const items = collections.value
      const activeItem = items.find(item => item === activeElement.value)
      const currentItem = items.find(
        item => item.id === currentTabStopId.value,
      )
      const candidateItems = [activeItem, currentItem, ...items].filter(
        Boolean,
      ) as typeof items
      focusFirst(candidateItems)
    }
  }

  isClickFocus.value = false
}

provide(ROVING_FOCUS_INJECTION_KEY, {
  loop,
  dir,
  orientation,
  currentTabStopId,
  onItemFocus: (tabStopId) => {
    currentTabStopId.value = tabStopId
  },
  onItemShiftTab: () => {
    isTabbingBackOut.value = true
  },
  onFocusableItemAdd: () => {
    focusableItemsCount.value++
  },
  onFocusableItemRemove: () => {
    focusableItemsCount.value--
  },
})
</script>

<template>
  <Primitive
    ref="primitiveElement"
    :tabindex="isTabbingBackOut || focusableItemsCount === 0 ? -1 : 0"
    :data-orientation="orientation"
    :as="as"
    :as-child="asChild"
    style="outline: none"
    @mousedown="isClickFocus = true"
    @focus="handleFocus"
    @blur="isTabbingBackOut = false"
  >
    <slot />
  </Primitive>
</template>
