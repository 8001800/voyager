<template lang='pug'>
mixin tx-container-sent
  .tx-container
    .tx-element.tx-coins
      .tx-coin(v-for='coin in coinsSent')
        .key {{ coin.denom.toUpperCase() }}
        .value {{ pretty(coin.amount) }}
    div
      .tx-element.tx-date(v-if="devMode") {{ date }}
      .tx-element.tx-address(v-if="!sentSelf") Sent to {{ receiver }}
      .tx-element.tx-address(v-if="sentSelf") You sent this amount to yourself.
.tm-li-tx(v-if="sentSelf" @click="() => devMode && viewTransaction()")
  .tx-icon: i.material-icons swap_horiz
  +tx-container-sent

.tm-li-tx.tm-li-tx-sent(v-else-if="sent" @click="() => devMode && viewTransaction()")
  .tx-icon: i.material-icons remove_circle
  +tx-container-sent

.tm-li-tx.tm-li-tx-received(v-else @click="() => devMode && viewTransaction()")
  .tx-icon: i.material-icons add_circle
  .tx-container
    .tx-element.tx-coins
      .tx-coin(v-for='coin, key in coinsReceived' :key="key")
        .key {{ coin.denom.toUpperCase() }}
        .value {{ pretty(coin.amount) }}
    div
      .tx-element.tx-date(v-if="devMode") {{ date }}
      .tx-element.tx-address Received from {{ sender }}
</template>

<script>
import moment from "moment"
import numeral from "numeral"

const defaultCoin = {
  denom: null,
  amount: null
}

const defaultInput = {
  address: null,
  coins: [defaultCoin, defaultCoin, defaultCoin]
}

const defaultTransaction = {
  tx: {
    value: {
      msg: [
        {
          value: {
            time: null,
            inputs: [defaultInput, defaultInput, defaultInput],
            outputs: [defaultInput, defaultInput, defaultInput]
          }
        }
      ]
    }
  }
}

export default {
  name: "tm-li-transaction",
  computed: {
    tx() {
      return this.transaction.tx.value.msg[0].value
    },
    // HERE FOR DOCUMENTATION
    // transactionHeight() {
    //   return this.transaction.height
    // },
    // TODO: sum relevant inputs/outputs
    sentSelf() {
      return this.tx.inputs[0].address === this.tx.outputs[0].address
    },
    sent() {
      return this.tx.inputs[0].address === this.address
    },
    sender() {
      return this.tx.inputs[0].address
    },
    coinsSent() {
      return this.tx.inputs[0].coins
    },
    receiver() {
      return this.tx.outputs[0].address
    },
    coinsReceived() {
      return this.tx.inputs[0].coins
    },
    date() {
      try {
        return moment(this.transaction.time).format("MMMM Do YYYY, h:mm:ss a")
      } catch (error) {
        return null
      }
    }
  },
  data: () => ({
    devMode:
      process.env.PREVIEW !== undefined
        ? JSON.parse(process.env.PREVIEW)
        : process.env.NODE_ENV === "development"
  }),
  methods: {
    pretty(num) {
      return numeral(num).format("0,0.00")
    },
    viewTransaction() {
      // console.log("TODO: implement tx viewer")
    }
  },
  props: {
    transaction: {
      type: Object,
      default: () => defaultTransaction
    },
    address: {
      type: String,
      default: null
    }
  }
}
</script>

<style lang="stylus">
@require '~variables'

.tm-li-tx
    display flex
    font-size sm
    border-bottom 1px solid var(--bc-dim)

    &:nth-of-type(2n-1)
        background var(--app-fg)

    .tx-icon
        padding 0 0.5rem
        background var(--app-fg)
        display flex
        align-items center
        justify-content center

    .tx-container
        flex-direction column
        flex-wrap nowrap
        padding 0.5rem 0
        margin 0.5rem 0
        display flex
        width 100%
        min-width 0 // fix text-overflow

    .tx-element
        padding 0 2rem 0 1.5rem
        line-height 1.5rem

    .tx-coin
        .value
            flex 0 0 100%
            font-size sm
            color var(--dim)

            &:before
                content ''
                display inline

        .key
            font-weight 500
            font-size m

        .value, .key
            line-height 1.5rem

    .tx-address
        white-space nowrap
        overflow hidden
        text-overflow ellipsis
        color var(--dim)
        font-size sm

    &.tm-li-tx-sent
        .tx-coin .value
            &:before
                content '-'

    &.tm-li-tx-received
        .tx-icon
            background var(--app-fg)

        .tx-coin .value
            color success

            &:before
                content '+'

    &:hover
        cursor pointer
        background var(--hover-bg)

@media screen and (min-width: 700px)
    .tm-li-tx
        font-size 0.875rem

        .tx-container
            flex-direction row

            .tx-coins
                flex 0 0 9rem
                padding 0
                min-width 0

                .tx-coin
                    padding 0 1.5rem 0

                    .key
                        white-space nowrap
                        overflow hidden
                        text-overflow ellipsis
</style>
