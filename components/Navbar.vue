<template>
  <b-navbar
    :active.sync="isBurgerMenuOpened"
    :class="{ 'navbar-shrink': !showTopNavbar }"
    close-on-click
    fixed-top
    mobile-burger
    spaced
    wrapper-class="container">
    <template #brand>
      <b-navbar-item :to="{ path: '/' }" class="logo" tag="nuxt-link">
        <img
          :src="logoSrc"
          alt="First NFT market explorer on Kusama and Polkadot"
          width="143"
          height="42" />
      </b-navbar-item>
      <div
        class="is-hidden-desktop is-flex is-flex-grow-1 is-align-items-center is-justify-content-flex-end"
        @click="closeBurgerMenu">
        <!-- <HistoryBrowser class="navbar-item" /> -->
        <img
          v-if="showSearchOnNavbar"
          class="mobile-nav-search-btn mr-2"
          :src="
            isDarkMode
              ? '/search-mobile-navbar-dark.svg'
              : '/search-mobile-navbar.svg'
          "
          @click="showMobileSearchBar" />

        <div v-show="openMobileSearchBar">
          <div
            class="fixed-stack is-flex is-align-items-center is-justify-content-space-between p-2">
            <Search
              ref="mobilSearchRef"
              hide-filter
              class="is-flex-grow-1 mt-3" />
            <b-button class="cancel-btn" @click="hideMobileSearchBar">
              {{ $t('cancel') }}
            </b-button>
          </div>
        </div>
      </div>
    </template>
    <template #start>
      <div v-if="showSearchOnNavbar" class="navbar-item is-expanded">
        <Search
          class="search-navbar is-flex-grow-1 pb-0 is-hidden-touch"
          hide-filter
          search-column-class="is-flex-grow-1" />
      </div>
    </template>
    <template #end>
      <ExploreDropdown
        v-if="!isMobile"
        class="navbar-explore custom-navbar-item"
        data-cy="explore" />
      <MobileExpandableSection v-if="isMobile" :title="$t('explore')">
        <NavbarExploreOptions />
      </MobileExpandableSection>

      <CreateDropdown
        v-show="isCreateVisible"
        class="navbar-create custom-navbar-item ml-0"
        data-cy="create"
        :is-mobile="isMobile"
        :chain="chain" />
      <StatsDropdown
        class="navbar-stats custom-navbar-item"
        data-cy="stats"
        :is-mobile="isMobile"
        :chain="chain" />

      <ChainSelectDropdown
        v-if="!isMobile"
        id="NavChainSelect"
        class="navbar-chain custom-navbar-item"
        data-cy="chain-select" />
      <template v-if="isMobile">
        <MobileLanguageOption v-if="!account" />
        <MobileExpandableSection
          v-if="account"
          :no-padding="true"
          :title="$t('account')"
          icon="user-circle"
          icon-family="fa">
          <b-navbar-item
            :to="`/${urlPrefix}/u/${account}`"
            data-cy="hot"
            tag="nuxt-link">
            {{ $t('profile.page') }}
          </b-navbar-item>
          <b-navbar-item
            :to="{ name: 'identity' }"
            data-cy="hot"
            tag="nuxt-link">
            {{ $t('identity.page') }}
          </b-navbar-item>
          <b-navbar-item data-cy="hot" tag="nuxt-link" to="/settings">
            {{ $t('settings') }}
          </b-navbar-item>
          <MobileLanguageOption />
          <MobileNavbarProfile
            id="NavProfile"
            @closeBurgerMenu="closeBurgerMenu" />
        </MobileExpandableSection>
        <div
          v-if="account"
          class="navbar-item"
          @click.stop="openWalletConnectModal">
          <span>
            {{ $t('wallet') }}
            <b-icon icon="wallet" />
          </span>
          <b-icon class="icon--right" icon="chevron-right" pack="fas" />
        </div>
        <ColorModeButton class="navbar-item" />

        <div v-if="!account" id="NavProfile">
          <ConnectWalletButton
            class="button-connect-wallet"
            variant="connect"
            @closeBurgerMenu="closeBurgerMenu" />
        </div>
      </template>
      <ProfileDropdown
        v-if="!isMobile"
        id="NavProfile"
        :chain="chain"
        data-cy="profileDropdown"
        @closeBurgerMenu="closeBurgerMenu" />
    </template>
  </b-navbar>
</template>

