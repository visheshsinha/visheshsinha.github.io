---
title: "Deploy a Smart Contract on Ethereum network"
date: 2021-07-15
draft: true
author: Gitansh Wadhwa
ShowShareButtons: true
ShowReadingTime: true
ShowToc: true
ShowBreadCrumbs: true
tags: ['Ethereum', 'Blockchain']
cover:
    image: "images/eth.jpg"
    alt: "Ethereum"
    relative: false
editPost:
    URL: "https://github.com/gitanshwadhwa28/gitanshwadhwa28.github.io/tree/main/content"
    Text: "Suggest Changes" 
    appendFilePath: true 
---

> Published on GeeksforGeeks: [Link](https://www.geeksforgeeks.org/how-to-simply-deploy-a-smart-contract-on-ethereum/)

Smart contracts are blocks of code that reside on the blockchain. It is like an Ethereum account but there is a critical difference between an external account and a smart contract. Unlike a smart contract, an external account can connect to multiple Ethereum networks (Rinkebey, Kovan, main, etc.) whereas a smart contract is only specific to one individual network (the network it is deployed on). When a smart contract is deployed, it creates an instance (contract account) on the network. One can create multiple instances of a smart contract on the network or multiple networks. Deployment of a smart contract is done by sending a transaction to the network with bytecode.


### Deploying to a Local Network

An emulator can be used to deploy a smart contract on a local network eg. Ganache-cli. It takes care of everything and the user doesn’t have to worry about the security and the gas amount required for transactions since everything is happening on a local test network. All one has to do is pass the ganache provider as an argument to the web3 instance(web3 facilitates the connection between the blockchain network and the js application). 

### Deploying to an actual Ethereum Network

Before deploying a smart contract to an actual Ethereum network make sure the account has some ether in it. Deploying a contract is like sending a transaction and it needs some gas amount to process. Unlike deploying on a local network, transactions will take some time to complete (anywhere between 15 seconds to 5 minutes). Web3 is used to interact with the network the same way it is done in local deployment except customize the provider that will be passed into the web3 instance. Instead of creating our own node that connects to the Ethereum network, one can use Infura. It is a public API that gives access to the Infura node that is already hosted on the Ethereum network. Simply sign-up for Infura and get an endpoint that will be used in the code to deploy the smart contract.

example.sol- Below is the sample solidity code used for testing. All it does is set a public variable as the address of the sender.

```
// Solidity program to implement
// the above approach
pragma solidity ^0.8.4;
 
// Creating a contract named Example
contract Example
{
    // Public variable of type address
    address public manager;
 
    // Constructor function to set manager
    // as address of sender
     constructor()
     {
        manager = msg.sender;
     }
}
```

**Step 1**- Install the required dependencies by running the following commands

> npm i solc@0.8.4 truffle-hdwallet-provider@1.0.17 web3@1.3.5

Make sure to install the same versions for the following scripts to run successfully.

**Step 2**- Sign up for Infura and create a project on a particular Ethereum network to get access to the endpoint. The endpoint will be required to deploy the smart contract on the infura node that is already hosted on the Ethereum network. To create a project on infura-

- Click on create a new project.
- Give it a name.
- Select the network to deploy the smart contract on. 

**Step 3**– Get access to Bytecode and ABI (Compile the smart contract). Solidity compiler gives a huge piece of code as output, one can print the output to console if required. Only the relevant part (relevant for deployment) i.e., bytecode and interface are extracted from the output in the following script.

```
// Javascript file to implement
// the above approach
const path = require("path");
const fs = require("fs");
const solc = require("solc");
 
// remember to change line 8 to your
// own file path. Make sure you have your
// own file name or contract name in line
// 13, 28 and 30 as well.
 
const examplePath = path.resolve(__dirname, "contracts", "example.sol");
const source = fs.readFileSync(examplePath, "utf-8");
 
var input = {
    language: 'Solidity',
    sources: {
        'example.sol': {
            content: source
        }
    },
    settings: {
        outputSelection: {
            '*': {
                '*': ['*']
            }
        }
    }
};
 
var output = JSON.parse(solc.compile(JSON.stringify(input)));
 
var interface = output.contracts["example.sol"]["example"].abi;
 
var bytecode = output.contracts['example.sol']["example"].evm.bytecode.object;
 
module.exports = { interface, bytecode };
```

**Step 4**– Once have access to bytecode and interface, all that is required is to create a provider with own mnemonic phrase and infura endpoint using the truffle-hdwallet-provider that was installed earlier. Create a web3 instance and pass the provider as an argument. Finally, use the deploy method with bytecode as an argument to deploy the smart contract.

```
const HDWalletProvider = require("truffle-hdwallet-provider");
 
// Web3 constructor function.
const Web3 = require("web3");
 
// Get bytecode and ABI after compiling
// solidity code.
const { interface, bytecode } = require("file-path");
 
const provider = new HDWalletProvider(
  "mnemonic phrase",
  // Remember to change this to your own phrase!
  "-"
  // Remember to change this to your own endpoint!
);
 
// Create an instance of Web3 and pass the
// provider as an argument.
const web3 = new Web3(provider);
 
const deploy = async () => {
  // Get access to all accounts linked to mnemonic
  // Make sure you have metamask installed.
  const accounts = await web3.eth.getAccounts();
 
  console.log("Attempting to deploy from account", accounts[0]);
 
  // Pass initial gas and account to use in the send function
  const result = await new web3.eth.Contract(interface)
    .deploy({ data: bytecode })
    .send({ gas: "1000000", from: accounts[0]});
 
  console.log("Contract deployed to", result.options.address);
};
 
deploy();
 
// The purpose of creating a function and
// calling it at the end -
// so that we can use async await instead
// of using promises
```

Output:
> Contract is deployed to 0x8716443863c87ee791C1ee15289e61503Ad4443c

Now the contract is deployed on the network, its functionality can be tested using remix IDE or one can create an interface to interact with the smart contract on the network.