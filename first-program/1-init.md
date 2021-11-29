We'll dip our toes in today and initialize our first Anchor program. We'll learn
a little bit about the structure of Anchor projects, how to test them, and how
we bridge between the frontend and the program code.

The code samples for this can be found
[here](https://github.com/CamdenClark/anchor-book-code/tree/main/first-program)

# Creating a new program

First, we'll initialize a new anchor project.

```
anchor init first-program --typescript
```

This command will generate a new anchor project under the directory
`first-program`, with the tests written in typescript.

# The main pieces

There are two core pieces of the initialized anchor program:

1. The Solana program itself. This is found at
   `./programs/first-program/src/lib.rs`.
2. The Anchor framework integration tests, which are found at
   `./tests/first-program.ts`. Anchor introduces some utilities for interating
   fast using integration tests in your program.

# Diving into the program

Open `./program/first-program/src/lib.rs` and you'll see the following:

```rust
use anchor_lang::prelude::*;

declare_id!("Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS");

#[program]
mod first_program {
    use super::*;
    pub fn initialize(_ctx: Context<Initialize>) -> ProgramResult {
        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize {}
```

Let's step through each piece of this.

```rust
use anchor_lang::prelude::*;
```

Similar to Rust itself, Anchor has a prelude. This imports all the pieces of the
anchor framework that we'll need to define our program.

```rust
declare_id!("Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS");
```

`declare_id!` bakes the program's account id into our program's binary. It
allows for a consistent program id everywhere we deploy (locally, devnet,
testnet, mainnet-beta).

One other benefit, which we'll get into later, is ensuring that you are calling
the right program if calling this from another program (see cross-program
invocation).

Note that if we actually deploy this, anchor would instead pick a different
account id, because we will have to generate a new keypair.

(There's discussion about a more ergonomic way of doing this
[here](https://github.com/project-serum/anchor/issues/695)).

```rust
#[program]
mod first_program {
    use super::*;
    pub fn initialize(_ctx: Context<Initialize>) -> ProgramResult {
        Ok(())
    }
}
```

First, we define a
[Rust module](https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html)
called first_program. Since this module has the
[program attribute](https://github.com/project-serum/anchor/blob/master/lang/attribute/program/src/lib.rs),
anchor will know that this module contains all the program instruction handlers.

In your editor, try deleting `use super::*;` and run `anchor build`. This will
fail to compile! This is because `use super::*;` brings all the things we
imported in from the anchor prelude into the current module's context.

Finally, we get to the initialize handler, which is the only instruction that
we've defined as part of this Anchor program. Our initialize handler takes in
one parameter, `_ctx` which has type `Context<Initialize>`. This is a really
stripped down example, so we won't perform any business logic inside this
instruction handler. In a non-trivial instruction handler, this parameter would
hold our account data, which we would be able to manipulate. Here, we just
return `Ok(())`, which returns a successful result back to the client.

```rust
#[derive(Accounts)]
pub struct Initialize {}
```

Finally, we have the last piece of the program, which describes the account
context. In a non-trivial program, this would define _all_ (and I mean _all_)
the accounts that are manipulated by the instruction handler. Since we aren't
actually using any accounts in this example, we don't need to add anything.
However, we still need to define a struct that holds all the accounts and signal
to the instruction handler what we should expect from the client.
