# Interlude: Using Curl

`digitalcoin-cli` is ultimately just a wrapper. It's a way to interface with `digitalcoind` from the command line, providing simplified access to its many RPC commands. But RPC can, of course, be accessed directly. That's what this interlude is about: directly connecting to RPC with the `curl` command.

It won't be used much in the future chapters, but it's an important building block that you can see as an alternative access to `digitalcoind` is you so prefer.

## Know Your Curl

`curl`, short for "see URL", is a command-line tool that allows you to directly access URLs in a programmatic way. It's an easy way to interact with servers like `digitalcoind` that listen to ports on the internet and that speak a variety of protocols. Curl is also available as a library for many programming languages, such as C, Java, PHP, and Python. So, once you know how to work with Curl, you'll have a strong foundation for using a lot of different API.

In order to use `curl` with `digitalcoind`, you must know three things: the standard format, the user name and password, and the correct port.

### Know Your Format

The `digitalcoin-cli` commands are all linked to RPC commands in `digitalcoind`. That makes the transition from using `digitalcoin-cli` to using `curl` very simple. In fact, if you look at any of the help pages for `digitalcoin-cli`, you'll see that they list not only the `digitalcoin-cli` commands, but also parallel `curl` commands. For example, here is `digitalcoin-cli help getmininginfo`:
```
$ digitalcoin-cli help getmininginfo
getmininginfo

Returns a json object containing mining-related information.
Result:
{                              (json object)
  "blocks" : n,                (numeric) The current block
  "currentblockweight" : n,    (numeric, optional) The block weight of the last assembled block (only present if a block was ever assembled)
  "currentblocktx" : n,        (numeric, optional) The number of block transactions of the last assembled block (only present if a block was ever assembled)
  "difficulty" : n,            (numeric) The current difficulty
  "networkhashps" : n,         (numeric) The network hashes per second
  "pooledtx" : n,              (numeric) The size of the mempool
  "chain" : "str",             (string) current network name (main, test, regtest)
  "warnings" : "str"           (string) any network and blockchain warnings
}

Examples:
> digitalcoin-cli getmininginfo 
> curl --user myusername --data-binary '{"jsonrpc": "1.0", "id": "curltest", "method": "getmininginfo", "params": []}' -H 'content-type: text/plain;' http://127.0.0.1:8332/
```
And there's the `curl` command, at the end of the help screen! This somewhat lengthy command has four major parts: (1) a listing of your user name; (2) a `--data-binary` flag; (3) a JSON object that tells `digitalcoind` what to do, including a JSON array of parameters; and (4) an HTTP header that includes the `digitalcoind` URL.

When you are working with `curl`, most of these arguments to `curl` will stay the same from command to command; only the `method` and `params` entries in the JSON array will typically change. However, you need to know how to fill in your username and your URL address in order to make it work in the first place!

_Whenever you're unsure about how to curl an RPC command, just look at the digitalcoin-cli help and go from there._

### Know Your User Name

In order to speak with the `digitalcoind` port, you need a user name and password. These were created as part of your initial Digitalcoin setup, and can be found in `~/.digitalcoin/digitalcoin.conf`.

For example, here's our current setup:
```
$ cat ~/.digitalcoin/digitalcoin.conf
server=1
dbcache=1536
par=1
maxuploadtarget=137
maxconnections=16
rpcuser=StandUp
rpcpassword=8eaf562eaf45c33c3328bc66008f2dd1
rpcallowip=127.0.0.1
debug=tor
prune=550
testnet=1
mintxfee=0.001
txconfirmtarget=1
[test]
rpcbind=127.0.0.1
rpcport=18332
[main]
rpcbind=127.0.0.1
rpcport=8332
[regtest]
rpcbind=127.0.0.1
rpcport=18443
```
Our user name is `StandUp` and our password is `8eaf562eaf45c33c3328bc66008f2dd1`.

> **WARNING:** Clearly, it's not very secure to have this information in a plain text file. As of Digitalcoin Core 0.12, you can instead omit the `rpcpassword` from your `digitalcoin.conf` file, and have `digitalcoind` generate a new cookie whenever it starts up. The downside of this is that it makes use of RPC commands by other applications, such as the ones detailed in this chapter, more difficult. So, we're going to stick with the plain `rpcuser` and `rpcpassword` information for now, but for production software, consider moving to cookies.

The secure way to RPC with `digitalcoind` is as follows:
```
$ curl --user StandUp --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getmininginfo", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:18332/
Enter host password for user 'bitcoinrpc':
```
As noted, you will be prompted for your password.

> :link: **TESTNET vs MAINNET:** Testnet uses a URL with port 18332 and mainnet uses a URL with port 8332. Take a look in your `digitalcoin.conf`, it's all laid out there.

