<template>
  <div class="home">
    <b-modal :active.sync="openConfirm" has-modal-card trap-focus aria-role="dialog" aria-modal>
      <div class="modal-card" style="width: 650px; text-align:left">
        <header class="modal-card-head">
            <p class="modal-card-title">Create asset</p>
        </header>
        <section class="modal-card-body">
          <b>Name</b>
          : {{ asset.name }}
          <br />
          <b>Symbol</b>
          : {{ asset.symbol }}
          <br />
          <b>Supply</b>
          : {{ supply }} {{ asset.symbol }}
          <br />
          <b>Decimals</b>
          : {{ asset.decimals }}
          <br />
          <b>Reissuable</b>
          : {{ asset.reissuable }}
          <br />
          <b>Burnable</b>
          : {{ asset.burnable }}
          <br />
          <b>Permissioned</b>
          : {{ asset.permissioned }}
          <br />
          <br />
          <strong style="color:#f00">
            This operation will create this asset and will send the asset to your wallet:<br>{{ address }}.
            <br />Please make sure all the informations are correct.
          </strong>
          <br /><br>
          <b-button v-if="!isUploading" v-on:click="createSidechain" type="is-primary" size="is-large">CREATE</b-button>
        </section>
      </div>
    </b-modal>
    <div class="page-body">
      <div class="columns">
        <div class="column"></div>
        <div class="column is-two-thirds">
          <h1 style="text-align:center; margin:0">
            <strong>CREATE YOUR OWN DIGITAL ASSET</strong>
          </h1>
          <span v-if="wallet" style="font-size:13px">
            using
            <i>{{ address }}</i>
          </span>
          <br><br>
          <div class="columns">
            <div class="column is-four-fifths" style="text-align:left">
              <b-field label="Asset public name">
                <b-input v-model="asset.name" required></b-input>
              </b-field>
            </div>
            <div class="column">
              <b-field label="Symbol" style="text-align:left">
                <b-input v-model="asset.symbol" required></b-input>
              </b-field>
            </div>
          </div>

          <b-field label="Total issued supply" style="text-align:left">
            <b-input v-model="supply" required></b-input>
          </b-field>

          <p style="margin-top: 30px;"><strong>Set the decimals division</strong></p>
          <range-slider class="slider" style="width:100%" min="0" max="8" v-on:change="fixStep" step="1" v-model="asset.decimals"></range-slider>
          {{ asset.decimals }}

          <div style="text-align:left">
            <div class="field">
              <b-checkbox v-model="asset.reissuable">Reissuable</b-checkbox>
              <p>You can set your asset as "reissuable" (owner can add any amount of supply also after the asset is issued)-</p>
            </div>
            <br>
            <div class="field">
              <b-checkbox v-model="asset.burnable">Burnable</b-checkbox><br>
              <p>You can set your asset as "burnable" (anyone can destroy a portion or the whole supply of the asset sending assets to a proper burn address).</p>
            </div>
            <br>
            <div class="field">
              <b-checkbox v-model="asset.permissioned">Permissioned</b-checkbox><br>
              <p>You can set your asset as "permissioned" (all users must be enabled to operate by the owner or a validator, only you will be able to create validators).</p>
            </div>
          </div>
          <div v-if="wallet">
            <div style="text-align:center; margin-top: 25px; margin-bottom: 25px;" v-if="!issueResponse">
              <b-button
                type="is-primary"
                size="is-large"
                v-if="!isUploading"
                v-on:click="showConfirm"
              >CONFIRM CONFIGURATION</b-button>
              <span v-if="isUploading">Creating asset, please wait..</span>
            </div>
          </div>
          <div v-if="!wallet" style="font-size:18px; width:100%; display:block; margin-top:15px">
              <b-notification
                type="is-danger"
                has-icon
                aria-close-label="Close notification"
                role="alert">
                You must login with your ID BDCash to create a Sidechain.<br>
                <a href="/#/login"> 
                  <b-button size="sm" class="btn-warning my-2 my-sm-0">LOGIN NOW</b-button>
                </a>
            </b-notification>
          </div>
        </div>
        <div class="column"></div>
      </div>
    </div>
    <b-loading :is-full-page="true" :active.sync="isLogging" :can-cancel="false"></b-loading>
  </div>
