<template>
  <div class="trans-history">
    <!-- {{transaction}} -->
    <trans-detail
        :visible.sync="transDetailVisible"
        :detail="currentTrans"></trans-detail>
    <div class="trans-history-title">
      {{ $t('account.transHistory.title') }}
          <img @click="refresh" src="../../static/refresh.png" />
      </div>
    <ul class="trans-list" v-if="transaction && transaction.length > 0">
      <li class="trans-item"
          @click="handleClick(trans)"
          v-for="(trans, index) in transaction"
          :key="index">
        <div class="first-row">
          <span class="trans-hash">{{trans.Outputs[0]}}</span>
          <span class="trans-amount">{{ formatAmount(trans.Value) }} {{ trans.symbol }}</span>
          
        </div>
        <div class="second-row">
          <span class="trans-time">{{ formatTime(trans.CreateTime * 1000 || trans.CreateTime) }}</span>
          <span class="trans-type">{{ formatType(trans.Type) }}</span>
          <span class="trans-status"> {{ formatStatus(trans) }} </span>
        </div>
      </li>
    </ul>
    <div v-else class="empty-block">
      {{ $t('account.transHistory.emptyBlock') }}
    </div>
  </div>
</template>

<script>
  import TransDetail from './trans-detail'
  import transUtil from './trans-util'

  export default {
    name: 'account-trans-history',

    props: [
      'transaction',
      
    ],

    components: {
      TransDetail
    },

    methods: {
      formatType: transUtil.formatType,

      formatTime: transUtil.formatTime,

      formatAmount: transUtil.formatAmount,

      formatStatus: transUtil.formatStatus,

      handleClick (trans) {
        this.transDetailVisible = true
        this.currentTrans = trans
      },
      refresh(){
        this.$emit('refresh')
      }
    },
    mounted(){
      console.log('transaction',this.transaction)
    },
    data () {
      return {
        transDetailVisible: false,
        currentTrans: {}
      }
    }
  }
</script>

<style lang="scss" scoped>
  .empty-block {
    text-align: center;
  }

  .trans-history-title {
    font-size: 13px;
    color: #B4BCCC;
    padding-bottom: 10px;
    margin-bottom: 12px;
    border-bottom: 1px solid rgba(180, 188, 204, 0.3);
    img{
      width:6%;
      float:right;
    }
  }

  .trans-list {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .trans-item {
    cursor: pointer;
    padding-bottom: 12px;
    border-bottom: 1px solid #ECEEF2;

    .first-row {
      display: flex;
      flex-direction: row;
    }

    .second-row {
      line-height: 12px;
    }

    .trans-hash {
      text-overflow: ellipsis;
      overflow: hidden;
      flex: 1 0 0;
      margin-right: 12px;
      color: #5B5F67;
      font-size: 15px;
    }

    .trans-amount {
      color: #5B5F67;
      font-size: 17px;
    }

    .trans-time, .trans-type, .trans-status {
      color: #B4BCCC;
      font-size: 12px;
    }

    .trans-time {
      display: inline-block;
      width: 140px;
    }

    .trans-status {
      float: right;
    }
  }
</style>
