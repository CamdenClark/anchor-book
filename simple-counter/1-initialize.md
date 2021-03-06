# Initializing a counter

## Extending our first program

The first program that we generated with `anchor init` doesn't really _do_ anything.

When you call the `initialize` instruction, nothing about the state of the blockchain really changes, except the fact that we called the instruction.

What if we wanted to build a simple counter on the Solana blockchain? How would we be able to initialize the counter, then increment the counter later, and have all that data be stored on the blockchain?

In this section, we'll build a program that can do that.

We mentioned earlier that programs are accounts marked executable. Specifically, the data that the account holds is the program's compiled bytecode.

We can also use accounts to store any binary data we want. So, here, we'll use an account to store the data for the counter.

## Getting started

Let's start fresh with a new project.

```bash
anchor init simple-counter
```

{% hint style="info" %}
You can find the code that covers this section [here](https://github.com/CamdenClark/anchor-book-code/tree/main/simple-counter-1)
{% endhint %}

Like [our first anchor program](../programs/2-program.md), our newly initialized simple-counter project has two high-level components, the program itself (at `programs/simple-counter/src/lib.rs`) and the integration tests (found at `tests/simple-counter.ts`).

## The program

First, we'll build up the program. Open `programs/simple-counter/src/lib.rs`. Remove everything, and paste the following:

```rust
use anchor_lang::prelude::*;

declare_id!("Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS");

#[program]
pub mod simple_counter {
    use super::*;
    pub fn initialize(ctx: Context<Initialize>) -> ProgramResult {
        let counter = &mut ctx.accounts.counter;
        counter.count = 0;
        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(init, payer = user, space = 8 + 8)]
    pub counter: Account<'info, Counter>,
    #[account(mut)]
    pub user: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[account]
pub struct Counter {
    pub count: u64,
}
```

We'll dissect each piece of this individually. Let's start from the bottom, with the `Counter` account struct.

### The account struct

```rust
#[account]
pub struct Counter {
    pub count: u64,
}
```

Accounts are how you store persistent data on the Solana blockchain. They can store arbitrary binary data.

From the [Solana docs](https://docs.solana.com/developing/programming-model/accounts#executable):

> Accounts are similar to files in operating systems such as Linux in that they may hold arbitrary data that persists beyond the lifetime of a program. Also like a file, an account includes metadata that tells the runtime who is allowed to access the data and how.

Public keys (from the ed25519 curve) are the address of the account. The address would be, in the analogy of the filesystem, the file path.

Another way of thinking about accounts is that the Solana blockchain is a giant hash-map, with account public key as the keys, and the account data as the value.

Anchor will do the serialization and deserialization of our data to and from binary in our program for us. But we have to tell Anchor what shape of data to expect so it can actually do that.

This is the purpose of the `#[account]` Rust decorator. This is a special Anchor decorator which says this struct can be used to model an account's data. Under the hood, Anchor will serialize and deserialize the data for any account marked with this struct into this data structure.

Since we're only implementing a simple counter, all we need is one property, which is the `count`. This will store the current value of the counter associated with that account.

We'll move on to the account context, which is where we define which accounts that our instruction should expect.

### The Accounts context

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

Above is the `Initialize` accounts context. It describes all the accounts that need to be passed in to complete the instruction. Account contexts are typically coupled with one instruction handler. The account context is the most dense and "magic" part of Anchor: you have a lot of power to do things behind the scenes.

We pass in three different accounts when calling this instruction: the counter, the user, and the system program. We'll go through each and describe what's going on behind the scenes for each.

#### The counter account

The counter is typed with `Account<'info, Counter>`, where `Counter` is the struct that we described above. This tells Anchor to serialize and deserialize the account data in the shape of the `Counter` struct. So, later, when we are working on the actual instruction, we'll be able to have access to the data as a `Counter`.

#### The system\_program account

If you look at the `Initialize` instruction handler above, you'll notice that the system\_program account is never explicitly used.

Try removing the `system_program`.

You'll see the program won't compile. That's because we've defined, in this struct, `counter` as an account that is to be initialized. As a result, Anchor requires us to include the `system_program` account because Anchor needs to use the system program to initialize the account.

#### The user account

We also don't explicitly use the user account in the instruction handler for `Initialize`. Let's try removing it (along with the `#[account(mut)])` attribute).

This won't compile either. That's because the account that's paying for the rent is the user, so we'll get a compilation error.

The type of the user account isn't `Program` or `Account`, it's `Signer`. The Signer struct just tells anchor that the user account has signed the transaction, but shouldn't get or deserialize the data on the account (because we don't need to access that data).

### The instruction handler

Finally, we have the instruction handler:

```rust
pub fn initialize(ctx: Context<Initialize>) -> ProgramResult {
	let counter = &mut ctx.accounts.counter;
	counter.count = 0;
	Ok(())
}
```

The name of this instruction is `initialize`. On the client, we'll be able to call this business logic by calling the initialize instruction.

#### The `ctx` parameter

This method only has one parameter, `ctx`, with type `Context<Initialize>`. You can go to `Context` in your editor. What's happening here is that anchor injects all things known about this instruction's context into this parameter. We have to specify the `Initialize` account context that we created earlier because that tells Anchor the _full list_ of accounts that need to be accessed in our instruction.

#### When you call this instruction

When you actually call this instruction, Anchor will do a bunch of heavy lifting:

1. Anchor will make sure that the `user` account is one of the signers of the transaction this instruction is called in (because of the `Signer` type)
2. Anchor will call out to the system program and tell it to initialize the account for the counter. It will call out to the system program and tell it to initialize the account for the counter with `8 + 8` bytes of space in its data, with rent paid by the `user` account.

{% hint style="info" %}
Note that all of this happens because of this decorator:

```rust
#[account(init, payer = user, space = 8 + 8)]
```
{% endhint %}

1. It will deserialize account data for the `counter` into the `Counter` struct that we created earlier

#### The business logic

Next, we have the actual business logic of the instruction handler.

What we do here is just get a mutable reference to the counter account's data. Notice that `ctx.accounts.counter` has the `Counter` type as we specified below. That means that once we grab a mutable reference to `ctx.accounts.counter`, we can set its `count` property to 0.

`Ok(())` tells the runtime that the instruction handler was successfully invoked.

{% hint style="info" %}
In this program we never have to "commit" our changes to account data in any way. All we do is mutate the data using our mutable reference to the account's data. Anchor will serialize the data back to bytes and update the account's data for us.
{% endhint %}

Next, we'll move on to writing our integration tests which verify that we have initialized our counter correctly.
