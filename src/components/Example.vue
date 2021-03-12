<template>
  <div>
    <div v-if="web3">
      <h1>Connect wallet success!</h1>
      <p>Address: {{ this.activeAddress }}</p>
      <button v-on:click="() => sendBnb()">Send 0.1 BNB to 0x8D7fa74e4c11a13F82412B391F2D9e1EEeA3326A</button>
      <button v-on:click="() => sendDfy()">Send 0.1 DFY to 0x8D7fa74e4c11a13F82412B391F2D9e1EEeA3326A</button>
      <button v-on:click="() => sendNft()">Send YLGR to 0x8D7fa74e4c11a13F82412B391F2D9e1EEeA3326A</button>
    </div>
    <div v-else-if="web3KeyStore">
      <h1>Connect wallet success!</h1>
      <p>Address: {{ this.activeAddress }}</p>
      <br/>
      <input v-model="keyStorePassword" placeholder="password keystore" type="password"/>
      <br/>
      <button v-on:click="() => sendBnbKeyStore()">Send 0.1 BNB to 0x8D7fa74e4c11a13F82412B391F2D9e1EEeA3326A</button>
      <button v-on:click="() => sendDfyKeyStore()">Send 0.1 DFY to 0x8D7fa74e4c11a13F82412B391F2D9e1EEeA3326A</button>
    </div>
    <div v-else>
      <input v-model="inputMnemonic" placeholder="Mnemonic"/>
      <button v-on:click="() => inputWalletByMnemonic()">Wallet by Mnemonic</button>
      <input v-model="inputPrivateKey" placeholder="Private key"/>
      <button v-on:click="() => inputWalletByPrivateKey()">Wallet by Private key</button>
      <br/>
      <p>Upload keystore</p>
      <input type="file" @change="inputKeystore">
      <br/>
      <h1>Or</h1>
      <br/>
      <button v-on:click="() => generateWallet()">Create wallet</button>
      <br/>
      <p v-if="createWalletInfo">Mnemonic: {{ createWalletInfo.mnemonic }}</p>
      <p v-if="createWalletInfo">Private key: {{ createWalletInfo.privateKey }}</p>
      <p v-if="createWalletInfo">Address: {{ createWalletInfo.address }}</p>
      <br/>
      <div v-if="createWalletInfo">
        <input v-model="createKeyStorePassword" placeholder="password keystore" type="password"/>
        <button v-on:click="createKeyStore">Create keystore</button>
      </div>
    </div>
  </div>
</template>

<script>
import {generateMnemonic} from 'bip39';
import Web3 from 'web3';
import BigNumber from 'bignumber.js';
import HDWalletProvider from '@truffle/hdwallet-provider';
import dfyAbi from '../contracts/dfy.abi';
import nftAbi from '../contracts/nft.abi';
import fileDownload from 'js-file-download';

