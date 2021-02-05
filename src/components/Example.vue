<template>
    <div>
        <div v-if="!web3">
            <input v-model="inputMnemonic" placeholder="Mnemonic"/>
            <button v-on:click="() => inputWalletByMnemonic()">Wallet by Mnemonic</button>
            <input  v-model="inputPrivateKey" placeholder="Private key"/>
            <button v-on:click="() => inputWalletByPrivateKey()">Wallet by Private key</button>
            <br/>
            <h1>Or</h1>
            <br/>
            <button v-on:click="() => generateWallet()">Create wallet</button>
            <br/>
            <p v-if="createWalletInfo">Mnemonic: {{createWalletInfo.mnemonic}}</p>
            <p v-if="createWalletInfo">Private key: {{createWalletInfo.privateKey}}</p>
            <p v-if="createWalletInfo">Address: {{createWalletInfo.address}}</p>
        </div>
        <div v-else>
            <h1>Connect wallet success!</h1>
            <p>Address: {{this.activeAddress}}</p>
            <button v-on:click="() => sendBnb()">Send 0.1 BNB to 0x469EA41396a3cD5D4c5aA96D8C9eaBd38f85d35d</button>
            <button v-on:click="() => sendDfy()">Send 0.1 DFY to 0x469EA41396a3cD5D4c5aA96D8C9eaBd38f85d35d</button>
        </div>
    </div>
</template>

<script>
    import { generateMnemonic} from 'bip39';
    import Web3 from 'web3';
    import BigNumber from 'bignumber.js';
    import HDWalletProvider from '@truffle/hdwallet-provider';
    import dfyAbi from '../contracts/dfy.abi';

    export default {
        name: 'Example',
        data: function () {
            return {
                web3: null,
                dfyContract: null,
                inputMnemonic: '',
                inputPrivateKey: '',
                activeAddress: '',
                createWalletInfo: null
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
            initDfyContract() {
                this.dfyContract = new this.web3.eth.Contract(
                    dfyAbi,
                    '0xB6bd9Bba44C8369D47f07CcC9032e65E811A112d'
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
                    to: '0x469EA41396a3cD5D4c5aA96D8C9eaBd38f85d35d',
                    value: "1000000000000000000000",
                    gas: 21000,
                    gasPrice: gasPrice
                }
                const signed = await this.web3.eth.signTransaction(tx)
                console.log('signed: ', signed)
                const receipt = await this.web3.eth.sendSignedTransaction(signed.rawTransaction)
                console.log('receipt: ', receipt)
            },
            async sendDfy() {
                const gasPrice = await this.web3.eth.getGasPrice()
                const txData = this.dfyContract.methods.transfer(
                    '0x469EA41396a3cD5D4c5aA96D8C9eaBd38f85d35d',
                    new BigNumber(
                        0.1
                    ).multipliedBy(10 ** 18).integerValue()
                ).encodeABI();
                const tx = {
                    to: '0xB6bd9Bba44C8369D47f07CcC9032e65E811A112d',
                    value: 0,
                    gas: 100000,
                    gasPrice: gasPrice,
                    data: txData
                };
                const signed = await this.web3.eth.signTransaction(tx)
                console.log('signed: ', signed)
                const receipt = await this.web3.eth.sendSignedTransaction(signed.rawTransaction)
                console.log('receipt: ', receipt)
            }
        }
    }
</script>

