---
layout: post
title:  "Hands-on with blockchain - Ethereum"
description: "Brief about Blockchain and hands-on with Ethereum"
date:   2018-01-24 22:00:00
comments: true
image: /images/blockchain.png
categories: [Blockchain, Ethereum]
keywords: "Blockchain, Ethereum, Ubuntu"
---
<h3>What is Blockchain?</h3>

The blockchain is a distributed ledger of economic transactions, which can be programmed to record not only crypto-currency transactions but anything having value. There are many implementation technologies of blockchain but I am going to provide detail on Ethereum on this brief hands-on post. The chain word in blockchain makes it's definition more clear where one block of transactions connects with another block using some part of signature of previous block. The blockchain will start from first block, which is called genesis block and from that block onwards chain of block starts to create whole blockchain network, you can refer below image explaining same concepts.

<image src="/images/ethereum-node-chain.png"></image>

So the blockchain has to be started by someone using genesis block and we will discuss further how genesis block can be configured using genesis.json file when we start our private blockchain locally. The application which runs on blockchain is called DApp (Decentralized Application); whichever current ICO (Initial Coin Offering) startups essentially are DApp offering their own tokens. When we talk about Ethereum, it is one of the blockchain networks with its own protocol. In blockchain, each block is time bound and miners will mine each block of transactions using Proof of Work algorithm to verify authenticity. The current mining is based on Proof of Work (PoW), which requires very high computing power to solve particular mathematical formula/puzzle and whoever first mines the block is the winner while there exists another approach called Proof of Stake (PoS). In PoS, the creator of block is chosen deterministic way so that it is mined by one entity rather than multiple miners are competing with each other to solve the puzzle. The determination of miner can be based on its wealth or stake and that is the reason it is called Proof of Stake and future of blockchain would be based on this approach because it is very cost effective. In this post, we would be using Ethereum as a blockchain platform to run our first simple smart contract; which uses PoW to mine the block.

<h3>Development Environment for Ethereum</h3>

1. Operating System: Ubuntu 16.04 LTS (I have used Windows 10 as well for the same app and it works exactly the same but this post is about Linux)
2. Geth: Using geth, you can run full Ethereum node. It is written in GoLang and it is also known as go-ethereum, you can download it from [here][gethdownload].
3. Mist: It is a kind of blockchain browser where you can browse the DApp, deploy your smart contracts, open Remix (it is a Solidity editor using which you can write smart contract), manage your accounts etc. You can download it from [here][mistdownload], go ahead and select Mist-linux32-0-9-3.deb or Mist-linux64-0-9-3.deb based on your OS architecture.

You do not need any other tools for this hands-on demo but for your real world use cases, you may need other tools.

<h4>Installing Geth on Ubuntu</h4>

You can use below commands to install Geth on ubuntu and for Windows, there is an exe on download page mentioned above; which installs required dependency and prepare command line tool to work with Geth.

{% highlight PowerShell %}

sudo apt-get install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum

{% endhighlight %}

<h4>Installing Mist on Ubuntu</h4>

It is a .deb file and you can double click on it to install Mist on your Ubuntu or you can use dpkg to install downloaded .deb file of Mist.

<h3>Running full Ethereum node locally - Private Network</h3>

Before we start running different commands, you need to create one data directory where you want your private blockchain network to store data. As we are going to create first blockchain node, we need to start it as a genesis block and for that we need to prepare genesis.json file as mentioned below.

{% highlight javascript %}

