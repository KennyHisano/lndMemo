-Lnd memo

 ##General
https://www.youtube.com/watch?v=wIhAmTqXhZQ
https://fs.bitcoinmagazine.com/img/articles/understanding-the-lightning-network-part-building-a-bidirectional-payment-channel-76.jpg

 - 1  only Parent is bloadcasted. any number of child tx can be done offchain.
 - 2  softfork allows exchange of signiture of child tx before parent if confirmed.
 - 3  RSMC is applied to the one who is broadcasting the transaction.
 - 4  the ols state of transaction is invalid due to RSMC. In case Alice pay 0.1 to Bob -> Alice 0.4 and Bob 0.6, if Bob bloadcastd old tx 0.5 -> alice and 0.5 -> bob, Bob still loose all the money. 

## intension of this read is to keep track of coding structure of lightning network based on the local setup tutorial post below.


through lnd tutorial.
http://dev.lightning.community/tutorial/01-lncli/index.html

it works with testnet from btcd.
<testnet>
mining fund (btcd) -> alice -> bob -> charlie

## Note: required knowledge:
-protobuf rpc with go:
https://github.com/minaandrawos/Go-Protobuf-Examples.git

this project gives bootstrap in case if you are not familiar with it




### Q1 how generation of address works

##### wallet
first run client and d 


```
alice$ lnd --rpcport=10001 --peerport=10011 --restport=8001
```

go to lnd.go func main: 443
lndMain() -> loadConfig() -> config.go
starts program and read ports


```
alice$ lncli --rpcserver=localhost:10001 --no-macaroons create
```

go to commands.go: func create 732

- establish connection  
(lncli)func create -> main.go:41 getWalletUnlockerClient() -> getClientconn() -> func Dial() conn.go 35

- create wallet
(lncli)func create -> req := &lnrpc.CreateWalletRequest -> rpc.proto 31->
(json) \rpc.swagger.json 181 ->
(now lnd)  (access btcd, then creates wallet there)

(lncli) func create -> createwallet -> /walletunlocker/service.go: 45 createwallet->
loader:= wallet.Newloader ->
https://github.com/Roasbeef/btcwallet/wallet/loader.go NewLoader


##### address 

```
alice$ lncli --rpcserver=localhost:10001 --no-macaroons newaddress np2wkh
{
    "address": <ALICE_ADDRESS>
}
```

Three options 
```
   - p2wkh:  Push to witness key hash
   - np2wkh: Push to nested witness key hash
   - p2pkh:  Push to public key hash (can't be used to fund channels)

```


go  to commands.go: func newAddress 98

-establish connection
funcnewAddress -> getclient: main.go 51 -> getclientconn -> func dial

-take arguments and pass 
func newaddress -> client.NewAddress-> rpcservergo: func newAddress 271 + 

rpc.proto: 126"
```

    /** lncli: `newaddress`
    NewAddress creates a new address under control of the local wallet.
    */
    rpc NewAddress (NewAddressRequest) returns (NewAddressResponse);

    /**
```
"

-> validateMacaroon(skip) rpcserver.go: 276 ->  r.server.cc.wakllet.NewAddress -> \lnwallet\btcwallet\btcwallet.go func newAddewss addrType waddrmgr ->
github.com/roasbeef/btcwallet/waddrmgr -> 

https://github.com/Roasbeef/btcwallet/wallet/wallet.go func NewAddress: 2095

-> https://github.com/Roasbeef/btcwallet/wallet/wallet.go func newAddress :2105

->/waddrmgr/address.go 42: func addressTypeInternal
-> fetch address, learn diagram here for more detail.
https://dev.visucore.com/bitcoin/doxygen/index.html
it's a bitcore diagram.

### q2 how alice creares paymentchannel to bob

-"Creating the P2P Network
Now that Alice and Charlie have some simnet Bitcoin, let’s start connecting them together.

Connect Alice to Bob:"

```
# Get Bob's identity pubkey:
bob$ lncli-bob getinfo
{
    ----->"identity_pubkey": <BOB_PUBKEY>,
    "alias": "",
    "num_pending_channels": 0,
    "num_active_channels": 0,
    "num_peers": 0,
    "block_height": 450,
    "block_hash": "2a84b7a2c3be81536ef92cf382e37ab77f7cfbcf229f7d553bb2abff3e86231c",
    "synced_to_chain": true,
    "testnet": false,
    "chains": [
        "bitcoin"
    ]
}

# Connect Alice and Bob together
alice$ lncli-alice connect <BOB_PUBKEY>@localhost:10012
{
    "peer_id": 0
}

```

### a2

first, keet track of the method of fetching Bob's pubKey

lncli-bob getinfo(lncli-bob is alias of lncli)

1. go to lnclid/commands.go 



from lnrpc/rpc.proto
data serialization is done by google protobuf

```
  /** lncli: `getinfo`
    GetInfo returns general information concerning the lightning node including
    it's identity pubkey, alias, the chains it is connected to, and information
    concerning the number of open+pending channels.
    */
    rpc GetInfo (GetInfoRequest) returns (GetInfoResponse) {
        option (google.api.http) = {
            get: "/v1/getinfo"
        };
    }


```


2. 
