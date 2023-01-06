+++
title = "Cosmos SDK Introduction and First Step for Building Your Own Blockchain"
date = 2023-01-01
[taxonomies]
tags = ["CosmosSDK", "IgniteCLI"]
+++

この記事では以下について解説します。
- Cosmos SDKの概要
- Ignite CLIを用いた基本的な開発フロー
<!-- more -->
## Cosmos SDK Overview
Cosmos SDKは、Proof-of-Stake(PoS)やProof-of-Authority(PoA)のブロックチェーンを開発するためのオープンソースフレームワークで、あるアプリケーションに特化したApplication-specific Blockchainを開発するこができます。  

Application-specific Blockchainは、Cosmos SDKにおいて中心となる思想です。
Ethereumなどの汎用チェーンでdAppsを開発する場合、開発者は自身の開発するdAppのGovernanceの他に、そのdAppの基盤となるEthereumチェーン自体のGovernanceも考慮しなければなりません。(2 LayerのGovernance)  
一方Application-specific BlockchainではBlockchainレイヤーからアプリケーションを開発できるため、1 LayerのGovernanceのみを考慮すれば良いことになります。  

dApp同士のTokenの互換性はどうでしょうか？  
Ethereumでは[ERC-20](https://ethereum.org/ja/developers/docs/standards/tokens/erc-20/)や[ERC-721](https://ethereum.org/ja/developers/docs/standards/tokens/erc-721/)というToken規格に従っていれば、dApps間でそれぞれのTokenを扱うことができ、同じチェーン上にdAppが存在しているためTransferなどの操作も可能です。  
一方Application-specific Blockchainを含む異なるGovernanceのチェーン間のToken操作はBridgeが必用となり厄介です。
そこで生み出されたのが、 [Inter-Blockchain Communication Protocol (IBC)](https://ibcprotocol.org/)です。
IBCは、2つのブロックチェーン間でPermissionlessに認証とデータ転送を処理するためのProtocolで、機能については[Interchain Standard(ICS)](https://github.com/cosmos/ibc#interchain-standards)で定められています。
例えば[ICS-20](https://github.com/cosmos/ibc/blob/main/spec/app/ics-020-fungible-token-transfer/README.md)では、ERC-20のようなFungible Token Transferが定義されています。
Cosmos SDKを用いればIBCの実装も容易に可能です。

次にApplication-specific Blockchainのアーキテクチャについて述べます。  
アーキテクチャーはモジュラーに設計されており、以下のような構成になります。  

![](https://tutorials.cosmos.network/resized-images/600/academy/2-cosmos-concepts/images/architecture_overview.png)  
Reference: [Cosmos SDK Developer Portal](https://tutorials.cosmos.network/academy/2-cosmos-concepts/1-architecture.html)


- [Terndermint Core](https://tendermint.com/core/)  
Nodeのネットワーキング機能の担うNetworkking LayerとそのNodeの合意形成(PoS)のを担うConsensus Layerを担います。
Byzantine Fault Tolerance(BFT)があり、Nodeの1/3が動作しなくともアプリケーションが機能することを保証します。  
Terndermint Coreを用いることで、開発者はNetworking LayerとConsensus Layerを実装する必用がなくなります。
またTendermint Coreでは、TransactionはBlockの作成時にFinalizeされForkは作成されません。

- ABCI  
Application LayerとコミュニケーションするためのInterfaceを定義します。
アプリケーションとTendermintの実装言語が異なる場合Socketを提供します。


- Cosmos SDK  
Application Layerを担います。
多数の[Module](https://docs.cosmos.network/main/modules)が提供されており、それらを用いて独自のアプリケーションを開発することができます。





```
proto
└── {project_name}
    └── {module_name}
        └── {proto_version}
            ├── {module_name}.proto
            ├── event.proto
            ├── genesis.proto
            ├── query.proto
            └── tx.proto
```

```
x/{module_name}
├── client
│   ├── cli
│   │   ├── query.go
│   │   └── tx.go
│   └── testutil
│       ├── cli_test.go
│       └── suite.go
├── exported
│   └── exported.go
├── keeper
│   ├── genesis.go
│   ├── grpc_query.go
│   ├── hooks.go
│   ├── invariants.go
│   ├── keeper.go
│   ├── keys.go
│   ├── msg_server.go
│   └── querier.go
├── module
│   └── module.go
├── simulation
│   ├── decoder.go
│   ├── genesis.go
│   ├── operations.go
│   └── params.go
├── spec
│   ├── 01_concepts.md
│   ├── 02_state.md
│   ├── 03_messages.md
│   └── 04_events.md
├── {module_name}.pb.go
├── abci.go
├── codec.go
├── errors.go
├── events.go
├── events.pb.go
├── expected_keepers.go
├── genesis.go
├── genesis.pb.go
├── keys.go
├── msgs.go
├── params.go
├── query.pb.go
└── tx.pb.go
```

[Cosmos SDK Document - Recommended Folder Structure](https://docs.cosmos.network/main/building-modules/structure)

core concepts
- Accounts



## Ignite CLI
Based on 
https://docs.ignite.com/guide/blog

- Scaffolding Chain
- Query
- Message
- Event



## References
- [Cosmos SDK Document](https://docs.cosmos.network/main)
- [Cosmos SDK - Developer Portal](https://tutorials.cosmos.network/)
- [Ignite CLI | Ignite Cli Docs](https://docs.ignite.com/)

