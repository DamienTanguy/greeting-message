<template>
  <div>
    <p>{{ greeting }}</p>
    <input v-model="greeting" placeholder="Set greeting" />
    <button v-on:click="setGreeting"> Set Greeting </button>
  </div>
</template>

<script>
  import { ethers } from 'ethers';
  import Greeter from '@/artifacts/contracts/Greeter.sol/Greeter.json'; //import of the ABI to interact with the smart contract

  export default {
    name: 'Greeting',
    data() {
      return {
        greeting: '',
        greeterAddress: '0x0165878A594ca255338adfa4d48449f69242Eb8F' //address of the contract deployement with npx hardhat run scripts/deploy.js ...
      }
    },
    mounted(){
      this.fetchGreeting();
    },
    methods : {
      requestAccount : async function(){ //the user has to connect to our webiste with Metamask
        await window.ethereum.request({method: 'eth_requestAccounts'})
      },
      fetchGreeting : async function(){ //to get the message stored in the blockchain
        if(typeof window.ethereum !== 'undefined') { //Metamask needs to be connected
          const provider = new ethers.providers.Web3Provider(window.ethereum);
          const contract = new ethers.Contract(this.greeterAddress, Greeter.abi, provider); //new instance of the contract to interact with the function of the contract
          try {
            this.greeting = await contract.greet(); //function of the contract
          }
          catch(err) {
            console.log(err);
          }
        }
      },
      setGreeting: async function(){ //to modify the message stored in the blockchain
        if(!this.greeting) return
        if(typeof window.ethereum !== "undefined"){
          await this.requestAccount();
          const provider = new ethers.providers.Web3Provider(window.ethereum);
          const signer = provider.getSigner(); // need a signer to sign the transaction
          const contract = new ethers.Contract(this.greeterAddress, Greeter.abi, signer);

          const transaction = await contract.setGreeting(this.greeting); //function of the contract
          await transaction.wait();
          this.fetchGreeting();
        }
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
