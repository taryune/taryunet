+++
title = "Cosmos SDK Introduction and First Step for Building Your Own Blockchain"
date = 2023-01-10
[taxonomies]
tags = ["CosmosSDK", "IgniteCLI"]
+++
## TL;DR
- Cosmos SDK Overview
  - Application-specific Blockchain
  - Inter-blockchain Communication Protocol
- Building the blockchain with Ignite CLI
  - Installing Ignite CLI
  - Store Object
  - Create a Message
  - Error Handling
  - Event Handling
<!-- more -->

## Cosmos SDK Overview
Cosmos SDK is an open source framework for developing Proof-of-Stake (PoS) and Proof-of-Authority (PoA) blockchains. Application-specific Blockchain is an open source framework for developing application-specific blockchains.  

Application-specific Blockchain is the core concept behind the Cosmos SDK.
When developing dApps on a general-purpose chain such as Ethereum, developers must consider not only the governance of their own dApp, but also the governance of the underlying Ethereum chain itself. (2 Layers Governance)  
On the other hand, application-specific blockchain allows applications to be developed from the Blockchain layer, so only 1 Layer of Governance needs to be considered.

How about Token compatibility between dApps?  
In Ethereum, there are token standards such as [ERC-20](https://ethereum.org/ja/developers/docs/standards/tokens/erc-20/) and [ERC-721](https://ethereum.org/ja/developers/docs/standards/tokens/erc-721/), so it is possible to be handled between dApps in on the same chain.
On the other hand, Token operations between different Governance chains, including application-specific blockchains, require a Bridge, which can be troublesome.
This is where the  [Inter-Blockchain Communication Protocol (IBC)](https://ibcprotocol.org/) comes in.
IBC is a protocol for handling permissionless authentication and data transfer between two blockchains, and its functions are defined in the Interchain Standard (ICS).
For example, [ICS-20](https://github.com/cosmos/ibc/blob/main/spec/app/ics-020-fungible-token-transfer/README.md) defines fungible token transfer like ERC-20.
And, IBC can be easily implemented with Cosmos SDK.

### Architecture
Next, we describe the architecture of the application-specific blockchain.  
The architecture is designed to be modular.

{{ figure(src="https://tutorials.cosmos.network/resized-images/600/academy/2-cosmos-concepts/images/architecture_overview.png",
          style="width: 100%;",
          caption="Reference: Cosmos SDK Developer Portal - Introduction to Cosmos",
          caption_link="https://tutorials.cosmos.network/academy/2-cosmos-concepts/1-architecture.html") }}

- [Terndermint Core](https://tendermint.com/core/)  
Tendermint modules attend to consensus and networking, which are important components of any blockchain.
This frees developers to focus on the application level without descending into lower-level blockchain concerns such as peer discovery, consensus, and transaction finalization.  
Without Tendermint, developers would be forced to build software to address these concerns, which would add additional time, complexity, and cost to the development of their applications.  
Also, with Terndermint Core, Transactions are finalized at block creation and fork is never created.

- ABCI  
ABCI defines interfaces to communicate with the Application Layer.
Provides a Socket when the implementation language of the Application and Tendermint are different.

- Cosmos SDK  
Cosmos SDK provides a rich set of [modules](https://docs.cosmos.network/main/modules) such as governance, tokens, and IBC.
So, developers are able to build their application-specific blockchain easily by using these modules.

## Building the Blockchain
[Ignite CLI](https://ignite.com/) is a feature-packed tool to accelarate blockchain development.
It allows you to scaffold a blockchain with a single command and provides local execution environment.

In this tutorial, you create a blog chain, let you write and read blog post from the blockchain.
The version of Ignite CLI is `0.25.2`.  
Full source code is here ->
[GitHub Repository](https://github.com/taryune/cosmos-sdk-tutorial-blog)

### Install Ignite CLI
First, install Ignite CLI using the following command:
```console
curl https://get.ignite.com/cli! | bash
```
Using macOS? write permisson is required.
Run the following command:

```console
sudo curl https://get.ignite.com/cli! | sudo bash
```

If you wanna install specific veriosn of Ignite CLI, like this:

```console
curl https://get.ignite.com/cli@v0.25.2! | bash
```


### Create your blog chain
Now, create a new blockchain named `blog` with `ignite scaffold` command.
You can see more details about `ignite scaffold` [here](https://docs.ignite.com/cli#ignite-scaffold)
```console
ignite scaffold chain blog
```
Then the following structure file will be created at `blog/`.  
A description of each folder can be found [here](https://docs.cosmos.network/v0.47/building-modules/structure).  
The main directories you editting are `proto/` and `x/`.
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

### Store Object
Create the following store that store data about a blog:  
`PostCount`: The post counter, `uint64`  
`StoredPost`: Blog posts,`Map`  

**PostCount**
```proto
message PostCount {
  uint64 count = 1; 
}
```
Scaffold `PostCount` with Ignite CLI.  
Be sure to `git commit` before and after executing the command.
Ignite CLI does not provide a command to revert a scaffold command, so you need to manage it with Git.

```console
ignite scaffold single postCount count:uint \
    --module blog \
    --no-message
```
To scaffold CRUD for data stored in a single location run `ignite scaffold single NAME [field]...`  

You must add `--no-message.`, if you wanna control object by the application.
If you omit it, Ignite CLI creates an sdk.Msg and an associated service whose purpose is to overwrite your `PostCount` object.

Also, in `ignite scaffold`, the default type of `field` is string, so declare an integer type with `:int`.  

You can find more detail about `ignite scaffold single` [here](https://docs.ignite.com/cli#ignite-scaffold-single)

**StoredPost**
```proto
message StoredPost {
  string index = 1; 
  string title = 2; 
  string body = 3; 
}
```
Run `ignite scaffold map NAME [field]...` in the same way.  
Index field can be specified with `--index`. In this case, `index` is used.
```console
ignite scaffold map storedPost title body \
  --index index \
  --module blog  \
  --no-message
```

Set the default genesis value start.  
In default, `PostCount` is set as nullable. It is not correct. So you need to fix like this:

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
When you edit `.proto` files, you need to recompile them using following command:
```console
ignite generate proto-go
```

Then, set genesis value, `PostCount.Count = 0`.
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

Fix other errors, due to the change of type.
[Diffs](https://github.com/taryune/cosmos-sdk-tutorial-blog/commit/458c3a2192ffeb724b7ab5df3caf362d7c7a7428?diff=split)
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

### Create a Message
Create a message to write to `StoredPost`.
Run following command to create `createPost`:
```console
ignite scaffold message createPost title body \
  --module blog \
  --response postIndex
```
You can set response field by using `--response` option. ([more details](https://docs.ignite.com/cli#ignite-scaffold-message))
By running `ignite scaffold message`, `/x/[module]/keeper/msg_server_[name].go` will be created. (`msg_server_create_post.go` at this time)  
Then, impliment `CreatePost`.

**About Keeper**  
[Mutistore and Keepers](https://tutorials.cosmos.network/academy/2-cosmos-concepts/7-multistore-keepers.html)  
Keeper is responsible for accessing all stores in a Module.  
Stores defined in the module are read/written only by the methods defined in the keeper.
If you come from a module-view-controller (MVC) world, then it helps to think of the keeper as the controller.

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
Create `msg_server_create_post_test.go`, and write a test.
([Diffs](https://github.com/taryune/cosmos-sdk-tutorial-blog/commit/aa52afc817c66047d0f9ab6df4eba54f6dd30752))
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

Run test:
```console
cd x/blog/keeper && go test
```
Let's get the chain up and running in your local environment.
```console
ignite chain serve -r
```
Sending `create-post` transaction by running following command:  
You can refer to `blogd tx blog -h` for executable commands.
```console
blogd tx blog create-post "Test" "This is a test"  --from alice
```

Check if there is the blog post:
You can refer to `blogd q blog -h` for executable commands.
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
Congrats!

### Handle Errors
Impliment validation `Title` and `Body` of the `CreatePost`.([Diffs](https://github.com/taryune/cosmos-sdk-tutorial-blog/commit/6df92a8c682fd199ec281be34f83339332de1f8d))  
Define the errors.
- [x/blog/types/errors.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/6df92a8c682fd199ec281be34f83339332de1f8d/x/blog/types/errors.go)
```go
var (
	ErrSample           = sdkerrors.Register(ModuleName, 1100, "sample error")
	ErrMissingPostTitle = sdkerrors.Register(ModuleName, 1101, "title is missing")
	ErrMissingPostBody  = sdkerrors.Register(ModuleName, 1102, "body is missing")
)

```
Create `x/blog/types/full_post.go` and implement `Validate()`.
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

Add validation before storing objects.
- [x/blog/keeper/msg_server_create_post.go](https://github.com/taryune/cosmos-sdk-tutorial-blog/blob/6df92a8c682fd199ec281be34f83339332de1f8d/x/blog/keeper/msg_server_create_post.go)
```go
err := storedPost.Validate()
if err != nil {
  return nil, err
}
```

Write some tests.
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
Run test.
```console
cd x/blog/keeper && go test
```

### Handle Events
By emitting an event, it is possible to log the event information in the transaction.
([Diffs](https://github.com/taryune/cosmos-sdk-tutorial-blog/commit/03df1c10a8b4425c431f8dbe36248c20c61d923c))
Let's define `new-post-created` event.
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

Emit event after storing objects.
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

Write a test.
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

Check if the event is emitted.
```console
blogd tx blog create-post "Test" "This is a test"  --from alice
```

```console
...
txhash: FB80FE1BF5A3FA7CBA588F9A7A064B72A1B29CFC0D8B16AB52180BFE19119CF9
```
Query `txhash`.
```console
blogd q tx FB80FE1BF5A3FA7CBA588F9A7A064B72A1B29CFC0D8B16AB52180BFE19119CF9 --output json | jq
```
`new-post-created` was emitted!
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
