<template>
  <div>
    <pre>
      @@@@@@ @@@       @@@@@@  @@@  @@@  @@@ @@@      @@@ @@@     
 !@@     @@!      @@!  @@@ @@!  @@!  @@! @@!      @@! !@@     
  !@@!!  @!!      @!@  !@! @!!  !!@  @!@ @!!       !@!@!      
     !:! !!:      !!:  !!!  !:  !!:  !!  !!:        !!:       
 ::.: :  : ::.: :  : :. :    ::.:  :::   : ::.: :   .:        
                                                              
 @@@@@@@@  @@@@@@  @@@@@@@  @@@ @@@  @@@  @@@@@@@             
 @@!      @@!  @@@ @@!  @@@ @@! @@!@!@@@ !@@                  
 @!!!:!   @!@!@!@! @!@  !@! !!@ @!@@!!@! !@! @!@!@            
 !!:      !!:  !!! !!:  !!! !!: !!:  !!! :!!   !!:            
  :        :   : : :: :  :  :   ::    :   :: :: :             
                                                              
  @@@@@@   @@@  @@@ @@@ @@@@@@@@ @@@@@@@ @@@      @@@ @@@     
 @@!  @@@  @@!  @@@ @@! @@!        @@!   @@!      @@! !@@     
 @!@  !@!  @!@  !@! !!@ @!!!:!     @!!   @!!       !@!@!      
 !!:!!:!:  !!:  !!! !!: !!:        !!:   !!:        !!:       
  : :. :::  :.:: :  :   : :: :::    :    : ::.: :   .:  
</pre
    >
    <div class="terminal">
      <a
        href="#"
        @click="deployContract"
        v-if="!started"
        style="margin-top: 2em"
        >Deploy Contract</a
      >
      <pre>{{ terminalContent }}</pre>
    </div>
  </div>
</template>

<script>
import { ethers } from "ethers";
import contractABI from "./contract/contract.json";

export default {
  data() {
    return {
      terminalContent: "",
      started: false,
      isLoading: false,
    };
  },
  methods: {
    async deployContract() {
      if (!window.ethereum) {
        alert("MetaMask not detected!");
        return;
      }

      try {
        const provider = new ethers.BrowserProvider(window.ethereum)
        const signer = await provider.getSigner();
        const network = await provider.getNetwork();

        console.log(network.chainId);

        if (network.chainId !== 80002n && network.chainId !== 1n) {
          // Sepolia and Mainnet chain IDs
          alert(
            "Please switch to the Sepolia or Ethereum mainnet in your MetaMask."
          );
          return;
        }

        this.started = true;
        this.isLoading = true;

        const contractFactory = new ethers.ContractFactory(
          contractABI.abi,
          contractABI.bytecode,
          signer
        );

        this.terminalContent +=
          "\n\n\nDeploying contract with the account " +
          (await signer.getAddress());
        this.terminalContent +=
          "\n\nPlease sign the transaction to deploy the contract ";

        await this.delay(1500);

        const contract = await contractFactory.deploy();

        this.terminalContent += "\n\nHold on, we're deploying the contract";
        await contract.deploymentTransaction().wait(2);

        const address = await contract.getAddress();
        this.terminalContent +=
          "\n\nThe contract is deployed at " + address;
        this.isLoading = false;
      } catch (err) {
        this.terminalContent += `\nError: ${err.message}`;
        this.isLoading = false;
      }
    },
    delay(ms) {
      return new Promise((resolve) => setTimeout(resolve, ms));
    },
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  background-color: black;
}

html {
  height: 100vh;
  background-color: black;
}

html,
body {
  min-height: 100vh;
  height: 100vh;
  color: #55ff55;
  font-family: monospace;
}

body a,
body a:visited,
body a:hover,
body a:active,
body a:focus {
  color: #55ff55;
  border: 1px dashed #55ff55;
  display: inline-block;
  padding: 0.5em 1em;
}
.terminal {
  display: block;
  width: 100%;
}
</style>
