<template>
  <v-app>
    <v-main>
      <v-container>
        <v-card class="pa-3">
          <h2 class="text-center">Simple Dapp</h2>

          <v-btn
            v-if="metamaskConnected"
            @click="connectMetamask()"
            class="mt-4"
            color="primary"
          >
            Connect Wallet
          </v-btn>
          <v-btn
            v-if="!metamaskConnected"
            @click="disconnectMetamask()"
            class="mt-4"
            color="primary"
          >
            Disconnect Wallet
          </v-btn>

          <div></div>
        </v-card>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import Web3 from "web3";

export default {
  name: "App",
  data() {
    return {
      metamaskConnected: false,
    };
  },
  mounted() {
    this.initialize();
  },
  methods: {
    async initialize() {
      // check if metamask is installed
      if (typeof window.ethereum !== "undefined") {
        return;
      }

      console.log("MetaMask is installed!");

      console.log("ethereum", window.ethereum);
      var web3 = new Web3("ws://localhost:8546");
      console.log(web3);
      console.log("ethereum", window.ethereum);

      console.log();
    },
    async connectMetamask() {
      //Will Start the metamask extension
      const accounts = await window.ethereum.request({
        method: "eth_requestAccounts",
      });
      const account = accounts[0];
      if (account) {
        this.metamaskConnected = true;
      }
      console.log("accounts", accounts, account);
    },
    async disconnectMetamask() {
      console.log("disconnect");
    },
  },
};
</script>
