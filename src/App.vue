<template>
  <v-app>
    <v-main>
      <v-container>
        <v-card class="pa-3">
          <h2 class="text-center">Simple Dapp</h2>
          <div class="text-center">
            Connected Accounts: {{ accounts.join(", ") }}
          </div>

          <!-- <pre>{{ methodsList }}</pre> -->

          <v-row class="my-4">
            <v-col cols="12" md="6">
              <h4>Contract Address: {{ contractAddress }}</h4>
            </v-col>
            <v-col cols="12" md="6" class="text-right">
              <v-btn
                v-if="!metamaskConnected"
                @click="connectMetamask()"
                class="mt-4"
                color="primary"
              >
                Connect Wallet
              </v-btn>
              <v-btn
                v-if="metamaskConnected"
                @click="disconnectMetamask()"
                class="mt-4"
                color="primary"
              >
                Disconnect Wallet
              </v-btn>
            </v-col>
          </v-row>

          <v-expansion-panels class="mt-5">
            <v-expansion-panel v-for="(item, i) in methodsList" :key="i">
              <v-expansion-panel-header>
                {{ item.name }}
              </v-expansion-panel-header>
              <v-expansion-panel-content>
                <div v-if="item.inputs && item.inputs.length > 0">
                  <div v-for="(method, j) in item.inputs" :key="j">
                    <!-- {{ method }} -->
                    <v-text-field
                      outlined
                      :name="method.name || ''"
                      :label="method.name || ''"
                      :placeholder="method.internalType || ''"
                    ></v-text-field>
                  </div>
                </div>
                <v-btn class="float-right" color="success">Run</v-btn>
              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>

          <!-- 
          <v-tabs v-model="tab">
            <v-tab> Read Contract </v-tab>
            <v-tab> Write Contract </v-tab>
          </v-tabs>

          <v-tabs-items v-model="tab">
            <v-tab-item>
              <v-card flat>
                <v-card-text>
                  <v-expansion-panels>
                    <v-expansion-panel>
                      <v-expansion-panel-header>
                        Item
                      </v-expansion-panel-header>
                      <v-expansion-panel-content>
                        Lorem ipsum dolor sit amet, consectetur adipiscing elit,
                        sed do eiusmod tempor incididunt ut labore et dolore
                        magna aliqua. Ut enim ad minim veniam, quis nostrud
                        exercitation ullamco laboris nisi ut aliquip ex ea
                        commodo consequat.
                      </v-expansion-panel-content>
                    </v-expansion-panel>
                  </v-expansion-panels>
                </v-card-text>
              </v-card>
            </v-tab-item>
            <v-tab-item>
              <v-card flat>
                <v-card-text>
                  Lorem ipsum, dolor sit amet consectetur adipisicing elit.
                  Accusamus, ratione. Odit aspernatur assumenda molestias dolor
                  similique ipsam dolorum dolorem, sequi officiis eveniet fugit,
                  ea neque veniam! Eos consequatur corporis perferendis.
                </v-card-text>
              </v-card>
            </v-tab-item>
          </v-tabs-items> -->
        </v-card>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import Web3 from "web3";
// import Contract from "web3-eth-contract";

