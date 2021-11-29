Up to this point, we've made it so that our counter could be incremented or
initialized to any value.

What if we wanted to make it so that the counter could only be initialized or
incremented to a value under 100? If you tried to initialize a counter at 100 or
above, it would throw a specific error to the user. And, if you tried to
increment a counter from 99 to 100, it would report a similar error.

Let's see how we could change our program to support this feature.

# Initialization

We'll tackle making it so you can't initialize a counter to 100 or above first.

## Updating our tests

We should add a test for this new piece of functionality before we make the
adjustment so we don't break it in the future.

In our test file, add above the very first test:

```js
it("Initializing counter to 100 or above throws an error", async () => {
  let transactionFailed = false;

  try {
    await program.rpc.initialize(new anchor.BN(100), {
      accounts: {
        counter: counter.publicKey,
        user: provider.wallet.publicKey,
        systemProgram: anchor.web3.SystemProgram.programId,
      },
      signers: [counter],
    });
  } catch {
    transactionFailed = true;
  }

  assert.ok(transactionFailed);
});
```

This is the exact same as our existing initialize functionality, but our test
will fail if we don't throw an error when we attempt to initialize to 100.

Let's run `anchor test`.

```
  1 passing (721ms)
  3 failin>g

  1) simple-counter
       Initializing counter to 100 or above throws an error:
     AssertionError: expected false to be truthy
     ...snip...

  2) simple-counter
       Initializes the counter to 2:
     Error: failed to send transaction: Transaction simulation failed: Error processing Instruction 0: custom program error: 0x0
      at Connection.sendEncodedTransaction (node_modules/@solana/web3.js/src/connection.ts:3689:13)
      at processTicksAndRejections (node:internal/process/task_queues:96:5)
      at Connection.sendRawTransaction (node_modules/@solana/web3.js/src/connection.ts:3649:20)
      at sendAndConfirmRawTransaction (node_modules/@solana/web3.js/src/util/send-and-confirm-raw-transaction.ts:27:21)
      at Provider.send (node_modules/@project-serum/anchor/src/provider.ts:114:18)
      at Object.rpc [as initialize] (node_modules/@project-serum/anchor/src/program/namespace/rpc.ts:19:23)

  3) simple-counter
       Increments the counter:
     AssertionError: expected false to be truthy
     ...snip...
```

We see that we have 3 failing tests. The reason for this is that, since the
first call in our new test to initialize still succeeds, the assumptions our
other tests are based on fails.

What we are doing here is an anti-pattern in constructing tests: we want our
test state to be as isolated as possible. What we could have done instead is
construct tests that construct their own counter account data and only depend on
that state, not on other tests passing. We'll rectify this later in our
tutorials, but showing the anti-pattern here helps you see why this is a good
idea!

Either way, let's try to make the test we created pass.

## Make the test pass

First, we'll create our error enum. Anchor has a special decorator that allows
you to describe an enum as an error. Let's write some code:

```rust
#[error]
pub enum ErrorCode {
    #[msg("You can't initialize a counter to 100 or above")]
    CounterTooHigh,
}
```

We've established one of our error codes to be `CounterTooHigh`, which has an
associated message that will be sent to the user when you make this mistake.

In our `initialize` instruction handler:

```rust
if value >= 100 {
	return Err(ErrorCode::CounterTooHigh.into())
}
```

We reference the `ErrorCode::CounterTooHigh` enum value, then cast that into an
error result.

Now, let's run `anchor test` again.

```
  4 passing (851ms)
```

Awesome!

# The final test

We made it so the counter can't be initialized to 100 or above, but we also
wanted to make it so you can't increment it to 100 or above either. Let's adjust
our existing tests, then add an additional test for this.

```js
it("Initializes the counter to 98", async () => {
  await program.rpc.initialize(new anchor.BN(98), {
    accounts: {
      counter: counter.publicKey,
      user: provider.wallet.publicKey,
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
      authority: provider.wallet.publicKey,
    },
  });

  const counterData = await program.account.counter.fetch(coun>
});

it("Incrementing above 99 fails", async () => {
  let transactionFailed = false;
  try {
    await program.rpc.increment({
      accounts: {
        counter: counter.publicKey,
        authority: provider.wallet.publicKey,
      },
    });
  } catch {
    transactionFailed = true;
  }

  assert.ok(transactionFailed);
});
```

Now, we can add a similar check in the `increment` instruction handler.

```rust
if counter.count >= 99 {
	return Err(ErrorCode::CounterTooHigh.into());
}
```

and if we run `anchor test`, we should see all of our tests pass.

... _TODO_: Show passing tests