{
    "config": {
        "chainId": 2000,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
    "coinbase": "0x0000000000000000000000000000000000000001",
    "difficulty": "400",
    "extraData": "",
    "gasLimit": "0x8000000",
    "nonce": "0x0000000000000042",
    "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "timestamp": "0x0",
    "alloc": {
    	"0x89fd6f157c3fb1a40566ca170986cdc49025f9df": {"balance": "0x1337000000000000000000"},
      "0xc6a729e1e3d869e2fcf9199f00764ef00fad023c": {"balance": "0x2337000000000000000000"}
    }
}

{% endhighlight %}

The content of your genesis file should be something similar to above and file should be stored in data directory you created previously. I am going to explain only few properties of genesis file here and for rest you can look online. The 'chainId' is the id you want to give to your block/network, it can be anything above 3 as 1 to 3 are reserved by blockchain for different purpose. The 'difficulty' indicates the effort required to discover valid block, statistically more calculations a Miner must perform to discover a valid block. We should keep 'difficulty' level as low as possible in test network so that our transactions get completed faster. The 'alloc' is the place where we will put default accounts with ether balance when we start our private network. You need to change both account ids as per generated by your geth, below is the command to create new account before starting the node.

{% highlight PowerShell %}

geth --datadir /path/to/your/data/folder account new

Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase: 
Repeat passphrase: 
Address: {9bc0fae0baeb01fe0e441a980acdf61b85e3db9e}


{% endhighlight %}

You need to provide password to account and note the address(id)/password for the account, go ahead and repeat this one more time to generate another account. Once you have two accounts, you can replace two ids in genesis file with these two accounts you generated.

<h4>Initializing genesis Node</h4>

We have now genesis file under data folder and we need to tell Geth about it so next command is to initialize genesis node with genesis file.

{% highlight PowerShell %}
geth --datadir /path/to/your/data/folder init /path/to/your/data/folder/genesis.json
{% endhighlight %}

The above command is the one time initialization of your genesis node, from where your blockchain can be started.

<h4>Running private Node/Network</h4>

{% highlight PowerShell %}
geth --datadir /path/to/your/data/folder --ipcpath ~/.ethereum/geth.ipc
{% endhighlight %}

In above command, you are noticing ipcpath option, which tells geth to put IPC file to specific location. The geth will start with above command and you are running your private blockchain network locally. 

<h3>Using Mist with Private Network</h3>

The Mist comes with its own private blockchain node so when you open Mist, it will start the private node by itself. In our case we want Mist to use our private network rather than starting new one so the default location from where Mist looks for geth.ipc file is ~/.ethereum/geth.ipc. So, now you understand that why we started our geth with ipcpath command line option. Once Mist is opened, you will find that two accounts in Wallet tab: (1) Main Account and (2): Account1. These accounts were created by us initially before we initialized our Geth. We now need to create our first smart contract using Remix - a Solidity editor.

 <h4>Creating first Smart Contract</h4>

 Once Mist is open, click on 'Open Remix IDE' under Develop menu from main toolbar. The Remix editor will open with default contract, which you can override with below contract. The sample contract is for E-Voting where owner of the contract can add candidate and others can vote for the chosen candidate, we can also find total votes of specific candidate by supplying name of the candidate.

{% highlight javascript %}

pragma solidity ^0.4.18;

contract EVoting {
 
  address public owner = msg.sender;
  mapping (bytes32 => uint8) public votes;
  
  bytes32[] public candidates;

  function EVoting() public {
  
  }

  function totalVotesFor(bytes32 candidate) view public returns (uint8) {
    require(validCandidate(candidate));
    return votes[candidate];
  }

  function voteForCandidate(bytes32 candidate) public {
    require(validCandidate(candidate));
    votes[candidate] += 1;
  }
  
  function addCandidate(bytes32 candidate) public returns (bool) {
    if(owner == msg.sender){
        candidates.push(candidate);
        return true;
    }
    return false;
}

  function validCandidate(bytes32 candidate) view public returns (bool) {
    for(uint i = 0; i < candidates.length; i++) {
      if (candidates[i] == candidate) {
        return true;
      }
    }
    return false;
  }
}

{% endhighlight %}


Following are the important functions of above smart contract

Note: We are using Byte32 instead of string data type for candidate names because as of this writing Solidity does not support dynamic data type in mapping where we wanted to map candidate and number of votes.
  
  1. addCandidate - is the function using which contract owner can add new candidate, here we are making sure that only contract owner can add candidate
  2. voteForCandidate - is the function to vote for a specific candidate by supplying candidate name
  3. validCandidate - is the function to check whether candidate is valid candidate by looking at the candidate list
  4. totalVotesFor - it returns number of votes for a specific candidate

You can also notice that read functions have 'view' in its definition and other functions are write functions. The write function will need ether to carry out transaction in blockchain, we will look into it once we deploy the contract. Also Remix editor comes with its own debugger and you can run full contract within javascript based virtual blockchain node but I will not cover that part in this post instead we will deploy and run contract in private blockchain network.

<h3>Deploying Smart Contract to private network using Mist</h3>

Now we have our smart contract ready and we want to deploy it to our network but before doing that we want to make sure that miner is started on our private network. To start the miner, you need to open another terminal window and run below command.

{% highlight PowerShell %}
geth attach
{% endhighlight %}

The above command will open interactive JavaScript console to interact with running blockchain. Once you are in console, you will need to type below command and press enter.

{% highlight PowerShell %}
miner.start(2)
{% endhighlight %}

The number in start method indicates number of threads so you can put any number above 0 based on your processor architecture. There is also stop method as mentioned below to stop the miner.

{% highlight PowerShell %}
miner.stop()
{% endhighlight %}

Now again you can switch back to Mist and find the 'CONTRACTS' tab and click on it. Once you are in 'CONTRACTS' tab you will find 'DEPLOY NEW CONTRACT' button, which you can click. You can select Main account in 'FROM' field and copy paste the smart contract we developed in to 'SOLIDITY CONTRACT SOURCE CODE' field. Once the contract is pasted, from right hand side dropdown select 'EVoting' as contract under 'SELECT CONTRACT TO DEPLOY' panel. Now you can click on 'DEPLOY' button, which will open up a dialog box where you need to provide transaction password for main account you created using 'Geth Account new' command initially. 

<h3>Running Smart Contract using Mist</h3>

Once the contract is deployed, you can verify that your contract is deployed successfully from 'WALLETS' tab by looking at 'LATEST TRANSACTIONS'. You need to allow contract execution to complete with all confirmations and then you can select your contract from 'CONTRACTS' tab. The contract will open up and now you can interact with your contract. You will find two sections there (1) READ FROM CONTRACT and (2) WRITE TO CONTRACT. The second option is where you will be interacting with your contract, you can select 'ADD CANDIDATE' to add new candidate and 'VOTE FOR CANDIDATE' to vote specific candidate. Here we need to remember that the name we need to supply in byte32 encoding, you can use [Hex Converter][hexconverter] to covert string to byte32. You can use read methods similar ways to know total number of votes for a specific candidate. When you execute write method, you will have to provide transaction password to carry out that specific transaction and in real world there would be a transaction charge in terms of ether for different types of transactions.

I have not attached all screenshots for the all above steps to keep post length minimum but if you need screenshots of all above steps, please write it in comments and I will send you across.

[gethdownload]: https://geth.ethereum.org/downloads/
[mistdownload]: https://github.com/ethereum/mist/releases
[hexconverter]: https://codebeautify.org/string-hex-converter
