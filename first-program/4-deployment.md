# Deploying a program

So far we haven't really done anything with our program, we've only ran tests.
What does it mean to actually deploy a program, and how can we deploy our
program right now?

There are 4 environments you can deploy your program to: local, devnet, testnet,
and mainnet-beta.

In local, devnet, and testnet environments, you are able to airdrop yourself
SOL. This allows you to test deployment of your programs without significant
cost. Locally, other users won't be able to see or interact with your program.
If you deploy to devnet or testnet, other users on those networks can see your
program and all transactions you make. However, because these aren't the real
Solana, transactions operating on devnet or testnet shouldn't have any real
repercussions.

**On the other hand, mainnet-beta is the _real_ Solana.** Deploying a program to
mainnet-beta effectively makes it publicly available to anyone on the Solana
network. That means that deploying to mainnet is something we should be really
careful about!

Today, we'll show you how to deploy to your local environment, then to devnet.
This should be sufficient to develop and share your programs with others.

# Getting set up

If you haven't used Solana before, you should generate a wallet.

```bash
solana-keygen new
```

This will generate a new keypair that will serve as your wallet.

# Deploying locally

Deploying locally gives us the freedom to test our program locally as if we were
running it in one of the real environments.

However, in order to deploy locally, we need to be running a validator locally.

Solana packages a local test validator for us, so let's kick it off.

```bash
solana-test-validator
```

You should see the test validator start up.

Recall that `declare_id!` call we discussed earlier?

```rust
declare_id!("Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS");
```

Every program has a keypair associated with it. We don't have the private key
associated with this public key. Anchor will generate its own keypair to deploy
our program with. For us, this means that we'll have to generate the keypair
first, then add the new public key here, and re-build.

Run these two commands.

```bash
anchor build
solana-keygen pubkey target/deploy/first_program-keypair.json
```

First, anchor builds the project. It generates a keypair at
`target/deploy/first_program-keypair.json`. Then, we use the `solana-keygen`
utility to output the public key of that keypair.

Copy that public key and paste in place of the old `declare_id!`.

Now, we're ready to build, then deploy!

```bash
anchor build
anchor deploy
```

{% hint style="warning" %} Make sure to stop your local validator if you go back
to using `anchor test`. The integration tests don't work if you're running your
own validator. {% endhint %}

# Deploying to devnet

Now, we can try to deploy our program to devnet

```bash
anchor deploy --provider.cluster devnet
```

You should see something like this:

```
Error: Account ... has insufficient funds for spend (1.06007064 SOL) + fee (0.003345 SOL)
```

We don't have any SOL to deploy! This would be an expensive deploy on mainnet.
On devnet, we can just give ourselves some SOL.

```
solana airdrop 5 -u devnet
```

Now, we should have enough money for our deploy!

```
anchor deploy --provider.cluster devnet
```

Note that deploying to devnet took a while, upwards of 10 seconds (on my
machine). This can affect your iteration speed when working, so working locally
and with tests will help you in active development.

Devnet is a great environment to share your experiments and give a risk-free way
for people to test out new pieces of your program's code. For active
development, you might find it limiting, but your development environment is
yours!
