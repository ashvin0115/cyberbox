<template>
  <div class="modal">
    <div class="modal__block sell" :class="{ success: showSuccessModal }">
        <div class="modal__sell-header">
          <button class="modal__sell-header-button" @click="closeModal"><img src="/close.svg" alt="close"></button>
        </div>
        <h2 class="modal__title">{{ modalTitle }}</h2>
        <div class="modal__sell">
          <div class="modal__sell-block" v-if="!showSuccessModal">
            <div class="modal__sell-info">
              <img class="modal__image" :src="getNFTImage(nft)">
              <div class="modal__sell-nft">
                <p class="modal__sell-collection">{{ collectionName }}</p>
                <p class="modal__sell-description">{{ sellDescription }}</p>
              </div>
            </div>
            <div class="modal__sell-form">
              <p class="modal__sell-form-price">Price</p>
              <div class="modal__sell-form-inputbox">
                <input class="modal__sell-form-input" ref="sellInput" type="number" min="0" :readonly="pending || successSellToken" placeholder="0" v-model="nftPrice">
              </div>
              <p class="modal__sell-form-unit">${{ dollarPrice }}</p>
              <p class="modal__sell-form-refioffset">
                <img src="/plant.svg" alt="plant" class="modal__sell-form-refioffset-img"> Successful NFT sale offset<span class="modal__sell-form-refioffset-amount">{{ refiOffset }} ton CO2</span>
              </p>
              <p class="modal__sell-form-description">Item will be on sale until you cancelled.</p>
              <p class="modal__sell-form-feeinfo">Once sold, the following fees will be deducted:<br/>{{ nftServiceFee }}% service fee | {{ nftRoyalty }}% creator royalty | {{nftProducerFee}}% ReFi fee</p>
            </div>
            <p class="modal__sell-content">List your NFT to sell for {{ nftPrice || 0 }} CELO</p>
          </div>
          <div class="modal__sell-success" v-else>
            <p class="modal__sell-description">{{ sellDescription }}</p>
            <img class="modal__sell-success-icon" src="/success.svg" alt="success">
          </div>
          <div class="modal__sell-buttons">
            <button class="modal__sell-buttons-button modal__sell-buttons-button-done" @click="closeSuccessModal" v-if="showSuccessModal">Done</button>
            <div class="modal__sell-buttons-box" v-else>
              <button class="modal__sell-buttons-button btn-approve" :class="{ disabled: !nftPrice || tokenApproved, pending: !tokenApproved && pending }" @click="submitApproveToken" v-if="!approved">
                Approve
                <img class="modal__sell-buttons-button-loading" src="/loading-button.svg" alt="loading" v-if="!tokenApproved && pending">
              </button>
              <button class="modal__sell-buttons-button" :class="{ disabled: !nftPrice || !tokenApproved, pending: tokenApproved && pending }" @click="sellToken">
                Confirm
                <img class="modal__sell-buttons-button-loading" src="/loading-button.svg" alt="loading" v-if="tokenApproved && pending">
              </button>
            </div>
          </div>
          <div class="modal__step" v-if="!approved && !showSuccessModal">
            <span class="modal__step-status" :class="{ active: nftPrice, approved: tokenApproved }">
              <span v-if="!tokenApproved">1</span>
              <img src="/check-circle.svg" alt="check" v-else>
            </span>
            <span class="modal__step-line" :class="{ active: tokenApproved }"></span>
            <span class="modal__step-status" :class="{ active: tokenApproved, approved: successSellToken }">
              <span v-if="!successSellToken">2</span>
              <img src="/check-circle.svg" alt="check" v-else>
            </span>
          </div>
        </div>
    </div>
  </div>  
</template>
<script>