<script lang="ts">
import { Component, Ref, Watch, mixins } from 'nuxt-property-decorator'
import BasicImage from '@/components/shared/view/BasicImage.vue'
import MobileExpandableSection from '@/components/navbar/MobileExpandableSection.vue'
import NavbarExploreOptions from '@/components/navbar/NavbarExploreOptions.vue'
import ProfileDropdown from '~/components/navbar/ProfileDropdown.vue'
import Search from '@/components/search/Search.vue'
import ExploreDropdown from '~/components/navbar/ExploreDropdown.vue'
import CreateDropdown from '~/components/navbar/CreateDropdown.vue'
import KodaBetaDark from '@/assets/Koda_Beta_dark.svg'
import KodaBeta from '@/assets/Koda_Beta.svg'
import PrefixMixin from '@/utils/mixins/prefixMixin'
import ColorModeButton from '~/components/common/ColorModeButton.vue'
import MobileLanguageOption from '~/components/navbar/MobileLanguageOption.vue'
import { createVisible } from '@/utils/config/permision.config'
import AuthMixin from '@/utils/mixins/authMixin'
import ExperimentMixin from '@/utils/mixins/experimentMixin'
import ChainSelectDropdown from '~/components/navbar/ChainSelectDropdown.vue'
import StatsDropdown from '~/components/navbar/StatsDropdown.vue'
import MobileNavbarProfile from '~/components/navbar/MobileNavbarProfile.vue'
import ConnectWalletButton from '~/components/shared/ConnectWalletButton.vue'
import { getKusamaAssetId } from '~/utils/api/bsx/query'
import { NeoButton } from '@kodadot1/brick'
import { ConnectWalletModalConfig } from '@/components/common/ConnectWallet/useConnectWallet'

@Component({
  components: {
    NeoButton,
    Search,
    BasicImage,
    MobileExpandableSection,
    ProfileDropdown,
    ExploreDropdown,
    CreateDropdown,
    ChainSelectDropdown,
    StatsDropdown,
    MobileNavbarProfile,
    NavbarExploreOptions,
    ConnectWalletButton,
    ColorModeButton,
    MobileLanguageOption,
  },
})
export default class NavbarMenu extends mixins(
  PrefixMixin,
  AuthMixin,
  ExperimentMixin
) {
  public showTopNavbar = true
  public openMobileSearchBar = false
  private fixedTitleNavAppearDistance = 85
  private lastScrollPosition = 0
  private artistName = ''
  public isBurgerMenuOpened = false
  private isMobile = window.innerWidth < 1024

  @Ref('mobilSearchRef') readonly mobilSearchRef

  get account() {
    return this.$store.getters.getAuthAddress
  }

  get chain(): string {
    return this.urlPrefix
  }

  get tokens() {
    return ['', getKusamaAssetId(this.urlPrefix)]
  }

  get isCreateVisible(): boolean {
    return createVisible(this.urlPrefix)
  }

  get currentCollection() {
    return this.$store.getters['history/getCurrentlyViewedCollection'] || {}
  }

  get currentGalleryItemName() {
    return this.$store.getters['history/getCurrentlyViewedItem']?.name || ''
  }

  get isLandingPage() {
    return this.$route.name === 'index'
  }

  get isDarkMode() {
    return (
      this.$colorMode.preference === 'dark' ||
      document.documentElement.className.includes('dark-mode')
    )
  }

  get logoSrc() {
    return this.isDarkMode ? KodaBetaDark : KodaBeta
  }

  get showSearchOnNavbar(): boolean {
    return !this.isLandingPage || !this.showTopNavbar || this.isBurgerMenuOpened
  }

  public openWalletConnectModal(): void {
    this.closeBurgerMenu()

    this.$buefy.modal.open({
      parent: this,
      ...ConnectWalletModalConfig,
    })
  }

  @Watch('isBurgerMenuOpened')
  onBurgerMenuOpenedChanged() {
    this.setBodyScroll(!this.isBurgerMenuOpened)
  }

  onScroll() {
    const currentScrollPosition = document.documentElement.scrollTop
    const searchBarPosition = document
      .getElementById('networkList')
      ?.getBoundingClientRect()?.top
    if (currentScrollPosition <= 0) {
      this.showTopNavbar = true
      return
    }
    if (Math.abs(currentScrollPosition - this.lastScrollPosition) < 30) {
      return
    }
    if (this.isLandingPage && searchBarPosition) {
      this.showTopNavbar = searchBarPosition > this.fixedTitleNavAppearDistance
    } else {
      this.showTopNavbar =
        currentScrollPosition < this.fixedTitleNavAppearDistance
    }
    this.lastScrollPosition = currentScrollPosition
  }
  setBodyScroll(allowScroll: boolean) {
    this.$nextTick(() => {
      const body = document.querySelector('body') as HTMLBodyElement
      if (allowScroll) {
        body.classList.remove('is-clipped')
      } else {
        body.classList.add('is-clipped')
      }
    })
  }

  showMobileSearchBar() {
    this.openMobileSearchBar = true
    this.$nextTick(() => {
      this.mobilSearchRef?.focusInput()
    })
    this.setBodyScroll(false)
  }

  hideMobileSearchBar() {
    this.openMobileSearchBar = false
    this.setBodyScroll(true)
  }

  closeBurgerMenu() {
    this.isBurgerMenuOpened = false
  }

  handleResize() {
    this.isMobile = window.innerWidth < 1024
  }

  mounted() {
    window.addEventListener('scroll', this.onScroll)
    document.body.style.overflowY = 'initial'
    window.addEventListener('resize', this.handleResize)
  }

  beforeDestroy() {
    window.removeEventListener('scroll', this.onScroll)
    this.setBodyScroll(true)
    document.documentElement.classList.remove('is-clipped-touch')
    window.removeEventListener('resize', this.handleResize)
  }
}
</script>
