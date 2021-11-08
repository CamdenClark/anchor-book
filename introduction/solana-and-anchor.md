# What is Solana?

Solana, at its core, is a public proof-of-stake blockchain with some really key optimizations that give it high transaction throughput for low cost.

[The Solana docs](https://docs.solana.com/cluster/overview) explain the core architecture in detail.

For our purposes, developers are able to deploy programs onto this blockchain. These programs form the foundation for decentralized applications, spanning
use cases from DeFi to NFTs.

# What is Anchor?

Anchor intends to be the Rails of the Solana ecosystem: a framework for writing Solana programs that are easier to understand and iterate on.
Anchor wants to minimize the cognitive load on programmers to develop Solana applications.

Writing programs on Solana without using Anchor is really difficult. There's a ton of boilerplate, you have to serialize and deserialize to bytes yourself.
