# BNC Chain Go SDK


The Binance Chain GO SDK provides a thin wrapper around the BNC Chain API for readonly endpoints, in addition to creating and submitting different transactions.
It includes the following core components:

* **client** - implementations of Binance Chain transaction types and query, such as for transfers and trading.
* **common** - core cryptographic functions, uuid functions and other useful functions.
* **e2e** - end-to-end test package for go-sdk developer. For common users, it is also a good reference to use go-sdk. 
* **keys** - implement `KeyManage` to manage private key and accounts.
* **types** - core type of Binance Chain, such as `coin`, `account`, `tx` and `msg`.

## Install

### Use go mod(recommend)

Add "github.com/binance-chain/go-sdk" dependency into your go.mod file. Example:
```go
require (
	github.com/binance-chain/go-sdk latest
)
```

### Use go get

Use go get to install sdk into your `GOPATH`:
```bash
go get github.com/binance-chain/go-sdk
```

### Use dep
Add dependency to your Gopkg.toml file. Example:
```bash
[[override]]
  name = "github.com/binance-chain/go-sdk"
```

## Usage 

### Key Manager

Before start using API, you should construct a Key Manager to help sign the transaction msg or verify signature.
Key Manager is an Identity Manger to define who you are in the bnbchain. It provide following interface:

```go
type KeyManager interface {
	Sign(tx.StdSignMsg) ([]byte, error)
	GetPrivKey() crypto.PrivKey
	GetAddr() txmsg.AccAddress
	
	ExportAsMnemonic() (string, error)
    ExportAsPrivateKey() (string, error)
    ExportAsKeyStore(password string) (*EncryptedKeyJSON, error)
}
```

We provide three construct functions to generate Key Manger:
```go
NewKeyManager() (KeyManager, error)

NewMnemonicKeyManager(mnemonic string) (KeyManager, error)

NewKeyStoreKeyManager(file string, auth string) (KeyManager, error)

NewPrivateKeyManager(priKey string) (KeyManager, error) 

```
- NewKeyManager. You will get a new private key without provide anything, you can export and save this `KeyManager`.
- NewMnemonicKeyManager. You should provide your mnemonic, usually is a string of 24 words.
- NewKeyStoreKeyManager. You should provide a keybase json file and you password, you can download the key base json file when your create a wallet account.
- NewPrivateKeyManager. You should provide a Hex encoded string of your private key.

Examples:

From mnemonic:
```Go
mnemonic := "lock globe panda armed mandate fabric couple dove climb step stove price recall decrease fire sail ring media enhance excite deny valid ceiling arm"
keyManager, _ := keys.NewMnemonicKeyManager(mnemonic)
```

From key base file:
```GO
file := "testkeystore.json"
keyManager, err := NewKeyStoreKeyManager(file, "Zjubfd@123")

```

From raw private key string:
```GO
priv := "9579fff0cab07a4379e845a890105004ba4c8276f8ad9d22082b2acbf02d884b"
keyManager, err := NewPrivateKeyManager(priv)
```

We provide three export functions to persistent a Key Manger:

```go
ExportAsMnemonic() (string, error)

ExportAsPrivateKey() (string, error)

ExportAsKeyStore(password string) (*EncryptedKeyJSON, error)
```

Examples:
```go
km, _ := NewKeyManager()
encryPlain1, _ := km.GetPrivKey().Sign([]byte("test plain"))
keyJSONV1, err := km.ExportAsKeyStore("testpassword")
bz, _ := json.Marshal(keyJSONV1)
ioutil.WriteFile("TestGenerateKeyStoreNoError.json", bz, 0660)
newkm, _ := NewKeyStoreKeyManager("TestGenerateKeyStoreNoError.json", "testpassword")
encryPlain2, _ := newkm.GetPrivKey().Sign([]byte("test plain"))
assert.True(t, bytes.Equal(encryPlain1, encryPlain2))
```


### Init Client

```GO
import sdk "github.com/binance-chain/go-sdk/client"

mnemonic := "lock globe panda armed mandate fabric couple dove climb step stove price recall decrease fire sail ring media enhance excite deny valid ceiling arm"
//-----   Init KeyManager  -------------
keyManager, _ := keys.NewMnemonicKeyManager(mnemonic)

//-----   Init sdk  -------------
client, err := sdk.NewDexClient("https://testnet-dex.binance.org", types.TestNetwork, keyManager)

```
For sdk init, you should know the famous api address. Besides, you should know what kind of network the api gateway is in, since we have different configurations for 
test network and production network.

|  ChainNetwork |  ApiAddr | 
|-------------- |----------------------------------|
|   TestNetwork | https://testnet-dex.binance.org  |  
|   ProdNetwork | https://dex.binance.org          |                                |

If you want broadcast some transactions, like send coins, create orders or cancel orders, you should construct a key manager.


### Example

Create a `buy` order: 
```go
createOrderResult, err := client.CreateOrder(tradeSymbol, nativeSymbol, txmsg.OrderSide.BUY, 100000000, 100000000, true)
```

For more API usage documentation, please check the [wiki](https://github.com/binance-chain/go-sdk/wiki)..