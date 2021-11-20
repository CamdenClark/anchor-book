Currently, when we initialize our counter, it always starts from 0. But what if
we wanted to start the counter from a specific number? That would require
passing in the value that we want to initialize the counter to as an argument.

You'll learn how to pass in a value to our `initialize` instruction, which will
teach you how to pass arguments more generally to your instructions.

Let's start with the tests!

# Adjusting our tests

Our test changes here will be really simple. We are going to add the value to
the instruction caller, and then change our assertions to reflect that.

```js
describe("simple-counter", () => {
  // Configure the client to use the local cluster.
  const provider = anchor.Provider.env();
  anchor.setProvider(provider);

  const counter = anchor.web3.Keypair.generate();
  const program = anchor.workspace.SimpleCounter;

  it("Initializes the counter to 2", async () => {
    await program.rpc.initialize(2, {
      accounts: {
        counter: counter.publicKey,
        user: anchor.Provider.env().wallet.publicKey,
        systemProgram: anchor.web3.SystemProgram.programId,
      },
      signers: [counter],
    });

    const counterData = await program.account.counter.fetch(counter.publicKey);

    assert.ok(counterData.count.eq(new anchor.BN(2)));
  });

  it("Increments the counter", async () => {
    await program.rpc.increment({
      accounts: {
        counter: counter.publicKey,
      },
    });

    const counterData = await program.account.counter.fetch(counter.publicKey);

    assert.ok(counterData.count.eq(new anchor.BN(3)));
  });
});
```

There are three changes here:

```js
it("Initializes the counter to 2", async () => {
  await program.rpc.initialize(new anchor.BN(2), {
    // ...
  });
});
```

First, we add in a parameter to our `initialize` function before the accounts
that represents the number that we want to initialize the counter to.

```js
assert.ok(counterData.count.eq(new anchor.BN(2)));
//...
assert.ok(counterData.count.eq(new anchor.BN(3)));
```

Then, we change our assertions to represent the updated values: 2 for the
initial counter state, and 3 after incrementing it.

# Updating our instruction

Since we're not changing anything about our account structure, we don't need to
update either of the account context or the `Counter` account struct. We aren't
changing the `increment` instruction handler either since it's not dependent on
the initial value of the counter.

Our change is isolated to the instruction handler for `initialize`. Let's make
the adjustment:

```rust
pub fn initialize(ctx: Context<Initialize>, value: u64) -> ProgramResult {
	let counter = &mut ctx.accounts.counter;
	counter.count = value;
	Ok(())
}
```

By convention, the context parameter always goes first, followed by arguments.
`value` is added as the second argument to the `initialize` handler.

Finally, we set `counter.count` to the value that's passed in to initialize.

Let's run our tests with `anchor test`!

```bash
  simple-counter
    ✓ Initializes the counter to 2 (252ms)
    ✓ Increments the counter (422ms)


  2 passing (678ms)
```
