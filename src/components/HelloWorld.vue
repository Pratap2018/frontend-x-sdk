<template>
  <div>

    <p>{{ accounts }}</p>
    <button @click="createDidMeta">Connect Metamask </button>

    <button @click="createDidkeplr">Connect keplr</button>
    <hr>

    <textarea v-model="didDocument" id="" cols="200" rows="50"></textarea>

  </div>
</template>

<script>
/* eslint-disable */
import { HypersignDID } from 'hs-ssi-sdk'
import loadWeb3 from '../mixins/getweb3'
export default {
  name: 'HelloWorld',
  data() {
    return {
      didDocument: null,

      hypersignDID: null,
      web3: null,
      keplr: null,
      accounts: new Map()
    }
  },
  mixins: [loadWeb3],
  mounted() {


  },
  methods: {
    initialise() {
      const offlineSigner = window.keplr.getOfflineSigner('jagrat')
      this.hypersignDID = new HypersignDID({
        offlineSigner,
        nodeRestEndpoint: 'http://localhost:1317',
        nodeRpcEndpoint: 'http://localhost:26657',
        namespce: 'testnet'

      })

    },
    async createDidMeta() {
      if (this.hypersignDID === null) {
        this.initialise()

      }
      this.web3 = await loadWeb3(1)
      const accounts = await this.web3.eth.getAccounts();

      this.accounts.set(1, accounts[0])
      const hexChainId = this.web3.utils.toHex(1)
      const didDoc = await this.hypersignDID.createByClientSpec({
        methodSpecificId: accounts[0],
        address: accounts[0],
        chainId: hexChainId,
        keyType: 'EcdsaSecp256k1RecoveryMethod2020'
      })
      const signatureAndDoc=await this.hypersignDID.signByClientSpec({
        didDocument:didDoc,
        clientSpec:'eth-personalSign',
        address:accounts[0],
        web3:this.web3,
        chainId:1
      })

      this.didDocument=JSON.stringify(signatureAndDoc,null,2)

    },
    async createDidkeplr() {
      if (this.hypersignDID === null) {
        this.initialise()

      }
      const keplrObj = await window.keplr.getKey('jagrat')
    
      this.keplr = window.keplr
      this.accounts.set('jagrat', keplrObj.bech32Address)
      const publicKeyMultibase = 'z'+this.uint8ArrayToBase58(keplrObj.pubKey)
      const didDoc = await this.hypersignDID.createByClientSpec({
        methodSpecificId: keplrObj.bech32Address,
        publicKey: publicKeyMultibase,
        address: keplrObj.bech32Address,
        chainId: 'jagrat',
        keyType: 'EcdsaSecp256k1VerificationKey2019'
      })

      const signatureAndDoc=await this.hypersignDID.signByClientSpec({
        didDocument:didDoc,
        clientSpec:'cosmos-ADR036',
        address:keplrObj.bech32Address,
        web3:this.keplr,
        chainId:'jagrat'
      })
      this.didDocument=JSON.stringify(signatureAndDoc,null,2)
    },

    uint8ArrayToBase58(uint8array) {
      const ALPHABET = '123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz';
      const BASE = BigInt(ALPHABET.length);
      const ZERO = BigInt(0);

      // Count leading zeros
      let leadingZeros = 0;
      while (uint8array[leadingZeros] === 0) {
        leadingZeros++;
      }

      let number = BigInt(0);
      for (let i = 0; i < uint8array.length; i++) {
        number = number * BigInt(256) + BigInt(uint8array[i]);
      }

      let encoded = '';
      while (number > ZERO) {
        const remainder = number % BASE;
        encoded = ALPHABET[Number(remainder)] + encoded;
        number = (number - remainder) / BASE;
      }

      // Add leading zeros
      for (let i = 0; i < leadingZeros; i++) {
        encoded = ALPHABET[0] + encoded;
      }

      return encoded;
    }


  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
