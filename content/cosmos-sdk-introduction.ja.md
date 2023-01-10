+++
title = "Cosmos SDK Introduction and First Step for Building Your Own Blockchain"
date = 2023-01-01
[taxonomies]
tags = ["CosmosSDK", "IgniteCLI"]
+++
## TL;DR
この記事では以下について解説します。
- Cosmos SDKの概要
  - Application-specific Blockchain
  - Inter-blockchain Communication Protocol
- Ignite CLIを用いた基本的な開発フロー
  - Ignite CLIのインストール
  - Store Objectの定義
  - Messageの作成
  - Error Handling
  - Event Handling
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
Blogの情報を保存する以下のようなStoreを作成します。

`PostCount`: ブログの記事数のCounter, `uint64`  
`StoredPost`: ブログ記事の`Map`  

**PostCount**
```proto
message PostCount {
  uint64 count = 1; 
}
```
Ignite CLIでScaffoldします。
このとき注意する点としては、**コマンド実行前後にかならずCommitを行うようにしましょう。**
Ignite CLIでは、ScaffoldをRevertするコマンドが用意されていないので、Gitで管理する必用があります。
```console
ignite scaffold single postCount count:uint \
    --module blog \
    --no-message
```
単一のStoreを作成するときは、`ignite scaffold single NAME [field]...` を用います。  
`no-message`のフラグを省略すると、`PostCount` Objectを上書きするサービスも作成されていまします。
今回はアプリケーション内でコードを記述し制御したいため追加する必用があります。
また、`ignite scaffold` では`field`のdefault typeはstringとなっているため`:int`で整数型を宣言します。
`ignite scaffold`についてのドキュメントは[こちら](https://docs.ignite.com/cli#ignite-scaffold)です。

**StoredPost**
```proto
message StoredPost {
  string index = 1; 
  string title = 2; 
  string body = 3; 
}
```
同様に`ignite scaffold map NAME [field]...` を実行します。  
Indexの変数については`--index`で指定可能です。今回は`index`としています。
```console
ignite scaffold map storedPost title body \
  --index index \
  --module blog  \
  --no-message
```

次に`PostCount`のGenesis valueを設定します。
デフォルトでは、`PostCount`が`nullable`に設定されているので修正します。

- [proto/blog/blog/genesis.proto](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/458c3a2192ffeb724b7ab5df3caf362d7c7a7428/proto/blog/blog/genesis.proto)
```proto
// GenesisState defines the blog module's genesis state.
message GenesisState {
  Params params = 1 [(gogoproto.nullable) = false];
  PostCount postCount = 2 [(gogoproto.nullable) = false];
  repeated StoredPost storedPostList = 3 [(gogoproto.nullable) = false];
  // this line is used by starport scaffolding # genesis/proto/state
}
```
protoファイルを変更したので、再コンパイルします。
```console
ignite generate proto-go
```
次にGenesisでのValueを設定します。今回は`PostCount.Count=0`としています。
- [x/blog/types/genesis.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/458c3a2192ffeb724b7ab5df3caf362d7c7a7428/x/blog/types/genesis.go)
```go
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

型の修正を行います。
- [x/blog/genesis.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/458c3a2192ffeb724b7ab5df3caf362d7c7a7428/x/blog/genesis.go)
```go
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
```go
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
その他テストなどの修正差分は[こちら](https://github.com/taryune/cosmos-sdk-tutorial-blog/commit/458c3a2192ffeb724b7ab5df3caf362d7c7a7428?diff=split)です。

### Create a Message
Storeを変更するMessage作成します。
以下のコマンドで`createPost`というMessageを作成します。
`--response`で返すfield名を指定できます。（[Document](https://docs.ignite.com/cli#ignite-scaffold-message)）
```console
ignite scaffold message createPost title body \
  --module blog \
  --response postIndex
```
`ignite scaffold message`を実行すると`/x/[module]/keeper/msg_server_[name].go`が作成されるので実装していきます。
(今回は`msg_server_create_post.go`)  

**Keeperについて**  
[Mutistore and Keepers](https://tutorials.cosmos.network/academy/2-cosmos-concepts/7-multistore-keepers.html)  
KeeperはModule内のすべてのStoreのアクセスを担います。
Module内で定義されたStoreは、Keeperで定義されたMethodによってのみ読み書きが行われます。
Module-View-Controller(MVC)で言うControllerの役割というとイメージが容易でしょう。

- [x/blog/keeper/msg_server_create_post.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/aa52afc817c66047d0f9ab6df4eba54f6dd30752/x/blog/keeper/msg_server_create_post.go)
```go
func (k msgServer) CreatePost(goCtx context.Context, msg *types.MsgCreatePost) (*types.MsgCreatePostResponse, error) {
	ctx := sdk.UnwrapSDKContext(goCtx)

  // Get postCount from store
	postCount, found := k.Keeper.GetPostCount(ctx)
	if !found {
		panic("postCount is not found")
	}

	newIndex := strconv.FormatUint(postCount.Count, 10)
	storedPost := types.StoredPost{
		Index: newIndex,
		Title: msg.Title,
		Body:  msg.Body,
	}

  // Store storedPost and postCount
	k.Keeper.SetStoredPost(ctx, storedPost)
	postCount.Count++
	k.Keeper.SetPostCount(ctx, postCount)

  // Return newIndex
	return &types.MsgCreatePostResponse{
		PostIndex: newIndex,
	}, nil
}
```
`msg_server_create_post_test.go`を作成し、テストを書いていきます。
- [x/blog/keeper/msg_server_create_post_test.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/aa52afc817c66047d0f9ab6df4eba54f6dd30752/x/blog/keeper/msg_server_create_post_test.go)
```go
package keeper_test

import (
	keepertest "blog/testutil/keeper"
	"blog/x/blog"
	"blog/x/blog/keeper"
	"blog/x/blog/types"
	"context"
	"testing"

	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/stretchr/testify/require"
)

func setupMsgServerCreatePost(t testing.TB) (types.MsgServer, keeper.Keeper, context.Context) {
	k, ctx := keepertest.BlogKeeper(t)
	blog.InitGenesis(ctx, *k, *types.DefaultGenesis())
	return keeper.NewMsgServerImpl(*k), *k, sdk.WrapSDKContext(ctx)
}

func TestCreatePostSuccess(t *testing.T) {
	msgServer, _, context := setupMsgServerCreatePost(t)
	createResponse, err := msgServer.CreatePost(context, &types.MsgCreatePost{
		Title: "Test",
		Body:  "This is a test",
	})
	require.Nil(t, err)
	require.EqualValues(t, types.MsgCreatePostResponse{
		PostIndex: "0",
	}, *createResponse)
}
```
差分は[こちら](https://github.com/taryune/cosmos-sdk-tutorial-blog/commit/aa52afc817c66047d0f9ab6df4eba54f6dd30752)  
テストを実行し、結果を確認します。
```console
cd x/blog/keeper && go test
```
またローカル環境でチェーンを立ち上げて実行してみましょう。
```console
ignite chain serve -r
```
`create-post`を実行します。  
`blogd tx blog`でmessageを送ることができます。
実行可能なコマンドに関しては`blogd tx blog -h`で参照できます。
```console
blogd tx blog create-post "Test" "This is a test"  --from alice
```
queryでpostがあるか確認します。  
`blogd q query`でqueryを送ることができます。
実行可能なコマンドに関しては`blogd q blog -h`で参照できます。

```console
blogd q blog list-stored-post
```
```console
pagination:
  next_key: null
  total: "0"
storedPost:
- body: This is a test
  index: "0"
  title: Test
```


### Handle Errors
今回は`CreatePost`の`Title`と`Body`をValidateするコードを記述します。  

エラー定義を記述します。
- [x/blog/types/errors.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/6df92a8c682fd199ec281be34f83339332de1f8d/x/blog/types/errors.go)
```go
var (
	ErrSample           = sdkerrors.Register(ModuleName, 1100, "sample error")
	ErrMissingPostTitle = sdkerrors.Register(ModuleName, 1101, "title is missing")
	ErrMissingPostBody  = sdkerrors.Register(ModuleName, 1102, "body is missing")
)

```
`x/blog/types/full_post.go`を作成しValidateのコードを記述します。
- [x/blog/types/full_post.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/6df92a8c682fd199ec281be34f83339332de1f8d/x/blog/types/full_post.go)
```go
package types

import "fmt"

func (storedPost StoredPost) GetPostTitle() (title string, err error) {
	if len(storedPost.Title) <= 0 {
		return title, ErrMissingPostTitle.Wrap(fmt.Sprintf("index = %s", storedPost.Index))
	}
	return title, nil
}

func (storedPost StoredPost) GetPostBody() (body string, err error) {
	if len(storedPost.Body) <= 0 {
		return body, ErrMissingPostBody.Wrap(fmt.Sprintf("index = %s", storedPost.Index))
	}
	return body, nil
}

func (storedPost StoredPost) Validate() (err error) {
	_, err = storedPost.GetPostTitle()
	if err != nil {
		return err
	}
	_, err = storedPost.GetPostBody()
	if err != nil {
		return err
	}
	return err
}
```

Validateのコードを追記します。
- [x/blog/keeper/msg_server_create_post.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/6df92a8c682fd199ec281be34f83339332de1f8d/x/blog/keeper/msg_server_create_post.go)
```go
err := storedPost.Validate()
if err != nil {
  return nil, err
}
```
テストを追記します。
- [x/blog/keeper/msg_server_create_post_test.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/6df92a8c682fd199ec281be34f83339332de1f8d/x/blog/keeper/msg_server_create_post_test.go)
```go
func TestCreatePostBadTitle(t *testing.T) {
	msgServer, _, context := setupMsgServerCreatePost(t)
	createResponse, err := msgServer.CreatePost(context, &types.MsgCreatePost{
		Title: "",
		Body:  "This is a test",
	})
	require.Nil(t, createResponse)
	require.EqualError(t, err, "index = 0: title is missing")
}

func TestCreatePostBadBody(t *testing.T) {
	msgServer, _, context := setupMsgServerCreatePost(t)
	createResponse, err := msgServer.CreatePost(context, &types.MsgCreatePost{
		Title: "Test",
		Body:  "",
	})
	require.Nil(t, createResponse)
	require.EqualError(t, err, "index = 0: body is missing")
}
```
テストを実行し、結果を確認します。
```console
cd x/blog/keeper && go test
```
差分は[こちら](https://github.com/taryune/cosmos-sdk-tutorial-blog/commit/6df92a8c682fd199ec281be34f83339332de1f8d)

### Handle Events
Eventを発火させることで、Transactoin logにEvent情報を残すことが可能です。
実際に`new-post-created`というEventを発火させてログを確認していきます。

Eventを定義します。
- [x/blog/types/keys.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/03df1c10a8b4425c431f8dbe36248c20c61d923c/x/blog/types/keys.go)
```go
// CreatePost Events
const (
	PostCreatedEventType = "new-post-created"
	PostCreatedCreator   = "creator"
	PostCreatedPostIndex = "post-index"
	PostCreatedTitle     = "title"
	PostCreatedBody      = "body"
)
```

Storeに書き込んだあとEventを発火させます。
- [x/blog/keeper/msg_server_create_post.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/03df1c10a8b4425c431f8dbe36248c20c61d923c/x/blog/keeper/msg_server_create_post.go)
```go
ctx.EventManager().EmitEvent(
		sdk.NewEvent(types.PostCreatedEventType,
			sdk.NewAttribute(types.PostCreatedCreator, msg.Creator),
			sdk.NewAttribute(types.PostCreatedPostIndex, newIndex),
			sdk.NewAttribute(types.PostCreatedTitle, msg.Title),
			sdk.NewAttribute(types.PostCreatedBody, msg.Body),
		),
	)
```
テストを記述していきます。

- [testutil/constants.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/03df1c10a8b4425c431f8dbe36248c20c61d923c/testutil/constants.go)
``` go
package testutil

const (
	Alice = "cosmos1jmjfq0tplp9tmx4v9uemw72y4d2wa5nr3xn9d3"
	Bob   = "cosmos1xyxs3skf3f4jfqeuv89yyaqvjc6lffavxqhc8g"
	Carol = "cosmos1e0w5t53nrq7p66fye6c8p0ynyhf6y24l4yuxd7"
)
```

- [x/blog/keeper/msg_server_create_post_test.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/03df1c10a8b4425c431f8dbe36248c20c61d923c/x/blog/keeper/msg_server_create_post_test.go)
```go
func TestCreatePostEmitted(t *testing.T) {
	msgServer, _, context := setupMsgServerCreatePost(t)
	_, err := msgServer.CreatePost(context, &types.MsgCreatePost{
		Creator: testutil.Alice,
		Title:   "Test",
		Body:    "This is a test",
	})
	require.Nil(t, err)

	ctx := sdk.UnwrapSDKContext(context)
	require.NotNil(t, ctx)
	events := sdk.StringifyEvents(ctx.EventManager().ABCIEvents())
	require.Len(t, events, 1)
	event := events[0]
	require.EqualValues(t, sdk.StringEvent{
		Type: "new-post-created",
		Attributes: []sdk.Attribute{
			{Key: "creator", Value: testutil.Alice},
			{Key: "post-index", Value: "0"},
			{Key: "title", Value: "Test"},
			{Key: "body", Value: "This is a test"},
		},
	}, event)
}

```
差分は[こちら](https://github.com/taryune/cosmos-sdk-tutorial-blog/commit/03df1c10a8b4425c431f8dbe36248c20c61d923c)

CLIで動作を確認します。

```console
blogd tx blog create-post "Test" "This is a test"  --from alice
```

```console
...
txhash: FB80FE1BF5A3FA7CBA588F9A7A064B72A1B29CFC0D8B16AB52180BFE19119CF9
```
`txhash`をqueryします。
```console
blogd q tx FB80FE1BF5A3FA7CBA588F9A7A064B72A1B29CFC0D8B16AB52180BFE19119CF9 --output json | jq
```
Eventが発火していることを確認します。
```json
...
  "logs": [
    {
      "msg_index": 0,
      "log": "",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "action",
              "value": "/blog.blog.MsgCreatePost"
            }
          ]
        },
        {
          "type": "new-post-created",
          "attributes": [
            {
              "key": "creator",
              "value": "cosmos1yf2aal544uf8n0ruecl3ytx066glfjjphskhdh"
            },
            {
              "key": "post-index",
              "value": "1"
            },
            {
              "key": "title",
              "value": "Test"
            },
            {
              "key": "body",
              "value": "This is a test"
            }
          ]
        }
      ]
    }
  ],
...
```



## References
- [Cosmos SDK Document](https://docs.cosmos.network/main)
- [Cosmos SDK - Developer Portal](https://tutorials.cosmos.network/)
- [Ignite CLI | Ignite Cli Docs](https://docs.ignite.com/)

