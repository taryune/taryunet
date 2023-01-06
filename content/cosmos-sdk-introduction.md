+++
title = "Cosmos SDK Introduction and First Step for Building Your Own BlockChain"
date = 2023-01-01
[taxonomies]
tags = ["CosmosSDK", "IgniteCLI"]
+++

# What is the Cosmos SDK

The Cosmos SDK is an open-source framework for building multi-asset public Proof-of-Stake (PoS) blockchains, like the Cosmos Hub, as well as permissioned Proof-of-Authority (PoA) blockchains. Blockchains built with the Cosmos SDK are generally referred to as application-specific blockchains.

The goal of the Cosmos SDK is to allow developers to easily create custom blockchains from scratch that can natively interoperate with other blockchains. We envision the Cosmos SDK as the npm-like framework to build secure blockchain applications on top of Tendermint. SDK-based blockchains are built out of composable modules, most of which are open-source and readily available for any developers to use. Anyone can create a module for the Cosmos SDK, and integrating already-built modules is as simple as importing them into your blockchain application. What's more, the Cosmos SDK is a capabilities-based system that allows developers to better reason about the security of interactions between modules. For a deeper look at capabilities, jump to Object-Capability Model.
