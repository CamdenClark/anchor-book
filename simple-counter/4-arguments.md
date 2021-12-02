Currently, when we initialize our counter, it always starts from 0. But what if
we wanted to start the counter from a specific number? That would require
passing in the value that we want to initialize the counter to as an argument.

You'll learn how to pass in a value to our `initialize` instruction, which will
teach you how to pass arguments more generally to your instructions.

Let's start with the tests!

{% hint style="info" %} You can find the code that covers this section
[here](https://github.com/CamdenClark/anchor-book-code/tree/main/simple-counter-3)
{% endhint %}

# Adjusting our tests

Our test changes here will be really simple. We are going to add the value to
the instruction caller, and then change our assertions to reflect that.

```js
import * as anchor from "@project-serum/anchor";
import { assert } from "chai";

const provider = anchor.Provider.env();
anchor.setProvider(provider);

const program = anchor.workspace.SimpleCounter;

const initializeCounter = async (amount: number) => {
  const counter = anchor.web3.Keypair.generate();

  await program.rpc.initialize(new anchor.BN(amount), {
    accounts: {
      counter: counter.publicKey,
      user: provider.wallet.publicKey,
      systemProgram: anchor.web3.SystemProgram.programId,
    },
    signers: [counter],
  });

  return counter.publicKey;
};

describe("simple-counter", () => {
  it("Initializes the counter to 2", async () => {
    const counter = await initializeCounter(2);

    const counterData = await program.account.counter.fetch(counter);

    assert.ok(counterData.count.eq(new anchor.BN(2)));
  });
  it("Increments the counter", async () => {
    const counter = await initializeCounter(2);

    await program.rpc.increment({ accounts: { counter } });

    const counterData = await program.account.counter.fetch(counter);

    assert.ok(counterData.count.eq(new anchor.BN(3)));
  });
});
```

First, we add in a parameter to our `initializeCounter` function that is the
number the counter should be initialized to. Then, in the `initialize`
instruction call, we add the amount parameter. We have to pass it in as
`anchor.BN`, so we have to do `new anchor.BN(amount)` to get it to work on the
other side.

```js
const counter = await initializeCounter(2);
```

We set the parameter we created for `initializeCounter` to `2` in both tests.

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
