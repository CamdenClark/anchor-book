# Solana's account model

Solana's account model is the most important part of understanding how to write
programs, so strap in!

Accounts are how you store persistent data on the Solana blockchain. When creating
an account, you have to declare a size in bytes, and you can store arbitrary binary
data inside those bytes.

From the [Solana docs](https://docs.solana.com/developing/programming-model/accounts#executable):

> Accounts are similar to files in operating systems such as Linux in that they may hold arbitrary data
> that persists beyond the lifetime of a program. Also like a file, an account includes metadata that
> tells the runtime who is allowed to access the data and how.

Another way of thinking about accounts is that the Solana blockchain is a giant hash-map,
with accounts as the keys, and the account data as the value.

Solana assigns public keys (from the ed25519 curve) as the address of the account. The address would
be similar to, in the analogy of the filesystem, would be the file path for an account on the Solana
blockchain.

### Ownership and Authority

**Accounts can only be owned by programs!** Accounts are owned by the system program by default.
The system program is a built-in program to the Solana runtime. The system program allows clients
to transfer lamports and change the owner of an account to another program id.

Accounts can be assigned a new owner only if the account is writable, not executable, and the data is
zero-initialized or empty.

If a program doesn't own an account, the program can only read data and send lamports into the account.
Programs can't take lamports out of an account if they don't own it.

Authority is a distinct concept from ownership. Ownership is assigned by the system program as part of
on-chain data. Authority just means that the account's private key signed the transaction. The account
is referred to as a "signer" in this case.

### Programs

We mentioned earlier that programs are just accounts marked executable with their bytecode as data.
When you deploy a program, you call a special program called the BPF Loader, which marks itself as
the _owner_ of the executable and program data accounts it creates to store your program.

More info [on deployment](https://docs.solana.com/cli/deploy-a-program),
[on the BPF Loader](https://docs.solana.com/developing/runtime-facilities/programs#bpf-loader),
and on [executable accounts](https://docs.solana.com/developing/programming-model/accounts#executable).

### Read-only

### Storing Value

Accounts can also store lamports (the smallest unit of SOL). This allows accounts to store value as well
as arbitrary data.

Accounts also [need rent](https://docs.solana.com/developing/programming-model/accounts#rent) in order
to keep accounts alive. Luckily, if you pay up enough rent for 2 years, you are now _rent-exempt_.
Executable accounts (programs) are required to be rent-exempt.

We'll talk more about rent later!

# Why this?

One reason that Solana chose this account model is that it makes parallel processing of transactions
really fast and easy to reason about. Accounts can be marked as
[read-only](https://docs.solana.com/developing/programming-model/accounts#read-only) when being
processed inside transactions. Read-only accounts can be read concurrently by multiple programs.

You can have one program that operates on a lot of different types of accounts.

We'll talk more about the SPL Token program later, but this
[example from an article on Solana is great](https://2501babe.github.io/posts/solana101.html)

> one advantage of the program/account model is you can have one generic program that operates on various data. the best example of this is the spl token program. to create a new token, you dont need to deploy code like you do on ethereum. you create an account that can mint tokens, and more accounts that can receive them. the mint address uniquely determines the token type, and these are all passed as arguments to one static program instance
