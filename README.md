# cosmos-notes

1. `Object Capability Model` for security: 
    1. Cosmos has a principle that there will always be some modules that are faulty
    2. For security, we need to follow OCap model: It is a security paradigm used for managing access to resources. It combines the principles of Object oriented programming and capabilities (permissions) to manage access. Two rules:
    3. An object A can send a message to B only if object A holds a reference to B.
    4. An object A can obtain a reference to C only if object A receives a message containing a reference to C
    5. Example: 
    
    ```rust
    // Violation of OCap: ComputeSumValue is given a pointer which gives it 
    // access to modify account
    type AppAccount struct {...}
    account := &AppAccount{
        Address: pub.Address(),
        Coins: sdk.Coins{sdk.NewInt64Coin("ATM", 100)},
    }
    sumValue := externalModule.ComputeSumValue(account)
    
    // Correct way
    sumValue := externalModule.ComputeSumValue(*account)
    ```
    
2. `IBC`: **Inter-Blockchain Communication Protocol**
    1. Blockchains with different applications and architecture specifications become interoperable whether or not they share a validator set.
    2. ![image](https://github.com/user-attachments/assets/2790c12d-6d77-446d-827e-021044ec5c49)
    3. The interchain implements a **modular architecture with two blockchain classes**: **hubs** and **zones**.
        1. **Zones** are heterogeneous blockchains carrying out the authentication of accounts and transactions, the creation and distribution of tokens, and the execution of changes to the chain.
        2. **Hubs** are blockchains designed to connect the so-called zones. Once a zone connects to a hub through an IBC connection, it gets automatic access to the other zones connected to that hub. At this point, data and value can be sent and received between the zones without risk of, for example, double-spending tokens.
3. `CometBFT`: CometBFT is a blockchain application platform which supports state machines in any language
    1. CometBFT is software for securely and consistently replicating an application on many machines. By securely, we mean that CometBFT works as long as less than 1/3 of machines fail in arbitrary ways. By consistently, we mean that every non-faulty machine sees the same transaction log and computes the same state.
    2. CometBFT consists of two chief technical components: a blockchain consensus engine and a generic application interface.
        1. The consensus engine, which is based on Tendermint consensus algorithm, ensures that the same transactions are recorded on every machine in the same order. 
        2. The application interface, called the Application BlockChain Interface (ABCI), enables the transactions to be processed in any programming language.
4. `ABCI`
    1. CometBFT packages the networking and consensus layers of a blockchain and presents an interface to the application layer, the **Application Blockchain Interface (ABCI)**.
    2. The consensus engine runs in one process and controls the state machine, while the application runs in another process. CometBFT is connected to the application by a socket protocol.
    3. ABCI provides a socket for applications written in other languages. If the application is written in the same language as the CometBFT implementation, the socket is not used.
       ![image](https://github.com/user-attachments/assets/631c66e2-3e54-4693-838f-7d37103cd012)  
    4. CometBFT passes confirmed transactions to the application layer through the **Application Blockchain Interface (ABCI)**. The application layer must implement ABCI, which is a socket protocol.
5. Transactions and blocks utilize several key methods and message types:
    1. **`CheckTx`:** CometBFT is agnostic to the transaction interpretation. That’s why application layer needs to implement this method. CometBFT uses this method to ask the application layer if a transaction is valid. Applications implement this function.
    2. **`DeliverTx`:** CometBFT calls the `DeliverTx` method to pass block information to the application layer for interpretation and possible state machine transition.
    3. **`BeginBlock` and `EndBlock` :** `BeginBlock` and `EndBlock` messages are sent through the ABCI even if blocks contain no transactions. This provides positive confirmation of basic connectivity and helps identify time periods with no operations. These methods facilitate the execution of scheduled processes that should always run because they call methods at the application level, where developers define processes.
6. In the Cosmos SDK, keys are stored and managed in an object called a **`keyring`**. The keyring object stores and manages multiple accounts. 
7. Three types of addresses specify a context when an account is used:
    1. [**`AccAddress`**](https://github.com/cosmos/cosmos-sdk/blob/1dba6735739e9b4556267339f0b67eaec9c609ef/types/address.go#L129) identifies users, which are the sender of a message.
    2. [**`ValAddress`**](https://github.com/cosmos/cosmos-sdk/blob/23e864bc987e61af84763d9a3e531707f9dfbc84/types/address.go#L298) identifies validator operators.
    3. [**`ConsAddress`**](https://github.com/cosmos/cosmos-sdk/blob/23e864bc987e61af84763d9a3e531707f9dfbc84/types/address.go#L448) identifies validator nodes that are participating in consensus. Validator nodes are derived using the [**ed25519**](https://www.cryptosys.net/pki/manpki/pki_eccsafecurves.html) curve.
8. In the Cosmos SDK, a **transaction** contains **one or more `messages`**. Modules process the messages after the transaction is included in a block by the consensus layer.
    1. Transactions contain messages, which are processed after being finalized by CometBFT, with each message routed to the appropriate module via MsgServiceRouter.
    2. A message service in the Cosmos SDK is a Protobuf-defined system that manages how modules handle state-changing messages in transactions. It routes messages to the appropriate module logic and Each module has exactly one Protobuf `Msg` service defined in `tx.proto`.
9. A `query` is a request for information (which could be about the network, about an application, or about the application's state) that is made by end-users of an application through an interface.
10. `Modules` are functional components that address application-level concerns such as token management or governance. They can be considered state machines within the larger state machine.
11. Modules implement several elements:
    1. `Interfaces` facilitate communication between modules and the composition of multiple modules into coherent applications.
    2. `Protobuf` provides one `Msg` service to handle messages and one gRPC `Query` service to handle queries.
    3. A `Keeper` is a controller that defines the state and presents methods for updating and inspecting the state.
12. [**`gRPC`**](https://grpc.io/) is a modern, open-source, high-performance framework that supports multiple languages. It is the recommended standard for external clients such as wallets, browsers, and backend services to interact with a node.
13. `Keepers` are the gatekeepers to any stores in the module. It is mandatory to go through a module’s keeper to access a store. A keeper encapsulates the knowledge about the layout of the storage within the store and contains methods to update and inspect it.
14. Protocol Buffers (`Protobuf`) is an open-source, extensible, cross-platform, and language-agnostic method of serializing object data, primarily for network communication and storage.
15. A Cosmos SDK application's core mainly consists of type definitions and constructor functions, comprising a reference to the `BaseApp`, a list of store keys, a list of each module's keepers, a reference to the codec used, and a reference to the module manager.
16. Each Cosmos SDK application contains a state at its root, the `Multistore`. It is subdivided into separate compartments managed by each module in the application. The `Multistore` is a store of `KVStore`s that follows the [**`Multistore interface`**](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L104-L133)
    1. A [**`CommitMultistore`**](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L141-L184) is a `Multistore` with a committer. This is the main type of multistore used in the Cosmos SDK. The underlying `KVStore` is used primarily to restrict access to the committer.
    2. The [**`rootMulti.Store`**](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/rootmulti/store.go#L43-L61) is the go-to implementation of the `CommitMultiStore` interface. It is a base-layer multistore built around a database on top of which multiple `KVStore`s can be mounted. It is the default multistore store used in `BaseApp`.
    3. A [**`cachemulti.Store`**](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/cachemulti/store.go#L17-L28) is used whenever the `rootMulti.Store` needs to be branched. This is used primarily to create an isolated store, typically when it is necessary to mutate the state but it might be reverted later.
    4. `Transient.Store` is a `KVStore` that is discarded automatically at the end of each block.
17. Some more `KVStore` wrappers:
    1. The `GasKv.Store` is a `KVStore` wrapper that enables automatic gas consumption each time a read or write to the store is made.
    2. `tracekv.Store` is a `KVStore` wrapper which provides operation tracing functionalities over the underlying `KVStore`. It is applied automatically by the Cosmos SDK on all `KVStore`s if tracing is enabled on the parent `MultiStore`.
    3. `prefix.Store` is a `KVStore` wrapper which provides automatic key-prefixing functionalities over the underlying `KVStore`
18.  Inclusion of the `AnteHandler` component is recommended to authenticate transactions before their internal messages are processed. It defends against spam and other wasteful transaction events, performs preliminary stateful validity checks, and is involved in collecting transaction fees. The most widely used `AnteHandler` is the auth module.
19. `BaseApp` is a boilerplate implementation of a Cosmos SDK application that handles ABCI functionality, state management, and service routing.
    1. **ABCI Implementation**: BaseApp provides a ready-to-use implementation of the CometBFT ABCI, including functions like `DeliverTx`, `CheckTx`, `InitChain`, `BeginBlock`, and `EndBlock`.
    2. **State Machine**: Implements a state machine with canonical (persistent) state and two volatile states (`checkState` and `deliverState`), managed via `CommitMultiStore`.
    3. **Service Routing**: Includes a `msgServiceRouter` for routing transactions to appropriate modules and a `grpcQueryRouter` for processing gRPC queries.
    4. **Transaction Handling**:
        1. **CheckTx**: Verifies transactions and updates `checkState`.
        2. **DeliverTx**: Executes transactions, updating `deliverState`, which is committed to the main state on success.
    5. **Consensus Parameters**: Manages parameters like `minGasPrices` and validator information for consensus operations.
    6. **Bootstrapping Parameters**:
        1. **CommitMultiStore**: Central store for application state.
        2. **Database**: For state persistence.
        3. **TxDecoder**: Decodes raw transactions.
        4. **ParamStore**: Manages consensus parameters, configurable via on-chain governance.
    7. **Constructor**: Allows customization through various options like setting pruning or gas prices.
    8. **State Updates**:
        1. **InitChain**: Sets up initial states.
        2. **Commit**: Writes state transitions to disk, finalizing changes.
20. An event is an object that contains information about the execution of applications. Events are used by service providers like block explorers and wallets to track the execution of various messages and index transactions.
    1. **Structure**: Consists of a type and key-value attributes.
    2. **Definition**: Defined per module in `/types/events.go` and documented in `spec/xx_events.md`.
    3. **EventManager**: Manages and emits events during transaction execution.
    4. **Subscription**: Events can be subscribed to via Tendermint's WebSocket for categories like `NewBlock`, `Tx`, and `ValidatorSetUpdates`.
21. The `context` is a data structure that carries information about the current state of the application. It provides access to a branched storage (a safe branch of the entire state) as well as useful objects and information like `gasMeter`, `block height`, `consensus parameters` and more.
