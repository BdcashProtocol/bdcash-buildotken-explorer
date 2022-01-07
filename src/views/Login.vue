<template>
  <div id="app">
      <section class="hero">
        <div class="hero-body" style="padding: 0;">
          <div class="container" id="create" style="margin-top:50px;">
            <div class="card">
              <div style="padding: 50px 20px;">
                <h1 class="title is-1">Chain Devs Platform</h1>
                <h2 class="subtitle">
                  <img src="img/logo.png" class="img-responsive img-fluid">
                  <br />Enter with ID BDCash Extension or create a <a href="https://web.bdcashprotocol.com" target="_blank">new account here</a>.
                  <br />
                  <br />
                  <b-upload v-model="file" v-on:input="loadWalletFromFile" drag-drop>
                    <section class="section">
                      <div class="content has-text-centered">
                        <p>Drag and drop here or click to upload</p>
                      </div>
                    </section>
                  </b-upload>
                </h2>
              </div>
            </div>
          </div>
        </div>
      </section>
    <b-loading :is-full-page="true" :active.sync="isLogging" :can-cancel="false"></b-loading>
  </div>
</template>

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
    app.bdcash.staticnodes = true;
    app.wallet = await app.bdcash.importBrowserSID();
    app.wallet = await app.bdcash.returnDefaultIdentity();
    if (app.wallet.length > 0) {
      let SIDS = app.wallet.split(":");
      app.address = SIDS[0];
      let identity = await app.bdcash.returnIdentity(app.address);
      app.wallet = identity;
      app.isLogging = false;
      window.location = '/#/create'
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
              setTimeout(function(){
                location.reload()
              },20)
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
      location.reload();
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