export default {
  name: 'Example',
  data: function () {
    return {
      web3: null,
      web3KeyStore: null,
      dfyContract: null,
      nftContract: null,
      inputMnemonic: '',
      inputPrivateKey: '',
      activeAddress: '',
      createWalletInfo: null,
      createKeyStorePassword: null,
      keyStorePassword: null,
      keystoreFile: null
    }
  },
  methods: {
    inputWalletByMnemonic() {
      const provider = new HDWalletProvider({
        mnemonic: {
          phrase: this.inputMnemonic
        },
        providerOrUrl: "https://data-seed-prebsc-1-s1.binance.org:8545"
      })
      this.web3 = new Web3(provider)
      this.activeAddress = Object.keys(provider.wallets)[0]
      this.initDfyContract()
    },
    inputWalletByPrivateKey() {
      const provider = new HDWalletProvider({
        privateKeys: [this.inputPrivateKey],
        providerOrUrl: "https://data-seed-prebsc-1-s1.binance.org:8545"
      })
      this.web3 = new Web3(provider)
      this.activeAddress = Object.keys(provider.wallets)[0]
      this.initDfyContract()
    },
    inputKeystore(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.readAsText(file);
        reader.onloadend = (event) => {
          const fileContent = JSON.parse(event.target.result)
          if (fileContent && fileContent.address) {
            this.activeAddress = '0x' + fileContent.address
            this.keystoreFile = fileContent
            this.web3KeyStore = new Web3("https://data-seed-prebsc-1-s1.binance.org:8545")
          }
        };
      }
    },
    initDfyContract() {
      this.dfyContract = new this.web3.eth.Contract(
          dfyAbi,
          '0xB6bd9Bba44C8369D47f07CcC9032e65E811A112d'
      )
    },
    initNftContract() {
      this.nftContract = new this.web3.eth.Contract(
          nftAbi,
          '0x1eF2dB5a32A302Fba03466f3D490591e051d7ec7'
      )
    },
    generateWallet() {
      const mnemonic = generateMnemonic()
      console.log('mnemonic: ', mnemonic)
      const provider = new HDWalletProvider({
        mnemonic: {
          phrase: mnemonic
        },
        providerOrUrl: "https://data-seed-prebsc-1-s1.binance.org:8545"
      });
      const address = Object.keys(provider.wallets)[0]
      const wallet = provider.wallets[address]
      this.createWalletInfo = {
        mnemonic: mnemonic,
        privateKey: wallet.getPrivateKeyString(),
        address: address
      }
    },
    async sendBnb() {
      const gasPrice = await this.web3.eth.getGasPrice()
      const tx = {
        from: this.activeAddress,
        to: '0x8D7fa74e4c11a13F82412B391F2D9e1EEeA3326A',
        value: new BigNumber(
            0.1
        ).multipliedBy(10 ** 18).toString(),
        gas: 21000,
        gasPrice: gasPrice
      }
      const signed = await this.web3.eth.signTransaction(tx, this.activeAddress)
      console.log('signed: ', signed)
      const receipt = await this.web3.eth.sendSignedTransaction(signed.raw)
      console.log('receipt: ', receipt)
      alert('send success: ' + receipt.transactionHash)
    },
    async sendDfy() {
      const gasPrice = await this.web3.eth.getGasPrice()
      const txData = this.dfyContract.methods.transfer(
          '0x8D7fa74e4c11a13F82412B391F2D9e1EEeA3326A',
          new BigNumber(
              0.1
          ).multipliedBy(10 ** 18).integerValue()
      ).encodeABI();
      const tx = {
        from: this.activeAddress,
        to: '0xB6bd9Bba44C8369D47f07CcC9032e65E811A112d',
        value: 0,
        gas: 100000,
        gasPrice: gasPrice,
        data: txData
      };
      const signed = await this.web3.eth.signTransaction(tx, this.activeAddress)
      console.log('signed: ', signed)
      const receipt = await this.web3.eth.sendSignedTransaction(signed.raw)
      console.log('receipt: ', receipt)
      alert('send success: ' + receipt.transactionHash)
    },
    async sendNft() {
      const gasPrice = await this.web3.eth.getGasPrice()
      this.initNftContract()
      const txData = this.nftContract.methods.safeTransferFrom(
          this.activeAddress,
          '0x8D7fa74e4c11a13F82412B391F2D9e1EEeA3326A',
          2
      ).encodeABI();
      const tx = {
        from: this.activeAddress,
        to: '0x1eF2dB5a32A302Fba03466f3D490591e051d7ec7',
        value: 0,
        gas: 700000,
        gasPrice: gasPrice,
        data: txData
      };
      const signed = await this.web3.eth.signTransaction(tx, this.activeAddress)
      console.log('signed: ', signed)
      const receipt = await this.web3.eth.sendSignedTransaction(signed.raw)
      console.log('receipt: ', receipt)
      alert('send success: ' + receipt.transactionHash)
    },
    async sendBnbKeyStore() {
      try {
        const account = this.web3KeyStore.eth.accounts.decrypt(this.keystoreFile, this.keyStorePassword)
        console.log('account: ', account)
        const gasPrice = await this.web3KeyStore.eth.getGasPrice()
        const tx = {
          from: this.activeAddress,
          to: '0x8D7fa74e4c11a13F82412B391F2D9e1EEeA3326A',
          value: new BigNumber(
              0.1
          ).multipliedBy(10 ** 18).toString(),
          gas: 21000,
          gasPrice: gasPrice
        }
        const signed = await account.signTransaction(tx)
        console.log('signed: ', signed)
        const receipt = await this.web3KeyStore.eth.sendSignedTransaction(signed.rawTransaction)
        console.log('receipt: ', receipt)
        alert('send success: ' + receipt.transactionHash)
      } catch (e) {
        alert(e.message)
      }

    },
    async sendDfyKeyStore() {
      try {
        const account = this.web3KeyStore.eth.accounts.decrypt(this.keystoreFile, this.keyStorePassword)
        console.log('account: ', account)
        const gasPrice = await this.web3KeyStore.eth.getGasPrice()
        const txData = this.dfyContract.methods.transfer(
            '0x8D7fa74e4c11a13F82412B391F2D9e1EEeA3326A',
            new BigNumber(
                0.1
            ).multipliedBy(10 ** 18).integerValue()
        ).encodeABI();
        const tx = {
          from: this.activeAddress,
          to: '0xB6bd9Bba44C8369D47f07CcC9032e65E811A112d',
          value: 0,
          gas: 100000,
          gasPrice: gasPrice,
          data: txData
        };
        const signed = await account.signTransaction(tx)
        console.log('signed: ', signed)
        const receipt = await this.web3KeyStore.eth.sendSignedTransaction(signed.rawTransaction)
        console.log('receipt: ', receipt)
        alert('send success: ' + receipt.transactionHash)
      } catch (e) {
        alert(e.message)
      }
    },
    createKeyStore() {
      if (this.createKeyStorePassword) {
        const web3 = new Web3('https://data-seed-prebsc-1-s1.binance.org:8545')
        const keystore = web3.eth.accounts.encrypt(this.createWalletInfo.privateKey, this.createKeyStorePassword)
        fileDownload(JSON.stringify(keystore), 'keystore.json')
      } else alert('please input password first!')
    }
  }
}
</script>