The insecure way to do so is as follows:
```
$ curl --user StandUp:8eaf562eaf45c33c3328bc66008f2dd1 --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getmininginfo", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:18332/
```
> **WARNING:** Entering your password on the command line may put your password into the process table and/or save it into a history. This is even less recommended than putting it in a file, except for testing on testnet. If you want to do it anywhere else, make sure you know what you're doing!

### Know Your Command & Parameters

With all of that in hand, you're ready to send off standard RPC commands with `curl` ... but you still need to know how to incorporate the two elements that tend to change in the `curl` command.

The first is `method`, which is the RPC method being used. This should generally match the command names you've been feeding into `digitalcoin-cli` for ages.

The second is `params`, which is a JSON array of parameters. These are the same as the arguments (or named arguments) that you've been using. They're also the most confusing part of `curl`, in large part because they're a structured array rather than a simple list.

Here's what some parameter arrays will look like:

  * `[]` — An empty array
  * `["000b4430a7a2ba60891b01b718747eaf9665cb93fbc0c619c99419b5b5cf3ad2"]` — An array with data
  * `["'$signedhex'"]` — An array with a variable
  * `[6, 9999999]` — An array with two parameters
  * `{}` - An empty object
  * `[''[ { "txid": "'$utxo_txid'", "vout": '$utxo_vout' } ]'', ''{ "'$recipient'": 0.298, "'$changeaddress'": 1.0}'']` — An array with an array containing an object and a bare object

## Get Information

You can now send your first `curl` command by accessing the `getmininginfo` RPC:
```
$ curl --user StandUp:8eaf562eaf45c33c3328bc66008f2dd1 --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getmininginfo", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:18332/
{"result":{"blocks":1772428,"difficulty":10178811.40698772,"networkhashps":91963587385939.06,"pooledtx":61,"chain":"test","warnings":"Warning: unknown new rules activated (versionbit 28)"},"error":null,"id":"curltest"}```
Note that we provided the method, `getmininginfo`, and the parameter, `[]`, but that everything else was the standard `curl` command line.
```
> **WARNING:** If you get a result like "Failed to connect to 127.0.0.1 port 8332: Connection refused", be sure that a line like `rpcallowip=127.0.0.1` is in your ~/.digitalcoin/digitalcoin.conf. If things still don't work, be sure that you're allowing access to port 18332 (or 8332) from localhost. Our standard setup from [Chapter Two: Creating a Digitalcoin-Core VPS](02_0_Setting_Up_a_Digitalcoin-Core_VPS.md) should do all of this.

The result is another JSON array, which is unfortunately ugly to read if you're using `curl` by hand. Fortunately, you can clean it up  simply by piping it through `jq`:
```
$ curl --user StandUp:8eaf562eaf45c33c3328bc66008f2dd1 --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getmininginfo", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:18332/ | jq -r '.'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   295  100   218  100    77  72666  25666 --:--:-- --:--:-- --:--:-- 98333
{
  "result": {
    "blocks": 1772429,
    "difficulty": 10178811.40698772,
    "networkhashps": 90580030969896.44,
    "pooledtx": 4,
    "chain": "test",
    "warnings": "Warning: unknown new rules activated (versionbit 28)"
  },
  "error": null,
  "id": "curltest"
}
```
You'll see a bit of connectivity reporting as the data is downloaded, then when that data hits `jq`, everything will be output in a correctly indented form. (We'll be omitting the download information in future examples.)

## Manipulate Your Wallet

Though you're accessing `digitalcoind` directly, you'll still get access to wallet functionality, because that's largely stored in `digitalcoind` itself.

### Look Up Addresses

Use the `getaddressesbylabel` RPC to list all of your current addresses:
```
$ curl --user StandUp:8eaf562eaf45c33c3328bc66008f2dd1 --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getaddressesbylabel", "params": [""] }' -H 'content-type: text/plain;' http://127.0.0.1:18332/ | jq -r '.'
{
  "result": {
    "mi25UrzHnvn3bpEfFCNqJhPWJn5b77a5NE": {
      "purpose": "receive"
    },
    "mjehC2KHzXcBDcwTd4LhZ2GzyzrZ3Kd3ff": {
      "purpose": "receive"
    },
    "moKVV6XEhfrBCE3QCYq6ppT7AaMF8KsZ1B": {
      "purpose": "receive"
    },
    "mwJL7cRiW2bUnY81r1thSu3D4jtMmwyU6d": {
      "purpose": "receive"
    },
    "tb1q5gnwrh7ss5mmqt0qfan85jdagmumnatcscwpk6": {
      "purpose": "receive"
    },
    "tb1qmtucvjtga68kgrvkl7q05x4t9lylxhku7kqdpr": {
      "purpose": "receive"
    }
  },
  "error": null,
  "id": "curltest"
}
```
This is our first example of a real parameter, `""`. This is the required `label` parameter for `getaddressesbylabel`, but all of our addresses are under the default label, so nothing special was required here.

The result is a list of all the addresses that have been used by this wallet ... some of which presumably contain funds.

### Look Up Funds

Use the `listunspent` RPC to list the funds that you have available:
```
$ curl --user StandUp:8eaf562eaf45c33c3328bc66008f2dd1 --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "listunspent", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:18332/ | jq -r '.'
{
  "result": [
    {
      "txid": "e7071092dee0b2ae584bf6c1ee3c22164304e3a17feea7a32c22db5603cd6a0d",
      "vout": 1,
      "address": "mk9ry5VVy8mrA8SygxSQQUDNSSXyGFot6h",
      "scriptPubKey": "76a91432db726320e4ad170c9c1ee83cd4d8a243c3435988ac",
      "amount": 0.0009,
      "confirmations": 4,
      "spendable": true,
      "solvable": true,
      "desc": "pkh([d6043800/0'/1'/2']02881697d252d8bf181d08c58de1f02aec088cd2d468fc5fd888c6e39909f7fabf)#p6k7dptk",
      "safe": true
    },
    {
      "txid": "91261eafae15ea53dedbea7c1db748c52bbc04a85859ffd0d839bda1421fda4c",
      "vout": 0,
      "address": "mjehC2KHzXcBDcwTd4LhZ2GzyzrZ3Kd3ff",
      "label": "",
      "scriptPubKey": "76a9142d573900aa357a38afd741fbf24b075d263ea6e088ac",
      "amount": 0.00022,
      "confirmations": 19,
      "spendable": true,
      "solvable": true,
      "desc": "pkh([d6043800/0'/0'/3']0278608b54b8fb0d8379d3823d31f03a7c6ab0adffb07dd3811819fdfc34f8c132)#nhjc3f8y",
      "safe": true
    }
  ],
  "error": null,
  "id": "curltest"
}
```
This is almost exactly the same output that you receive when you type `digitalcoin-cli listunspent`, showing how closely tied the two interfaces are. If no cleanup or extra help is needed, then `digitalcoin-cli` just outputs the RPC. Easy!

### Create an Address

After you know where your funds are, the next step in crafting a transaction is to get a change address. By now you've probably got the hang of this, and you know that for simple RPC commands, all you need to do is adjust the `method` is the `curl` command:
```
$ curl --user StandUp:8eaf562eaf45c33c3328bc66008f2dd1 --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getrawchangeaddress", "params": ["legacy"] }' -H 'content-type: text/plain;' http://127.0.0.1:18332/ | jq -r '.'
{
  "result": "mrSqN37TPs89GcidSZTvXmMzjxoJZ6RKoz",
  "error": null,
  "id": "curltest"
}

