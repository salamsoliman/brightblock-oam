<template>
  <mdb-dropdown>
    <mdb-dropdown-toggle tag="a" navLink color="" slot="toggle"  waves-fixed>
      <img class="avatar" :src="avatarUrl"/>
      <!--<mdb-icon icon="user-circle" />-->
    </mdb-dropdown-toggle>

    <mdb-dropdown-menu class="dropdown-menu-right" id="userMainMenu" >
      <mdb-dropdown-item tag="div">
       <strong>{{ username }}</strong>
      </mdb-dropdown-item>

      <mdb-dropdown-item>
        <router-link class="dropdown-item" to="/profile/update"><a @click="closeMenu">Settings</a></router-link>
      </mdb-dropdown-item>

      <mdb-dropdown-item>
        <router-link class="dropdown-item" to="/admin/settings" v-if="showAdmin"><a @click="closeMenu">Admin</a></router-link>
      </mdb-dropdown-item>

      <mdb-dropdown-item>
        <a href="#" class="dropdown-item" @click.prevent="logout">Logout</a>
      </mdb-dropdown-item>

    </mdb-dropdown-menu>
  </mdb-dropdown>

</template>

<script>
import { mdbBadge, mdbNavbar, mdbNavItem, mdbNavbarNav, mdbNavbarToggler, mdbBtn, mdbContainer, mdbCollapse, mdbDropdown, mdbDropdownItem, mdbDropdownMenu, mdbDropdownToggle, mdbInput, mdbNavbarBrand, mdbIcon, mdbRow } from 'mdbvue';
import myAccountService from "@/services/myAccountService";

export default {
  name: 'AccountLinks',
  components: {
    mdbNavbar,
    mdbNavItem,
    mdbNavbarNav,
    mdbNavbarToggler,
    mdbBtn,
    mdbContainer,
    mdbDropdown,
    mdbCollapse,
    mdbDropdownItem,
    mdbDropdownMenu,
    mdbDropdownToggle,
    mdbInput,
    mdbNavbarBrand,
    mdbBadge,
    mdbIcon,
    mdbRow
  },
  data() {
    return {
      showSearch: false,
      query: null
    };
  },
  mounted() {
  },
  computed: {
    showAdmin() {
      return this.$store.state.myAccountStore.myProfile.showAdmin;
    },
    hasGalleries() {
      let galleries = this.$store.getters["galleryStore/getMyGalleries"];
      if (!galleries || !galleries.records) {
        return 0;
      }
      return galleries.records.length;
    },
    username() {
      return this.$store.state.myAccountStore.myProfile.name;
    },
    avatar() {
      let myProfile = this.$store.getters["myAccountStore/getMyProfile"];
      if (myProfile.loggedIn) {
        return (
          '<img style="width: 30px; height: 30px; border-radius: 30px;" src="' +
          myProfile.avatarUrl +
          '"/>'
        );
      } else {
        return '<span class="icon-user"></span>';
      }
    },
    avatarUrl() {
      let myProfile = this.$store.getters["myAccountStore/getMyProfile"];
      return myProfile.avatarUrl;
    },
    loggedIn() {
      let myProfile = this.$store.getters["myAccountStore/getMyProfile"];
      return myProfile.loggedIn;
    }
  },
  methods: {
    logout() {
      localStorage.clear();
      sessionStorage.clear();
      myAccountService.logout();
      this.$router.push("/");
      this.$emit("closeMenu");
    },
    closeMenu() {
      this.$emit("closeMenu");
    }
  }
};

</script>
<style scoped>
.avatar {
  width: 25px;
  height: 25px;
  border-radius: 25px;
}
.nav-link {
  text-transform: capitalize;
  font-size: 1.3em;
  font-weight: 900;
}
.dropdown-menu {
  min-width: 200px;
  border: none;
  top: 11px;
  border-radius: 0;
  margin-top: -5px;
}

.btn.dropdown-toggle { color: black; }
.dropdown .dropdown-menu .dropdown-item,
.dropdown .dropdown-menu .dropdown-item:hover {
  background: initial;
  box-shadow: none;
  color: initial!important;
}

.dropdown-submenu {
  position:relative;
}

.dropdown-submenu .dropdown-menu {
  top:0;
  left:100%;
}

.dropdown-item {
  font-size: 0.9375rem;
  font-weight: 900;
}
.navbar .dropdown-menu a {
  font-weight: 900;
}

.dropdown-item > a {
  padding: 0;
}

.fa-user-circle { font-size: 1.7rem; padding-bottom: 4px;}

</style>
