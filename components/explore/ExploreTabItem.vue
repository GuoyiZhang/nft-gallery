<template>
  <div
    class="explore-tabs field has-addons is-flex is-align-items-center"
    data-cy="tabs">
    <div class="mr-4">
      <a
        class="is-hidden-mobile"
        :class="{ disabled: selectedTab === TabType.COLLECTION }"
        @click="toggleSidebarFilters">
        <b-icon
          :icon="
            isSidebarFiltersOpen && selectedTab !== TabType.COLLECTION
              ? 'times'
              : 'bars'
          "
          size="is-medium" />
      </a>
      <a
        class="is-hidden-tablet"
        :class="{ disabled: selectedTab === TabType.COLLECTION }"
        @click="openMobileFilters">
        <b-icon :icon="'bars'" size="is-medium" />
      </a>
    </div>
    <p class="control">
      <NeoButton
        class="explore-tabs-button"
        tag="nuxt-link"
        :active="selectedTab === TabType.COLLECTION"
        to="collectibles">
        <span> {{ $t('collections') }}</span>
        <img v-if="selectedTab === TabType.COLLECTION" src="/checkmark.svg" />
      </NeoButton>
    </p>
    <p class="control">
      <NeoButton
        class="explore-tabs-button"
        tag="nuxt-link"
        :active="selectedTab === TabType.ITEMS"
        to="items">
        <span> {{ $t('items') }}</span>
        <img v-if="selectedTab === TabType.ITEMS" src="/checkmark.svg" />
      </NeoButton>
    </p>
  </div>
</template>

<script setup lang="ts">
import { NeoButton } from '@kodadot1/brick'

enum TabType {
  COLLECTION = 'collectibles',
  ITEMS = 'items',
}

const route = useRoute()
const { $store } = useNuxtApp()

const selectedTab = computed(() => route?.name?.split('-')[2])
const isSidebarFiltersOpen = computed(
  () => $store.getters['preferences/getsidebarFilterCollapse']
)

const toggleSidebarFilters = () => {
  $store.dispatch(
    'preferences/setSidebarFilterCollapse',
    !isSidebarFiltersOpen.value
  )
}
const openMobileFilters = () => {
  $store.dispatch('preferences/setMobileFilterCollapse', true)
}

watch(selectedTab, () => {
  if (selectedTab.value === TabType.COLLECTION) {
    $store.dispatch('preferences/setSidebarFilterCollapse', false)
  }
})
</script>

<style lang="scss" scoped>
@import '@/styles/abstracts/variables';
a.disabled {
  opacity: 0.3;
  cursor: not-allowed;
  pointer-events: none;
}
.explore-tabs {
  &-button {
    width: 15rem;
  }

  .control,
  &-button {
    @include until-widescreen {
      width: 12rem;
    }

    @include mobile {
      width: 100%;
    }
  }
}

.dark-mode .explore-tabs-button.active img {
  filter: brightness(0%);
}
</style>
