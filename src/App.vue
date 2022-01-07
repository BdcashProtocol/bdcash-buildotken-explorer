<template>
  <div id="app">
    <b-navbar style="position:fixed; top: 0; left:0; width:100%; border-bottom:1px solid #ddd;">
      <template slot="brand">
        <b-navbar-item tag="router-link" :to="{ path: '/' }">
          <img src="/logo.png" />
        </b-navbar-item>
      </template>
      <template slot="start">
        <b-navbar-item href="/#/">Explorer tokens</b-navbar-item>
        <b-navbar-item v-if="wallet" href="/#/create">Create your token</b-navbar-item>
        <b-navbar-item v-if="wallet" href="/#/manage">Manage your token</b-navbar-item>
        <b-navbar-item href="https://docs.bdcashprotocol.com/" target="_blank">Docs</b-navbar-item>
      </template>

      <template slot="end">
        <b-navbar-item tag="div" v-if="wallet">
          <div class="buttons">
            <a :href="'/#/address/' + address">
              <v-gravatar :email="address" style="margin-right: 10px; height: 80px;max-height: 37px;float: right;margin-top: -8px;border-radius: 4px;"/>
            </a>
            <a v-on:click="logout" class="button is-primary">
              <strong>Logout</strong>
            </a>
          </div>
        </b-navbar-item>
        <b-navbar-item tag="div" v-if="!wallet">
          <div class="buttons">
            <a href="/#/login" class="button is-primary">
              <strong>Login</strong>
            </a>
          </div>
        </b-navbar-item>
      </template>
    </b-navbar>
    <router-view style="padding-top: 80px;" />
    <hr />Devs Chain Platform open-source project by
    <a href="https://bdcashprotocol.com" target="_blank">BDCash Protocol</a>.
    <br />
    <br />
    <b-loading :is-full-page="true" :active.sync="isLogging" :can-cancel="false"></b-loading>
  </div>
</template>

<style>
  .media-content{
    text-align:left!important;
  }
  .table th{
    padding:10px 0 10px 0 !important;
  }
  .table td{
    padding:10px 0 5px 0!important;
    font-size:12px;
    text-align:left!important;
  }
  .table td button{
    width:100%;
    margin-top:-5px;
  }
  @media screen and (max-width: 767px){
    .table th{
      padding:10px !important;
    }
    .table td{
      padding:10px 10px 5px 10px!important;
      font-size:12px;
      text-align:left!important;
    }
  }
</style>
<script>
let BDCashCore = require("@bdcash-protocol/core");

export default {
  data() {
    return {
      bdcash: new BDCashCore(true),
      address: "",
      wallet: "",
      isLogging: true,
      file: [],
      isCreating: false,
      isUpdating: false,
      showCreateModal: false,
      password: "",
      passwordrepeat: ""
    };
  },
  async mounted() {
    const app = this;
    app.wallet = await app.bdcash.importBrowserSID();
    app.wallet = await app.bdcash.returnDefaultIdentity();
    if (app.wallet.length > 0) {
      let SIDS = app.wallet.split(":");
      app.address = SIDS[0];
      let identity = await app.bdcash.returnIdentity(app.address);
      app.wallet = identity;
      app.isLogging = false;
    } else {
      app.isLogging = false;
    }
  },
  methods: {
    loadWalletFromFile() {
      const app = this;
      const file = app.file;
      const reader = new FileReader();
      reader.onload = function() {
        var dataKey = reader.result;

        app.$buefy.dialog.prompt({
          message: `Enter wallet password`,
          inputAttrs: {
            type: "password"
          },
          trapFocus: true,
          onConfirm: async password => {
            let key = await app.bdcash.readKey(password, dataKey);
            if (key !== false) {
              app.bdcash.importPrivateKey(key.prv, password);
              localStorage.setItem("SID", dataKey);
              location.reload();
            } else {
              app.$buefy.toast.open({
                message: "Wrong password!",
                type: "is-danger"
              });
            }
          }
        });
      };
      reader.readAsText(file);
    },
    showCreate() {
      const app = this;
      app.showCreateModal = true;
    },
    logout() {
      localStorage.setItem("SID", "");
      window.location = '/#/'
    },
    async createUser() {
      const app = this;
      if (app.password !== "") {
        if (app.passwordrepeat === app.password) {
          app.isCreating = true;
          setTimeout(async function() {
            let id = await app.bdcash.createAddress(app.password, true);
            let identity = await app.bdcash.returnIdentity(id.pub);
            app.address = id.pub;
            app.wallet = identity;
            localStorage.setItem("SID", id.walletstore);
            app.showCreateModal = false;
            app.password = "";
            app.passwordrepeat = "";
            let tx = await app.bdcash.post("/init", {
              address: id.pub,
              airdrop: true
            });
            if (tx.airdrop_tx === false) {
              app.$buefy.toast.open({
                message: "Sorry, airdrop was not successful!",
                type: "is-danger"
              });
            }
            app.isCreating = false;
          }, 500);
        } else {
          app.$buefy.toast.open({
            message: "Passwords doesn't matches.",
            type: "is-danger"
          });
        }
      } else {
        app.$buefy.toast.open({
          message: "Write a password first!",
          type: "is-danger"
        });
      }
    }
  }
};
</script>

<style>
  #app {
    font-family: "Sen";
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #000000;
  }
  .home{
    padding:20px;
  }
  .is-140x140{
    width:140px;
    height:140px;
  }
  #nav {
    padding: 30px;
    background-color:#eca50b;
  }

  #nav a {
    font-weight: bold;
    color: #ffffff;
    background-color:#eca50b;
  }

  #nav a.router-link-exact-active {
    color: #ffffff;
    background-color:#eca50b;
  }
</style>