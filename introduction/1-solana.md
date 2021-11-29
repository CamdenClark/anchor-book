# What is Solana?

Solana, at its core, is a public proof-of-stake blockchain with some really key
optimizations that give it high transaction throughput for low cost.

[The Solana docs](https://docs.solana.com/cluster/overview) explain the core
architecture in detail.

For our purposes, developers are able to deploy programs onto the Solana
blockchain as accounts. These programs form the foundation for decentralized
applications, powering use cases from DeFi to NFTs.

This guide will teach you to write and deploy programs onto the Solana
blockchain, and write web frontends that interact with those programs.

# What is Anchor?

Anchor intends to be the Rails of the Solana ecosystem: a framework for writing
Solana programs that are easier to understand and support quicker iteration.
Anchor wants to minimize the cognitive load on programmers to develop Solana
applications.

Writing programs on Solana without using Anchor is really difficult. There's a
ton of boilerplate, you have to serialize and deserialize to bytes yourself.

Anchor is the main choice for developers currently writing their projects on
Solana, so it makes sense for beginners to Solana to pick Anchor to develop
their projects and learn about the platform.

Although Anchor abstracts away a lot of the complexity, it's important to
understand what Anchor is doing behind the scenes as we build our applications.

The goal of this guide is to use examples to empower you to build real
applications on the Solana blockchain, while also teaching the core concepts
underpinning programs on Solana.
