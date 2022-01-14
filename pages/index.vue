<template>
  <div>
    <!-- Navbar -->
    <div class="w-full bg-black text-white p-3 flex flex-row justify-between">
      <h1 class="text-xl">Hamood Token</h1>
      <div>
        {{ account }}
      </div>
    </div>

    <!-- Forms -->
    <div class="right-0 left-0 mx-auto w-1/2 mt-24 bg-gray-100 p-10 rounded-xl">
      <div class="flex flex-row justify-between">
        <button
          :class="type == 'buy' ? 'bg-purple-400 text-white' : ''"
          class="rounded-md py-1 px-3 border-2 border-purple-400"
          @click="type = 'buy'"
        >
          Buy
        </button>

        <button
          :class="type == 'sell' ? 'bg-purple-400 text-white' : ''"
          class="rounded-md py-1 px-3 border-2 border-purple-400"
          @click="type = 'sell'"
        >
          Sell
        </button>
      </div>
      <h1 class="text-2xl text-center mb-5 capitalize">{{ type }} hamood token</h1>
      <BuyForms
        v-if="type == 'buy'"
        :action="buyTokens"
        :ethBalance="ethBalance"
        :tokenBalance="tokenBalance"
      />
      <SellForms
        v-else
        :action="sellTokens"
        :ethBalance="ethBalance"
        :tokenBalance="tokenBalance"
      />
    </div>
  </div>
</template>

<script>
import { ethers, providers } from "ethers";

// GET ABIS
import EthSwap from "@/abis/EthSwap.json";
import Token from "@/abis/Token.json";

export default {
  name: "IndexPage",
  data() {
    return {
      type: "buy",
      account: "",
      ethBalance: "",
      tokenBalance: "0",
      token: {},
      ethswap: {},
      ethAmount: 0,
      loading: false,
    };
  },
  computed: {
    tokenAmount() {
      return this.ethAmount * 100;
    },
  },
  async mounted() {
    await this.loadWeb3();
    await this.loadContracts();
  },
  methods: {
    async loadWeb3() {
      // Load MetaMask Account
      if (window.ethereum) {
        console.log(ethers);
        window.web3 = new ethers.providers.Web3Provider(window.ethereum);
      } else {
        window.alert("Please Install MetaMask");
      }
    },
    async loadContracts() {
      const web3 = window.web3;

      const accounts = await window.ethereum.request({
        method: "eth_requestAccounts",
      });
      this.account = accounts[0];

      // Get Balance
      const balance = await web3.getBalance(this.account);
      this.ethBalance = ethers.utils.formatEther(balance);

      // Load Token contract
      const networkId = web3.network.chainId;
      const tokenData = Token.networks[networkId];
      const token = new ethers.Contract(tokenData.address, Token.abi, web3);
      if (token) {
        this.token = token;
        // Get The Balance of Connected Account
        const tokenBalance = await token.balanceOf(this.account);
        this.tokenBalance = ethers.utils.formatEther(tokenBalance);
      }

      // Load EthSwap contract
      const ethSwapData = EthSwap.networks[networkId];
      const ethswap = new ethers.Contract(
        ethSwapData.address,
        EthSwap.abi,
        web3
      );
      if (ethswap) {
        this.ethswap = ethswap;
      }
    },

    async buyTokens(ethAmount) {
      // Get A Signer
      const signer = window.web3.getSigner();
      const signContract = await this.ethswap.connect(signer);
      const amount = ethers.utils.parseEther(ethAmount.toString());

      const transaction = await signContract.buyTokens({
        value: amount,
        from: this.account,
      });
      // window.web3.once(transaction.hash, (transaction) => {
      //   console.log("DONNE")
      // })
      await transaction.wait().then((receipt) => {
        location.reload();
      });
    },

    async sellTokens(tokenAmount) {
      const signer = window.web3.getSigner();
      const ethSwapSign = await this.ethswap.connect(signer);

      let abi = ["function approve(address _spender, uint256 _value) public returns (bool success)"]
      const tokenContract = new ethers.Contract(this.token.address, abi, signer)

      const amount = ethers.utils.parseEther(tokenAmount.toString());
       console.log(tokenAmount)

      //First we have to approve this transaction
      const approval = await tokenContract.approve(this.ethswap.address, amount)
      await approval.wait()

      const transaction = await ethSwapSign.sellTokens(amount, {from: this.account})

      await transaction.wait()
    }
  },
};
</script>
