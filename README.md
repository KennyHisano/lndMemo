### intension of this read is to keep track of coding structure of lightning network based on the local setup tutporial post below/


through lnd tutorial.
http://dev.lightning.community/tutorial/01-lncli/index.html

it works with testnet from btcd.
<testnet>
mining fund (btcd) -> alice -> bob -> charlie


# q1 
how alice creares paymentchannel to bob


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

# a

first, keet track of the method of fetching Bob's pubKey

lnrpc rpc.pb.go
