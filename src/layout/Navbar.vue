<template>
<!-- Main navigation 424f95 -->
<header>
<login-tab-modal v-if="showModal" :modal="showModal" @closeModal="closeModal"/>
<nav class="navbar navbar-expand-lg py-0">

  <!-- mdbNavbar brand -->
  <span class="d-inline-block mr-3">
    <router-link to="/" class="navbar-brand d-inline-block" ><img :src="logo" height="40px" alt="radicle logo"></router-link>
  </span>
  <button id="btn-toggle" @click.prevent="toggleNav" class="navbar-toggler toggler-example black darken-3" type="button"
      data-toggle="collapse"
      data-target="#navbarSupportedContent"
      aria-controls="navbarSupportedContent"
      aria-expanded="false"
      aria-label="Toggle navigation">
      <span class="white-text"><i class="fas fa-bars fa-1x"></i></span>
  </button>

  <div :class="toggleClass" class="navbar-collapse collapse" id="navbarSupportedContent">
    <!--  <span class="dark-blue-text"><i class="fas fa-bars fa-1x"></i></span>
    <form class="md-form search-form ml-2" @submit.prevent="">
      <a type="button" @click.prevent="doSearch"><mdb-icon class="" icon="search" /></a>
      <input label="Search" type="text" class="mt-0 mb-0 ml-1" v-model="query" placeholder="Search" aria-label="Search" v-on:keyup.13="doSearch"/>
    </form>
    -->
    <ul class="navbar-nav ml-auto">
      <li class="nav-item" @click="closeMenu"><router-link class="nav-link navbar-link" to="/gallery">Gallery</router-link></li>
      <li class="nav-item" @click="closeMenu"><router-link class="nav-link navbar-link" to="/artists">Artists</router-link></li>
      <li v-if="featureAuctions" class="nav-item" @click="closeMenu"><router-link class="nav-link navbar-link" to="/online-auctions">Auctions</router-link></li>
      <upload-links v-if="loggedIn" @closeMenu="closeMenu"/>
      <li v-if="loggedIn && hasInvoices" class="nav-item" @click="closeMenu"><router-link class="nav-link navbar-link" to="/orders">Cart</i><sup><mdb-badge color="danger-color" class="ml-2">{{hasInvoices}}</mdb-badge></sup></router-link></li>
      <li class="nav-item" @click="closeMenu"><router-link to="/search"><mdb-icon icon="search" class="teal-text mt-3 mx-2" style="line-height: 10px;" aria-hidden="true" size="2x"/></router-link></li>
      <!-- <li class="nav-item" @click="closeMenu"><router-link v-if="!loggedIn" class="nav-link navbar-link" to="/login">Login</router-link></li> -->
      <portfolio-links v-if="loggedIn" @closeMenu="closeMenu"/>
      <account-links v-if="loggedIn" @closeMenu="closeMenu"/>
      <li class="nav-item" @click="closeMenu"><a v-on:click.prevent="loginMultiPlayer" v-if="!loggedIn" class="nav-link navbar-link">Login with Blockstack</a></li>
    </ul>
  </div>

</nav>
<!-- Main navigation -->
</header>
</template>

<script>
let resizeTimeout;
function resizeThrottler(actualResizeHandler) {
  // ignore resize events as long as an actualResizeHandler execution is in the queue
  if (!resizeTimeout) {
    resizeTimeout = setTimeout(() => {
      resizeTimeout = null;
      actualResizeHandler();

      // The actualResizeHandler will execute at a rate of 15fps
    }, 66);
  }
}

import { mdbBadge, mdbContainer, mdbIcon, mdbRow, mdbCol, mdbNavbar, mdbNavbarToggler, mdbNavbarNav, mdbNavItem, mdbInput, mdbBtn, mdbNavbarBrand } from 'mdbvue';
import { mdbDropdown, mdbDropdownItem, mdbDropdownMenu, mdbDropdownToggle } from 'mdbvue';
import AccountLinks from "@/layout/AccountLinks";
import PortfolioLinks from "@/layout/PortfolioLinks";
import UploadLinks from "@/layout/UploadLinks";
import artworkSearchService from "@/services/artworkSearchService";
import myAccountService from '../services/myAccountService';
import LoginTabModal from "@/views/components/utils/LoginTabModal";

