<template>
  <div class="flex flex-col items-start gap-4">
    <div
      class="flex md:flex-wrap overflow-hidden gap-4 overflow-x-scroll w-[calc(100%+1rem)] pr-4"
      :class="{ 'h-0': width < 768 }"
    >
      <component
        :is="item.component"
        v-for="item in components"
        ref="componentRef"
        :key="item.key"
        :name="item.key"
        :label="item.label"
      />
    </div>

    <div v-show="width >= 768" class="flex flex-wrap gap-4">
      <Boolean name="has_image" :label="t('controls.filter.hasImage')" :default="true" />
      <Boolean name="has_iip" :label="t('controls.filter.hasIip')" />
    </div>

    <FilterMobile
      ref="filterMobileRef"
      :is-opened="isMobileMenuOpened"
      :components="components"
      @toggleMenu="onToggleMobileMenu"
      @resetAll="onResetAll"
    />

    <div class="flex flex-wrap gap-3">
      <div
        v-for="item in selected"
        :key="`${item?.key}-${item?.value}`"
        class="bg-primary text-white flex items-center py-1 px-3 gap-2 cursor-pointer rounded-3xl"
        @click="item.toggle()"
      >
        <div class="text-xs">{{ item?.value }}</div>
        <Icon name="close" class="w-3" />
      </div>

      <div class="flex flex-wrap gap-3">
        <div
          v-if="selected.length"
          class="flex py-1 px-3 gap-2 border-2 border-dark items-center cursor-pointer rounded-3xl"
          @click="onResetAll"
        >
          <Icon class="w-4 h-4" name="rotate" />
          <div class="text-xs">{{ t('controls.filter.reset') }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useWindowSize } from '@vueuse/core'

import SelectMobile from '~/components/controls/mobile/Select.vue'
import RangeMobile from '~/components/controls/mobile/Range.vue'
import Icon from '~/components/general/Icon.vue'
import { useControls } from '~/composables/controls'
import FilterMobile from '~/components/general/FilterMobile.vue'
import Select from "~/components/controls/Select.vue";
import Range from "~/components/controls/Range.vue";
import Boolean from "~/components/controls/Boolean.vue";
import Hidden from "~/components/controls/Hidden.vue";

const isMobileMenuOpened = ref(false)

const { width } = useWindowSize()
const componentRef = ref<InstanceType<typeof Select | typeof Hidden>[]>([])
const { t } = useI18n()

const components = [
  {
    component: Select,
    mobileComponent: SelectMobile,
    key: 'author',
    label: t('item.attribute.author'),
  },
  {
    component: Select,
    mobileComponent: SelectMobile,
    key: 'work_type',
    label: t('item.attribute.workType'),
  },
  {
    component: Select,
    mobileComponent: SelectMobile,
    key: 'topic',
    label: t('item.attribute.topic'),
  },
  {
    component: Select,
    mobileComponent: SelectMobile,
    key: 'technique',
    label: t('item.attribute.technique'),
  },
  {
    component: Select,
    mobileComponent: SelectMobile,
    key: 'medium',
    label: t('item.attribute.medium'),
  },
  {
    component: Select,
    mobileComponent: SelectMobile,
    key: 'exhibition',
    label: t('item.attribute.exhibition'),
  },
  {
    component: Range,
    mobileComponent: RangeMobile,
    key: {
      min: 'date_earliest',
      max: 'date_latest',
    },
    label: t('item.attribute.dateRange'),
  },
  {
    component: Hidden,
    mobileComponent: Hidden,
    key: 'related_work',
    label: t('item.attribute.relatedWork'),
  },
]

// This is workaround, and will be removed once
// state management is moved from component to composable
const route = useRoute()
const selected = computed<any[]>(() => {
  const options: any[] = []

  Object.keys(route.query).forEach((key) => {
    if (components.some((c) => c.key === key)) {
      const items = String(route.query[key]).split('|')

      items.forEach((item) => {
        options.push({
          value: item,
          toggle: () => {
            componentRef.value.forEach((component) => {
              if (component.name === key) {
                component.toggle?.(item)
              }
            })
          },
        })
      })
    }
  })

  return options
})

const { reset } = await useControls()

const onToggleMobileMenu = (event: MouseEvent) => {
  if (width.value >= 768) {
    return
  }

  event.preventDefault()

  isMobileMenuOpened.value = !isMobileMenuOpened.value
}

const filterMobileRef = ref<InstanceType<typeof FilterMobile>>()
const onResetAll = () => {
  componentRef.value.forEach((component) => {
    component.onReset?.()
  })
}

onUnmounted(() => {
  reset()
})
</script>
