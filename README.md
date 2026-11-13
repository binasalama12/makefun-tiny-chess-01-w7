# Nimble Falcon Chess — a deterministic 4 KB engine

An original tiny chess engine written for the make.fun
**tiny-chess-01** competition. The whole engine is `main.py`:
a single Python file, standard library only, that talks to the
competition environment at `MAKEFUN_ENV_URL` and plays every game
of the gauntlet it is served.

## Design

- **Board**: 0x88 mailbox board in a flat 128-slot list; pieces are
  small integers (type in the low bits, colour in the high bit).
- **Move generation**: pseudo-legal generation with offset tables
  per piece type, sliding rays for bishop/rook/queen, pawn pushes,
  captures, en passant, promotions and both castles; legality is
  confirmed by making the move and testing the own king.
- **Search**: iterative-deepening negamax with alpha-beta windows,
  a Zobrist transposition table, null-move pruning, late-move
  reductions and killer-move ordering. The quiescence search only
  extends captures. Everything is budgeted in **nodes, never
  time**, so the same position always produces the same move.
- **Evaluation**: material plus formula-generated piece-square
  tables, a bishop-pair bonus, and an endgame king table blended
  in as non-pawn material disappears.

## This variant

Nimble Falcon trades depth for certainty. It runs the lightest
node schedule of the family (30k / 17k / 9k) but reduces late
moves by **two** plies once five moves have been tried at a node
— its siblings reduce by one — so the nodes it does spend go to
the lines that matter. Pawns grab a bigger central bonus (six
points on the centre squares), the middlegame king table is
sharper, the endgame king table is the steepest of all
(9c − 36), and the endgame blend stretches over 7,200 points of
non-pawn material, the slowest phase-in in the family. Piece
values are the family's lightest after the gecko's. Zobrist keys
come from its own seeded generator. Locally it is the fastest
build in the family per move and still passes every tactical
test the family uses.

## Files

- `main.py` — the engine (the readable source is packed into the
  file as an lzma + base85 payload under a two-line unpacker, a
  format the rules explicitly allow; both parts count toward the
  4,096-byte cap).
- `LICENSE` — MIT.
- `.makefun/owner.json` — the entrant wallet for this entry.

## Running

```sh
MAKEFUN_ENV_URL=http://127.0.0.1:PORT python3 main.py
```

The bot loops on `POST /game` and answers `POST /move` until the
environment reports the trial done.
