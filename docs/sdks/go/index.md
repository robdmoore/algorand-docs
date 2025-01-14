title: Your First Transaction

This section is a quick start guide for sending your first transaction on the Algorand TestNet network using the Go programming language. This guide installs the Go SDK, creates an account and submits a payment transaction. This guide also installs Algorand Sandbox, which provides required infrastructure for development and testing. 
 
# Install Algorand Sandbox
Algorand Sandbox is developer-focused tool for quickly spinning up the Algorand infrastructure portion of your development environment. It uses Docker to provide an `algod` instance for connecting to the network of your choosing and an `indexer` instance for querying blockchain data. APIs are exposed by both instances for client access provided within the SDK. Read more about [Algorand networks](../../get-details/algorand-networks/index.md), their capabilities and intended use.

!!! Prerequisites
    - Docker Compose ([install guide](https://docs.docker.com/compose/install/))
    - Git ([install guide](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git))

From a terminal window, install Algorand Sandbox connected to TestNet:

```bash
git clone https://github.com/algorand/sandbox.git
cd sandbox
./sandbox up testnet
```

!!! Warning
    The Algorand Sandbox installation may take a few minutes to complete in order to catch up to the current round on TestNet. To learn more about fast catchup, see [Sync Node Network using Fast Catchup](https://developer.algorand.org/docs/run-a-node/setup/install/#sync-node-network-using-fast-catchup).


# Install Go SDK
Algorand provides an SDK for Go. 

!!! Prerequisites
    - Go programming language ([install guide](https://golang.org/doc/install))

From a terminal window, install the Go SDK:

```bash
go get -u github.com/algorand/go-algorand-sdk/v2
```

- [SDK repository](https://github.com/algorand/go-algorand-sdk)
 
The SDK is installed and can now interact with the running Algorand Sandbox environment, as configured above.

# Create account
In order to interact with the Algorand blockchain, you must have a funded account on the network. To quickly create an account on Algorand TestNet create a new file **yourFirstTransaction.go** and insert the following code:

<!-- ===GOSDK_ACCOUNT_GENERATE=== -->
```go
newAccount := crypto.GenerateAccount()
passphrase, err := mnemonic.FromPrivateKey(newAccount.PrivateKey)

if err != nil {
	fmt.Printf("Error creating transaction: %s\n", err)
} else {
	fmt.Printf("My address: %s\n", newAccount.Address)
	fmt.Printf("My passphrase: %s\n", passphrase)
}
```
[Snippet Source](https://github.com/algorand/go-algorand-sdk/blob/examples/examples/kmd/main.go#L166-L175)
<!-- ===GOSDK_ACCOUNT_GENERATE=== -->

!!! Note 
    Lines 17 and 35 contain TODO: comments about inserting additional code. As you proceed with this guide, ensure the line numbers remain in sync.

!!! Tip
    Make sure to save the generated address and passphrase in a secure location, as they will be used later on.

!!! Warning 
    Never share your mnemonic passphrase or private keys. Production environments require stringent private key management. For more information on key management in community Wallets, click [here](https://developer.algorand.org/docs/community/#wallets). For the open source [Algorand Wallet](https://developer.algorand.org/articles/algorand-wallet-now-open-source/), click [here](https://github.com/algorand/algorand-wallet).

- [More Information](https://developer.algorand.org/docs/features/accounts/create/#standalone)
 
# Fund account
The code below prompts to fund the newly generated account. Before sending transactions to the Algorand network, the account must be funded to cover the minimal transaction fees that exist on Algorand. To fund the account use the [Algorand TestNet faucet](https://dispenser.testnet.aws.algodev.network/). 

!!! Info
    All Algorand accounts require a minimum balance to be registered in the ledger. To read more about Algorand minimum balance see [Account Overview](https://developer.algorand.org/docs/features/accounts/#minimum-balance)


# Instantiate client
You must instantiate a client prior to making calls to the API endpoints. The Go SDK implements the client natively using the following code:


<!-- ===GOSDK_ALGOD_CREATE_CLIENT=== -->
```go
// Create a new algod client, configured to connect to out local sandbox
var algodAddress = "http://localhost:4001"
var algodToken = strings.Repeat("a", 64)
algodClient, _ := algod.MakeClient(
	algodAddress,
	algodToken,
)

// Or, if necessary, pass alternate headers

var algodHeader common.Header
algodHeader.Key = "X-API-Key"
algodHeader.Value = algodToken
algodClientWithHeaders, _ := algod.MakeClientWithHeaders(
	algodAddress,
	algodToken,
	[]*common.Header{&algodHeader},
)
```
[Snippet Source](https://github.com/algorand/go-algorand-sdk/blob/examples/examples/overview/main.go#L18-L36)
<!-- ===GOSDK_ALGOD_CREATE_CLIENT=== -->
 
!!! Info
    This guide provides values for `algodAddress` and `algodToken` as specified by Algorand Sandbox. If you want to connect to a third-party service provider, see [Purestake](https://developer.purestake.io/code-samples) or [AlgoExplorer Developer API](https://algoexplorer.io/api-dev/v2) and adjust these values accordingly.
 

# Check account balance

Before moving on to the next step, make sure your account has been funded.
 
 <!-- ===GOSDK_ALGOD_FETCH_ACCOUNT_INFO=== -->
```go
acctInfo, err := algodClient.AccountInformation(acct.Address.String()).Do(context.Background())
if err != nil {
	log.Fatalf("failed to fetch account info: %s", err)
}
log.Printf("Account balance: %d microAlgos", acctInfo.Amount)
```
[Snippet Source](https://github.com/algorand/go-algorand-sdk/blob/examples/examples/overview/main.go#L51-L56)
 <!-- ===GOSDK_ALGOD_FETCH_ACCOUNT_INFO=== -->


# Build transaction
Communication with the Algorand network is performed using transactions. Create a payment transaction sending 1 ALGO from your account to the TestNet faucet address:

<!-- ===GOSDK_TRANSACTION_PAYMENT_CREATE=== -->
```go
sp, err := algodClient.SuggestedParams().Do(context.Background())
if err != nil {
	log.Fatalf("failed to get suggested params: %s", err)
}
// payment from account to itself
ptxn, err := transaction.MakePaymentTxn(acct.Address.String(), acct.Address.String(), 100000, nil, "", sp)
if err != nil {
	log.Fatalf("failed creating transaction: %s", err)
}
```
[Snippet Source](https://github.com/algorand/go-algorand-sdk/blob/examples/examples/overview/main.go#L59-L68)
<!-- ===GOSDK_TRANSACTION_PAYMENT_CREATE=== -->

!!! Info
    Algorand supports many transaction types. To see what types are supported see [Transactions](https://developer.algorand.org/docs/features/transactions/).


# Sign transaction
Before the transaction is considered valid, it must be signed by a private key. Use the following code to sign the transaction.

<!-- ===GOSDK_TRANSACTION_PAYMENT_SIGN=== -->
```go
_, sptxn, err := crypto.SignTransaction(acct.PrivateKey, ptxn)
if err != nil {
	fmt.Printf("Failed to sign transaction: %s\n", err)
	return
}
```
[Snippet Source](https://github.com/algorand/go-algorand-sdk/blob/examples/examples/overview/main.go#L71-L76)
<!-- ===GOSDK_TRANSACTION_PAYMENT_SIGN=== -->

!!! Info
    Algorand provides many ways to sign transactions. To see other ways see [Authorization](https://developer.algorand.org/docs/features/transactions/signatures/#single-signatures).


# Submit transaction
The signed transaction can now be broadcast to the network for validation and inclusion in a future block. The `waitForConfirmation` SDK method polls the `algod` node for the transaction ID to ensure it succeeded.

<!-- ===GOSDK_TRANSACTION_PAYMENT_SUBMIT=== -->
```go
pendingTxID, err := algodClient.SendRawTransaction(sptxn).Do(context.Background())
if err != nil {
	fmt.Printf("failed to send transaction: %s\n", err)
	return
}
confirmedTxn, err := transaction.WaitForConfirmation(algodClient, pendingTxID, 4, context.Background())
if err != nil {
	fmt.Printf("Error waiting for confirmation on txID: %s\n", pendingTxID)
	return
}
fmt.Printf("Confirmed Transaction: %s in Round %d\n", pendingTxID, confirmedTxn.ConfirmedRound)
```
[Snippet Source](https://github.com/algorand/go-algorand-sdk/blob/examples/examples/overview/main.go#L79-L90)
<!-- ===GOSDK_TRANSACTION_PAYMENT_SUBMIT=== -->
 
# Viewing the Transaction
To view the transaction, open the [Algorand Blockchain Explorer](https://testnet.algoexplorer.io/){:target="_blank"} or [Goal Seeker](https://goalseeker.purestake.io/algorand/testnet){:target="_blank"} and paste the transaction ID into the search bar or simply click on the funded transaction link on the dispenser page.)