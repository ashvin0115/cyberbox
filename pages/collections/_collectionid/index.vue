<template>
  <section class="collection">
    <img :src="collection.banner" alt="banner" class="collection__banner">
    <div class="collection__content container-xl">
      <div class="collection__header">
        <img :src="collection.logo" alt="avatar" class="collection__header-avatar">
        <h1 class="collection__header-title" ><span>{{ collection.name }}</span> <img src="/confirmed.svg" alt="confirm"></h1>
        <div class="collection__header-socials">
          <a :href="collection.discord" target="_blank" v-if="collection.discord"><img src="/socials/disckord.svg" alt="social"></a>
          <a :href="collection.telegram" target="_blank" v-if="collection.telegram"><img src="/socials/telegram.svg" alt="social"></a>
          <a :href="collection.twitter" target="_blank" v-if="collection.twitter"><img src="/socials/twitter.svg" alt="social"></a>
          <a :href="collection.website" target="_blank" v-if="collection.website"><img src="/socials/web.svg" alt="social"></a>
        </div>
        <div class="collection__header-info">
          <div class="collection__header-info-block" ref="itemsInfo">
            <h3 class="collection__header-info-block-title">{{ collectionInfo.mint_count ? collectionInfo.mint_count : 0 }}</h3>
            <h3 class="collection__header-info-block-subtitle">Items</h3>
          </div>
          <div class="collection__header-info-block">
            <h3 class="collection__header-info-block-title"><img src="/celo.svg" alt="celo">{{ (collectionInfo.sell_total_price ? collectionInfo.sell_total_price / 1000 : 0).toFixed(2) }}</h3>
            <h3 class="collection__header-info-block-subtitle">Volume traded</h3>
          </div>
         <div class="collection__header-info-block">
           <h3 class="collection__header-info-block-title"><img src="/celo.svg" alt="celo">{{ floorPrice }}</h3>
           <h3 class="collection__header-info-block-subtitle">Floor price</h3>
         </div>
         <div class="collection__header-info-block" ref="refiInfo">
           <h3 class="collection__header-info-block-title">{{ refiCO2Price }}</h3>
           <h3 class="collection__header-info-block-subtitle">Ton CO2 <img src="/plant.svg" alt="plant"></h3>
         </div>
        </div>
        <h3 class="collection__header-content">
          {{ collection.description }}
        </h3>
      </div>
      <div class="collection__filter" @click="changeFilter">
        <button
          class="collection__filter-button"
          :class="{'collection__filter-button-active': filter === 'All'}"
          @click="filter = 'All'"
        >
          All
        </button>
        <button
          class="collection__filter-button"
          :class="{'collection__filter-button-active': filter === 'listed'}"
          @click="filter = 'listed'"
        >
          Listings
        </button>
        <button
          class="collection__filter-button"
          :class="{'collection__filter-button-active': filter === 'bought'}"
          @click="filter = 'bought'"
        >
          Sold
        </button>
      </div>
      <div class="collection__sort" v-if="!isMultiNftCollection">
        <button
          class="collection__sort-button"
          :class="{'collection__sort-button-active': sort === 'mint-lowest'}"
          @click="changeSort('mint-lowest')"
          v-if="filter === 'All'"
        >
          <span>Token ID<span class="collection__sort-button-delimiter"> - </span><br class="collection__sort-button-breakline"/>Lowest</span>
        </button>
        <button
          class="collection__sort-button"
          :class="{'collection__sort-button-active': sort === 'mint-highest'}"
          @click="changeSort('mint-highest')"
          v-if="filter === 'All'"
        >
          <span>Token ID<span class="collection__sort-button-delimiter"> - </span><br class="collection__sort-button-breakline"/>Highest</span>
        </button>
        <button
          class="collection__sort-button"
          :class="{'collection__sort-button-active': sort === 'sold-latest'}"
          @click="changeSort('sold-latest')"
          v-if="filter === 'bought'"
        >
          <span>Sold<span class="collection__sort-button-delimiter"> - </span><br class="collection__sort-button-breakline"/>Latest</span>
        </button>
        <button
          class="collection__sort-button"
          :class="{'collection__sort-button-active': sort === 'price-lowest'}"
          @click="changeSort('price-lowest')"
          v-if="filter !== 'All'"
        >
          <span>Price<span class="collection__sort-button-delimiter"> - </span><br class="collection__sort-button-breakline"/>Lowest</span>  
        </button>
        <button
          class="collection__sort-button"
          :class="{'collection__sort-button-active': sort === 'price-highest'}"
          @click="changeSort('price-highest')"
          v-if="filter !== 'All'"
        >
          <span>Price<span class="collection__sort-button-delimiter"> - </span><br class="collection__sort-button-breakline"/>Highest</span>
        </button>
        <button
          class="collection__sort-button"
          :class="{'collection__sort-button-active': sort === 'rarity-rare'}"
          @click="changeSort('rarity-rare')"
          v-if="!isNomDomain"
        >
          <span>Rarity <span class="collection__sort-button-delimiter"> - </span><br class="collection__sort-button-breakline"/>Rare</span>
        </button>
        <button
          class="collection__sort-button"
          :class="{'collection__sort-button-active': sort === 'rarity-common'}"
          @click="changeSort('rarity-common')"
          v-if="!isNomDomain"
        >
          <span>Rarity <span class="collection__sort-button-delimiter"> - </span><br class="collection__sort-button-breakline"/>Common</span>
        </button>
        <button
          class="collection__sort-button"
          @click="showTraitsFilter = true"
          v-if="!isNomDomain"
        >
          <span class="collection__sort-button-title traits">Traits</span> <img src="/trait.svg" alt="trait" class="collection__sort-button-icon"> <span class="collection__sort-button-badge" v-if="filtersCount > 0">{{filtersCount}}</span>
        </button>
      </div>
