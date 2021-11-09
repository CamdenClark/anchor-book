{% hint style="info" %}
If you're familiar with the architecture of decentralized apps on Ethereum, you can probably skip this section.
{% endhint %}

# Traditional application infrastructure

In typical web applications, we (very) roughly have the following architecture:

Client (Browser) => Web server => Database / Storage

We have a client, like a browser or mobile app, that makes calls to a web server. That web server performs some
business logic, and calls out to a database or external storage system to retrieve or mutate persistent data.

# dApp structure

On the other hand, decentralized applications on Solana roughly have the following architecture:

Client (Browser) => RPC server => Blockchain (program)

Similarly, we have a client, like a browser or mobile app. The RPC server is the piece that receives transactions
and communicates them to the Solana network, where Solana clusters will verify the integrity of those transactions
and perform the instructions provided in the transaction. The persistent data for programs is stored within the blockchain.

# Comparing and contrasting

Traditional application infrastructure and dApps have significantly different constraints. Here are some examples of differences
and similarities.

## Storing lots of data

Storing lots of data is complicated in decentralized application structures. Every extra byte of storage has to be allocated
across all validator nodes, which means that storage is very expensive.

The Solana blockchain is not the place to store image files, for example. (It's probably technically possible by using many accounts,
but that would be prohibitively expensive)

The best way to store these things is off-chain somewhere, either using a decentralized service like IPFS/Arweave or using
a centralized service.

## Storing hidden data

Storing hidden data is also not a good idea on the blockchain. Since every transaction, and the contents of the data of each transaction,
are public knowledge, you can't permission _read access_ to program data.

This means applications such as 1-1 direct messaging aren't good candidates for Solana.

{% hint style="info" %}
You probably could do 1-1 direct messaging if both users had shared public PGP keys before messaging and only write encrypted messages
onto the blockchain. However, you'd still leak knoweldge that two different accounts are posting messages at a certain time. This
is more information than is exposed to third parties (other than the messaging platform) in centralized messaging platforms.
{% endhint %}

However, programs can permission _write access_ to account data.