export default {
  name: 'Navbar',
  data() {
    return {
      query: '',
      link1: "team",
      link2: "mission",
      link3: "contact",
      link4: "donate",
      title: "title",
      taglink2: '',
      toggleClass: '',
      showModal: false,
      logo: require("@/assets/img/logo/Radicle_logo-test.png")
    };
  },
  components: {
    LoginTabModal,
    UploadLinks,
    PortfolioLinks,
    AccountLinks,
    mdbContainer,
    mdbRow,
    mdbCol,
    mdbNavbar,
    mdbNavbarToggler,
    mdbNavbarNav,
    mdbNavItem,
    mdbInput,
    mdbBtn,
    mdbBadge,
    mdbIcon,
    mdbNavbarBrand,
    mdbDropdown,
    mdbDropdownItem,
    mdbDropdownMenu,
    mdbDropdownToggle
  },
  created() {
    this.getContent();
    window.addEventListener('resize', this.handleResize)
  },
  beforeDestroy: function () {
    window.removeEventListener('resize', this.handleResize)
  },
  computed: {
    featureAuctions() {
      return this.$store.state.constants.featureAuctions;
    },
    loggedIn() {
      let myProfile = this.$store.getters["myAccountStore/getMyProfile"];
      return myProfile.loggedIn;
    },
    hasInvoices() {
      let invoices = this.$store.getters["invoiceStore/getInvoices"];
      if (!invoices) {
        return 0;
      }
      return invoices.length;
    }
  },
  methods: {
    getContent() {
      /**
      this.$prismic.client.getSingle("navbar").then(document => {
        this.link1 = document.data.link1[0].text;
        this.link2 = document.data.link2[0].text;
        this.link3 = document.data.link3[0].text;
        this.link4 = document.data.link4[0].text;
        this.title = document.data.title[0].text;
        this.taglink2 = document.data.taglink2[0].text;
      });
      **/
    },
    handleResize (event) {
      this.fullWidth = document.documentElement.clientWidth;
      if (this.fullWidth > 1200) {
        this.toggleClass = "";
        document.getElementById("navbarSupportedContent").style.display = "block";
      } else {
        document.getElementById("navbarSupportedContent").style.display = "none";
      }
      console.log("width: " + this.fullWidth);
    },
    loginMultiPlayer: function () {
      let res = myAccountService.loginMultiPlayer();
      this.$router.push({ path: "/" });
      this.showModal = true;
      return res;
    },
    closeMenu() {
      //this.toggleClass = "";
      this.toggleNav();
    },
    closeModal: function() {
      this.showModal = false;
    },
    toggleNav() {
      let togglee = document.getElementById("navbarSupportedContent");
      if (this.toggleClass === "show") {
        this.toggleClass = "";
        togglee.style.display = "none";
      } else {
        this.toggleClass = "show";
        togglee.style.display = "";
        togglee.style.display = "block";
        //document.getElementById("navbarSupportedContent").attr("style", "display: block !important");
      }
    },
    doSearch() {
      let qString = this.query;
      if (!this.query || this.query.length === 0) {
        qString = "*";
      }
      artworkSearchService.newQuery({field: "title", query: qString});
      this.$router.push("/search?query=" + qString);
    },
    scrollListener() {
      resizeThrottler(this.handleScroll);
    },
    logout() {
      localStorage.clear();
      sessionStorage.clear();
      myAccountService.logout();
    }
  },
  mounted() {
  },
  beforeDestroy() {
  },
  watch: {
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.navbar {
  box-shadow: none;
  min-height: 60px;
}
.navbar-link {
  text-transform: capitalize;
  font-size: 1.3em;
  font-weight: 900;
}
.navbar-toggler-icon {
  background-color: blue;
}
.navbar .dropdown-menu a {
  font-weight: 900;
}
@media (max-width: 1200px) {
    .navbar {
      justify-content: space-between;
    }
    .navbar-expand-lg .navbar-toggler {
        display: block;
    }
    .navbar-header {
        float: none;
    }
    .navbar-toggler {
        display: block;
        float: right;
    }
    .navbar-collapse {
        border-top: 1px solid transparent;
        box-shadow: inset 0 1px 0 rgba(255,255,255,0.1);
    }
    .navbar-collapse.collapse {
      display: none;
    }
    .navbar-nav {
        float: none!important;
        margin: 7.5px -15px;
    }
    .navbar-nav>li {
        float: none;
    }
    .navbar-nav>li>a {
        padding-top: 10px;
        padding-bottom: 10px;
    }
}

.show {
  display: block;
  position: absolute;
  right: 0px;
  left: 0px;
  top: 60px;
  bottom: 0px;
  background-color: #4EAC9A;
  color: white;
  font-weight: 900;
  z-index: 10;
  padding: 10px 0 10px 30px;
  min-height: calc(100vh);
}

.navbar .md-form {
  margin: 0;
}

i.fa-search {
  color: gray;
  transform: rotate(90deg);
  font-size: 1.3em;
}
.search-form input, .login-link, .nav-cta { font-size: 1.0em; font-weight: normal; }
.search-form input { border-bottom: none; }
.search-form input:focus { box-shadow: none!important; border-bottom: 1pt solid gray!important; }

.search-form input::-webkit-input-placeholder { /* Chrome/Opera/Safari */
  color: grey;
  font-size: 1.3em;
  text-transform: capitalize;
}
.search-form input::-moz-placeholder { /* Firefox 19+ */
  color: grey;
  font-size: 1.3em;
  text-transform: capitalize;
}
.search-form input:-ms-input-placeholder { /* IE 10+ */
  color: grey;
  font-size: 1.3em;
  text-transform: capitalize;
}
.search-form input:-moz-placeholder { /* Firefox 18- */
  color: grey;
  font-size: 1.3em;
  text-transform: capitalize;
}
.search-form button.transparent { border: none; background: transparent; color: black; }


</style>
