Now, we're going to take everything we've learned up to this point and write a
tic-tac-toe program with a functional frontend.

How are we going to go about this?

Often trying to figure out every small technical decision up front when writing
our programs is really difficult and can run us into problems.

Here though, we know the problem space pretty well just by knowing how the game
works. We can discuss how we plan to model the data of the game of tic-tac-toe
using accounts and instructions.

# How does Tic-Tac-Toe (TTT) work?

Two players, the X player (who goes first), and the O player, play on a 3 x 3
grid. Each player takes turns selecting one of the grid items, and putting an X
or an O on that grid item.

There are three terminal conditions:

1. X player makes a row or column full of Xs (X wins)
1. O player makes a row or column full of Os (O wins)
1. All grid items are full and neither player wins

# What does our program have to do?

There's a major simplification we'll make with our TTT program: the first player
will generate an account used to store the TTT game state and send that account
to the second player, who will join after.

The player who initializes will always be X, and the player who joins second
will be O.

Each turn, the program will have to do a couple of things:

1. Block a user putting their symbol on a grid space that already has a symbol
1. Register their move in the internal game state
1. Check to see if the game state has reached a terminal condition
1. Keep track of whose move it is, and only allow that player to make the next
   move.

There's one last decision we have to make: do we pair initialization with the
first move, (and, for the O player, pair joining the game with their first
move)? This would save one transaction for both players.

This decision isn't super relevant since we won't be deploying to mainnet, and
the content of the first move isn't really that important. To simplify the user
experience, and the design of our program, we'll separate initializing/joining
and each player's first moves.

When you go build your own programs, it's important to keep these kinds of
tradeoffs in mind!
