<template lang="pug">
// TODO Сделать подгрузку инфы о токене с сервиса там о дапах который
div
  .row
    //.col(v-if="id == 26 && network.name == 'wax'").mb-2
      el-alert(title='TLM Market are closed from 6.04.2021 till 13.04.2021!' type='info' effect='dark')
        .lead Due to the opening of TLM teleport functionality trading is suspended until technical implementation is complete.
    .col(v-if='scam')
      .row.mb-2
        .col
          el-alert(type='error', show-icon)
            .lead {{ $t('Potential SCAM token') }}!

    .col(v-if='$store.state.market.quote_token.str == "DMT@shmothership"')
      .row.mb-2
        .col
          el-alert(type='error', show-icon)
            .lead {{ $t('The DMT token has been hacked. Any movement of the token (including cancellation of the order) takes 99% of the transfer amount. For any questions, please contact the DMT team') }} !

    .col(
      v-if='$store.state.market.base_token.contract == "bosibc.io" || $store.state.market.quote_token.contract == "bosibc.io"'
    )
      .row.mb-2
        .col
          el-alert(type='warning', show-icon)
            .lead {{ $t('Cross Chain transfers of BOSIBC tokens are temporary stopped! It is recommended to wait for the news before continuing trading') }}.

  client-only
    DesktopTrade(v-if='!isMobile')
    MobileTrade(v-else)
    nuxt-child
</template>

<script>
import { mapGetters, mapState } from 'vuex'

import DesktopTrade from '~/components/trade/DesktopTrade'
import MobileTrade from '~/components/trade/MobileTrade'

export default {
  components: {
    MobileTrade,
    DesktopTrade,
  },

  computed: {
    ...mapState(['network', 'markets']),
    ...mapState('market', ['symbol', 'id', 'stats', 'streaming', 'base_token', 'quote_token', 'scam']),
    ...mapGetters(['user']),
  },

  watch: {
  },

  mounted() {
    this.$store.dispatch('loadOrders', this.id)
    if (!this.streaming) {
      this.$store.dispatch('market/startStream', this.id)
    }
  },

  head() {
    const { symbol, quote_token, stats: { last_price } } = this.$store.state.market

    return {
      title: `${last_price.toFixed(7)} | ${symbol} | Alcor Exchange (Spot)`,

      meta: [
        {
          hid: 'description',
          name: 'description',
          content: `Trade ${symbol} on Biggest DEX at ${this.network.desc} blockchain. No KYC, no limits.`,
        },
        {
          // Will fail for non local images (ie. from eos-airdrops)
          hid: 'og:image',
          name: 'og:image',
          content: this.$tokenLogo(
            quote_token.symbol.name,
            quote_token.contract
          ),
        },
      ],
    }
  },
}
</script>