export default {
  props: ['nft', 'celoPrice', 'approved'],
  computed: {
    approveToken() {
      return this.$store.state.approveToken
    },
    tokenApproved() {
      return this.approved || this.approveToken === 'approve'
    },
    listToken() {
      return this.$store.state.listToken
    },
    successSellToken() {
      return this.listToken === true
    },
    modalTitle() {
      if (!this.showSuccessModal) {
        return this.nft.market_status !== 'LISTED' ? 'Sell your NFT' : 'Change sell price'
      } else {
        return 'Successfully listing'
      }
    },
    collectionName() {
      return this.$store.state.collectionList.find(item => item.route === this.nft.contract).name
    },
    sellDescription() {
      const nameSuffix = this.nft.contract !== 'nomdom' ? '' : '.nom'
      return `${this.nft.name}${nameSuffix}`
    },
    dollarPrice() {
      return this.nftPrice ? (this.nftPrice * this.celoPrice).toFixed(1) : 0
    },
    refiOffset() {
      const offsetValue = this.nftPrice * this.nft.refiOffset
      return offsetValue > 0 ? parseFloat(offsetValue).toFixed(3) : 0
    }
  },
  data() {
    return {
      nftPrice: null,
      pending: false,
      showSuccessModal: false,
      nftServiceFee: 0,
      nftRoyalty: 0,
      nftProducerFee: 0
    }
  },
  watch: {
    approveToken() {
      if (this.$store.state.approveToken === 'approve') {
        this.pending = false
      } else {
        this.closeModal()
      }
    },
    listToken() {
      if (this.$store.state.listToken === true) {
        this.pending = false
        setTimeout(() => this.showSuccessModal = true)
      } else {
        this.closeModal()
      }
    }
  },
  async mounted() {
    this.$store.commit('changelistToken', '')
    const collectionInfo = await this.$store.dispatch('getCollectionInfo')
    this.nftServiceFee = collectionInfo.marketFee / 10
    this.nftRoyalty = collectionInfo.createrFee / 10
    this.nftProducerFee = collectionInfo.producerFee / 10
    if (this.nft.market_status === 'LISTED') {
      this.nftPrice = this.nft.price
    }
  },
  methods: {
    closeModal() {
      this.$emit('closeModal', this.tokenApproved)
      this.$store.commit('changelistToken', '')
    },
    closeSuccessModal() {
      this.$emit('done', this.nftPrice ? parseFloat(this.nftPrice) : 0)
      this.$store.commit('changelistToken', '')
    },
    submitApproveToken() {
      if (this.pending) return
      if (this.nftPrice) {
        this.pending = true
        this.$store.dispatch('approveToken')
      } else {
        this.$refs.sellInput.focus()
      }
    },
    sellToken() {
      if (this.pending) return
      if (this.nftPrice) {
        this.pending = true
        if (this.nft.market_status !== 'LISTED') {
          this.$store.dispatch('listingNFT', this.nftPrice)
        } else {
          this.$store.dispatch('changeNFTPrice', this.nftPrice)
        }
      } else {
        this.$refs.sellInput.focus()
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.modal {
  &__block {
    background: $white;
  }
  &__title {
    text-align: left;
    font-size: 3.2rem;
    color: $titleColor;
  }
  @media(max-width: 460px) {
    &__title {
      font-size: 2.2rem;
    }
  }
}
</style>

<style lang="scss">
.modal {
  &__block {
    &.sell {
      width: 45.6rem;
      background: $white;
      padding: 2.1rem 1.6rem 2.4rem;
      cursor: default;
      &.success {
        width: 23.5rem;
        padding: 1.8rem 1.8rem 2.6rem;
        .modal__title {
          width: 100%;
          font-size: 1.8rem;
          text-align: center;
        }
        .modal__sell {
          padding-top: 0;
          text-align: center;
          &-header {
            display: none;
          }
          &-description {
            margin-top: 0.4rem;
            font-size: 1.4rem;
          }
          &-success-icon {
            padding: 2.1rem 0 2.9rem;
          }
          &-link {
            font-weight: 600;
            font-size: 1.6rem;
            color: $grayLight;
            img {
              width: 2rem;
              transform: translateY(4px);
            }
          }
        }
      }
    }
  }
  &__sell {
    padding-top: 1.8rem;
    &.success {
      padding-top: 0;
    }
    &-header {
      display: block;
      width: calc(100% + 3.2rem);
      height: 1.4rem;
      margin: 0 -1.6rem;
      padding: 0;
      position: relative;
      &-button {
        background: transparent;
        position: absolute;
        top: 0;
        right: 1.6rem;
        line-height: 1.4rem;
      }
    }
    &-info {
      display: flex;
      flex-wrap: wrap;
      .modal__image {
        display: block;
        width: 6.4rem;
        padding: 0;
        margin-right: 1.2rem;
        border-radius: 4px;
      }
    }
    &-collection {
      font-weight: 600;
      font-size: 1.4rem;
      color: $grayLight;
    }
    &-description {
      font-weight: 600;
      font-size: 1.8rem;
    }
    &-form {
      padding-top: 1.8rem;
      padding-bottom: 2.4rem;
      font-size: 1.8rem;
      &-price {
        padding-bottom: 1.6rem;
        font-size: 1.4rem;
        color: $grayLight;
      }
      &-input {
        width: calc(100% - 1.85rem);
        height: 2rem;
        padding: 1.2rem 1rem;
        border: 1px solid $border;
        font-size: 1.6rem;
        &:focus {
          border-color: $green;
        }
        &:read-only {
          background: transparent;
          border-color: $border;
          color: #9CA3AF;
        }
      }
      &-inputbox {
        position: relative;
        &::after {
          display: block;
          content: "CELO";
          position: absolute;
          top: 1.2rem;
          right: 1rem;
          font-size: 1.6rem;
        }
      }
      &-unit {
        padding-top: .8rem;
        font-size: 1.4rem;
        color: $grayLight;
      }
      &-refioffset {
        display: flex;
        align-items: center;
        width: fit-content;
        width: -moz-fit-content;
        margin-top: 1.8rem;
        padding: 0.8rem;
        border: 1px solid $modalColor;
        font-size: 1.2rem;
        color: $black;
        img {
          width: 1.4rem;
          margin-right: 0.9rem;
        }
        &-amount {
          margin-left: 1rem;
          font-weight: 600;
          font-size: 1.3rem;
          color: #63A60D;
        }
      }
      &-description {
        padding-top: 1.8rem;
        font-size: 1.4rem;
      }
      &-feeinfo {
        padding-top: 1.8rem;
        font-size: 1.3rem;
        color: $grayLight;
      }
    }
    &-content {
      margin: 0 -1.6rem;
      padding: 2.4rem 0;
      border-top: 1px solid $modalColor;
      text-align: center;
      font-size: 1.3rem;
      color: $grayDark;
    }
    &-buttons {
      display: flex;
      justify-content: flex-end;
      &-button {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 100%;
        background: $pink;
        padding: 1rem;
        margin-right: 1.6rem;
        border: 1px solid $pink;
        border-radius: 25px;
        text-align: center;
        font-size: 1.6rem;
        color: $white;
        &:last-child {
          margin: 0;
        }
        &.disabled {
          background: $white;
          border-color: $border;
          color: $border;
          pointer-events: none;
        }
        &.pending {
          background: $white;
          color: $pink;
          pointer-events: none;
        }
        &-loading {
          animation: loading 1s infinite;
          margin-left: 10px;
        }
      }
      &-box {
        display: flex;
        align-items: center;
        width: 100%;
        .modal__sell-buttons-button {
          flex: 1;
          &.btn-approve {
            margin-right: 1.6rem;
          }
        }
      }
    }
  }
  &__step {
    display: flex;
    align-items: center;
    margin-top: 2.4rem;
    padding: 0 13.7rem;
    &-status {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 24px;
      height: 24px;
      border: 2px solid $border2;
      border-radius: 50%;
      font-size: 1rem;
      color: $border2;
      &.active {
        border: 1px solid $pink;
        color: $pink;
        * {
          color: $pink;
        }
      }
      &.approved {
        border: 0;
      }
    }
    &-line {
      flex: 1;
      height: 2px;
      background: $border2;
      opacity: 0.65;
      &.active {
        background: linear-gradient(93.06deg,  #2CFF64 8.21%, #FC2EF5 100%);
        opacity: 1;
      }
    }
  }
}
@media screen and (max-width: 460px) {
  .modal {
    > div:first-child {
      margin: 0 .8rem;
    }
    &__block {
      &.sell {
        width: 100%;
        height: auto;
        padding: 0 .8rem 2.4rem !important;
        border-radius: 10px;
        overflow-y: auto;
        &.success {
          width: 23.5rem;
          padding: 1.8rem 1.8rem 2.6rem !important;
          .modal__sell {
            &-success-icon {
              padding: 2.2rem 0 1.4rem;
            }
            &-description, &-link {
              font-size: 1.4rem;
            }
          }
        }
      }
      &-container {
        text-align: center;
      }
    }
    &__sell {
      padding-top: 1.6rem;
      &-header {
        width: calc(100% + 1.6rem);
        height: 4rem;
        margin: 0 -0.8rem;
        &-button {
          top: 50%;
          right: 1.3rem;
          transform: translateY(-50%);
        }
      }
      &-info {
        .modal__image {
          width: 11.4rem;
        }
      }
      &-description {
        font-size: 1.6rem;
      }
      &-form {
        padding-bottom: 3.8rem;
        &-price {
          padding-bottom: .8rem;
          font-size: 1.4rem;
        }
        &-input {
          width: calc(100% - 2rem);
          padding: 1.2rem 1rem;
          border-color: $border2;
          font-size: 1.4rem;
        }
        &-unit {
          padding-top: 1.2rem;
        }
        &-description {
          padding-top: 1.6rem;
        }
        &-feeinfo {
          padding-top: 1.6rem;
          font-size: 1.2rem;
        }
      }
      &-content {
        width: 100%;
        padding: 1.6rem 0;
        margin: 0;
        font-size: 1.2rem;
      }
      &-buttons {
        &-button {
          width: 100%;
          &-cancel {
            display: none;
          }
        }
      }
    }
    &__step {
      padding: 0 4.5rem;
    }
  }
}
@media screen and (max-width: 460px) and (max-height: 630px) {
  .modal {
    &__sell {
      &-block {
        max-height: 65vh;
        overflow-x: hidden;
        overflow-y: auto;
        padding-right: 0.2rem;
        scrollbar-width: none;
        &::-webkit-scrollbar {
          display: none;
        }
      }
    }
  }
}
</style>
