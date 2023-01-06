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


### Architecture
次にApplication-specific Blockchainのアーキテクチャについて述べます。  
アーキテクチャーはモジュラーに設計されており、以下のような構成になります。  

{{ figure(src="https://tutorials.cosmos.network/resized-images/600/academy/2-cosmos-concepts/images/architecture_overview.png",
          style="width: 100%;",
          caption="Reference: Cosmos SDK Developer Portal - Introduction to Cosmos",
          caption_link="https://tutorials.cosmos.network/academy/2-cosmos-concepts/1-architecture.html") }}

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

## Building the Blockchain
[Ignite CLI](https://ignite.com/)を使って環境を作ってい行きます。
Ignite CLIはCosmos SDKを用いたBlockchainの開発環境です。
コマンド1つでBlockchainをScaffoldingできたり、Local環境での実行を行えたりします。

Blogを作成するシンプルなチュートリアルとなっています。
記事作成時点でのIgnite CLIのVersionは`0.25.2`です。  
完成版のRepositoryはこちら: 
[GitHub](https://github.com/taryune/cosmos-sdk-tutorial-blog)

### Install Ignite CLI
まずIginite CLIをインストールします。
```console
curl https://get.ignite.com/cli! | bash
```
MacOSXの場合は別途Permissionが必要なため以下の手順でインストールします。
```console
sudo curl https://get.ignite.com/cli! | sudo bash
```

記事作成時点でのIgnite CLIのVersionは`0.25.2`です。  
Versionを指定したい場合、以下のように指定することができます。
```console
curl https://get.ignite.com/cli@v0.25.2! | bash
```


### Create your blog chain
`blog`というChainをScaffoldingしていきます。  
`ignite scaffold`についてのドキュメントは[こちら](https://docs.ignite.com/cli#ignite-scaffold)
```console
ignite scaffold chain blog
```
すると`blog/`に以下のようなストラクチャのファイルが作成されます。  
各フォルダの解説については[こちら](https://docs.cosmos.network/v0.47/building-modules/structure)にあります。
主に編集するディレクトリは`proto/`と`x/`になります
```console
blog
|+ .git/
|+ .github/
|+ app/
|+ cmd/
|+ docs/
|- proto/
 |- blog/
  |- blog/
   |  genesis.proto
   |  params.proto
   |  query.proto
   |  tx.proto
|+ testutil/
|+ ts-client/
|+ vue/
|- x/
 |- blog/
  |+ client/
  |+ keeper/
  |+ simulation/
  |+ types/
  |  genesis.go
  |  genesis_test.go
  |  module.go
  |  module_simulation.go
|  .gitignore
|  config.yml
|  go.mod
|  go.sum
|  readme.md
```

### Create Stores

```console
ignite scaffold single postCount count:uint \
    --module blog \
    --no-message
```

```console
ignite scaffold map storedPost title body \
  --index index \
  --module blog  \
  --no-message
```

[diff](https://github.com/taryune/cosmos-sdk-tutorial-blog/commit/458c3a2192ffeb724b7ab5df3caf362d7c7a7428?diff=split)
- [proto/blog/blog/genesis.proto](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/458c3a2192ffeb724b7ab5df3caf362d7c7a7428/proto/blog/blog/genesis.proto)
```proto,  hl_lines=4
// GenesisState defines the blog module's genesis state.
message GenesisState {
  Params params = 1 [(gogoproto.nullable) = false];
  PostCount postCount = 2 [(gogoproto.nullable) = false];
  repeated StoredPost storedPostList = 3 [(gogoproto.nullable) = false];
  // this line is used by starport scaffolding # genesis/proto/state
}
```
- [x/blog/types/genesis.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/458c3a2192ffeb724b7ab5df3caf362d7c7a7428/x/blog/types/genesis.go)
```go, hl_lines=2-5
func DefaultGenesis() *GenesisState {
	return &GenesisState{
		PostCount: PostCount{
			Count: uint64(0),
		},
		StoredPostList: []StoredPost{},
		// this line is used by starport scaffolding # genesis/types/default
		Params: DefaultParams(),
	}
}
```


- [x/blog/genesis.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/458c3a2192ffeb724b7ab5df3caf362d7c7a7428/x/blog/genesis.go)
```go, hl_lines=3 20
// InitGenesis initializes the module's state from a provided genesis state.
func InitGenesis(ctx sdk.Context, k keeper.Keeper, genState types.GenesisState) {
	k.SetPostCount(ctx, genState.PostCount)
	// Set all the storedPost
	for _, elem := range genState.StoredPostList {
		k.SetStoredPost(ctx, elem)
	}
	// this line is used by starport scaffolding # genesis/module/init
	k.SetParams(ctx, genState.Params)
}
```
```go, hl_lines=9
// ExportGenesis returns the module's exported genesis
func ExportGenesis(ctx sdk.Context, k keeper.Keeper) *types.GenesisState {
	genesis := types.DefaultGenesis()
	genesis.Params = k.GetParams(ctx)

	// Get all postCount
	postCount, found := k.GetPostCount(ctx)
	if found {
		genesis.PostCount = postCount
	}
	genesis.StoredPostList = k.GetAllStoredPost(ctx)
	// this line is used by starport scaffolding # genesis/module/export

	return genesis
}
```

### Create a Message
```console
ignite scaffold message createPost title body \
  --module blog \
  --response postIndex
```



### Handle Errors


### Handle Events

## References
- [Cosmos SDK Document](https://docs.cosmos.network/main)
- [Cosmos SDK - Developer Portal](https://tutorials.cosmos.network/)
- [Ignite CLI | Ignite Cli Docs](https://docs.ignite.com/)