<!--      <attributesFilter />-->
      <div class="collection__loading" v-if="loading">
        <img src="/loading-button.svg" alt="load">
      </div>
      <div class="collection__info" v-else>
        <h3 class="collection__info-items">{{ countItems }} items</h3>
        <div class="collection__info-nft">
          <div class="collection__info-nft-search search-box" v-if="!isMultiNftCollection">
            <input class="search-box-input" :type="!isNomDomain ? 'number' : 'text'" min="1" :placeholder="fitlerPlaceholder" v-model="searchName" @input="searchNft">
            <img src="/search.svg" alt="search" class="search-box-img" v-if="!searchName">
            <img src="/close-bold.svg" alt="close" class="search-box-img icon-close" @click="clearSearch" v-else>
          </div>
          <div  class="collection__info-nft-filter" @click="changeMyNftStatus">
            <h3 class="collection__info-nft-text">My NFTs</h3>
            <div class="collection__info-nft-switcher" :class="{'collection__info-nft-switcher-active': myNft}">
              <div class="collection__info-nft-switcher-element" :class="{'collection__info-nft-switcher-element-active': myNft}"></div>
            </div>
          </div>
        </div>
      </div>
      <div class="collection__loading" v-if="nftLoading">
        <img src="/loading-button.svg" alt="load">
      </div>
      <div class="collection__items" v-else-if="nftList.length && !loading">
        <nft :nft="nft" :key="index"  v-for="(nft, index) of nftList" :filter="filter" :owner="nftOwned(nft)" :multiNft="isMultiNftCollection" :seller="false" :route="nftRoute(nft)"/>
      </div>
      <p class="collection__empty-items" v-else-if="!loading">There are no results matching your selected criteria</p>
    </div>
    <TraitsFilterModal
      :show="showTraitsFilter"
      :mintCount="collectionInfo.mint_count"
      :filtersCount="filtersCount"
      @updateFilter="updateTraitFilter"
      @close="showTraitsFilter = false"
      v-if="showTraitsFilter"
    />
  </section>