export default {
  name: "App",
  data() {
    return {
      metamaskConnected: false,
      accounts: [],
      contractAddress: "0x1f9840a85d5aF5bf1D1762F925BDADdC4201F984",
      contractAbi: [
        {
          inputs: [
            { internalType: "address", name: "account", type: "address" },
            { internalType: "address", name: "minter_", type: "address" },
            {
              internalType: "uint256",
              name: "mintingAllowedAfter_",
              type: "uint256",
            },
          ],
          payable: false,
          stateMutability: "nonpayable",
          type: "constructor",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: true,
              internalType: "address",
              name: "owner",
              type: "address",
            },
            {
              indexed: true,
              internalType: "address",
              name: "spender",
              type: "address",
            },
            {
              indexed: false,
              internalType: "uint256",
              name: "amount",
              type: "uint256",
            },
          ],
          name: "Approval",
          type: "event",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: true,
              internalType: "address",
              name: "delegator",
              type: "address",
            },
            {
              indexed: true,
              internalType: "address",
              name: "fromDelegate",
              type: "address",
            },
            {
              indexed: true,
              internalType: "address",
              name: "toDelegate",
              type: "address",
            },
          ],
          name: "DelegateChanged",
          type: "event",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: true,
              internalType: "address",
              name: "delegate",
              type: "address",
            },
            {
              indexed: false,
              internalType: "uint256",
              name: "previousBalance",
              type: "uint256",
            },
            {
              indexed: false,
              internalType: "uint256",
              name: "newBalance",
              type: "uint256",
            },
          ],
          name: "DelegateVotesChanged",
          type: "event",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: false,
              internalType: "address",
              name: "minter",
              type: "address",
            },
            {
              indexed: false,
              internalType: "address",
              name: "newMinter",
              type: "address",
            },
          ],
          name: "MinterChanged",
          type: "event",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: true,
              internalType: "address",
              name: "from",
              type: "address",
            },
            {
              indexed: true,
              internalType: "address",
              name: "to",
              type: "address",
            },
            {
              indexed: false,
              internalType: "uint256",
              name: "amount",
              type: "uint256",
            },
          ],
          name: "Transfer",
          type: "event",
        },
        {
          constant: true,
          inputs: [],
          name: "DELEGATION_TYPEHASH",
          outputs: [{ internalType: "bytes32", name: "", type: "bytes32" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [],
          name: "DOMAIN_TYPEHASH",
          outputs: [{ internalType: "bytes32", name: "", type: "bytes32" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [],
          name: "PERMIT_TYPEHASH",
          outputs: [{ internalType: "bytes32", name: "", type: "bytes32" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [
            { internalType: "address", name: "account", type: "address" },
            { internalType: "address", name: "spender", type: "address" },
          ],
          name: "allowance",
          outputs: [{ internalType: "uint256", name: "", type: "uint256" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: false,
          inputs: [
            { internalType: "address", name: "spender", type: "address" },
            { internalType: "uint256", name: "rawAmount", type: "uint256" },
          ],
          name: "approve",
          outputs: [{ internalType: "bool", name: "", type: "bool" }],
          payable: false,
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          constant: true,
          inputs: [
            { internalType: "address", name: "account", type: "address" },
          ],
          name: "balanceOf",
          outputs: [{ internalType: "uint256", name: "", type: "uint256" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [
            { internalType: "address", name: "", type: "address" },
            { internalType: "uint32", name: "", type: "uint32" },
          ],
          name: "checkpoints",
          outputs: [
            { internalType: "uint32", name: "fromBlock", type: "uint32" },
            { internalType: "uint96", name: "votes", type: "uint96" },
          ],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [],
          name: "decimals",
          outputs: [{ internalType: "uint8", name: "", type: "uint8" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: false,
          inputs: [
            { internalType: "address", name: "delegatee", type: "address" },
          ],
          name: "delegate",
          outputs: [],
          payable: false,
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          constant: false,
          inputs: [
            { internalType: "address", name: "delegatee", type: "address" },
            { internalType: "uint256", name: "nonce", type: "uint256" },
            { internalType: "uint256", name: "expiry", type: "uint256" },
            { internalType: "uint8", name: "v", type: "uint8" },
            { internalType: "bytes32", name: "r", type: "bytes32" },
            { internalType: "bytes32", name: "s", type: "bytes32" },
          ],
          name: "delegateBySig",
          outputs: [],
          payable: false,
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          constant: true,
          inputs: [{ internalType: "address", name: "", type: "address" }],
          name: "delegates",
          outputs: [{ internalType: "address", name: "", type: "address" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [
            { internalType: "address", name: "account", type: "address" },
          ],
          name: "getCurrentVotes",
          outputs: [{ internalType: "uint96", name: "", type: "uint96" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [
            { internalType: "address", name: "account", type: "address" },
            { internalType: "uint256", name: "blockNumber", type: "uint256" },
          ],
          name: "getPriorVotes",
          outputs: [{ internalType: "uint96", name: "", type: "uint96" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [],
          name: "minimumTimeBetweenMints",
          outputs: [{ internalType: "uint32", name: "", type: "uint32" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: false,
          inputs: [
            { internalType: "address", name: "dst", type: "address" },
            { internalType: "uint256", name: "rawAmount", type: "uint256" },
          ],
          name: "mint",
          outputs: [],
          payable: false,
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          constant: true,
          inputs: [],
          name: "mintCap",
          outputs: [{ internalType: "uint8", name: "", type: "uint8" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [],
          name: "minter",
          outputs: [{ internalType: "address", name: "", type: "address" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [],
          name: "mintingAllowedAfter",
          outputs: [{ internalType: "uint256", name: "", type: "uint256" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [],
          name: "name",
          outputs: [{ internalType: "string", name: "", type: "string" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [{ internalType: "address", name: "", type: "address" }],
          name: "nonces",
          outputs: [{ internalType: "uint256", name: "", type: "uint256" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [{ internalType: "address", name: "", type: "address" }],
          name: "numCheckpoints",
          outputs: [{ internalType: "uint32", name: "", type: "uint32" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: false,
          inputs: [
            { internalType: "address", name: "owner", type: "address" },
            { internalType: "address", name: "spender", type: "address" },
            { internalType: "uint256", name: "rawAmount", type: "uint256" },
            { internalType: "uint256", name: "deadline", type: "uint256" },
            { internalType: "uint8", name: "v", type: "uint8" },
            { internalType: "bytes32", name: "r", type: "bytes32" },
            { internalType: "bytes32", name: "s", type: "bytes32" },
          ],
          name: "permit",
          outputs: [],
          payable: false,
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          constant: false,
          inputs: [
            { internalType: "address", name: "minter_", type: "address" },
          ],
          name: "setMinter",
          outputs: [],
          payable: false,
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          constant: true,
          inputs: [],
          name: "symbol",
          outputs: [{ internalType: "string", name: "", type: "string" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [],
          name: "totalSupply",
          outputs: [{ internalType: "uint256", name: "", type: "uint256" }],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: false,
          inputs: [
            { internalType: "address", name: "dst", type: "address" },
            { internalType: "uint256", name: "rawAmount", type: "uint256" },
          ],
          name: "transfer",
          outputs: [{ internalType: "bool", name: "", type: "bool" }],
          payable: false,
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          constant: false,
          inputs: [
            { internalType: "address", name: "src", type: "address" },
            { internalType: "address", name: "dst", type: "address" },
            { internalType: "uint256", name: "rawAmount", type: "uint256" },
          ],
          name: "transferFrom",
          outputs: [{ internalType: "bool", name: "", type: "bool" }],
          payable: false,
          stateMutability: "nonpayable",
          type: "function",
        },
      ],
      tab: 0,
    };
  },
  computed: {
    methodsList() {
      let methods = this.contractAbi;

      methods = methods.filter(
        (methods) => methods.name && methods.type === "function"
      );
      // methods = methods.filter((methods) => methods.name === 'Approval');

      return methods;
    },
  },
  mounted() {
    this.initialize();
  },
  methods: {
    async initialize() {
      // check if metamask is installed
      if (typeof window.ethereum !== "undefined") {
        console.log("MetaMask is installed!");
      }

      const web3 = new Web3(window.ethereum);

      this.accounts = await web3.eth.getAccounts();

      if (this.accounts.length > 0) {
        this.metamaskConnected = true;
      }

      // contract to work on
      const contract = new web3.eth.Contract(
        this.contractAbi,
        this.contractAddress
      );

      // event handling for disconnect wallet
      window.ethereum.on("disconnect", () => {
        console.log("disconnected");
      });

      // check for metamask logged in
      web3.eth.getAccounts(function (err, accounts) {
        if (err != null) console.error("An error occurred: " + err);
        else if (accounts.length == 0)
          console.log("User is not logged in to MetaMask");
        else console.log("User is logged in to MetaMask");
      });

      // const test = contract.name;
      console.log("cotract name", contract.name);
      console.log("cotract methods", contract.methods);
      console.log("contract", contract);
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
      window.ethereum.close();
    },
  },
};
</script>