</template>

<script>
  import RangeSlider from "vue-range-slider";
  import "vue-range-slider/dist/vue-range-slider.css";
  let BDCashCore = require("@bdcash-protocol/core");
  let math = require("mathjs")

  export default {
    name: "create",
    mounted: async function() {
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
      } else {
        app.isLogging = false;
      }
    },
    watch: {
      supply: function () {
        const app = this
        setTimeout(function(){
          app.fixStep()
        },1000)
      }
    },
    data() {
      return {
        bdcash: new BDCashCore(true),
        math: math,
        address: "",
        wallet: { address: "", wallet: ""},
        sidechains: [],
        step: 0,
        isLogging: true,
        openConfirm: false,
        isLoading: true,
        issueResponse: "",
        passwordShow: false,
        public_address: "",
        isUploading: false,
        encrypted_wallet: "",
        nodesh: "",
        unlockPwd: "",
        user: "",
        supply: 0,
        asset: {
          name: "",
          symbol: "",
          decimals: 0,
          burnable: false,
          reissuable: false,
          extendable: false,
          permissioned: false
        }
      };
    },
    components: {
      RangeSlider
    },
    methods: {
      showConfirm(){
        const app = this
        if(app.asset.name !== "" && app.asset.symbol !== "" && app.supply > 0 && app.supply <= 1000000000000000){
          if(app.asset.symbol.length <= 5){
            app.openConfirm = true
          }else{
            this.$buefy.toast.open({
                message: 'Symbol is too long.',
                type: 'is-danger'
            })
          }
        }else{
          this.$buefy.toast.open({
              message: 'Please write all required fields.',
              type: 'is-danger'
          })
          if(app.supply >= 1000000000000000){
            this.$buefy.toast.open({
              message: 'You can\'t issue more than 1Quadrillion token at time!',
              type: 'is-danger'
            })
          }
        }
      },
      fixStep(){
        const app = this
        let decimals = app.asset.decimals
        let supply = app.supply.toString()
        if(supply.indexOf(',') !== -1){
          supply = app.supply.replace(',','.')
        }
        supply = parseFloat(supply)
        app.supply = app.math.round(supply, decimals)
      },
      createSidechain() {
        const app = this;
        app.bdcash.staticnodes=true
        app.$buefy.dialog.prompt({
          message: `Enter wallet password`,
          inputAttrs: {
            type: "password"
          },
          trapFocus: true,
          onConfirm: async password => {
            let key = await app.bdcash.readKey(password, app.wallet.wallet);
            if (key !== false) {
              app.isUploading = true;
              let resIssue = await app.bdcash.post(
                "/sidechain/issue",
                {
                  dapp_address: app.wallet.address,
                  private_key: key.prv,
                  pubkey: key.key,
                  version: 1,
                  name: app.asset.name,
                  symbol: app.asset.symbol,
                  supply: parseFloat(app.supply),
                  decimals: parseFloat(app.asset.decimals),
                  reissuable: app.asset.reissuable,
                  burnable: app.asset.burnable,
                  permissioned: app.asset.permissioned,
                  extendable: false
                }
              );
              if (
                resIssue.issued === true ||
                resIssue.issued === "true"
              ) {
                this.$buefy.toast.open({
                    message: 'Asset issued correctly!',
                    type: 'is-success'
                })
                setTimeout(function(){
                  window.location = '/#/'
                },500)
              }
            }
          }
        });
      }
    }
  };
</script>

<style>
  .node-badge {
    position: fixed;
    bottom: -3px;
    font-size: 10px;
    padding: 8px;
    right: 10px;
    z-index: 9999;
    border: 1px solid rgb(201, 171, 6);
    border-top-left-radius: 4px;
    border-top-right-radius: 4px;
  }
</style>