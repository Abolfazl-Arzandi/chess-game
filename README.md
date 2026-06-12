# ♞ Chess — Player vs Bot

A fully playable chess game with a built-in AI, written as a single self-contained HTML file. No frameworks, no libraries, no build step.

---

## How It Was Built

### Architecture

The entire project lives in one `index.html` file with inline CSS and JavaScript. Unlike the simpler canvas-based games, the board is rendered as a **CSS Grid of DOM elements** (8×8 `<div>` squares), which makes click handling and styling straightforward without needing a draw loop.

### Board Representation

The board is a flat `array[64]` of single characters:

```
index 0 = a8 (top-left) ... index 63 = h1 (bottom-right)
```

Pieces are encoded as characters — uppercase for white (`P N B R Q K`), lowercase for black (`p n b r q k`), and `'.'` for empty squares. The full game state is a plain object holding the board, whose turn it is, castling rights, en passant target square, and move clocks.

### Move Generation

Move generation follows the standard two-pass approach:

1. **`pseudoMoves(state)`** — generates all geometrically possible moves for the side to move (including potentially illegal ones that leave the king in check).
2. **`legalMoves(state)`** — filters pseudo moves by applying each one and checking whether the moving side's king ends up in check. Castling gets an extra check ensuring the king doesn't pass through an attacked square.

Each move is a plain object: `{from, to, piece, capture, promo, flag}` where `flag` encodes special cases (`'ep'`, `'castleK'`, `'castleQ'`, `'double'`).

### Special Rules

All standard chess rules are implemented:

- **En passant** — tracked via an `ep` square index in the state, set when a pawn advances two squares.
- **Castling** — both kingside and queenside for each color, with rights updated whenever king or rook moves or is captured.
- **Pawn promotion** — a UI overlay appears with piece choices (Q, R, B, N); the move is held in a `pendingPromo` buffer until the player selects.
- **50-move rule** — tracked via a `halfmove` clock, drawn when it reaches 100 half-moves.

### AI Engine

The bot uses **Negamax with alpha-beta pruning** — a standard minimax variant where both sides maximize their own score, avoiding the need for a separate `min` case.

```
negamax(state, depth, alpha, beta)
  └── if depth == 0: return evaluate(state)
  └── for each legal move:
        val = -negamax(apply(move), depth-1, -beta, -alpha)
        alpha-beta cutoff
```

**Difficulty levels:**

| Level | Behavior |
|---|---|
| 1 — Random | Picks a random legal move |
| 2 — Beginner | Greedy: best immediate material gain (depth 1) |
| 3 — Easy | Minimax depth 2 |
| 4 — Medium | Minimax depth 3 |
| 5 — Hard | Minimax depth 4 |

### Evaluation Function

The static evaluator sums up **material value** plus **positional bonuses** for every piece on the board. Positional bonuses come from hardcoded **Piece-Square Tables (PST)** — 8×8 grids of bonus/penalty values per piece type that encourage good placement (e.g. knights prefer the center, pawns are rewarded for advancing, the king stays safe near a corner).

```
score += pieceValue[type] + PST[type][square]   // for white
score -= pieceValue[type] + PST[type][mirror]   // for black
```

### Move Ordering

Before the alpha-beta search, moves are sorted using **MVV-LVA** (Most Valuable Victim / Least Valuable Attacker): captures of high-value pieces by low-value pieces are explored first. This dramatically improves pruning efficiency since good moves are found early.

### SAN Notation

The move history panel displays moves in **Standard Algebraic Notation** (`toSAN`). The function handles disambiguation (when two identical pieces can reach the same square), capture notation (`x`), promotion (`=Q`), check (`+`), and checkmate (`#`).

### Undo

Each move pushes a deep clone of the state onto a `history` stack. Undo pops up to two states (bot move + player move) to restore the player's last turn. Captured piece lists are recomputed by diffing piece counts between the starting position and the current board.

### Rendering

The board is re-rendered from scratch on every state change. Move hints are injected as child elements: a **dot** for empty target squares and a **ring** for captures. The board can flip orientation when the player chooses to play Black.

---

## How to Run

No installation or server required.

1. Download the file:
   ```
   index.html
   ```

2. Open `index.html` in any modern browser (Chrome, Firefox, Edge, Safari).

3. Click a piece to select it, then click a highlighted square to move.

That's it.

---

## Controls

| Action | How |
|---|---|
| Select / move piece | Click |
| Choose promotion piece | Click the piece in the popup |
| Undo last move | Undo button |
| New game | New Game button |
| Switch sides | "You play" dropdown (resets the game) |
| Change bot difficulty | "Bot level" dropdown |
