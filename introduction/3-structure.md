{% hint style="info" %} If you're familiar with the architecture of
decentralized apps on Ethereum, you can probably skip this section.
{% endhint %}

The structure and capability of decentralized applications is significantly
different from that of centralized applications. Let's look, at a

# Traditional application infrastructure

In typical web applications, we (very very) roughly have the following
architecture:

_TODO_: Diagram:

Client (Browser) => Web server => Database / Storage

We have a client, like a browser or mobile app, that makes calls to a web
server. That web server performs some business logic, and calls out to a
database or external storage system to create, read, update, or delete
persistent data.

# dApp structure

On the other hand, decentralized applications on Solana roughly have the
following architecture:

_TODO_: Diagram:

Client (Browser) => RPC server => Program (blockchain)

Similarly, decentralized applications have a client, like a browser or mobile
app.

You can think of the RPC server as the gateway to the blockchain for clients.
This is the piece that receives transactions and communicates them to the Solana
network, where Solana clusters will verify the integrity of those transactions
and perform the instructions provided in the transaction.

Finally, the code for the program is executed. It has the capability to perform
computation (business logic) and store data on the blockchain

# Comparing and contrasting

Traditional application infrastructure and dApps have significantly different
constraints. Here are some examples of differences and similarities.

## Storing lots of data

Storing lots of data is complicated in decentralized application structures.
Every extra byte of storage has to be allocated across all validator nodes,
which means that storage is very expensive.

The Solana blockchain is not the place to store image files, for example. (It's
probably technically possible by using many accounts, but that would be
prohibitively expensive)

The best way to store these things is off-chain somewhere, either using a
decentralized service like IPFS/Arweave or using a centralized service.

## Storing hidden data

Storing hidden data is also not a good idea on the blockchain. Since every
transaction, and the contents of the data of each transaction, are public
knowledge, you can't permission _read access_ to program data.

This means applications such as 1-1 direct messaging aren't _necessarily_ good
candidates for Solana.

{% hint style="info" %} You probably could do 1-1 direct messaging if both users
had shared public PGP keys before messaging and only write encrypted messages
onto the blockchain. However, you'd still leak knoweldge that two different
accounts are posting messages at a certain time. This is more information than
is exposed to third parties (other than the messaging platform) in centralized
messaging platforms. {% endhint %}

However, programs can permission _write access_ to account data. This is the
foundation of having programs on the blockchain: you can make permissioned edits
to the global state machine!

# Some kind of hybrid

Keep in mind that it's possible (and often necessary, due to technological
limitations, to support a user experience comparable to traditional web
applications) to make applications that have centralized components.

An example of this is `mirror.xyz`, which is a decentralized blogging platform.
Mirror stores posts on Arweave, so all posts are available off the centralized
platform. However, mirror makes use of having a traditional web server to
support publishing posts without publishing data onto the Ethereum blockchain.
You can read about
[mirror here](https://dev.mirror.xyz/J1RD6UQQbdmpCoXvWnuGIfe7WmrbVRdff5EqegO1RjI).