</template>
<script>
import _ from 'lodash'
import nft from '@/components/nft.vue'
import attributesFilter from '@/components/modals/attributesFilter'
import TraitsFilterModal from '@/components/modals/traitsFilterModal'
import {BigNumber} from 'ethers'
export default {
  data() {
    return {
      loading: false,
      nftLoading: false,
      showTraitsFilter: false,
      filter: 'All',
      activeRequest: 'getGraphData',
      sort: '',
      myNft: false,
      collectionInfo: {},
      floorPrice: '-',
      refiCO2Price: '-',
      traitFilters: null,
      searchName: '',
    }
  },
  head() {
    return {
      meta: [
        { hid: 'title', name: 'title', content: this.pageTitle },
        { hid: 'og:title', property: 'og:title', content: this.pageTitle },
        { hid: 'description', name: 'description', content: this.description },
        { hid: 'og:description', property: 'og:description', content: this.description },
        { hid: 'og:image', property: 'og:image', content: this.metaIcon }
      ]
    }
  },
  components: {
    nft,
    attributesFilter,
    TraitsFilterModal
  },
  computed: {
    pageTitle() {
      return `${this.collection.name} | Cyberbox ReFi NFT Marketplace`
    },
    description() {
      return this.collection.description
    },
    metaIcon() {
			let imageSrc = ''
      switch (this.$route.params.collectionid) {
        case 'cpunk': imageSrc = '/collections/Media_punks.png'
          break
        case 'ctoadz': imageSrc = '/collections/Media_toadz.png'
          break
        case 'knoxnft': imageSrc = '/collections/KnoxersDAO.png'
          break
        default: imageSrc = this.collection.image
          break
      }
      return imageSrc
    },
    isMultiNftCollection() {
      return this.$store.state.multiNftSymbols.includes(this.$route.params.collectionid)
    },
    isNomDomain() {
      return this.$route.params.collectionid === 'nomdom'
    },
    fitlerPlaceholder() {
      return !this.isNomDomain ? 'Mint number' : 'NOM name'
    },
    countItems() {
      if (!this.myNft) {
        switch (this.filter) {
          case 'All': return this.collectionInfo.filter_count || this.collectionInfo.mint_count;
          case 'listed': return this.collectionInfo.list_count;
          case 'bought': return this.collectionInfo.sell_count;
        }
      } else {
        return this.nftList.length
      }
    },
    collection() {
      return this.$store.state.collectionList.filter(item => item.route === this.$route.params.collectionid)[0]
    },
    nftList() {
      return this.$store.state.nftList
    },
    address() {
      return this.$store.state.address
    },
    fetchEnabled() {
      return this.address || (!this.address && !this.myNft)
    },
    filtersCount() {
      let sectionCount = 0
      this.$store.state.traitFilters.forEach(item => {
        const filteredValues = item.values.filter(filterItem => filterItem.checked)
        if (filteredValues.length > 0) {
          sectionCount++
        }
      })
      return sectionCount
    }
  },
  watch: {
    nftList() {
      if (this.$store.state.nftList.length === 0) {
        if (window.innerWidth < 836) {
          this.showFixedFooter(true)
        }
      } else {
        this.showFixedFooter(false)
      }
    },
    loading(newVal) {
      if (window.innerWidth < 836 && this.loading) {
        this.showFixedFooter(true)
      }
    },
    nftLoading() {
      if (window.innerWidth < 836 && this.nftLoading) {
        this.showFixedFooter(true)
      }
    }
  },
  methods: {
    nftRoute(nft) {
      if (!this.isMultiNftCollection) {
        return `/collections/${nft.contract}/${nft.contract !== 'nomdom' ? nft.contract_id : nft.image}`
      } else {
        return `/collections/${nft.nftSymbol}/${nft.id.split('/')[1].split('.')[0]}`
      }
    },
    nftOwned(nft) {
      return nft.owner && nft.owner.toLowerCase() === this.$store.state.fullAddress
    },
    showFixedFooter(show) {
      const footerEl = document.querySelector('.footer')
      if (show) {
        footerEl.classList.add('fixed')
      } else {
        footerEl.classList.remove('fixed')
      }
    },
    initNftListSetting() {
      this.$store.commit('setNewNftList', [])
      this.$store.commit('changeCountPage', 1)
      this.$store.commit('changeSortData', 'all')
    },
    async fetchNftsBySearch() {
      this.nftLoading = true
      const searchValue = !this.isNomDomain && this.searchName ? parseInt(this.searchName) : this.searchName
      if (!this.isNomDomain) {
        this.$store.commit('changeMintNumFilter', !searchValue ? null : searchValue)
      } else {
        this.$store.commit('changeNomNameFilter', !searchValue ? null : searchValue)
      }
      await this.$store.dispatch(this.activeRequest, this.$store.state.filteredTraits)
      this.nftLoading = false
    },
    searchNft: _.debounce(function() {
      if (this.searchName && parseInt(this.searchName) < 0) {
        this.searchName = ''
      } else {
        this.fetchNftsBySearch()
      }
    }, 500),
    clearSearch() {
      this.searchName = ''
      this.fetchNftsBySearch()
    },
    addCurrentPage() {
      const filteredTraits = this.$store.state.filteredTraits
      const count = this.$store.state.countPage
      const element = document.body
      if (element.scrollHeight <= window.pageYOffset + window.innerHeight && (!filteredTraits && count * 48 === this.nftList.length || filteredTraits) && this.nftList.length > 0) {
        this.$store.commit('changeCountPage', count + 1)
        this.$store.commit('changeSortData', 'pagination')
        this.$store.dispatch(this.activeRequest, filteredTraits)
      }
    },
    async fetchNftList() {
      if (this.fetchEnabled) {
        await this.$store.dispatch(this.activeRequest)
      } else {
        this.$store.commit('setNewNftList', [])
      }
    },
    changeCollectionSetting(setting) {
      const collectionSetting = this.$store.state.collectionSetting
      this.$store.commit('updateCollectionSetting', {
        ...collectionSetting,
        ...setting
      })
    },
    async changeSort(id) {
      this.sort = id
      if (this.isMultiNftCollection) return
      this.loading = true
      let sortPrefix = ''
      if (this.myNft) {
        if (this.address) {
          sortPrefix = 'myNft-'
        }
      }
      this.$store.commit('changeSortData', this.filter === 'bought' ? (sortPrefix + `${id}-sold`) : (sortPrefix + id))
      await this.fetchNftList()
      this.changeCollectionSetting({ sort: id })
      this.loading = false
    },
    changeMyNftFilter() {
      let sortMyNft = 'all'
      if (this.myNft) {
        switch (this.filter) {
          case 'All':
          case 'listed':
            sortMyNft = 'myNftAll'
            break;
          case 'bought':
            sortMyNft = 'myNftSold'
            break;
        }
      }

      if (this.address) {
        this.$store.commit('changeSortData', sortMyNft)
      }
      this.fetchNftList()
    },
    changeMyNftStatus() {
      this.myNft = !this.myNft
      if (!this.isMultiNftCollection) return
      this.changeMyNftFilter()
      this.changeCollectionSetting({ myNft: this.myNft })
    },
    changeFilter() {
      const filter = this.filter
      this.sort = '';
      this.$store.commit('setNewNftList', [])
      this.$store.commit('changeSortData', 'all')
      let activeRequest = 'getGraphData'
      switch (filter) {
        case 'All': activeRequest = 'getGraphData'
          break;
        case 'listed': activeRequest = 'getGraphDataListed'
          break;
        case 'bought': activeRequest = 'getGraphDataSells'
      }
      this.activeRequest = activeRequest
      this.changeCollectionSetting({
        filter: this.filter,
        fetchRequest: activeRequest
      })
      if (this.myNft) {
        this.changeMyNftFilter()
      } else {
        this.$store.dispatch(activeRequest, this.traitFilters)
      }
      this.$store.commit('changeCountPage', 1)
    },
    async updateTraitFilter(filters, filteredCount) {
      this.loading = true
      this.initNftListSetting()
      this.traitFilters = filters
      await this.$store.dispatch(this.activeRequest, filters)
      this.collectionInfo = {
        ...this.collectionInfo,
        filter_count: filteredCount
      }
      this.loading = false
    }
  },
  beforeMount() {
    window.addEventListener('scroll', this.addCurrentPage)
  },
  beforeDestroy() {
    window.removeEventListener('scroll', this.addCurrentPage)
  },
  async created() {
    if (process.browser && localStorage.getItem('move_back')) {
      localStorage.removeItem('move_back')
      const collectionSetting = this.$store.state.collectionSetting
      if (collectionSetting) {
        this.activeRequest = collectionSetting.fetchRequest || this.activeRequest
        this.filter = collectionSetting.filter || this.filter
        this.sort = collectionSetting.sort || this.sort
        this.myNft = collectionSetting.myNft || this.myNft
      }
      this.searchName = this.$store.state.mintNumFilter
    } else {
      this.$store.commit('setTraitFilters', [])
      this.loading = true
      this.initNftListSetting()
      this.$store.commit('changeMintNumFilter', null)
      this.$store.commit('updateCollectionSetting', null)
      await this.$store.dispatch(this.activeRequest)
      this.$store.dispatch('loadTraitFilters')
    }
    const collectionResult = await this.$store.dispatch('getCollectionInfo')
    collectionResult ? this.collectionInfo = collectionResult : this.collectionInfo = {}
    this.floorPrice = await this.$store.dispatch('getFloorPrice', this.$route.params.collectionid)
    if (this.$store.state.cMCO2Price > 0 && this.collectionInfo.producerFee > 0 && this.collectionInfo.sell_refi_price > 0) {
      const refiPrice = this.$store.state.cMCO2Price * this.collectionInfo.producerFee / 1000 * this.collectionInfo.sell_refi_price / 1000
      this.refiCO2Price = refiPrice > 1 ? Math.round(refiPrice).toLocaleString('EN-US') : refiPrice.toFixed(3)
    }
    this.loading = false
  },
  mounted() {
    if (this.isMobile()) {
      this.$refs.itemsInfo.parentNode.insertBefore(this.$refs.refiInfo, this.$refs.itemsInfo.nextSibling)
    }
  }
}
</script>
<style lang="scss">
.collection {
  padding-bottom: 20rem;
  &__banner {
    width: 100%;
    height: 22.4rem;
    object-fit: cover;
  }
  &__content {
    position: relative;
    top: -7rem;
  }
  &__header {
    display: flex;
    flex-direction: column;
    align-items: center;
    &-avatar {
      width: 12.6rem;
      height: 12.6rem;
      border-radius: 50%;
      border: .2rem solid $white;
    }
    &-title {
      display: flex;
      align-items: center;
      justify-content: center;
      flex-wrap: wrap;
      padding-top: 2rem;
      span {
        margin-right: 1rem;
        white-space: nowrap;
      }
      img {
        width: 2rem;
        transform: translateY(.2rem);
      }
    }
    &-socials {
      position: absolute;
      right: 0;
      top: 10rem;
      width: 17rem;
      display: flex;
      align-items: center;
      justify-content: space-between;
      :first-child {
        width: 2.1rem;
      }
      a {
        img {
          width: 2rem;
          cursor: pointer;
          &:hover {
            animation: shaking .5s;
          }
        }
      }
    }
    &-info {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding-top: 2rem;
      &-block {
        width: 14.6rem;
        height: 6.8rem;
        border: .1rem solid $lightGreen;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        margin-right: 1rem;
        &-title {
          font-family: OpenSans-SemiBold;
          font-size: 1.6rem;
          display: flex;
          align-items: center;
          img {
            width: 1.6rem;
            margin-right: .6rem;
          }
        }
        &-subtitle {
          font-family: OpenSans-Regular;
          font-size: 1.4rem;
          margin-top: .4rem;
          img {
            width: 1.2rem;
          }
        }
      }
    }
    &-content {
      display: flex;
      justify-content: center;
      color: $grayLight;
      width: 65rem;
      padding-top: 1rem;
      letter-spacing: 0.03em;
      font-size: 1.4rem;
    }
  }
  &__filter {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    border: .1rem solid $modalColor;
    border-radius: 2.5rem;
    margin-top: 4rem;
    &-button {
      width: 42.6rem;
      height: 5.4rem;
      border-radius: 2.5rem;
      background: none;
      &-active {
        background: $lightGreen;
      }
    }
  }
  &__sort {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    margin-top: 3.2rem;
    &-button {
      width: 100%;
      height: 4rem;
      box-shadow: 0 .4rem 1.2rem rgba(0, 0, 0, 0.05);
      border-radius: .8rem;
      margin-right: 2.0rem;
      background: none;
      font-family: OpenSans-Regular;
      display: flex;
      align-items: center;
      justify-content: center;
      &:last-child {
        margin-right: 0;
      }
      img {
        margin-left: 1.2rem;
        width: 2rem;
      }
      &-active {
        background: $modalColor;
      }
      &-breakline {
        display: none;
      }
      &-title {
        line-height: 1;
      }
      &-icon {
        margin-left: 4px !important;
      }
      &-icon, &-badge {
        margin-left: 4px;
      }
      &-badge {
        display: flex;
        align-items: center;
        justify-content: center;
        background: $border;
        width: 20px;
        height: 20px;
        border-radius: 50%;
        line-height: 1.1;
        font-weight: 600;
        font-size: 1.4rem;
        color: $white;
      }
    }
  }
  &__loading {
    display: flex;
    align-items: center;
    justify-content: center;
    padding-top: 20rem;
    img {
      width: 8rem;
      animation: loading 1s infinite;
    }
  }
  &__info {
    padding-top: 4rem;
    display: flex;
    align-items: center;
    justify-content: space-between;
    &-nft {
      display: flex;
      align-items: center;
      &-search {
        width: 15.3rem !important;
        margin-right: 1.84rem;
        .search-box-input {
          width: calc(100% - 32px) !important;
          font-size: 1.4rem;
        }
      }
      &-filter {
        display: flex;
        align-items: center;
      }
      &-text {
        margin-right: 1.6rem;
      }
      &-switcher {
        width: 5.3rem;
        height: 2.6rem;
        background: $border2;
        border-radius: 1.55rem;
        position: relative;
        cursor: pointer;
        transition: .5s;
        &-element {
          position: absolute;
          left: .2rem;
          top: .2rem;
          background: $white;
          width: 2.2rem;
          height: 2.2rem;
          border-radius: 1.5rem;
          transition: .5s;
          &-active {
            left: 2.8rem;
          }
        }
        &-active {
          background: $green;
        }
      }
    }
  }
  &__items {
    display: grid;
    grid-template-columns: 20rem 20rem 20rem 20rem 20rem 20rem;
    grid-column-gap: 2.4rem;
    grid-row-gap: 3.2rem;
    padding-top: 3.2rem;
  }
  &__empty-items {
    padding: 36px 0;
    text-align: center;
    font-size: 14px;
  }
}
@media screen and (max-width: 460px) {
  .collection {
    &__banner {
      height: 12rem;
    }
    &__content {
      top: -4.5rem;
    }
    &__header {
      &-avatar {
        width: 8rem;
        height: 8rem;
      }
      &-title {
        padding-top: 1rem;
      }
      &-subtitle {
        padding-top: .5rem;
      }
      &-socials {
        top: 17rem;
        left: auto;
        right: auto;
        width: 15rem;
      }
      &-info {
        width: 100%;
        padding-top: 7.2rem;
        flex-wrap: wrap;
        &-block {
          width: 14rem;
          height: 5.6rem;
          margin-bottom: .8rem;
          text-align: center;
          &-title {
            font-size: 1.4rem;
            justify-content: center;
            img {
              width: 1.6rem !important;
              margin-right: .8rem !important;
            }
          }
          &-subtitle {
            font-size: 1rem;
          }
          &:last-child {
            margin-right: 0;
          }
          &:nth-child(2), &:last-child {
            flex: 1;
            width: auto;
          }
        }
      }
      &-content {
        padding-top: 1rem;
        font-size: 1.4rem;
        text-align: left;
        width: auto;
      }
    }
    &__filter {
      margin-top: 2rem;
      &-button {
        font-size: 1.4rem;
      }
    }
    &__items {
      grid-template-columns: 14.4rem 14.4rem;
      grid-column-gap: 1.6rem;
      grid-row-gap: 1.6rem;
      padding-top: 2.2rem;
    }
    &__sort {
      padding-bottom: 1rem;
      overflow-x: auto;
      -ms-overflow-style: none;
      scrollbar-width: none;
      &::-webkit-scrollbar {
        display: none;
      }
      &-button {
        flex: 1;
        width: auto;
        height: 5.2rem;
        margin-right: .8rem;
        font-size: 1rem;
        &-delimiter {
          display: none;
        }
        &-breakline {
          display: inline-block;
        }
        &:last-child {
          margin: 0;
        }
        &-title.traits {
          display: none;
        }
      }
    }
    &__info {
      position: relative;
      padding-top: 6.3rem;
      .search-box {
        width: 100% !important;
        position: absolute;
        top: 1.6rem;
        left: 0;
        margin: 0;
      }
      &-items, &-nft-text {
        font-size: 1.4rem;
      }
    }
  }
}
@keyframes shaking {
  0% {
    transform: rotate(-10deg);
  }
  50% {
    transform: rotate(10deg);
  }
  100% {
    transform: rotate(-10deg);
  }
}
</style>