```
At this point, we can even revert to our standard practice of saving results to variables with additional help from `jq`:
```
$ changeaddress=$(curl --user StandUp:8eaf562eaf45c33c3328bc66008f2dd1 --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getrawchangeaddress", "params": ["legacy"] }' -H 'content-type: text/plain;' http://127.0.0.1:18332/ | jq -r '.result')
$ echo $changeaddress
mqdfnjgWr2r3sCCeuTDfe8fJ1CnycF2e6R
```
No need to worry about the downloading info. It'll go to `STDERR` and be displayed on your screen, while the results go to `STDOUT` and are saved in your variable.

## Create a Transaction

You're now ready to create a transaction with `curl`.

### Ready Your Variables

Just as with `digitalcoin-cli`, in order to create a transaction by curling RPC commands, you should first save your variables. The only change here is that `curl` creates a JSON object that includes a `result` key-value, so you always need to pipe through the `.result` tag before you do anything else.

This example sets up our variables for using the 1.2985 DGC in funds listed in the first unspent transaction above:
```
$ utxo_txid=$(curl --user StandUp:8eaf562eaf45c33c3328bc66008f2dd1 --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "listunspent", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:18332/ | jq -r '.result | .[0] | .txid')
$ utxo_vout=$(curl --user StandUp:8eaf562eaf45c33c3328bc66008f2dd1 --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "listunspent", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:18332/ | jq -r '.result | .[0] | .vout')
$ recipient=mwCwTceJvYV27KXBc3NJZys6CjsgsoeHmf
$ changeaddress=$(curl --user StandUp:8eaf562eaf45c33c3328bc66008f2dd1 --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getrawchangeaddress", "params": ["legacy"] }' -H 'content-type: text/plain;' http://127.0.0.1:18332/ | jq -r '.result')

$ echo $utxo_txid
e7071092dee0b2ae584bf6c1ee3c22164304e3a17feea7a32c22db5603cd6a0d
$ echo $utxo_vout
1
$ echo $recipient
mwCwTceJvYV27KXBc3NJZys6CjsgsoeHmf
$ echo $changeaddress
n2jf3MzeFpFGa7wq8rXKVnVuv5FoNSJZ1N
```

### Create the Transaction

The transaction created with `curl` is very similar to the transaction created with `digitalcoin-cli`, but with a few subtle differences:
```
$ curl --user StandUp:8eaf562eaf45c33c3328bc66008f2dd1 --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "createrawtransaction", "params": [''[ { "txid": "'$utxo_txid'", "vout": '$utxo_vout' } ]'', ''{ "'$recipient'": 0.0003, "'$changeaddress'": 0.0005}'']}' -H 'content-type: text/plain;' http://127.0.0.1:18332/ | jq -r '.'
{
  "result": "02000000010d6acd0356db222ca3a7ee7fa1e3044316223ceec1f64b58aeb2e0de921007e70100000000ffffffff0230750000000000001976a914ac19d3fd17710e6b9a331022fe92c693fdf6659588ac50c30000000000001976a9147021efec134057043386decfaa6a6aa4ee5f19eb88ac00000000",
  "error": null,
  "id": "curltest"
}
```
The heart of the transaction is, of course, the `params` JSON array, which we're putting to full use for the first time.

Note that the entire `params` is lodged in `[]`s to mark the parameters array.

We've also varied up the quoting from how things worked in `digitalcoin-cli`, to start and end each array and object within the `params` array with `''` instead of our traditional `'''`. That's because the entire set of JSON arguments already has a `'` around it. As usual, just take a look at the bizarre shell quoting and get used to it.

However, there's one last thing of note in this example, and it can be _maddening_ if you miss it. When you executed a `createrawtransaction` command with `digitalcoin-cli` the JSON array of inputs and the JSON object of outputs were each distinct parameters, so they were separated by a space. Now, because they're part of that `params` JSON array, they're separated by a comma (`,`). Miss that and you'll get a `parse error` without much additional information.

> **WARNING:** Ever having troubles debugging your `curl`? Add the argument `--trace-ascii /tmp/foo`. Full information on what's being sent to the server will be saved in `/tmp/foo` (or whatever file name you provide).

Having verified that things work, you probably want to save the hex code into a variable:
```
$ hexcode=$(curl --user StandUp:8eaf562eaf45c33c3328bc66008f2dd1 --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "createrawtransaction", "params": [''[ { "txid": "'$utxo_txid'", "vout": '$utxo_vout' } ]'', ''{ "'$recipient'": 0.0003, "'$changeaddress'": 0.0005}'']}' -H 'content-type: text/plain;' http://127.0.0.1:18332/ | jq -r '.result')
```

### Sign and Send

Signing and sending your transaction using `curl` is an easy use of the `signrawtransactionwithwallet` and `sendrawtransaction` RPC:

```
$ signedhex=$(curl --user StandUp:8eaf562eaf45c33c3328bc66008f2dd1 --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "signrawtransactionwithwallet", "params": ["'$hexcode'"] }' -H 'content-type: text/plain;' http://127.0.0.1:18332/ | jq -r '.result | .hex')

$ curl --user StandUp:8eaf562eaf45c33c3328bc66008f2dd1 --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "sendrawtransaction", "params": ["'$signedhex'"] }' -H 'content-type: text/plain;' http://127.0.0.1:18332/ | jq -r '.'
{
  "result": "eb84c5008038d760805d4d9644ace67849542864220cb2685a1ea2c64176b82d",
  "error": null,
  "id": "curltest"
}
```
## Summary: Accessing Digitalcoind with Curl

Having finished this section, you may feel that accessing `digitalcoind` via `curl` is very much like accessing it through `digitalcoin-cli` ... but more cumbersome. And, you'd be right. `digitalcoin-cli` has pretty complete RPC functionality, so anything that you do through `curl` you can probably do through `digitalcoin-cli`. Which is why we're going to continue concentrating on `digitalcoin-cli` following this digression.

But there are still reasons you'd use `curl` instead of `digitalcoin-cli`:

_What is the power of curl?_ Most obviously, `curl` takes out one level of indirection. Instead of working with `digitalcoin-cli` which sends RPC commands to `digitalcoind`, you're sending those RPC commands directly. This allows for more robust programming, because you don't have to worry about what unexpected things that `digitalcoin-cli` might do or how it might change over time. However, you're also taking your first steps toward using a more comprehensive programming language than the poor options offered by a shell script. As you'll see in the last few chapters of this, you might actually see curl libraries are other functions to access the RPC commands in a variety of programming languages: but that's still a long ways away.

## What's Next?

Learn one more way to "Send Digitalcoin Transactions" with [§4.5 Sending Coins with Automated Raw Transactions](04_5_Sending_Coins_with_Automated_Raw_Transactions.md).
