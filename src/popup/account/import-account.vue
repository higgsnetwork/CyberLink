<template>
  <nav-layout :title="$t('account.import.title')" class="u-full-height"> 
    <div class="warning-container">
      <div class="tip-title">{{ $t('common.warning') }}</div>
      <div class="tip-content">{{ $t('wallet.import.tip') }}</div>
    </div>

    <wallet-input :label="$t('account.import.typeLabel')" type="custom">
      <select v-model="type" class="display-block">
        <option value="mnemonic">{{ $t('account.import.mnemonicType') }}</option>
        <!-- <option value="privateKey">{{ $t('account.import.privateKeyType') }}</option> -->
      </select>
    </wallet-input>

    <wallet-input
        v-if="type === 'mnemonic'"
        v-model="mnemonic"
        :label="$t('account.import.mnemonicLabel')"
        :placeholder="$t('account.import.mnemonicPlaceholder')"
        type="textarea"></wallet-input>

    <wallet-input
        v-else
        v-model="privateKey"
        :label="$t('account.import.privateKeyLabel')"
        :placeholder="$t('account.import.privateKeyPlaceholder')"
        type="textarea"></wallet-input>

    <template slot="footer">
      <button
          class="display-block btn-primary"
          :disabled="!isValid"
          @click="confirm">{{ $t('common.confirm') }}</button>
    </template>
  </nav-layout>
</template>

<script>
  import WalletInput from '../components/input'
  import API from '../api'
  import formatError from '../api/format-error'
  import NavLayout from '../components/nav-layout'

  export default {
    name: 'import-account',

    components: {
      WalletInput,
      NavLayout
    },

    computed: {
      isValid () {
        return this.mnemonic || this.privateKey
      }
    },

    watch: {
      type (val) {
        if (val === 'mnemonic') {
          this.privateKey = null
        } else if (val === 'privateKey') {
          this.mnemonic = null
        }
      }
    },
 
    methods: {
      checkMnemonicCode(){
      
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
        this.$loading(this.$t('account.import.confirmLoading'))
        this.$loading.close()
        this.$router.push({
              name: 'welcome'
        })
        console.log(this.wallet);
        console.log(address);
        // return false
        // API.checkMnemonicCode(this.mnemonic).then((data) => {
        //   if(data===true){
        //     this.commit();
        //   }else{
        //     this.$toast(this.$t('wallet.import.correctMnemonic'), {type: 'center',duration: 5000,wordWrap: true})
        //   }
        // }, (error) => {
        //   this.$loading.close()
        //   this.$toast(this.$t('wallet.import.importFailure') + ' ' + formatError(error), {
        //     type: 'center',
        //     duration: 5000,
        //     wordWrap: true
        //   })
        // })
      },
      commit(){
        this.$loading(this.$t('account.import.confirmLoading'))
        setTimeout(() => {
          API.importAccount(this.mnemonic, this.privateKey).then(() => {
            this.$loading.close()
            this.$toast(this.$t('account.import.importSuccess'), {
              type: 'center'
            })

            this.$router.push({
              name: 'welcome'
            })
          }, (error) => {
            this.$loading.close()
            this.$toast(this.$t('account.import.importFailure') + ' ' + formatError(error), {
              type: 'center',
              duration: 5000,
              wordWrap: true
            })
          })
        }, 300)
      },
      confirm () {
        if(this.type==='mnemonic'){
          this.checkMnemonicCode()
        }else{
          this.commit();
        }
      }
    },

    data () {
      return {
        type: 'mnemonic',
        mnemonic: null,
        privateKey: null
      }
    }
  }
</script>

<style lang="scss" scoped>
  @import './warning.scss';
</style>
