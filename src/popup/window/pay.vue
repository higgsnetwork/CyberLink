<template>
  <div class="main-wrapper">
    <main-header @network-change="handleNetworkChange" :hide-menu-toggle="true"></main-header>
    <div class="content">
      <div class="page-title">{{ $t('window.transfer.title') }}</div>
      <div class="value-block">
        <div class="value">{{ value }}</div>
        <div class="desc" v-if="desc">{{ desc }}</div>
      </div>
      <div class="cells">
        <div class="cell">
          <label>{{ $t('window.transfer.addressLabel') }}</label>
          <span>{{ address }}</span>
        </div>
        <div class="cell">
          <label>{{ $t('window.transfer.destAddressLabel') }}</label>
          <span>{{ destAddress }}</span>
        </div>
        <div class="cell">
          <label>{{ $t('window.transfer.valueLabel') }}</label>
          <span>{{ value }} ELA</span>
        </div>
      </div>
    </div>
    <div class="footer">
      <fees-slider v-model="fees"></fees-slider>
      <div class="button-wrapper">
        <button @click="cancel">{{ $t('window.transfer.closeButton') }}</button>
        <button class="btn-primary" @click="onlyRaw ? confirmRaw() : confirm()">{{ $t('window.transfer.confirmButton') }}</button>
      </div>
    </div>
  </div>
</template>

<script>
import MainHeader from "../account/components/main-header";
import Main from "../account/main";
import FeesSlider from "../components/fees-slider";
import API from "../api";
import formatError from "../api/format-error";
import WindowMixin from "./mixin";
import axios from "axios";

export default {
  name: "contract",

  mixins: [WindowMixin],

  components: {
    Main,
    MainHeader,
    FeesSlider
  },

  created() {
    const query = this.$router.currentRoute.query;
    this.destAddress = query.destAddress;
    this.desc = query.desc;
    this.callbackId = query.callbackId;
    this.value = parseFloat(query.value) || 0;

    this.onlyRaw = query.onlyRaw;
  },
  mounted() {
    this.address = JSON.parse(localStorage.getItem('address'));
    this.mnemonic = JSON.parse(localStorage.getItem("mnemonic"));
    this.getBalance(this.address);
  },
  methods: {
    confirmRaw() {
      this.$loading(this.$t("window.transfer.confirmLoading"));

      API.callRaw("sendRaw", {
        network: this.network,
        address: this.address,
        destAddr: this.destAddress,
        value: this.value,
        fees: this.fees,
        desc: this.desc
      }).then(
        value => {
          this.$loading.close();
          this.$toast(this.$t("window.transfer.createSuccess"), {
            type: "center"
          });

          if (this.callbackId) {
            API.callPageCallback(this.callbackId, null, value);
          }

          setTimeout(() => {
            window.close();
          }, 300);
        },
        error => {
          this.$loading.close();
          this.$toast(
            this.$t("window.transfer.createFailure") + " " + formatError(error),
            {
              type: "center",
              duration: 5000,
              wordWrap: true
            }
          );

          if (this.callbackId) {
            API.callPageCallback(this.callbackId, error, null);
          }

          console.log(error);
        }
      );
    },
    //获取余额
    getBalance(address) {
      axios
        .get("https://api-wallet-ela.elastos.org/api/1/balance/" + this.address)
        .then(res => {
          this.balances = res.data.result;
          localStorage.setItem("balance", this.balances);
        })
        .catch(res => {
          console.log(res);
        });
    },
    confirm() {
     
      this.$loading(this.$t("window.transfer.confirmLoading"));

      if (this.value < 0.0001) {
        this.$toast(this.$t("errors.amountLessThanLimit"), {
          type: "center"
        });

        return;
      }

      if (this.balance && this.value > this.balance) {
        this.$toast(this.$t("errors.insufficientBalance"), {
          type: "center"
        });
        return;
      }

  
      let seed = window.getSeedFromMnemonic(this.mnemonic);
      let privateKey = window.getSinglePrivateKey(seed);
      let endpoint = "https://api-wallet-ela.elastos.org";
      let from = this.address;
      let to = this.destAddress;
      let num = this.value * 100000000;
      let tx = window.Transaction();


      try {
          tx.createTx(endpoint, from, to, num);
          tx.generateRawTransaction([privateKey]);
          let bktxt = tx.sendRawTx(endpoint);
          this.$toast(this.$t("window.transfer.createSuccess"), {
            type: "center"
          });
          if (this.callbackId) {
            API.callPageCallback(this.callbackId, null, bktxt);
          }
          this.$loading.close();
          setTimeout(() => {
            window.close();
          }, 300);
        } catch (error) {
          this.$loading.close();
          this.$toast(
            this.$t("window.transfer.createFailure") + " " + formatError(error),
            {
              type: "center",
              duration: 5000,
              wordWrap: true
            }
          );
          API.callPageCallback(this.callbackId, error, null);
      }

    }
  },

  data() {
    return {
      destAddress: null,
      address: '',
      desc: null,
      value: 0,
      balances: '',
      fees: 0.0001,
      mnemonic: ''
    };
  }
};
</script>

<style lang="scss" scoped>
@import "./common.scss";

.value-block {
  padding-bottom: 10px;
  background-color: #f2f5fc;

  .value {
    font-size: 38px;
    color: #3c78ea;
    margin-top: 10px;
    text-align: center;
    background: #fff;
  }

  .desc {
    color: #b4bccc;
    font-size: 15px;
    text-align: center;
    background-color: #fff;
    margin-bottom: 10px;
  }
}
</style>