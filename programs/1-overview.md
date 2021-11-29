# Architecture

A
[Solana program](https://docs.solana.com/developing/programming-model/overview)
exists on the Solana blockchain. You can send
[transactions](https://docs.solana.com/developing/programming-model/transactions)
containing
[instructions](https://docs.solana.com/developing/programming-model/transactions#instructions),
which will cause some computation to be performed on the Solana blockchain.

# How are programs stored?

Programs are stored as
[accounts marked executable](https://docs.solana.com/developing/programming-model/accounts#executable).
The program id of a program is equivalent to the public key of the account that
stores its data. The bytecode of the program itself is stored as the data under
the account.

We'll get more into accounts later, but for now knowing that programs are stored
in accounts is enough.
