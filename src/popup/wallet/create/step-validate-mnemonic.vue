<template>
  <nav-layout :title="$t('wallet.create.validate.title')" :path="backPath" class="u-full-height">
    <wallet-input
      style="margin-top: 16px;"
      v-model="currentMnemonic"
      :label="$t('wallet.create.validate.mnemonicLabel')"
      type="textarea"
      read-only
    ></wallet-input>

    <div>
      <mnemonic-tags
        :default-selection="defaultSelection"
        :mnemonics="shuffledMnemonics"
        @input="handleInput"
      ></mnemonic-tags>
    </div>

    <template slot="footer">
      <button
        class="display-block btn-primary"
        :disabled="!isCompleted || !isValid"
        @click="confirmCreate"
      >{{ $t('common.confirm') }}</button>
    </template>
  </nav-layout>
</template>

<style>
</style>

<script type="text/jsx">
import NavLayout from "../../components/nav-layout";
import WalletInput from "../../components/input";
import MnemonicTags from "../../components/mnemonic-tags";
import * as shuffle from "shuffle-array";
import API from "../../api";
import formatError from "../../api/format-error";
import eventBus from "../../account/bus";
// import mnemonic from "./mnemonic";
import mnemonic from '../../../../scripts/elastos-wallet-api'
export default {
  name: "validate-mnemonic",

  components: {
    NavLayout,
    WalletInput,
    MnemonicTags
  },

  props: {
    mnemonics: {
      type: Array,
      default: []
    }
  },

  created() {
    this.shuffledMnemonics = shuffle(this.mnemonics.slice());
 
    const route = this.$router.currentRoute;
    console.log("query:", route.query);
    const { password, mnemonic, currentMnemonic } = route.query;
    if (password) {
      this.password = password;
    }
    if (mnemonic) {
      this.mnemonic = mnemonic;
    }
    if (currentMnemonic) {
      this.selection = currentMnemonic.split(" ");
      this.currentMnemonic = currentMnemonic;
      this.defaultSelection = this.selection;
    }
    console.log("route", route);
    console.log("route.path", route.path);
    if (route && route.path) {
      this.isCreatingWallet = route.path.indexOf("wallet") !== -1;
    }
    console.log(this.isCreatingWallet);
    window.onunload = () => {
      localStorage.setItem(
        "RESTORE_PATH",
        JSON.stringify({
          name: route.name,
          query: {
            ...(route.query || {}),
            currentMnemonic: this.currentMnemonic
          }
        })
      );
    };
  },

  destroyed() {
    window.onunload = null;
  },

  computed: {
    isCompleted() {
      return (this.selection || []).length === 12;
    },

    backPath() {
      if (this.isCreatingWallet) {
        return {
          name: "backupWalletMnemonic"
        };
      } else {
        return {
          name: "createAccount"
        };
      }
    }
  },

  methods: {
    confirmCreate() {
      this.$loading(this.$t("wallet.create.validate.confirmLoading"));

      setTimeout(() => {
        let promise;
        console.log(this.isCreatingWallet);
        if (this.isCreatingWallet) {
          let seed = window.getSeedFromMnemonic(this.mnemonic);
          let privateKey = window.getSinglePrivateKey(seed);
          let publicKey = window.getSinglePublicKey(seed);
          let address = window.getAddress(publicKey);
          let obj = {
          seed: seed,
          privateKey: privateKey,
          publicKey: publicKey,
          address: address
        };
        this.wallet = obj;
         localStorage.setItem('mnemonic',JSON.stringify(this.mnemonic))
         localStorage.setItem('address',JSON.stringify(address))
         
            this.$loading.close();
            this.$toast(this.$t("wallet.create.validate.createSuccess"), {
              type: "center"
            }); 

            promise = API.createWallet(this.password, this.mnemonic);

          
        } else { 
          let seed = window.getSeedFromMnemonic(this.mnemonic);
          let privateKey = window.getSinglePrivateKey(seed);
          let publicKey = window.getSinglePublicKey(seed);
          let address = window.getAddress(publicKey);
          let obj = {
          seed: seed,
          privateKey: privateKey,
          publicKey: publicKey,
          address: address
        };
        this.wallet = obj;
         localStorage.setItem('mnemonic',JSON.stringify(this.mnemonic))
         localStorage.setItem('address',JSON.stringify(address))
        } 

          this.$loading.close();
          this.$toast(this.$t("wallet.create.validate.createSuccess"), {
            type: "center"
          });
         this.$router.push({
              name: "accountMain",
          });
       
        // return false;
        // promise.then(
        //   () => {
        //     this.$loading.close();
        //     this.$toast(this.$t("wallet.create.validate.createSuccess"), {
        //       type: "center"
        //     });

        //     if (!this.isCreatingWallet) {
        //       eventBus.$emit("header:state:refresh");
        //     }

        //     mnemonic.clear();

        //     this.$router.push({
        //       name: "accountMain",
        //       query:{
        //         wallet:this.wallet
        //       }
        //     });
        //   },
        //   error => {
        //     console.log("create wallet error:", error);
        //     this.$loading.close();
        //     this.$toast(
        //       this.$t("wallet.create.validate.createFailure") +
        //         " " +
        //         formatError(error),
        //       {
        //         type: "center",
        //         duration: 5000,
        //         wordWrap: true
        //       }
        //     );
        //   }
        // );
      }, 300);
    },

    validate() {
      const mnemonics = this.mnemonics;
      const selection = this.selection || [];

      let isValid = true;
      for (let i = 0; i < selection.length; i++) {
        if (mnemonics[i] !== selection[i]) {
          isValid = false;
        }
      }

      if (!isValid) {
        this.$toast(this.$t("wallet.create.validate.validateErrorTip"), {
          type: "center"
        });
      }

      return isValid;
    },

    handleInput(selection) {
      this.selection = selection;
      this.currentMnemonic = selection.join(" ");

      this.isValid = this.validate();
    }
  },

  data() {
    return {
      isCreatingWallet: true,
      defaultSelection: null,
      selection: [],
      currentMnemonic: "",
      shuffledMnemonics: [],
      isValid: true,
      mnemonic: "",
      password: null,
      wallet:{}
    };
  }
};
</script>
