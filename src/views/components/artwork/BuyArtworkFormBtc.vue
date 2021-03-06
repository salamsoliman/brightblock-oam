<template>
<div class="pt-1">
  <confirmation-modal v-if="showModal" :modal="showModal" :title="modalTitle" :content="modalContent" @closeModal="closeModal"/>
  <login-info-modal v-if="showLoginInfoModal" :modal="showLoginInfoModal" @closeLoginInfoModal="closeLoginInfoModal"/>
  <div>
    <div>
      {{artwork.saleData.amount}} EUR
      <br/><strong>{{btcMessage}} BTC</strong>
    </div>
    <div class="">
      <div class="my-3" v-if="!showModal">

        <!-- i am owner - send this user to my artwork / my artworks -->
        <router-link v-if="iamowner" :to="myArtworkUrl" class="inline-block">
          <mdb-btn class="btn teal lighten-1" size="sm" waves-light>manage artwork</mdb-btn>
        </router-link>

        <!-- order placed / purchase in progress - user has placed an order for this artwork or some other user? -->
        <div v-else-if="purchaseBegun">

          <router-link v-if="iambuyer" :to="invoiceUrl">
            <mdb-btn class="btn teal lighten-1 btn-lg">proceed to checkout</mdb-btn>
          </router-link>

          <mdb-btn v-else class="btn teal lighten-1 btn-lg" size="sm" waves-light :disabled="true">sold</mdb-btn>

          <router-link to="/gallery">
            <mdb-btn class="btn teal lighten-1 btn-lg">continue browsing</mdb-btn>
          </router-link>

        </div>

        <!-- purchase not in progress - user is free to place order? -->
        <mdb-btn v-else class="btn teal lighten-1 btn-lg m-0" @click="buyArtwork()">buy</mdb-btn>

        <p class="text-muted"><small class="text-muted">{{registerMessageBtc}}</small></p>
      </div>
      <div class="col-12 mt-3" v-else>
        <p>please wait - transferring to payment..</p>
      </div>
    </div>
    <div class="w-100"></div>
  </div>
</div>
</template>

<script>
import { mdbIcon, mdbBtn } from 'mdbvue';
import moneyUtils from "@/services/moneyUtils";
import { mdbContainer, mdbCol, mdbRow } from 'mdbvue';
import ConfirmationModal from "@/views/components/utils/ConfirmationModal";
import LoginInfoModal from "@/views/components/utils/LoginInfoModal";

// noinspection JSUnusedGlobalSymbols
export default {
  name: "BuyArtworkFormBtc",
  components: {
    ConfirmationModal,
    LoginInfoModal,
    mdbIcon,
    mdbBtn,
    mdbContainer, mdbCol, mdbRow
  },
  props: {
    purchaseState: {},
    myProfile: {},
    artwork: {
      type: Object,
      default() {
        return {
        };
      },
    }
  },
  data() {
    return {
      orderPlaced: false,
      showModal: false,
      showLoginInfoModal: false,
      modalTitle: "Transferring..",
      modalContent: "<p>Redirecting to payment department.</p>",
    };
  },
  methods: {
    buyArtwork() {
      if (this.myProfile.loggedIn) {
        this.showModal = true;
        this.$store.dispatch("invoiceStore/prepareNewInvoice", {artwork: this.artwork, saveInvoice: true}).then(invoice => {
          this.$router.push("/orders?orderId=" + invoice.invoiceId);
        })
      } else {
        this.showLoginInfoModal = true;
      }
    },
    closeModal: function() {
      this.showModal = false;
    },
    closeLoginInfoModal: function() {
      this.showLoginInfoModal = false;
    }
  },
  computed: {
    registerMessageBtc() {
      let artwork = this.artwork;
      let message;
      try {
        let registered = artwork.bitcoinTx;
        let price = artwork.saleData.amount > 0;
        if (!registered) {
          message = "Artwork not registered on blockchain."
        } else {
          if (price) {
            message = "Artwork is registered on blockchain."
          } else {
            message = "Artwork is registered on chain but is not currently for sale."
          }
        }
      } catch (e) {
          message = "Unregistered.";
      }
      return message;
    },
    myArtworkUrl() {
      return `/my-artworks/${this.artwork.id}`;
    },
    iamowner() {
      let profile = this.$store.getters["myAccountStore/getMyProfile"];
      return profile.username === this.artwork.owner;
    },
    iambuyer() {
      let invoice = this.$store.getters["invoiceStore/getInvoiceById"](this.artwork.id);
      let profile = this.$store.getters["myAccountStore/getMyProfile"];
      return invoice || profile.username === this.artwork.buyer;
    },
    invoiceUrl() {
      return `/orders?orderId=${this.artwork.id}`;
    },
    purchaseBegun() {
      let purchaseBegun = this.artwork.status === this.$store.state.constants.statuses.artwork.PURCHASE_BEGUN;
      let invoice = this.$store.getters["invoiceStore/getInvoiceById"](this.artwork.id);
      return invoice || purchaseBegun;
    },
    moneySymbol() {
      try {
        return moneyUtils.currencySymbol(this.artwork.saleData.fiatCurrency);
      } catch (e) {
        return "";
      }
    },
    btcMessage() {
      try {
        let value = moneyUtils.valueInBitcoin(this.artwork.saleData.fiatCurrency, this.artwork.saleData.amount);
        return value;
      } catch (e) {
        return "";
      }
    },
    label() {
      try {
        let myProfile = this.$store.getters["myAccountStore/getMyProfile"];
        if (this.artwork && myProfile.username === this.artwork.owner) {
          return "Payment Info";
        }
        let value = moneyUtils.valueInBitcoin(this.artwork.saleData.fiatCurrency, this.artwork.saleData.amount);
        return "BUY NOW " + value + " BTC";
      } catch (e) {
        console.log("Error formatting buy label: " + e);
        return "Buy Now";
      }
    }
  }
};
</script>
