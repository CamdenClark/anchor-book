Up to this point, we've made it so that our counter could be incremented or
initialized to any value.

What if we wanted to make it so that the counter could only be initialized or
incremented to a value under 100? If you tried to initialize a counter at 100 or
above, it would throw a specific error to the user. And, if you tried to
increment a counter from 99 to 100, it would report a similar error.

Let's see how we could change our program to support this feature.

{% hint style="info" %} You can find the code that covers this section
[here](https://github.com/CamdenClark/anchor-book-code/tree/main/simple-counter-5)
{% endhint %}

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
    await initializeCounter(100);
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
  3 passing (2s)
  1 failing

  1) simple-counter
       Initializing counter to 100 or above throws an error:
     AssertionError: expected false to be truthy
     ...snip...
```

We see that our newly created test fails.

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
  4 passing (3s)
```

Awesome!

# The final test

We made it so the counter can't be initialized to 100 or above, but we also
wanted to make it so you can't increment it to 100 or above either. Let's add
one last test for this.

```js
it("Incrementing above 99 fails", async () => {
  const counter = await initializeCounter(99);

  let transactionFailed = false;

  try {
    await program.rpc.increment({
      accounts: {
        counter,
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

```
  5 passing (2s)
```
