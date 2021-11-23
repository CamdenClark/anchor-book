The counter we've built will let _anyone_ call increment after the
initialization.

But what if we only wanted one account to be able to increment?

# Access control

How do we implement access control into our existing counter program?

At a high level, we need to maintain some state inside the counter account that
tracks who is authorized to increment the counter. Then, when we call the
`increment` instruction, we need to be able to check that the person who is
authorized to increment the counter signed our transaction. As part of the
`increment` instruction, we'll need to pass another account that represents the
authority, and do that check.

With those high-level pieces in place, let's add a test (that will fail
currently) that will validate our access control piece.

# Writing a test

We'll write some code and then explain. We'll edit our second test and add a
third test like so:

```js
it("Increments the counter", async () => {
  await program.rpc.increment({
    accounts: {
      counter: counter.publicKey,
      authority: provider.wallet.publicKey,
    },
  });

  const counterData = await program.account.counter.fetch(counter.publicKey);

  assert.ok(counterData.count.eq(new anchor.BN(3)));
});

it("Another user attempting to increment throws an error", async () => {
  let transactionFailed = false;

  try {
    await program.rpc.increment({
      accounts: {
        counter: counter.publicKey,
        authority: anchor.web3.Keypair.generate().publicKey,
      },
    });
  } catch {
    transactionFailed = true;
  }

  assert.ok(transactionFailed);
});
```

First, we add the `authority` account as part of the second
(`Increments the counter`) test. We set its value to `provider.wallet.publicKey`
because the original creator of the counter attmepting to increment the counter
should succeed.

Then, we add a new test that ensures that we throw an error if the authority
doesn't match the creator of the counter. This time, instead of setting the
authority to the `provider.wallet.publicKey`, we generate a new keypair, which
shouldn't have access to increment the counter.

Finally, we have one change to make so that the tests will actually run. In our
program, we have to change the account context `Increment` to include the
authority account.

```rust
#[derive(Accounts)]
pub struct Increment<'info> {
    #[account(mut)]
    pub counter: Account<'info, Counter>,
    pub authority: Signer<'info>,
}
```

If we run `anchor test`, we'll see the third test fail because we haven't
actually implemented access control: the `increment` instruction didn't throw an
error.

```bash
  simple-counter
    ✓ Initializes the counter to 2 (376ms)
    ✓ Increments the counter (422ms)
    1) Another user attempting to increment throws an error


  2 passing (1s)
  1 failing

  1) simple-counter
       Another user attempting to increment throws an error:
     AssertionError: expected false to be truthy
      at /home/camden/anchor-book-code/simple-counter/tests/simple-counter.ts:57:12
      at Generator.next (<anonymous>)
      at fulfilled (tests/simple-counter.ts:24:58)
      at processTicksAndRejections (node:internal/process/task_queues:96:5)
```

Our next job is to actually implement access control!

# Make the tests pass

We need to do two things:

1. Store who initialized the counter
2. Verify that the caller of the counter

## Storing who initialized the counter

Let's change our account struct to store what we'll call the `authority`.

```rust
#[account]
pub struct Counter {
    pub count: u64,
    pub authority: Pubkey,
}
```

Now, when we initialize the counter, the account data will also contain a
`Pubkey` field, `authority`.

Let's see how we'll store this in the `initialize` instruction handler.

```rust
pub fn initialize(ctx: Context<Initialize>, value: u64) -> ProgramResult {
	let counter = &mut ctx.accounts.counter;
	counter.authority = ctx.accounts.user.key();
	counter.count = value;
	Ok(())
}
```

The only change we've made here is to set `counter.authority` to
`ctx.accounts.user.key()`. We take the `user` account that we passed (in the
tests, the wallet signing the instruction), and set the counter `authority` to
its public key.

Let's run `anchor test`.

