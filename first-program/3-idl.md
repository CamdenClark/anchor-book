# The IDL

We were able to run the integration test suite for our program. But how did the
test code actually know about the structure of the program to be able to call it
in the right way?

The answer is in the Interface Description Language (IDL). When we run
`anchor build`, anchor outputs JSON that describes the structure of the program.

Then, in the test code, Anchor parses the IDL to provide a nice interface for us
to be able to interact with the program.

Let's run `anchor build`, then open `target/idl/first_program.json`.

You'll see the following:

```json
{
  "version": "0.0.0",
  "name": "first-program",
  "instructions": [
    {
      "name": "initialize",
      "accounts": [],
      "args": []
    }
  ]
}
```

The most important thing listed here is the set of instructions. Anchor has used
the program definition we created above to generate this IDL. We have one
instruction, with name `initialize` (the same as our instruction handler). It
has an empty set of accounts because we didn't specify any in the `Initialize`
account context struct. The IDL also has an empty set of `args` because the
`initialize` instruction handler only takes in the context, and hasn't specified
any other parameters.

This file is the key bridge between your Anchor program and consuming it in the
client and integration tests. It is how Anchor's frontend utilities know how to
call your program.
