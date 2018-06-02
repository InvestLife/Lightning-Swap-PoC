[ [index](/README.md), [previous](/LIGHTNING-02-connect.md), [next](/LIGHTNING-04-payment.md) ]

# Lightning Payment Channels

### Setup
We continue to use the `identity_pubkey`s of Exchange A and Exchange B which were set up [here](LIGHTNING-01-peers.md)

```
$ X_A_ID_PUBKEY=026374581ff7974975ffce20e65a04876ba33405502d1a13dc73c9a702b61aef31
$ X_B_ID_PUBKEY=0237cdf6b03cf17df8676af35b43da3ee0613b888bc5cd26a41064118f1241cc2f
```

### Exchange A opens 0.16 BTC payment channel to Exchange B
Check open payment channels for Exchange A
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons listchannels
{
    "channels": []
}
```

Exchange A tries to open `0.9 BTC` channel to Exchange B, getting limit error imposed by the [spec](https://github.com/lightningnetwork/lightning-rfc/blob/master/02-peer-protocol.md#requirements)
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons openchannel --node_key=$X_B_ID_PUBKEY --local_amt=90000000 --ticker=BTC
[lncli] rpc error: code = Unknown desc = funding amount is too large, the max channel size is: 0.16777216 BTC
```

Exchange A opens `0.16 BTC` channel to Exchange B with the following [BTC funding transaction](https://www.blocktrail.com/tBTC/tx/1f40907fc1968319cbb57955e06c7b11d4f3b9d413c633c1ca26288b9d2e033b)
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons openchannel --node_key=$X_B_ID_PUBKEY --local_amt=16000000 --ticker=BTC
{
        "funding_txid": "1f40907fc1968319cbb57955e06c7b11d4f3b9d413c633c1ca26288b9d2e033b"
}

```

Funding transaction must be confirmed before a channel is opened.
The default number of confirmations is 1.

Once the channel is opened, Exchange A lists the `Bitcoin` payment channel as follows
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons listchannels
{
        "channels": [
                {
                        "active": true,
                        "remote_pubkey": "0237cdf6b03cf17df8676af35b43da3ee0613b888bc5cd26a41064118f1241cc2f",
                        "channel_point": "1f40907fc1968319cbb57955e06c7b11d4f3b9d413c633c1ca26288b9d2e033b:0",
                        "chan_id": "1450271230199529472",
                        "capacity": "16000000",
                        "local_balance": "15991312",
                        "remote_balance": "0",
                        "commit_fee": "8688",
                        "commit_weight": "600",
                        "fee_per_kw": "12000",
                        "unsettled_balance": "0",
                        "total_satoshis_sent": "0",
                        "total_satoshis_received": "0",
                        "num_updates": "0",
                        "pending_htlcs": []
                }
        ]
}
```

Exchange B lists the `Bitcoin` payment channel as follows
```
$ lncli --rpcserver=localhost:10002 --no-macaroons listchannels
{
        "channels": [
                {
                        "active": true,
                        "remote_pubkey": "026374581ff7974975ffce20e65a04876ba33405502d1a13dc73c9a702b61aef31",
                        "channel_point": "1f40907fc1968319cbb57955e06c7b11d4f3b9d413c633c1ca26288b9d2e033b:0",
                        "chan_id": "1450271230199529472",
                        "capacity": "16000000",
                        "local_balance": "0",
                        "remote_balance": "15991312",
                        "commit_fee": "8688",
                        "commit_weight": "552",
                        "fee_per_kw": "12000",
                        "unsettled_balance": "0",
                        "total_satoshis_sent": "0",
                        "total_satoshis_received": "0",
                        "num_updates": "0",
                        "pending_htlcs": []
                }
        ]
}
```

Exchange A's `Bitcoin` balance decreased due to the committed amount in payment channel
```
$ lncli --rpcserver=localhost:10001 --no-macaroons walletbalance --ticker=BTC
{
        "balance": "16491564"
}
$ lncli --rpcserver=localhost:10001 --no-macaroons walletbalance --ticker=LTC
{
        "balance": "1000000000"
}
```


### Checking Swap Routes
Exchange A has no swap route to Exchange B yet
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons queryswaproutes --dest=$X_B_ID_PUBKEY --in_amt=1000 --in_ticker=BTC --out_ticker=LTC
[lncli] rpc error: code = Unknown desc = unable to find a path to destination
$ lncli --rpcserver=localhost:10001 --no-macaroons queryswaproutes --dest=$X_B_ID_PUBKEY --in_amt=1000 --in_ticker=LTC --out_ticker=BTC
[lncli] rpc error: code = Unknown desc = unable to find a path to destination

```

Exchange B has no swap route to Exchange A yet
```shell
$ lncli --rpcserver=localhost:10002 --no-macaroons queryswaproutes --dest=$X_A_ID_PUBKEY --in_amt=1000 --in_ticker=BTC --out_ticker=LTC
[lncli] rpc error: code = Unknown desc = unable to find a path to destination
$ lncli --rpcserver=localhost:10002 --no-macaroons queryswaproutes --dest=$X_A_ID_PUBKEY --in_amt=1000 --in_ticker=LTC --out_ticker=BTC
[lncli] rpc error: code = Unknown desc = unable to find a path to destination
```



### Exchange A opens 0.1 LTC payment channel to Exchange B and pushes 0.05 LTC to exchange B
Check open payment channels for Exchange A (only BTC channel to Exchange B exists)
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons listchannels
{
        "channels": [
                {
                        "active": true,
                        "remote_pubkey": "0237cdf6b03cf17df8676af35b43da3ee0613b888bc5cd26a41064118f1241cc2f",
                        "channel_point": "1f40907fc1968319cbb57955e06c7b11d4f3b9d413c633c1ca26288b9d2e033b:0",
                        "chan_id": "1450271230199529472",
                        "capacity": "16000000",
                        "local_balance": "15991312",
                        "remote_balance": "0",
                        "commit_fee": "8688",
                        "commit_weight": "600",
                        "fee_per_kw": "12000",
                        "unsettled_balance": "0",
                        "total_satoshis_sent": "0",
                        "total_satoshis_received": "0",
                        "num_updates": "0",
                        "pending_htlcs": []
                }
        ]
}
```

Exchange A opens `0.1 LTC` channel to Exchange B with the following [LTC funding transaction](https://chain.so/tx/LTCTEST/3c5b1d738e251819f0eaf263e73eb268e73a2d231e5da00cdfada76b3c66e8f7)
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons openchannel --node_key=$X_B_ID_PUBKEY --local_amt=10000000 --ticker=LTC
{
        "funding_txid": "3c5b1d738e251819f0eaf263e73eb268e73a2d231e5da00cdfada76b3c66e8f7"
}
```

Until confirmed Exchange B lists the new channel as pending channel.
```shell
$ lncli --rpcserver=localhost:10002 --no-macaroons pendingchannels --ticker=LTC
{
        "total_limbo_balance": "0",
        "pending_open_channels": [
                {
                        "channel": {
                                "remote_node_pub": "026374581ff7974975ffce20e65a04876ba33405502d1a13dc73c9a702b61aef31",
                                "channel_point": "3c5b1d738e251819f0eaf263e73eb268e73a2d231e5da00cdfada76b3c66e8f7:0",
                                "capacity": "10000000",
                                "local_balance": "5000000",
                                "remote_balance": "4963800"
                        },
                        "confirmation_height": 0,
                        "blocks_till_open": 0,
                        "commit_fee": "36200",
                        "commit_weight": "724",
                        "fee_per_kw": "50000"
                }
        ],
        "pending_closing_channels": [],
        "pending_force_closing_channels": []
}
```

Once confirmed, Exchange B lists the `Litecoin` payment channel as follows
```shell
$ lncli --rpcserver=localhost:10002 --no-macaroons listchannels
{
        "channels": [
                {
                        "active": true,
                        "remote_pubkey": "026374581ff7974975ffce20e65a04876ba33405502d1a13dc73c9a702b61aef31",
                        "channel_point": "1f40907fc1968319cbb57955e06c7b11d4f3b9d413c633c1ca26288b9d2e033b:0",
                        "chan_id": "1450271230199529472",
                        "capacity": "16000000",
                        "local_balance": "0",
                        "remote_balance": "15991312",
                        "commit_fee": "8688",
                        "commit_weight": "552",
                        "fee_per_kw": "12000",
                        "unsettled_balance": "0",
                        "total_satoshis_sent": "0",
                        "total_satoshis_received": "0",
                        "num_updates": "0",
                        "pending_htlcs": []
                },
                {
                        "active": true,
                        "remote_pubkey": "026374581ff7974975ffce20e65a04876ba33405502d1a13dc73c9a702b61aef31",
                        "channel_point": "3c5b1d738e251819f0eaf263e73eb268e73a2d231e5da00cdfada76b3c66e8f7:0",
                        "chan_id": "0",
                        "capacity": "10000000",
                        "local_balance": "5000000",
                        "remote_balance": "4963800",
                        "commit_fee": "36200",
                        "commit_weight": "724",
                        "fee_per_kw": "50000",
                        "unsettled_balance": "0",
                        "total_satoshis_sent": "0",
                        "total_satoshis_received": "0",
                        "num_updates": "0",
                        "pending_htlcs": []
                }
        ]
}
```

Exchange A lists the `Litecoin` payment channel as follows
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons listchannels
{
        "channels": [
                {
                        "active": true,
                        "remote_pubkey": "0237cdf6b03cf17df8676af35b43da3ee0613b888bc5cd26a41064118f1241cc2f",
                        "channel_point": "1f40907fc1968319cbb57955e06c7b11d4f3b9d413c633c1ca26288b9d2e033b:0",
                        "chan_id": "1450271230199529472",
                        "capacity": "16000000",
                        "local_balance": "15991312",
                        "remote_balance": "0",
                        "commit_fee": "8688",
                        "commit_weight": "600",
                        "fee_per_kw": "12000",
                        "unsettled_balance": "0",
                        "total_satoshis_sent": "0",
                        "total_satoshis_received": "0",
                        "num_updates": "0",
                        "pending_htlcs": []
                },
                {
                        "active": true,
                        "remote_pubkey": "0237cdf6b03cf17df8676af35b43da3ee0613b888bc5cd26a41064118f1241cc2f",
                        "channel_point": "3c5b1d738e251819f0eaf263e73eb268e73a2d231e5da00cdfada76b3c66e8f7:0",
                        "chan_id": "649885039294873600",
                        "capacity": "10000000",
                        "local_balance": "4963800",
                        "remote_balance": "5000000",
                        "commit_fee": "36200",
                        "commit_weight": "724",
                        "fee_per_kw": "50000",
                        "unsettled_balance": "0",
                        "total_satoshis_sent": "0",
                        "total_satoshis_received": "0",
                        "num_updates": "0",
                        "pending_htlcs": []
                }
        ]
}
```

### Checking Swap Routes
Note that until the funding transaction for both channels have 6 confirmations, `queryswaproutes` will return `[lncli] rpc error: code = Unknown desc = unable to find a path to destination` even though the channels may appear active in `listchannels`.

Exchange A has the following swap routes to Exchange B
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons queryswaproutes --dest=$X_B_ID_PUBKEY --in_amt=1000 --in_ticker=BTC --out_ticker=LTC
{
        "routes": [
                {
                        "total_time_lock": 591679,
                        "total_fees": "0",
                        "total_amt": "100000",
                        "hops": [
                                {
                                        "chan_id": "649885039294873600",
                                        "chan_capacity": "10000000",
                                        "amt_to_forward": "100000",
                                        "fee": "0",
                                        "expiry": 591103
                                },
                                {
                                        "chan_id": "1450271230199529472",
                                        "chan_capacity": "16000000",
                                        "amt_to_forward": "1000",
                                        "fee": "0",
                                        "expiry": 1319040
                                }
                        ]
                }
        ]
}
$ lncli --rpcserver=localhost:10001 --no-macaroons queryswaproutes --dest=$X_B_ID_PUBKEY --in_amt=1000 --in_ticker=LTC --out_ticker=BTC
{
        "routes": [
                {
                        "total_time_lock": 1319177,
                        "total_fees": "0",
                        "total_amt": "10",
                        "hops": [
                                {
                                        "chan_id": "1450271230199529472",
                                        "chan_capacity": "16000000",
                                        "amt_to_forward": "10",
                                        "fee": "0",
                                        "expiry": 1319033
                                },
                                {
                                        "chan_id": "649885039294873600",
                                        "chan_capacity": "10000000",
                                        "amt_to_forward": "1000",
                                        "fee": "0",
                                        "expiry": 591077
                                }
                        ]
                }
        ]
}
```

Exchange B has the following swap routes to Exchange A
```shell
$ lncli --rpcserver=localhost:10002 --no-macaroons queryswaproutes --dest=$X_A_ID_PUBKEY --in_amt=1000 --in_ticker=BTC --out_ticker=LTC
{
        "routes": [
                {
                        "total_time_lock": 591680,
                        "total_fees": "0",
                        "total_amt": "100000",
                        "hops": [
                                {
                                        "chan_id": "649885039294873600",
                                        "chan_capacity": "10000000",
                                        "amt_to_forward": "100000",
                                        "fee": "0",
                                        "expiry": 591104
                                },
                                {
                                        "chan_id": "1450271230199529472",
                                        "chan_capacity": "16000000",
                                        "amt_to_forward": "1000",
                                        "fee": "0",
                                        "expiry": 1319040
                                }
                        ]
                }
        ]
}
$ lncli --rpcserver=localhost:10002 --no-macaroons queryswaproutes --dest=$X_A_ID_PUBKEY --in_amt=1000 --in_ticker=LTC --out_ticker=BTC
{
        "routes": [
                {
                        "total_time_lock": 1319177,
                        "total_fees": "0",
                        "total_amt": "10",
                        "hops": [
                                {
                                        "chan_id": "1450271230199529472",
                                        "chan_capacity": "16000000",
                                        "amt_to_forward": "10",
                                        "fee": "0",
                                        "expiry": 1319033
                                },
                                {
                                        "chan_id": "649885039294873600",
                                        "chan_capacity": "10000000",
                                        "amt_to_forward": "1000",
                                        "fee": "0",
                                        "expiry": 591077
                                }
                        ]
                }
        ]
}
```

[ [index](/README.md), [previous](/LIGHTNING-02-connect.md), [next](/LIGHTNING-04-payment.md) ]
