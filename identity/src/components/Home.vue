<template>
	<div v-if="!runningInFrame" class="">
		<div class="wallet-details">
			<h1 class="">Wallet Aepp</h1>

			<div class="border">
				<div class="">
					<div class="">
						<span class="wallet-details-label">Public Key:</span> {{pub}}
					</div>
				</div>
				<div v-if="height" class="">
					<div class="p-2 w-1/4">
						<span class="wallet-details-label">Height:</span> {{height}}
					</div>
				</div>
				<div v-if="height" class="">
					<div class="p-2 w-1/4">
						<span class="wallet-details-label">Balance:</span> {{balance}}
					</div>
				</div>
			</div>
		</div>

		<div v-if="!aeppUrl" class="">
			Loading Aepp...
		</div>
		<!-- external app -->
		<iframe v-show="aeppUrl" ref="aepp" class="" src="about:blank" frameborder="1"></iframe>
	</div>
</template>

<script>
  // AE_SDK_MODULES is a webpack alias present in webpack.config.js
  import Wallet from '@aeternity/aepp-sdk/es/ae/wallet'
  import MemoryAccount from '@aeternity/aepp-sdk/es/account/memory'
  import account from '../account'

  export default {
    data() {
      return {
        runningInFrame: window.parent !== window,
        pub: account.pub,
        priv: account.priv,
        client: null,
        balance: null,
        height: null,
        url: account.url,
        internalUrl: account.internalUrl,
        compilerUrl: account.compilerUrl,
        aeppUrl: account.aeppUrl
      }
    },
    methods: {
      confirmDialog(method, params, { id }) {
        return Promise.resolve(window.confirm(`User ${ id } wants to run ${ method } ${ params }`))
      }
    },
    async created() {
      this.client = await Wallet({
        url: this.url,
        internalUrl: this.internalUrl,
        compilerUrl: this.compilerUrl,
        accounts: [MemoryAccount({ keypair: { secretKey: this.priv, publicKey: this.pub } })],
        address: this.pub,
        onTx: this.confirmDialog,
        onChain: this.confirmDialog,
        onAccount: this.confirmDialog,
        onContract: this.confirmDialog
      })

      if (!this.runningInFrame) this.$refs.aepp.src = this.aeppUrl
      else window.parent.postMessage({ jsonrpc: '2.0', method: 'ready' }, '*')

      this.height = await this.client.height()
      this.balance = await this.client.balance(this.pub).catch(() => 0)
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
	.wallet-details {
		border: 1px solid #F7286E;
		border-radius: 10px;
		padding: 20px 20px 40px 20px;
		background: #311b58;
		color: white;
		font-family: 'Avenir', Helvetica, Arial, sans-serif;
	}

	.wallet-details-label {
		font-weight: bold;
		color: #F7286E;
	}

	iframe {
		margin-top: 20px;
		width: 100%;
		height: 100vh
	}
</style>
