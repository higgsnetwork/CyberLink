<template>
  <div class="content-wrapper">
    <nav-bar :title="$t('account.send.title')"></nav-bar>
    <div class="content-body">
      <div class="from-title">{{ $t('account.send.fromLabel') }}</div>
      <div class="from-address">{{ address }}</div>
      <label class="transfer-limit">{{$t('account.sendToken.limit')}}&nbsp;{{balances}} &nbsp;ELA</label>
      <wallet-input
        v-model="destAddr"
        :label="$t('account.send.destLabel')"
        :placeholder="$t('account.send.destPlaceHolder')"
      ></wallet-input>

      <wallet-input
        v-model="value"
        type="number"
        postfix="ELA"
        :label="$t('account.send.valueLabel')"
        :placeholder="$t('account.send.valuePlaceHolder')"
      ></wallet-input>

      <wallet-input v-model="desc" :label="$t('account.send.descLabel')"></wallet-input>

      <fees-slider v-model="fees"></fees-slider>
    </div>

    <div class="content-footer">
      <button
        :disabled="!valid"
        class="display-block btn-primary"
        @click="confirmSend"
      >{{ $t('account.send.confirmButton') }}</button>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.content-wrapper {
  position: relative;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  overflow: auto;
}

.content-body {
  flex: 1 0 0;
  padding: 0 16px 7px;
  position: relative;
}
.transfer-limit {
  position: absolute;
  display: inline-block;
  top: 167px;
  right: 17px;
  z-index: 0;
  font-size: 12px;
  font-weight: 400;
  line-height: 22px;
  color: rgba(165, 174, 193, 1);
}
.content-footer {
  padding: 0 16px;
}

.from-title,
.from-address {
  color: #717680;
  font-size: 15px;
  padding: 2px;
}

.from-address {
  margin-bottom: 10px;
}
</style>

<script type="text/jsx">
import WalletInput from "../components/input";
import NavBar from "../components/nav-bar";
import FeesSlider from "../components/fees-slider";
import API from "../api";
import formatError from "../api/format-error";
import StateWatcher from "./state-watcher";
import axios from "axios";

export default {
  components: {
    WalletInput,
    FeesSlider,
    NavBar
  },

  mixins: [StateWatcher],

  created() {
    const { query } = this.$router.currentRoute;
    if (query.balance && !isNaN(parseFloat(query.balance))) {
      this.balance = parseFloat(query.balance);
    } else {
      this.balance = 0;
    }
  },
  mounted() {
    this.mnemonic = JSON.parse(localStorage.getItem("mnemonic"));
    let address = JSON.parse(localStorage.getItem('address'));
    this.address = address;
    this.balances = this.getBalance(address);
  },
  computed: {
    valid() {
      return this.destAddr && this.value;
    }
  },

  methods: {
    //获取余额
    getBalance(address) {
      axios
        .get("https://api-wallet-ela.elastos.org/api/1/balance/" + address)
        .then(res => {
          this.balances = res.data.result;
          localStorage.setItem("balance", this.balances);
        })
        .catch(res => {
          console.log(res);
        });
    },
    //转账提交
    confirmSend() { 
      //判断收款方地址不能为空
      if (this.destAddr == "") return;
      //判断转账金额是否大于0.0001
      if (this.value < 0.0001) {
        this.$toast(this.$t("errors.amountLessThanLimit"), {
          type: "center"
        });

        return;
      }
      //判断余额是否大于转账金额
      if (this.balance && this.value > this.balance) {
        this.$toast(this.$t("errors.insufficientBalance"), {
          type: "center"
        });
        return;
      }

      this.$loading(this.$t("account.send.confirmLoading"));
      // return false;
      let seed = window.getSeedFromMnemonic(this.mnemonic);
      let privateKey = window.getSinglePrivateKey(seed);
      let balances = Number(this.balances);
      let nums = this.value;
      let endpoint = "https://api-wallet-ela.elastos.org";
      let from = this.address;
      let to = this.destAddr;
      let num = this.value * 100000000;
      let Memo = this.desc;
      let tx = window.Transaction();

      // let tx = new Transaction;
      let testnet_privateKey1 = privateKey;
      // return false
      if (nums > balances) {
        this.$toast("余额不足！");
        this.$loading.close()
        // return false;
      } else {
        try {
          this.$loading(this.$t("account.send.confirmLoading"));
          tx.createTx(endpoint, from, to, num, Memo);
          tx.generateRawTransaction([testnet_privateKey1]);
          tx.sendRawTx(endpoint);
          this.$toast(this.$t("window.transfer.createSuccess"), {
            type: "center"
          });
          this.destAddr = null;
          this.value = null;
          this.desc = null;
          setTimeout(() => {
            this.$loading.close()
          }, 300);
        } catch (error) {
          this.$toast("请求处理异常！");
        }
      }

    }
  },

  data() {
    return {
      balance: null,
      destAddr: null,
      value: null,
      desc: null,
      fees: 0.0001,
      wallet: "",
      balances: "",
      address:''
    };
  }
};
</script>