```bash
  1 passing (192ms)
  2 failing

  1) simple-counter
       Initializes the counter to 2:
     Error: 163: Failed to deserialize the account
      at Function.parse (node_modules/@project-serum/anchor/src/error.ts:41:14)
      at Object.rpc [as initialize] (node_modules/@project-serum/anchor/src/program/namespace/rpc.ts:23:42)
      at processTicksAndRejections (node:internal/process/task_queues:96:5)

  2) simple-counter
       Increments the counter:
     Error: 167: The given account is not owned by the executing program
      at Function.parse (node_modules/@project-serum/anchor/src/error.ts:41:14)
      at Object.rpc [as increment] (node_modules/@project-serum/anchor/src/program/namespace/rpc.ts:23:42)
      at processTicksAndRejections (node:internal/process/task_queues:96:5)
```

Hm, we failed to deserialize the account in the initialization test. What
happened here?

Recall when we had to set the `space` property on the `Initialize` account
context.

```rust
#[account(init, payer = user, space = 8 + 8)]
pub counter: Account<'info, Counter>,
```

The space is set to `8 + 8` because of the 8 byte discriminator, and the 8 byte
count property. We've now added a new property, so the amount of space we
allocate isn't enough to store our `Pubkey` object. But what's the right amount
to store?

Solana uses 256-bit public keys, and 256 bits is equal to 32 bytes. That seems
like a good thing to try! Let's set our space to `8 + 8 + 32`.

```rust
#[account(init, payer = user, space = 8 + 8 + 32)]
pub counter: Account<'info, Counter>,
```

Now, let's run `anchor test`

```
simple-counter
    ✓ Initializes the counter to 2 (360ms)
    ✓ Increments the counter (412ms)
    1) Another user attempting to increment throws an error


  2 passing (1s)
  1 failing

  1) simple-counter
       Another user attempting to increment throws an error:
     AssertionError: expected false to be truthy
      at /home/camden/anchor-book-code/simple-counter/tests/simple-counter.ts:57:12
      at Generator.next (<anonymous>)
      at fulfilled (tests/simple-counter.ts:24:58)
      at processTicksAndRejections (node:internal/process/task_queues:96:5)
```

Now we're back to the original state of the tests: only the last one fails
because we haven't actually checked if the user calling increment matches the
authority we're storing.

## Checking the authority

We might be tempted to do something like the following inside the `increment`
instruction handler itself to verify that the authority is equal.

```rust
pub fn increment(ctx: Context<Increment>) -> ProgramResult {
	let counter = &mut ctx.accounts.counter;
	if (counter.authority != ctx.accounts.authority) {
		// Throw some error
	}
	counter.count += 1;
	Ok(())
}
```

We don't need to do this ourselves in the instruction handler. Anchor has a
mechanism to do access control for us inside of our account handlers.

```rust
#[derive(Accounts)]
pub struct Increment<'info> {
    #[account(mut, has_one=authority)]
    pub counter: Account<'info, Counter>,
    pub authority: Signer<'info>,
}
```

Our only change here is to add `has_one = authority` to the decorator. What's
going on here? According to the
[Anchor docs](https://docs.rs/anchor-lang/0.11.1/anchor_lang/derive.Accounts.html):

> `#[account(has_one = <target>)]` Checks the target field on the account
> matches the target field in the struct deriving Accounts.

What that means here is that Anchor will enforce that the `authority` field on
the `counter` account data will be equal to the `authority` account passed in to
the increment instruction. Functionally, this means that if the user's account
doesn't match the one set in the counter account, Anchor will throw an error.

Let's run `anchor test` one more time.

```
simple-counter
    ✓ Initializes the counter to 2 (345ms)
    ✓ Increments the counter (418ms)
Transaction simulation failed: Error processing Instruction 0: custom program error: 0x8d
    Program Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS invoke [1]
    Program log: Custom program error: 0x8d
    Program Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS consumed 3205 of 200000 compute units
    Program Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS failed: custom program error: 0x8d
     ...
     snip
     ...
    ✓ Another user attempting to increment throws an error (118ms)


  3 passing (889ms)
```

Awesome! Now we have created counters that only the original creator can
increment!
