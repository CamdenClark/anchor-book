Now that we can initialize our counter to zero, let's implement the ability to
increment the counter.

We'll start with the tests that test this functionality, then actually implement
it in the program and watch the tests pass. This is a good way to ensure that
you're thinking about the design constraints ahead of time!

# At a high level

Our initialize instruction creates a new account and sets its `count` data to 0.
What does it mean to increment our counter?

We want to mutate the `count` data in our existing counter account to increase
its value by one. Specifically, we want to do this in a new instruction. We
don't want to increment the value of our counter in the same instruction that we
initialize it in.

What would our tests look like? We will want to run the initialization of a
counter account first, like our existing tests. But then, we'll need to reuse
the same account, run our new instruction, and verify that the value of the
count has increased by one.

What accounts will we need for our instruction? For the initialize instruction,
we needed to include three accounts: the counter, the user, and the system
program. We'll definitely need to include the counter account again. However, we
won't need to include the user and system program accounts. We only included
these because the counter account wasn't actually initialized. We needed to
reference the user account to pay for rent on the counter account
initializiation, and we needed to reference the system program account because
it controls the initialization of accounts. Since our counter account is already
initialized, we don't need the system program for anything.

# Writing our new test

Let's write our new test:

```js
import * as anchor from "@project-serum/anchor";
import { assert } from "chai";

describe("simple-counter", () => {
  // Configure the client to use the local cluster.
  anchor.setProvider(anchor.Provider.env());

  const counter = anchor.web3.Keypair.generate();
  const program = anchor.workspace.SimpleCounter;

  it("Initializes the counter to 0", async () => {
    await program.rpc.initialize({
      accounts: {
        counter: counter.publicKey,
        user: anchor.Provider.env().wallet.publicKey,
        systemProgram: anchor.web3.SystemProgram.programId,
      },
      signers: [counter],
    });

    const counterData = await program.account.counter.fetch(counter.publicKey);

    assert.ok(counterData.count.eq(new anchor.BN(0)));
  });

  it("Increments the counter", async () => {
    await program.rpc.increment({
      accounts: {
        counter: counter.publicKey,
      },
    });

    const counterData = await program.account.counter.fetch(counter.publicKey);

    assert.ok(counterData.count.eq(new anchor.BN(1)));
  });
});
```

We've made a few changes here from the last chapter.

First, we've broken out the counter and the program out of the `it` block, so
they can be referenced across multiple `it` blocks.

We add a new `it` block to describe what incrementing the counter should look
like.

Inside this it block, we first call our new instruction: `increment`. We only
include the counter account, like we described above.

Similar to the initialization test, we fetch the data, and assert that the
`count` value is at `1` instead of `0`, which means that our instruction has
successfully incremented.

Let's run `anchor test` to see what happens:

```bash
  simple-counter
    ✓ Initializes the counter to 0 (351ms)
    1) Increments the counter


  1 passing (368ms)
  1 failing

  1) simple-counter
       Increments the counter:
     TypeError: program.rpc.increment is not a function
```

Our first test passes, but then the second test fails because `increment` is not
a function. This is expected: we haven't created our new instruction handler.
Let's implement this on the plet counter = &mut ctx.accounts.counter;
counter.count += 1; Ok(())rogram side now.

# Implementing the `increment` instruction handler

Let's open our program again.

Previously, we added a `Counter` account struct, added a new account context
called `Initialize`, and then implemented the instruction handler for the
`initialize` instruction.

We don't need to add a new account struct since we're just mutating the data in
an existing `Counter` account.

However, we will need to add a new account context, and implement the
instruction handler for `increment`.

## Account context

Let's add, below the `Initialize` account context, the following code:

```rust
#[derive(Accounts)]
pub struct Increment<'info> {
    #[account(mut)]
    pub counter: Account<'info, Counter>,
}
```

We omit the user and system program instructions as described above. We also
change the decorator we used when initializing the `counter` account previously.
We don't need to do any account initialization, rent paying, or space
allocation, but we do need to get a mutable reference to the `counter` data.
Tagging the account with `#[account(mut)]` will make sure that Anchor saves our
changes after we mutate the data inside the counter.

## Instruction handler

Below the previous `initialize` instruction handler, add the following code:

```rust
pub fn increment(ctx: Context<Increment>) -> ProgramResult {
	let counter = &mut ctx.accounts.counter;
	counter.count += 1;
	Ok(())
}
```

Like the initialize handler, we get a mutable reference to the counter account
data. We increment the count value, and return `Ok(())`. Anchor, under the hood,
will save our changes to the counter account data.

Let's re-run our tests with `anchor test`.

```bash
  simple-counter
    ✓ Initializes the counter to 0 (415ms)
    ✓ Increments the counter (411ms)


  2 passing (830ms)
```

Awesome!
