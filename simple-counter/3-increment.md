Now that we can initialize our counter to zero, let's implement the ability to
increment the counter.

We'll start with the tests that test this functionality, then actually implement
it in the program and watch the tests pass.

{% hint style="info" %} You can find the code that covers this section
[here](https://github.com/CamdenClark/anchor-book-code/tree/main/simple-counter-2)
{% endhint %}

# At a high level

Our initialize instruction creates a new account and sets its `count` data to 0.
What does it mean to increment our counter?

We want an `increment` instruction that allows us to mutate the `count` data in
our existing counter account, increasing its value by one.

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

```javascript
it("Increments the counter", async () => {
  const counter = await initializeCounter();

  await program.rpc.increment({ accounts: { counter } });

  const counterData = await program.account.counter.fetch(counter);

  assert.ok(counterData.count.eq(new anchor.BN(1)));
});
```

We reuse the same function that will initialize a fresh counter for us.

We then call our new instruction: `increment`. We only include the counter
account, as we described above.

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
Let's implement this on the program side now.

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
changes after we mutate the data inside the counter. **If you don't have that
tag, the data won't save after being mutated in the instruction handler!**

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
data. Instead of setting the value to 0, we increment the count value, and
return `Ok(())`. Anchor, under the hood, will save our changes to the counter
account data because we grabbed a mutable reference to the counter account.

Let's re-run our tests with `anchor test`.

```bash
  simple-counter
    ✓ Initializes the counter to 0 (415ms)
    ✓ Increments the counter (411ms)


  2 passing (830ms)
```

Awesome!
