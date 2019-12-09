## Preface:

This smart contract is a work for the 1st stage of the [<u>Blockchain contest</u>](https://contest.com/blockchain/) started *24 September 2019* and ended *15 October 2019*. The competition was officially held by **TON Issuer Inc.** and [**Telegram Group Inc.**](https://telegram.org) 

**[This work](https://contest.com/blockchain/entry347) took third place!**

------



## Multi-signature wallet smart contract:

### Quick start:

A main file, that contains a whole code of a smart contract is `smart-contract.fc`. It is already compiled into `smart-contract.fif`, so you do not need to startup a func compiler before all (**if whatever you need to recompile a smart contract, please, <u>read an UPD below</u>**).

So, to create an initialization message file of a smart contract you can smply run:

```bash
fift -s new-wallet.fif <workchain-id> <n> <k> [<filename-base>]
```

Where `<workchain-id>` is 0 or -1; `<n>` is total number of public/private keypairs; `<k>` is a min number of signatures, needed to send an order; `<filename-base>` is a name of a file for message, private keys, addr file and etc (by default == 'new-wallet').

**You have 600 seconds (10 minutes) to deploy a contract**, because of `valid_until` field, value is setted up to `now + 600` (*you can change this value on 58 line of 'new-wallet.fif' file*).

Deploying of a smart contract is performing by easily sending command `sendfile new-wallet-query.boc` into a blockchain.

------

**UPD:** **<u>I have found a bug in FunC compilator</u>**  (that's why I have added to an archive a file 'smart-contract.fif' - already compiled smart-contract from 'smart-contract.fc' file). I have [described this bug on GitHub](https://github.com/ton-blockchain/ton/issues/102). This is a problem with some stack registers (s16, s17, s18 and so on). Please, look at *fiftbase.pdf*, *7.2* on page *60*:

> • s0. . . s15 ( – s), words similar to s1, but pushing the Slice represent- ing other “stack registers” of TVM. **Notice that s16...s255 must be accessed using the word s().**
> • s() (x – s), takes an Integer argument 0 ≤ x ≤ 255 and returns a special Slice used by the Fift assembler to represent “stack register” s(x).

So, FunC compiler have bring to me some code:

```asm
s1 s17 XCHG
s0 s17 XCHG
```

It doesn't work. I have changed it to:

```assembly
s1 17 s() XCHG
s0 17 s() XCHG
```

And everything become fine! So if you will make a compilation from .fc file, like this:

```bash
func -o smart-contract.fif lib/stdlib.fc smart-contract.fc
```

Then you must, before using 'new-wallet.fif', change two lines of code in `smart-contract.fif`  with `s17` stack register. Or you can simply make some 'find and replace' in any text editor, using some regular expressions:   `/s(16|17|18)/` -> `$1 s()`

------

### List of wallet interface get methods:

1. **seqno** *(no arguments)*:

   Return a current `seqno` value of a smart contract (stored in persistent storage).

2. **orders** *(no arguments)*:

     Return list of ALL pending orders from a persistent storage of a smart contract.

    **Example:**

    `runmethod 0QCJNbaYdpvr-gcjwgGMyY-sHiReq9mKadN6pECt5XxtLpZi orders`

    **Return:**

    ![](https://habrastorage.org/webt/zh/04/ag/zh04agrmzpquvvfxy4kubquh0rq.png)

    **Schema:**

    `[ [ <ext_message_slice_hex> <number_of_valid_signatures> [ <wc_of_a_destination_address> <256_bit_integer_of_a_dest_address> ] <amount_in_nanograms> ] [ ... next pending message ... ] ]`

3. **orders_signature_check** *(arguments: int direction, int key_id)*:

    **Arguments:**

    1. **direction**: can be -1 or 0. If -1 user will get all pending orders, that signed by his private key. If 0 user will get all pending orders, NOT SIGNED by his private key;
    2. **key_id**: can be from 1 to 99. It is an ID of a private/public keypair. For example, user has created a smart contract wallet with 5 public/private key pairs, so he have files like: `new-wallet1.pk`, `new-wallet2.pk`, ..., `new-wallet5.pk`. So ID of a first private key is 1, second is 2 and so on.

    **Example:**

    `runmethod 0QCJNbaYdpvr-gcjwgGMyY-sHiReq9mKadN6pECt5XxtLpZi orders_signature_check -1 1`

    **Return:**

    The same as in previous method. If there is 0 finded pending orders, then result will be:

    `[ (null) ]`

------

### List of wallet interface scripts:

1. `new-wallet.fif` - **to create of a new wallet smart contract**;

    `usage: new-wallet.fif <workchain-id> <n> <k> [<filename-base>] 
    Creates a new multi-signature wallet (at least k of n signatures are required for a transfer order to be processed by the multisig wallet) in specified workchain, with private keys saved to or loaded from files with names like this: <filename-base>1.pk, <filename-base>2.pk, ..., <filename-base><n>.pk
    ('new-wallet.pk' by default)`

    **Example:**

    `fift -s new-wallet.fif 0 10 3`

2. `send-order.fif` - **to create (BoC files with serialized) external messages with completely new orders**;

    `usage: send-order.fif <filename-base> <dest-addr> <seqno> <amount> <valid-until> <signatures-number> <pkid1> <pkid2> ... <pkidN>`
    `Creates a request to multi-signature wallet created by new-wallet.fif, with <signatures-number> signatures, generated using private keys, loaded from files like: <filename-base><pkid1>.pk, <filename-base><pkid2>.pk, ..., <filename-base><pkidN>.pk and address from <filename-base>.addr, and saves it into wallet-query.boc`

    **Example:**

    `fift -s send-order.fif 'new-wallet' 0QBAdPgydUqY9uaL0i_aHmSpExuu6xIH7_6gCwWu2PMJKYAy 1 2.5 1570686177 2 3 4`  

    *(seqno=1; amount=2.5GRM; valid_until=1570686177 + 2 signatures: with 'new-wallet3.pk' and 'new-wallet4.pk')*

3. `parse-order.fif` - **to extract and show the internal message (especially its destination address and value) and the list of signatures from a previously serialized external messages (loaded from a file)**;

    `usage: parse-order.fif <filename-base>`

    `Parse all information from previously serialized external messages <filename-base>.boc file:(especially its destination address and value) and the list of signatures.`

    **Example:**

    `fift -s parse-order.fif wallet-query`

    *(parse 'walet-query.boc' file with an external message)*

4. `add-signatures.fif` - **to add new signatures to such external messages using a local private key file**;

    `usage: add-signatures.fif <filename-base> <private-key-filename-base> <signatures-number> <pkid1> <pkid2> ... <pkidN>
    To an external message from file <filename-base>.boc add new <signatures-number> signatures, generated using private keys, loaded from files like: <private-key-filename-base><pkid1>.pk, <private-key-filename-base><pkid2>.pk, ..., <private-key-filename-base><pkidN>.pk.`

    **Example:**

    `fift -s add-signatures.fif wallet-query new-wallet 1 8`

    *(add to a message from file 'wallet-query.boc' one signature with private key from file new-wallet8.pk)*

5. `merge-orders.fif` - **to merge two external messages with the same body, but with different signature sets, into one external message with the union of these signature sets**;

    `usage: merge-orders.fif <filename-base> <filename-base-2> <result-filename>`

    `Merge two messages from files: <filename-base>.boc, <filename-base-2>.boc and write it into <result-filename>.boc`

    **Example:**

    `fift -s merge-orders.fif wallet-query wallet-query2 result-query` 

    *(add to wallet-query.boc all new signatures from wallet-query2.boc and write resulted message to a file 'result-query.boc')*

6. `generate-order.fif` - **to create and sign a "new order" external message corresponding to a partially signed order recovered from the current state of the blockchain using one of the get-methods**.

    `usage: generate-order.fif <filename-base> <private-key-filename-base> <message-hex-filename-base> <seqno> <valid_until> <signatures-number> <pkid1> <pkid2> ... <pkidN>
    `

    `Create and sign a 'new order' external message corresponding to a partially signed order, recovered from the current state of the blockchain using one of the get-methods. <message-hex-filename-base>.fif - file, where you must store 'x{HEX_OF_MESSAGE_RETURNING_BY_GET_METHOD}'; <signatures-number> - number of signatures; <pkid1> <pkid2> ... <pkidN> - ids of private keys; a message will be saved into <filename-base>.boc file; <private-key-filename-base> - filebase of private keys file, like new-wallet1.pk.`

    **Example:**

    If you run some get method (`orders` or `orders_signature_check`) and get:

    `result:  [ [ [ CS{Cell{00704200203a7c193aa54c7b7345e917ed0f3254898dd7758903f7ff500582d76c798494a4a817c8000000000000000000000000000054455354} bits: 0..448; refs: 0..0} 2 [ 0 29154689562286904619410045527805962066422501564504660593886284751512775362857 ] 2500000000 ] (null) ] ]`

    Then you need copy the whole *HEX* value of `CS{Cell{HEX}}`, in this example it will be: `00704200203a7c193aa54c7b7345e917ed0f3254898dd7758903f7ff500582d76c798494a4a817c8000000000000000000000000000054455354`, then store it in some .fif file, for example, in `message-hex.fif`:

    `x{00704200203a7c193aa54c7b7345e917ed0f3254898dd7758903f7ff500582d76c798494a4a817c8000000000000000000000000000054455354}`

    After that you can run:

    `fift -s generate-order.fif wallet-query new-wallet message-hex 1 1580804686 2 3 4`

    This command will generate a new message with the same internal msg, that you get from list of pending orders of smart contract, but with new signatures, seqno and valid_until. For example if in your pending orders you have order with 2 signatures and you need 6 signatures for successful sending of a message, then you can sign a message with only 4 OTHER signatures, send it, and smart contract will automatically resend it to needed destination. It's because of signatures accumulation functionality in smart contract.

    *(creates a message with a message body readed from file message-hex.fif (in format: 'x{HEX_OF_A_MESSAGE}'). 0xHEX you can get from any of two get methods of a smart-contract)*

    ------

    ### Working example:

    **Smart contract address:** 
    **0:479e46c71628ba1b240c23665fa0a01509bd20fa11c0b19793f66cc3322f1039** 
    *Non-bounceable*: **0QBHnkbHFii6GyQMI2ZfoKAVCb0g-hHAsZeT9mzDMi8QObQd**
    *Bounceable*: **kQBHnkbHFii6GyQMI2ZfoKAVCb0g-hHAsZeT9mzDMi8QOenY**

    **<u>N = 20</u>; <u>K = 6</u>**

    https://test.ton.org/testnet/account?account=0QBHnkbHFii6GyQMI2ZfoKAVCb0g-hHAsZeT9mzDMi8QObQd