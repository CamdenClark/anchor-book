Let's dive in to the program and use the integration tests as a tool to see what
changing different pieces of the program would do.

# The account struct

```rust
#[account]
pub struct MyAccount {
    pub data: u64,
}
```

This struct defines what shape the account data will have. This specific account
will be a struct with a field `data` of type `u64`.

Try removing the `#[account]` attribute and re-running `anchor test`.

```
... snip ...

error[E0277]: the trait bound `MyAccount: Clone` is not satisfied
  --> programs/basic-1/src/lib.rs:34:21
   |
34 |     pub my_account: Account<'info, MyAccount>,
   |                     ^^^^^^^^^^^^^^^^^^^^^^^^^ the trait `Clone` is not implemented for `MyAccount`
   |
  ::: /home/camden/anchor/lang/src/account.rs:12:78
   |
12 | pub struct Account<'info, T: AccountSerialize + AccountDeserialize + Owner + Clone> {
   |                                                                              ----- required by this bound in `anchor_lang::Account`
```

When we define an Account, the associated account data type must have these
traits. The important thing here is that any account struct must have the
`#[account]` attribute so Anchor knows how to work with that struct.

Add back the `#[account]` attribute before continuing.

### Allocating space

Let's add in a new field on the struct now:

```
#[account]
pub struct MyAccount {
    pub data: u64,
    pub another_data: u64,
}
```

Then run `anchor test`

```
... snip ...

  logs: [
    'Program Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS invoke [1]',
    'Program 11111111111111111111111111111111 invoke [2]',
    'Program 11111111111111111111111111111111 success',
    'Program log: Custom program error: 0xa3',
    'Program Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS consumed 9553 of 200000 compute units',
    'Program Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS failed: custom program error: 0xa3'
  ]
}
    1) Creates and initializes an account in a single atomic transaction (simplified)
    2) Updates a previously created account


  0 passing (67ms)
  2 failing

... snip ...
```

Our logs from invoking Solana aren't super helpul here. We definitely reached
the system program (11....11) but we failed to initialize.

There aren't a ton of good clues here, but what did we actually change? We added
another field to the struct, which ends up increasing the amount of space we
need.

The fix here is to look above to where it says `space = 8 + 8`. We can change
that to `space = 8 + 8 + 8`, and run `anchor test` again, which will pass.

What did we learn? We have to be careful about the space we allocate for our
account data and make sure that we allocate the right amount of space. We have
to allocate 8 bytes for the discrimnator, 8 bytes for data, and 8 bytes for
another_data.

Let's revert the changes we've made up to this point.

# The Initialize accounts

The initialize accounts are defined here:

```
#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(init, payer = user, space = 8 + 8)]
    pub my_account: Account<'info, MyAccount>,
    #[account(mut)]
    pub user: Signer<'info>,
    pub system_program: Program<'info, System>,
}
```

Solana requires that we specify _all_ the accounts that are used in the
instruction handler because validators need to be able to operate on many
accounts in parallel. Getting a mutable reference to an account will rqeuire
blocking on any further changes to that account.

## The system program account

If you look at the Initialize handler above, you'll notice that the
system_program account is never explicitly used. However, try removing the
system_program and the program won't compile. That's because we've defined, in
this struct, `my_account` as an account that is to be initialized. As a result,
anchor requires us to include the system_program because the system_program is
the program capable of initializing accounts.

If we look at the Update accounts, we'll see that there is no system_program
defined. That is because we are just updating that account's data and not
initalizing it, so anchor doesn't require us to have access to the
system_program.

## The user account

We also don't explicitly use the user account in the instruction handler for
Initialize. Let's try removing it (along with the `#[account(mut)])` attribute).

This won't compile either. That's because the account that's paying for the rent
is the user, so we'll get a compilation error.

#### Why does the user account have the mutable attribute?

Let's re-add the user account line, but leave off the `#[account(mut)]` piece.
This compiles, so let's run `anchor test`.

This actually works. I don't know if anchor test isn't testing all the
constraints we expect, but I don't think this would work in a client setting. My
expectation would be that this doesn't work because the user account needs to be
mutable for the program to debit the account to pay for the space allocation.

**Open question--don't quite get this...**

#### The Signer struct

The Signer struct just tells anchor that the user account has signed the
transaction. Anchor knows that it doesn't have to fetch or deserialize the data
associated with this account.

## my_account

Finally, we have the account that actually holds the data for our program.

We've already covered the pieces of the attribute that this is tagged with, so
let's look at the account itself.

```rust
pub my_account: Account<'info, MyAccount>
```

This tells anchor that we have an account here with data. Anchor should expect
to deserialize the data into a MyAccount object, which we already went over.

# The Update accounts

Very simply, the Update instruction only requires one account, which is the
original account that we stored the data in. This has the mutable attribute,
because we want to be able to update the account data.

# The instruction handlers

Finally, we can move on to the instruction handlers!

These are virtually identical in how they are implemented in the code. They both
take two parameters, take a mutable reference to my_account, and then update the
data.

What happens under the hood is very different though, and the differences are
entirely because of the different account specifications.

When we call initailize, anchor, under the hood, is making a call to the system
program to initialize the account at my_account's address. Only after
initializing the account does it update the data.

The update instruction takes an already initailized account with this type of
data and just updates it.

Thanks for reading this deep dive. I'll be continuing to do more deep dives on
these anchor examples.
