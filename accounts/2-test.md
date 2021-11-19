# Composing our tests

Let's look at our tests at `tests/simple-counter.ts`

```js
import * as anchor from '@project-serum/anchor';

describe('simple-counter', () => {

  // Configure the client to use the local cluster.
  anchor.setProvider(anchor.Provider.env());

  it('Is initialized!', async () => {
    // Add your test here.
    const program = anchor.workspace.SimpleCounter;
    const tx = await program.rpc.initialize();
    console.log("Your transaction signature", tx);
  });
```

What changes do we need to make to our test to model our program?

Recall that as part of our transaction, we need to add in every account that is
used. Which accounts do we need to include here, and how do we include them?

Let's look back at what the accounts we need for the initialize instruction:

```rust
#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(init, payer = user, space = 8 + 8)]
    pub counter: Account<'info, Counter>,
    #[account(mut)]
    pub user: Signer<'info>,
    pub system_program: Program<'info, System>,
}
```

We have three accounts to include: the counter, the user, and the system
program. Let's write some code first, then we can explain in detail what's going
on at each step.

```js
import * as anchor from "@project-serum/anchor";

describe("simple-counter", () => {
  // Configure the client to use the local cluster.
  const provider = anchor.Provider.env();
  anchor.setProvider(provider);

  it("Is initialized!", async () => {
    // Add your test here.
    const program = anchor.workspace.SimpleCounter;

    const counter = anchor.web3.Keypair.generate();

    const tx = await program.rpc.initialize({
      accounts: {
        counter: counter.publicKey,
        user: provider.wallet.publicKey,
        systemProgram: anchor.web3.SystemProgram.programId,
      },
      signers: [counter],
    });
    console.log("Your transaction signature", tx);
  });
});
```

We've changed a few things here, so let's dive in.

```js
const provider = anchor.Provider.env();
anchor.setProvider(provider);
```

First, we set our provider to a variable so we can reference it here and below.

```js
const counter = anchor.web3.Keypair.generate();
```

The next change we made is to generate a new keypair. This reaches in to the
anchor library for the `Keypair` class and calls `generate()` to create a new
one.

By keypair, anchor means it is generating a new public and private key. This
keypair will represent the account that we are going to store our counter in. We
then get the public key of that keypair.

```js
const tx = await program.rpc.initialize({
  accounts: {
    counter: counter.publicKey,
    user: provider.wallet.publicKey,
    systemProgram: anchor.web3.SystemProgram.programId,
  },
  signers: [counter],
});
```

Finally, we edit our instruction call to include our accounts. Note that
whenever we include an account in the set of accounts, we have to reference the
public key.

The first account is including the counter public key that we generated earlier.
This will tell our program to initialize an account with this public key
address.

The second account is our user. When we spin up our provider earlier, Anchor
generates a wallet that is used for testing these transactions. This wallet is
basically what you would use when you transact on Solana normally, and has its
own public and private keys. We include the public key of the wallet's keypair
here to tell our program that we want to use the Anchor generated wallet for
initializing our program.

The third account is the system program. Anchor includes, as part of its
package, a constant value for the program id of the system program. We include
that there as well.

Finally, we also include a signers block. Any account that is initialized by the
system program has to be included as a signer of the transaction where it is
created.

If we look back to our program, we'll see that the user account has the `Signer`
type. Why don't we have to include it as a signer? That's because our
`provider.wallet` is already signing the transaction, so the user account is
implicitly a signer.

Let's run our tests!

```bash
anchor test
```

We should see the following output:

```
  simple-counter
Your transaction signature <some-hash>
    ✓ Is initialized! (359ms)


  1 passing (361ms)
```

Our tests are currently passing, as expected!

# Did we actually test anything?

We successfully ran the transaction, but we didn't actually check to see if the
counter has the data we expect! This is important to ensuring that, as we extend
our program, we can ensure that the right accounts have the right data.

To do that, we'll need to add an assertion to our tests. The first thing we need
to do is import `assert` from `chai`. At the top of our file, add:

```js
import { assert } from "chai";
```

Finally, below where we call our transaction, we can add the following code:

```js
const counterData = await program.account.counter.fetch(counter.publicKey);

assert.ok(counterData.count.eq(new anchor.BN(0)));
```

Inside the IDL, Anchor outputs the shape of the accounts that we defined in our
program. Therefore, Anchor knows how to deserialize data from our counter
account in our tests. The `counterData` object will have the same shape as the
`Counter` account struct we defined in our Rust program.

Then we have our assertion. We want to ensure that the value of count is equal
to 0, since that's what we're initializing our counter at.

`new anchor.BN(0)` just represents the binary value `0`. Anchor doesn't support
directly casting between javascript's number type and anchor's deserialized
integer. That means using `0` directly wouldn't work here.

Let's run `anchor test` again.

```bash
  simple-counter
    ✓ Is initialized! (388ms)


  1 passing (391ms)
```

Just for fun, let's see what happens if we change `anchor.BN(0)` to
`anchor.BN(1)` and re-run our tests.

```bash
simple-counter
    1) Is initialized!


  0 passing (507ms)
  1 failing

  1) simple-counter
       Is initialized!:
     AssertionError: expected false to be truthy
     ...
```

We can be pretty sure at this point that we are initializing our counter with
the correct value! Let's change it back to `anchor.BN(0)` and move on.
