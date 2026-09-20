# Chess LUX

**English** · [Türkçe](#chess-lux-türkçe)

▶ **[Play in your browser](https://cuneytinann.github.io/Chess-LUX/)** · [Benchmarking page](https://cuneytinann.github.io/Chess-LUX/BestArbiter.html)

**The world's most rule-accurate chess arbiter, in a single 7,688-byte HTML file.**

Chess LUX is a browser-based chess app for two players sharing a single screen. Its real job is arbitration: in line with the FIDE Laws of Chess, it determines whether each move is legal, whether the game is over and what the result is. In particular, it performs a check that most chess sites skip when a flag falls or a position locks up.

## At a glance

| | |
|---|---|
| **Size** | one file, 7,688 bytes: game, clock, interface and arbiter in one |
| **Rules** | checkmate, stalemate, dead positions, flag fall, resignation, the 50- and 75-move rules, threefold and fivefold repetition, draw claims and offers |
| **On Lichess data** | the correct verdict in 201,048 of 201,060 games wrongly decided on time (99.9940%) |
| **In locked positions** | the correct verdict in 50,514 of the 50,526 locked-structure games among them (99.98%) |
| **False draws** | 0 on the test positions |
| **Speed** | flag verdict in a median of 0.1 ms on real games; about half a second on the hardest constructed position |
| **Installation** | none: play online, or download the file and play offline |

## Try it now

1. Open [cuneytinann.github.io/Chess-LUX](https://cuneytinann.github.io/Chess-LUX/), or download `index.html` and open it locally; the tab will read "FideLite".
2. In the dialog that appears, keep the starting position and time control (10 min + 5 s) or enter your own FEN, then press **Play**.
3. Move the pieces by clicking or dragging. Right-drag across the board to mark a line of squares, right-click one square to mark it alone, and any left click clears the marks. Use ½ to claim or offer a draw and ⚐ to resign; once the game is over, ↺ starts a new one.

Any up-to-date browser will do: Chrome/Edge 85+, Firefox 98+, Safari 15.4+ (2022 or later).

Chess LUX is a prototype: the focus is on the rules layer, with the interface taking a back seat. Because the rules layer is independent of the interface, anyone who wishes can take it and adapt it to their own system. An interface-free version is embedded in `BestArbiter.html` as `engine_4x.js`.

## Why it matters

Under the FIDE Laws of Chess (Article 6.9), a player who runs out of time does not always lose: if the opponent could not checkmate them by any possible series of legal moves, even with both sides cooperating, the game is drawn. For a long time, chess servers largely failed to apply this rule.

A real example: the Lichess game [ijyj0mHa](https://lichess.org/ijyj0mHa#120) (one of chasolver's examples).

```
5b2/p7/Pp3k2/1Pp1pBp1/2P1P1P1/5K2/8/8 w - -
```

White ran out of time and the game was recorded as a loss for White. Yet the pawn wall is permanently locked, and Black cannot deliver mate even with White's cooperation. Chess LUX rules this position a draw (`TM`). Interestingly, had Black run out of time in the same position, White would have been awarded the win, because a mate for White does exist.

[chasolver](https://chasolver.org) scanned billions of Lichess games and found **201,060** games decided wrongly in exactly this way. Chess LUX reaches the correct verdict, a draw, in **201,048** of them. In the remaining 12 it cannot find a proof and errs on the side of caution by awarding the win. These are not wrong verdicts but undetected draws: the game ends in a win, exactly as it does today. Amounting to roughly 1 in 16,750 of the wrongly decided games, these 12 positions are listed in the technical appendix as targets still to conquer. Since the Lichess database of standard rated games contains more than 7 billion games, the verdicts Chess LUX misses come to fewer than 2 per billion games, or less than 1 in every 500 million.

Dead positions tell a similar story. Under FIDE rules (Article 5.2.2), the game is drawn the moment a position arises in which neither player can checkmate. According to chasolver's statistics, the position was already dead in 57.5% of the wrongly decided timeouts: those games were effectively over long before, yet dragged on until a flag fell. Chess LUX declares a dead position, to the extent it can recognize one, on the very move it arises.

## How the arbiter decides

**After every move**, it checks whether the game is over. Checkmate, stalemate, dead positions (insufficient material or a locked structure), fivefold repetition and the 75-move rule are all applied automatically.

**When a flag falls or a player resigns**, it asks a single question: could the opponent checkmate this player, even if both sides cooperated? It looks for the answer in three steps:

1. **Material:** Insufficient mating material is recognized directly.
2. **Locked structures:** Pawn walls, immobile pieces and squares out of the bishops' reach are analyzed.
3. **Search:** Every move for both sides is tried up to a depth of 99 plies, and the search stops after at most 50,000 nodes. Every line must be closed by a proof: material, a locked structure, stalemate, a repeated position or the 75-move limit.

If no proof is found, the side with time remaining wins. The overriding priority is never to award a false draw; missing a draw is the lesser evil.

**The clock is settled before every action.** A move, a draw claim and a resignation all begin by charging the elapsed time to the player on move; if that empties their clock, the flag verdict is issued and the action is refused. The 99 ms display tick therefore only delays what the players see, never what the arbiter counts: elapsed time is always measured from the last reading, so nothing is lost between two ticks and nothing is charged to the wrong side. Measurement uses `performance.now()`, a monotonic source, so a system-clock correction cannot give or take time either.

**The time control.** Both fields take any non-negative number, fractions included: 2.5 minutes is a 150-second game and 2.5 seconds is a 2.5-second increment, with no silent rounding on either side. Anything that is not a finite number, or a starting time of 24 hours or more, or an increment of an hour or more, is refused before the game starts. The clock reads `MM:SS` and grows an hour field only when it needs one, so `15:00` stays `15:00` and a 90-minute control reads `1:30:00`.

**The 75-move rule also feeds into the flag verdict.** If mate is only possible after the 75-move limit, a flag fall ends in a draw:

```
4k1n1/8/8/8/8/8/8/4K1N1 w - - 146 1   if White's flag falls: draw, as the 75-move limit is reached four plies later
4k1n1/8/8/8/8/8/8/4K1N1 w - - 0 1     same position, clock at 0: Black wins
```

**The layers complement each other.** Consider this position:

```
1kb5/1p1p4/1P1P4/8/8/4p1p1/4P1P1/5BK1 w - - 0 1
```

Both bishops are hemmed in behind their own pawns; the white king can only shuttle between g1 and h1, and the black king between b8 and a8. Neither side can ever deliver mate, so under FIDE rules the position is dead from the very first move. Because the lock analysis does not yet treat trapped bishops as blockers, it does not declare the dead position on move one. The arbiter still reaches the right result: if a flag falls, the search sees the position recur within four plies and rules a draw (`TM`); if the kings keep shuffling, the game is drawn by fivefold repetition (`5R`) after 16 plies. Recognizing this position as `DP` on the first move is on the list of targets still to conquer.

### Draw claims and offers: semi-automatic

Threefold-repetition and fifty-move claims are neither fully manual nor fully automatic; the ½ button mirrors the over-the-board procedure set out in the FIDE Laws of Chess:

- **Pressing ½ directly:** The claim is assessed for the current position (9.2.2, 9.3.2).
- **Making a move with ½ checked:** The claim is assessed for the position the move will produce (9.2.1, 9.3.1). This is the equivalent of writing the move down and declaring your intention to the arbiter over the board.
- **The board is read first:** A claim is only assessed once the position it produces has been examined. If that move gives checkmate or stalemate, or the position is dead, the game ends there and the claim never arises — mate outranks the fifty-move rule, threefold repetition and a draw agreement alike.
- **If the claim is incorrect:** It goes to the opponent as a draw offer (9.1.2.3). An offer sent together with a move corresponds to an over-the-board offer made after moving and before pressing the clock (9.1.2.1).
- **The opponent's response:** If the opponent presses ½, the game ends in a draw by agreement; moving without pressing it declines the offer.

Under a fully manual scheme, a player would have to catch the exact moment to claim, and a fast opponent or a premove could cost them that right. Under a fully automatic scheme, the game would end in a draw at a moment the player never chose. With the semi-automatic approach, the intention is registered before the move, the verdict is delivered with the move, and the decision to claim always rests with the player.

The ½ button is available from the very first move, so in a position set up from a FEN the fifty-move rule can be claimed right away. A draw by agreement is the one thing it cannot produce that early: under FIDE 5.2.3 the players may only agree once both of them have made a move, so until then the offer is recorded but never reaches the opponent. Disabling the button on the first ply would have been the simpler fix, but it would have taken the fifty-move claim down with it.

### Result codes

| code | meaning |
|---|---|
| `W#` / `B#` | White / Black delivered checkmate |
| `SM` | stalemate |
| `DP` | dead position |
| `5R` / `75` | fivefold repetition / 75-move rule |
| `3R` / `50` | threefold-repetition / fifty-move claim |
| `DA` | draw by agreement |
| `WT` / `BT` | White / Black won on time |
| `TM` | flag fell, but the opponent cannot mate: draw |
| `WR` / `BR` | opponent resigned: White / Black won |
| `RM` | resigned, but the opponent cannot mate: draw (FIDE 5.1.2, 2023) |

## chasolver and Chess LUX

[chasolver](https://github.com/miguel-ambrona/chasolver) (Miguel Ambrona) is a solver for adjudicating flag falls correctly, grounded in a peer-reviewed paper ([FUN 2022](https://chasolver.org/FUN22-full.pdf)). Its aim is to prevent servers from issuing wrong results on time. It addresses the question purely from the position; rules that depend on the history of the game, such as the 75-move rule and fivefold repetition, lie outside its scope.

Chess LUX additionally sets out to automate every way a game can end: dead positions should end without dragging on, draw claims should work as FIDE intends, and players should not lose time. Being meticulous about FIDE compliance, I also built the 75-move rule and fivefold repetition into the arbitration; the 75-move rule even feeds into the flag verdict. I did not leave Chess960 out either: neither the lock analysis nor the search relies on the castling rules. These functions can readily be ported to a Chess960-capable engine, or this engine can be adapted to Chess960; today's `index.html`, however, is written for standard chess.

I developed the algorithm independently, but without Ambrona's work I could never have taken it this far. I made direct use of the data he distilled by downloading and analyzing billions of games, and of the challenging positions he gathered from the community; his work is the yardstick for this project's correctness.

### Comparison with FideLite

Chess LUX builds on the most advanced version of the FideLite engine at [fidelite.art](https://fidelite.art), where the rules and functions are documented in detail. FideLite's lock detector only recognizes locked structures consisting of kings and pawns. With roughly 1 KB of additional code, Chess LUX extends it to every piece; since knights, rooks and queens can only be accounted for when they are immobile, the extension effectively amounts to adding bishops. In short, this repository is FideLite with a looser byte budget and higher rule accuracy. The naming follows the L3 build: every name L3 already had keeps it — `N` is `indexOf`, `Q` is `innerHTML`, the pawn sets inside the lock detector are still an adjacent pair (`m`/`n`, since `l` now names the detector itself) — and the declarations appear in L3’s order, so the two sources can be read side by side. Only what Chess LUX adds carries new names.

The difference is clearest in the locked-structure class of chasolver's data ("Blocked", 50,526 games):

| version | undetected | coverage |
|---|---|---|
| FideLite (fidelite.art, king + pawn) | 3,567 | 92.94% |
| Chess LUX (+ bishops, about 1 KB) | 12 | 99.98% |
| difference | 3,555 | 7.04 percentage points |

## Verify it yourself: BestArbiter.html

`BestArbiter.html` is a benchmarking page that lets you put the claims to the test yourself; the verdicts come from the very same engine that runs the game.

1. Open [BestArbiter.html](https://cuneytinann.github.io/Chess-LUX/BestArbiter.html) online, or download it and open it locally.
2. Download `chasolver-data-until-08-2026.csv` (or one of the two `.txt` files) from this repository and drag it onto the page.
3. Press **judge the files**. **run both builds** runs the two engine builds back to back and shows that their verdicts match.

Expected results with the default settings (99 plies, 50,000 nodes):

| file | expected |
|---|---|
| `chasolver-data-until-08-2026.csv` | 201,048 correct draws, 12 without a verdict, 0 contradictions |
| `chasolver-blocked-until-08-2026.csv` | 50,514 correct draws, the same 12 without a verdict, 0 contradictions |
| `chasolver-positions.txt` | 0 false draws across the 1,945 positions the opponent can win |
| `chasolver-lichess.txt` | 0 false draws |
| between the two engine builds | 0 differing verdicts |

Large files can take a few minutes to process; since the defaults were raised to 99 plies and 50,000 nodes, the two `.txt` sets — where most positions admit no proof — take noticeably longer than they used to. To try a single position, simply paste its FEN into the box on the page.

## Files

| file | size | contents | source |
|---|---|---|---|
| `index.html` | 7,688 B | game and arbiter | this project |
| `BestArbiter.html` | 68,839 B | benchmarking page with both engine builds embedded | this project |
| `chasolver-data-until-08-2026.csv` | 22.9 MB | the 201,060 wrongly decided timeouts chasolver found on Lichess (through August 2026) | [chasolver.org](https://chasolver.org/unfair-games) |
| `chasolver-blocked-until-08-2026.csv` | 6.2 MB | the "Blocked" (locked-structure) class of the same data, 50,526 games | [chasolver.org](https://chasolver.org/unfair-games) |
| `chasolver-positions.txt` | 152 KB | chasolver's 3,414 labeled challenging test positions | [chasolver `tests/positions.txt`](https://github.com/miguel-ambrona/chasolver/blob/main/tests/positions.txt) (MIT) |
| `chasolver-lichess.txt` | 3.3 MB | 65,536 labeled Lichess positions | [chasolver `tests/lichess.txt`](https://github.com/miguel-ambrona/chasolver/blob/main/tests/lichess.txt) (MIT) |
| `LICENSE` | | MIT License | this project |

The four `chasolver-*` data files are Miguel Ambrona's work. They are included here with attribution so that the measurements can be reproduced; the originals are linked in the source column.

## Counterexamples, contributions and license

**LUX: the fairest chess in the universe.** I stand by that claim, and if there's anything better out there, bring it on :)

Jokes aside, the claim is open to challenge. If you find a winnable position where Chess LUX awards a draw, or one that makes BestArbiter report **FALSE TM** or **found mate**, please share it by opening an issue on GitHub. Even a single counterexample is valuable to us.

Chess LUX is released under the MIT License; see the `LICENSE` file for details.

## Acknowledgments and credits

- **[chasolver](https://github.com/miguel-ambrona/chasolver), Miguel Ambrona:** unwinnability analysis, test positions and data on wrongly decided Lichess games ([chasolver.org](https://chasolver.org)).
- **[Lichess open database](https://database.lichess.org/)** (CC0).
- **FideLite:** the foundation of this engine; the rules and functions are documented at [fidelite.art](https://fidelite.art).

---

## Technical appendix

This section is for anyone who wants to modify the engine or reproduce the measurements.

<details>
<summary><b>Engine: board, state and functions</b></summary>

`b` is a 64-element array: index 0 = a1, 63 = h8. Each square holds `type*2 + color`, where the color bit is 1 for White and 0 for Black, and an empty square is 0:

| type | bishop | rook | queen | pawn | king | knight |
|---|---|---|---|---|---|---|
| code (Black/White) | 2/3 | 4/5 | 6/7 | 8/9 | 10/11 | 12/13 |

The starting position is built from a hexadecimal string: ``5d37b3d5${10n**40n-10n**32n}888888884c26a2c4``.

| global | meaning |
|---|---|
| `t` | side to move (1 White, 0 Black) |
| `c` | castling rights: 1 White kingside, 2 White queenside, 4 Black kingside, 8 Black queenside |
| `e` | en passant target square; set only when a legal capture exists, otherwise -1 |
| `n` | halfmove clock |
| `R`, `$` | repetition table (key `b+t+e+c`) and the repetition count of the current position |
| `o` | draw offers (bits 1, 2) and which side has already moved (bits 4, 8) |
| `z` | result (0 = game in progress) |
| `N`, `Q` | the strings `'indexOf'` and `'innerHTML'`, as in L3 |
| `Y` | the constant -1: no en passant square, no selected square, no such piece |
| `Hn`, `Ht` | search node budget and transposition table |

| name | role | 1x | 4x |
|---|---|---|---|
| `G(i,f,T)` | move geometry and castling; attack mode when `T` is set | 263 | 264 |
| `V(s,u)` | whether square `u` is attacked by `s`'s opponent; with `u` omitted, whether `s`'s king is in check | 50 | 53 |
| `L(i)` | legal target squares of a piece | 104 | 105 |
| `C(i)` | castling rights affected by a square | 27 | 27 |
| `M(i,f,u)` | makes a move on a copy of the board; in 1x it also clears the castling rights the squares touch | 234 | 219 |
| `I(g)` | insufficient material | 85 | 85 |
| `Im(g)` | material shortcut for the flag verdict | 75 | 75 |
| `Z()` | result after a move | 73 | 76 |
| `A(i,f,u)` | plays a move | 60 | 75 |
| `D(g)` | draw claims and offers | 43 | 43 |
| `H(g,d)` | helpmate search | 222 | 252 |
| `F(g,k)` | flag or resignation verdict | 43 | 43 |
| `l(a,G)` | dead-position and lock detector | 1,213 | 1,326 |
| **engine file** | | **2,644** | **2,795** |

`index.html` carries the 4x definitions (`G V L C M I Im H l`) byte-for-byte identical to those in `engine_4x.js`; `Z D F A` are interface versions that report the result as a text code. On top of the material test come two layers that most sites lack: `l` (1,326 bytes) and `H` (252 bytes), 1,578 bytes combined.

**Castling rights in the search.** The two builds solve this differently. 4x updates `c` inside `H`, right before `M`, and leaves `M`, `L` and `A` untouched. 1x puts the line into `M` itself, so `L` has to carry `c` through its save/restore pair and `A` drops its own copy — ten bytes cheaper, but the line then also runs on every pseudo-legal move `L` tries out, about 3% slower over 833 positions. Either way the transposition key becomes `b+t+e+c` and the search snapshot carries `c`.

**1x and 4x.** Everything shipped to users is written in 4x style: bytes come first, but a few extra bytes are spent whenever they buy a multiplicative speedup. 1x is the shortest source with identical behavior and is offered only as an experimental option on the benchmarking page. The two builds must return the same verdict on every input; on real games 4x stays under 100 ms, whereas 1x can take seconds.

**Priorities.** (1) Soundness: the arbiter never awards a false draw. (2) Bytes. (3) Speed. 100% completeness, meaning never missing a draw, is the goal but is not claimed.

</details>

<details>
<summary><b>Search H: flag fall and resignation</b></summary>

If `H(g,d)` returns true, `g`'s opponent cannot mate. A node is proven in the following order:

1. `Im(g)`, the material shortcut;
2. `Ht[b+t+e+c]>=d`, meaning the same position has already been examined to at least this depth;
3. `l(2-g)`, the one-sided lock detector that asks only about the opponent's winning chances.

If none of these applies, all legal moves and promotion choices (`[3,2,1,6]`) are tried.

- **75 moves:** At a node where the halfmove clock exceeds 149, every move is treated as proven.
- **Leaf with no legal moves:** If `g` is checkmated, the branch returns false; on stalemate or the opponent's checkmate it returns true.
- **Repetition:** A position that recurs along the same line is cut off the second time it appears; the shortest route to mate never repeats a position.
- **Castling rights:** They are updated along the line, and `c` is part of both the snapshot and the transposition key, so the search never generates a castling move after the right has been lost.
- **Budget and depth:** If either runs out, the branch returns false and the verdict defaults to a win. The game history (fivefold repetition) does not enter the flag search. Since the depth limit was raised to 99 plies, the budget is in practice the only bound: a position with no proof spends the full 50,000 nodes, roughly a second in a browser.

| where | budget | depth |
|---|---|---|
| `index.html` | 50,000 nodes | 99 plies |
| `F` in the embedded engine | 50,000 nodes | 99 plies |
| benchmarking page | adjustable, default 50,000 | adjustable, default 99 |

</details>

<details>
<summary><b>Lock detector l</b></summary>

**Interface.** `l(a=3, G=0)` operates on the globals `b` and `e`; returning true constitutes a certificate.
- **`a`:** Whose winning chances are queried: 2 for White, 1 for Black, 3 for both. Dead positions use 3; `H` calls `l(2-g)`.
- **`G`:** Frozen-king level. 0 is normal; at 1 the black king and at 2 the white king is assumed immobile, and the assumption is verified at the end.
- **En passant:** No certificate is issued while an en passant capture is available.

**Bitboards.** Each set is a 64-bit `BigInt`; bit `i` represents square `i`.
- **Masks:** `f` is the whole board, `y` everything but the a-file, `x` everything but the h-file, `d` the light squares.
- **Helpers:** `S` horizontal neighbors, `N` the 3×3 neighborhood including the square itself, `O` orthogonal neighbors, `T` diagonal neighbors.
- **Occupancy:** `M` all White men, `D` all Black men; `m`/`n` are the uncapturable pawns of each side within one iteration, and `I`/`J` the two sides' blocked-square sets.

**Fixed point.** Each iteration grows or shrinks the sets:
- **Blocked pawns:** `P`/`Q` only ever shrink; what remains are the pawns the enemy king can never capture and whose way forward is permanently blocked.
- **Pawn cones:** `C`/`Z`, the squares the pawns can advance to.
- **King floods:** `K`/`L`, the squares the kings can reach; they only grow. An enemy pawn touched by the flood counts as capturable via `r`/`q`.
- **Bishop floods:** `R`/`c`, the squares the bishops can reach; they stop at fixed squares.
- **Boxed-in pieces:** A knight, rook or queen whose neighborhood opens up, or which the opponent can reach, records its escape in `Y`; the first escape fails that level.

In 4x the loop stops as soon as the `z` checksum (`C+Z+K+L+r+q-P-Q+R+c`) stops changing (at most 769 iterations; never more than 69 in fuzzing). 1x always runs 768 iterations.

**Lock conditions.**
- no promotion;
- no pawn captures and no pawn checks;
- the king assumed frozen really is immobile;
- no piece has escaped its box (`!Y`);
- if bishops are present, the bishop layer holds.

**Bishop layer.** The king floods must not intersect. For each side and color there may be at most one bishop, and the CAPTURE test must pass: the bishop must not touch any piece or pawn cone. Beyond that, one of two conditions is required: either REGION (the bishop never touches the defending king's flood) or, with all of the defender's pawns blocked, the COLOR certificate.

**COLOR certificate.** A bishop can deliver mate only by checking along a diagonal on a square of its own color. The certificate shows that on every such square within the defending king's reach, an escape square remains even when the king is in check.
- **Hard blockers:** The defender's fixed pieces, the attacker's pawn attacks and the squares adjacent to the attacking king. They can seal any number of squares at once.
- **Soft blocker:** The defender's single bishop of that color, which can seal at most one square at a time. That is why two non-hard squares, or one free square, guarantee an escape.

1x source:

```js
|!(o?B^Q:W^P)&(!(a&1+o)|!(i=o?c:R,j=f^(p&~i|(o?S(C)<<8n|N(K):S(Z)>>8n|N(L))|t&~I),u=j&~i,s=j&I,A=u&I,m&I&~(E(9n)&E(7n)|O(u)|j>>8n&j<<8n|j>>1n&j<<1n&x&y|(j>>8n|j<<8n)&S(j))))
```

- `E=k=>s>>k&s<<k|A>>k|A<<k`: an escape on one diagonal pair;
- `i`: the defender's bishop flood;
- `j`: non-hard squares;
- `u`: free squares;
- `s`, `A`: copies restricted to the bishop's color.

**Invariants** (a change that breaks any of them invalidates the COLOR certificate):
1. at most one bishop in each combination;
2. the defender's pawns are permanently blocked;
3. no boxed-in piece has escaped;
4. no promotion, so shifts never exceed 64 bits; this block must not be moved ahead of the promotion check;
5. a checking bishop cannot control diagonal neighbors that lie off the checking line.

**Closed soundness holes.** The old detector issued false certificates in the positions below; today `l(3)/l(2)/l(1)` returns:

| hole | position | today |
|---|---|---|
| `T` was mistakenly written with XOR | `6k1/8/8/p1p1p1p1/PpPpPpPp/1P1P1P1P/2B5/2B3K1 w - - 0 1` | 0/0/0 |
| the capture test ignored advancing pawns | `4k3/7p/4p1pP/3pP1P1/2pPp3/1pP5/pP2P3/K5B1 w - - 0 1` | 0/0/0 |
| same family | `4k3/7p/4p1pP/3pP1P1/2pP2p1/1pP5/pP4P1/K5B1 w - - 0 1` | 0/0/0 |
| COLOR overlooked a king on the edge (…Bd1#) | `8/1k6/p1p1p1p1/P1p1P1P1/K1p1p1p1/P1P1P1P1/4b3/8 b - - 0 1` | 0/1/0 |
| same family (…Bc6#) | `b7/k7/p3p3/P1p1p3/K1p1p1p1/P1P1P1P1/8/8 b - - 0 1` | 0/1/0 |
| same family (Bd8#) | `8/4B3/p1p1p1p1/k1P1P1P1/p1P1p1p1/P1P1P1P1/1K6/8 w - - 0 1` | 0/0/1 |

**Origins.** The core of `l` is FideLite's king-and-pawn lock detector. That core was fuzzed with 5.8 million unique positions and 2,405,996 chasolver queries without turning up a single counterexample. All three known holes lay in the bishop layer.

</details>

<details>
<summary><b>Verification details and measurements</b></summary>

| measurement | result |
|---|---|
| chasolver data: proven draws | 201,048 / 201,060 (99.9940%) |
| locked-structure class: proven draws | 50,514 / 50,526 (99.98%) |
| no verdict | 12; all in the locked-structure class, and in each the search runs out of depth |
| mates found by the search (contradicting the reference) | 0 |
| `chasolver-positions.txt`: false draws across the 1,945 positions the opponent can win | 0 |
| `chasolver-positions.txt`: proven / missed draws (1,469 targets) | 796 / 673 |
| `chasolver-lichess.txt`: false draws | 0 |
| differing verdicts between 1x and 4x | 0 |

**Independent cross-checks**
- **Perft:** Identical to chess.js 1.4.0 on 19 tricky positions at depths 3–4.
- **Random games:** Across 450 games and 123,453 plies, the legal move set, board, castling rights and halfmove clock matched chess.js at every step.
- **FEN:** On 14,005 FENs, positions set up through `index.html`'s FEN path produced the same legal moves as chess.js.
- **`l`:** Mutation-based fuzzing on about 375,000 locked positions yielded bit-for-bit identical verdicts from the old and the streamlined engine.

**Speed**
- **Real games:** On a 10% sample of the chasolver data (20,106 positions), median 0.1 ms, maximum 60 ms.
- **Hardest position:** `B6b/pr6/8/8/8/4p1pp/P3Pp1p/2b2K1k b - -`. It uses 20,107 nodes and returns the correct verdict (WT); about 0.4 s in a browser on a mid-range laptop. A position with no proof at all now spends the full 50,000-node budget, roughly a second.
- **What drives the time:** The cost per node. Each node involves legal move generation, a board copy and a call to `l`.
- **Native code:** Ported to C++ or Rust with 64-bit integers, or compiled to WebAssembly, the engine would shed the memory-allocation overhead of BigInt. Typical positions could drop to microseconds and the hardest one to milliseconds; in practice, though, there is no need.

**FEN validation.** The following are checked:
- 64 squares and valid piece letters;
- exactly one king per side;
- no pawns on the first or eighth rank;
- the side not to move is not in check;
- a halfmove clock between 0 and 255.

Castling rights are set only when the king and rook stand on their original squares, and the en passant square only when a capture is actually possible. Known limitation: rank lengths are not checked individually.

</details>

<details>
<summary><b>Twelve positions left to conquer, roadmap and verification queue</b></summary>

These 12 positions are not wrong verdicts but draws that have yet to be detected. In each of them the search spends 130–240 nodes before hitting the depth limit; no position along the line can be certified by the material or lock analysis.

| gameId | flagged | n | FEN |
|---|---|---|---|
| GyWj6Ymz | Black | 8 | `8/4k1p1/4p1Pp/1p1pPp2/pP1P1P1P/P7/8/4K3 b - - 8 63` |
| 3KkCirHD | White | 0 | `8/1p6/4k3/1P1p2p1/1p1P2P1/1P1K4/8/8 w - - 0 46` |
| chikSuUd | Black | 11 | `4K3/8/6p1/5pP1/5P1k/5P1p/7P/8 b - - 11 59` |
| azjorRHB | Black | 0 | `8/4k1p1/4p1Pp/1p1pPp2/pP1P1P1P/P4K2/8/8 b - - 0 48` |
| lBSDAx07 | White | 0 | `8/4k3/1p4p1/pP1p1pP1/2pP1P2/P1P1KP2/8/8 w - - 0 49` |
| RwnQJq1k | White | 11 | `8/7p/5p1P/5p1K/5Pp1/6P1/b3k3/8 w - - 11 51` |
| hPiwD75i | White | 7 | `1n6/2Bp4/p1pPp1k1/PpP1Pp1p/1P3P1P/2K5/8/8 w - - 7 53` |
| vcaVIyhj | White | 4 | `8/1p1k4/4p3/1P1pP1p1/1p1P2Pp/1P1K3P/8/8 w - - 4 47` |
| pw3hB0Tp | Black | 0 | `8/1p2k3/8/1P1p2p1/1p1P2P1/1P3K2/8/8 b - - 0 44` |
| q2K0VFaV | White | 0 | `8/6k1/6p1/p1p1p1P1/P1P1P1p1/6K1/6P1/8 w - a6 0 38` |
| FKr42ZRT | Black | 13 | `8/8/7p/5p1P/5p1K/5Pp1/6P1/5kb1 b - - 13 63` |
| o2conOyc | White | 1 | `8/2p2kp1/1pPp4/pP1Pp1P1/P3P1p1/6P1/8/5K2 w - - 1 44` |

**Roadmap**
1. Teach the lock analysis to recognize these 12 structures.
2. Treat trapped bishops as blockers: `1kb5/1p1p4/1P1P4/8/8/4p1p1/4P1P1/5BK1 w - - 0 1` should be `DP` on the first move. Today the flag verdict is correct (`TM`) and the game ends by fivefold repetition, but the dead position is not declared on move one.
3. Reduce the 673 missed draws in `chasolver-positions.txt`.
4. Bring the game history (fivefold repetition) into the flag search.
5. Adapt the engine to Chess960.

**Verification queue**
- Independent verification of the "most rule-accurate arbiter" claim.
- Counting the Lichess games that should have been drawn under the 75-move rule.

</details>

<details>
<summary><b>Making changes: checklist, pitfalls, measuring with Node</b></summary>

**Checklist**
1. **Soundness rationale:** Write down why a new certificate makes mate impossible.
2. **4x and 1x:** Code shipped to users is written in 4x style; 1x is updated separately as the shortest source with identical behavior.
3. **Three copies:** Apply every change to `index.html` and to both `src_x4` and `src_nm` in `BestArbiter.html`; `index.html` and `src_x4` must stay identical. When porting to 1x, mind the difference between `&` and `&&`.
4. **Build equivalence:** The two builds must return the same verdict on every input (run both builds).
5. **Regression:** The positions in the closed-holes table must never be certified in the wrong direction.
6. **Soundness check:** 0 false draws on both `.txt` files and 0 found mates on the chasolver data.
7. **No completeness regression:** At least 201,048 correct draws on the chasolver data and at most 673 missed draws on `chasolver-positions.txt`.
8. **New certificate families:** Generate targeted positions and cross-check them against chasolver; ready-made test sets do not cover every corner case of a certificate.

**Pitfalls**
- **Ready-made test sets are not enough:** The known holes were found by targeted fuzzing: moving the king to the edge, adding locked pawn pairs, bishops on both colors, shifting the board, swapping colors.
- **Forgetting to count:** "A piece can reach this square" does not mean "this square can be sealed"; a single piece seals only one square at a time.
- **Negative BigInt:** `~x` yields a negative number; mask it down to 64 bits with `f&~(…)` before shifting.
- **File wraparound:** Horizontal shifts always need a mask; on a set restricted to one color, diagonal shifts are correct even without one.
- **Operator precedence:** `&&` and `||` bind more loosely than `|` and `&`; when you convert one, check its neighbors too.
- **Early-exit checksum:** Growing sets enter with a plus sign, shrinking ones (`P`, `Q`) with a minus sign.
- **`M` is not side-effect-free in 1x:** there it also clears castling rights, so `L` must carry `c` through its save/restore pair. In 4x `M` leaves `c` alone and the search updates it instead. Never port one half of that pair without the other.

**Measuring with Node**

```js
const fs = require('fs'), vm = require('vm');
const html = fs.readFileSync('BestArbiter.html', 'utf8');
const blocks = [...html.matchAll(/<script[^>]*>([\s\S]*?)<\/script>/g)].map((m) => m[1]);
const SRC = html.match(/id="src_x4">([\s\S]*?)<\/script>/)[1].trim();   // src_nm for 1x
const chunks = (t) => { const r = []; let d = 0, st = 0; for (let i = 0; i < t.length; i++) { const c = t[i]; if ('([{'.includes(c)) d++; else if (')]}'.includes(c)) d--; else if (c === ',' && d === 0) { r.push(t.slice(st, i)); st = i + 1; } } r.push(t.slice(st)); return r; };
const mirror = (x) => { const y = x.replace(/^H=/, 'Hd=').split('Ht[').join('Hs[').replace(',H(g,d-1)', ',Hd(g,d-1)'); const k = y.lastIndexOf('&&'); return y.slice(0, k + 2) + '(' + y.slice(k + 2, y.length - 1) + '||(HM=1,0)))'; };
const HD = chunks(SRC).filter((c) => c.startsWith('H=')).map(mirror)[0];
const ctx = { Math, BigInt, performance, console }; ctx.window = ctx; vm.createContext(ctx);
vm.runInContext(blocks[2], ctx);
vm.runInContext(SRC + ';' + HD + ';HM=0;Hs={};Ht={};Hn=0', ctx);
ctx.judge('8/8/4k3/3p2p1/1p1P1pP1/1P3P2/8/3K4 b - - 47 92', 99, 50000).code;   // 'DP'
```

Run with `node --stack-size=4000`.

</details>

---

# Chess LUX (Türkçe)

[English](#chess-lux) · **Türkçe**

▶ **[Tarayıcıda hemen oyna](https://cuneytinann.github.io/Chess-LUX/)** · [Ölçüm sayfası](https://cuneytinann.github.io/Chess-LUX/BestArbiter.html)

**7.688 baytlık tek bir HTML dosyasında, dünyanın kural doğruluğu en yüksek satranç hakemi.**

Chess LUX, tarayıcıda açılan ve aynı ekranda iki kişinin oynadığı bir satranç uygulaması. Asıl işi hakemlik: her hamlenin legal olup olmadığını, oyunun bitip bitmediğini ve sonucu FIDE kurallarına göre belirler. Özellikle bayrak düşmesinde ve kilitli pozisyonlarda, çoğu satranç sitesinin yapmadığı bir denetim yapar.

## Bir bakışta

| | |
|---|---|
| **Boyut** | tek dosya, 7.688 bayt: oyun, saat, arayüz ve hakem bir arada |
| **Kurallar** | mat, pat, ölü pozisyon, bayrak düşmesi, terk, 50 ve 75 hamle kuralı, üçlü ve beşli tekrar, beraberlik talebi ve teklifi |
| **Lichess verisiyle** | bayrak düşmesiyle haksız sonuçlanmış 201.060 oyunun 201.048'inde doğru hüküm (%99,9940) |
| **Kilitli yapılarda** | bunların 50.526'sını oluşturan kilitli yapı sınıfında 50.514 doğru hüküm (%99,98) |
| **Yanlış beraberlik** | test pozisyonlarında 0 |
| **Hız** | bayrak hükmü gerçek oyunlarda medyan 0,1 ms; en zor kurgulanmış pozisyonda yaklaşık yarım saniye |
| **Kurulum** | yok: çevrimiçi oynayın ya da dosyayı indirip internetsiz oynayın |

## Hemen dene

1. [cuneytinann.github.io/Chess-LUX](https://cuneytinann.github.io/Chess-LUX/) adresini açın ya da `index.html`'i indirip tarayıcıda açın. Sekmede "FideLite" başlığı görünür.
2. Açılan pencerede başlangıç pozisyonunu ve süreyi (10 dk + 5 sn) olduğu gibi bırakın ya da kendi FEN'inizi girin, ardından **Play**'e basın.
3. Taşları tıklayarak ya da sürükleyerek oynayın. Sağ tuşla tahtada sürüklerseniz aradaki bütün kareler işaretlenir, tek kareye sağ tıklarsanız yalnız o kare işaretlenir; sol tıklama işaretleri siler. ½ beraberlik talebi ve teklifi, ⚐ terk içindir; oyun bitince ↺ yeni bir oyun açar.

Güncel bir tarayıcı yeterli: Chrome/Edge 85+, Firefox 98+, Safari 15.4+ (2022 ve sonrası).

Chess LUX bir prototip; ağırlık kural katmanında, arayüz ikinci planda. Kural katmanı arayüzden bağımsız olduğu için isteyen onu alıp kendi sistemine uyarlayabilir. Arayüzsüz hâli, `BestArbiter.html` içinde `engine_4x.js` adıyla duruyor.

## Neden önemli

FIDE kurallarına göre (madde 6.9) süresi biten oyuncu her zaman kaybetmez: rakibi, iki taraf el ele verse bile onu hiçbir legal hamle dizisiyle mat edemiyorsa oyun berabere biter. Satranç sunucuları bu kuralı uzun süre büyük ölçüde uygulamadı.

Gerçek bir örnek: Lichess'te oynanan [ijyj0mHa](https://lichess.org/ijyj0mHa#120) oyunu (chasolver'ın örneklerinden).

```
5b2/p7/Pp3k2/1Pp1pBp1/2P1P1P1/5K2/8/8 w - -
```

Beyazın süresi bitti ve oyun beyazın yenilgisiyle sonuçlandı. Oysa piyon duvarı kalıcı olarak kilitli; siyah, beyaz yardım etse bile mat edemez. Chess LUX burada beraberlik (`TM`) verir. İlginç olan şu: aynı pozisyonda süresi biten siyah olsaydı hüküm beyazın galibiyeti olurdu, çünkü beyazın mat edebileceği bir yol var.

[chasolver](https://chasolver.org), Lichess'in milyarlarca oyununu tarayarak bu şekilde haksız sonuçlanmış **201.060** oyun buldu. Chess LUX bunların **201.048'inde** doğru hükmü, yani beraberliği veriyor. Kalan 12 pozisyonda kanıt bulamıyor ve güvenli tarafta kalarak galibiyet veriyor. Bunlar yanlış hüküm değil, yakalanamayan beraberlikler: oyun, bugünkü uygulamada olduğu gibi galibiyetle sonuçlanıyor. Haksız sonuçlanan oyunların yaklaşık 16.750'de 1'i olan bu 12 pozisyon, fethedilecek hedefler olarak teknik ekte listeleniyor. Lichess'in standart dereceli oyun veritabanı 7 milyarı aşkın oyun içeriyor; buna göre Chess LUX'ın kaçırdığı hükümler milyar oyunda 2'nin, yani her 500 milyon oyunda 1'in altında kalıyor.

Ölü pozisyonda da benzer bir durum var. FIDE'ye göre (madde 5.2.2) iki taraf için de matın imkânsız olduğu bir pozisyon oluştuğu anda oyun berabere biter. chasolver'ın istatistiklerine göre haksız bayrak sonuçlarının %57,5'inde pozisyon zaten ölüydü; bu oyunlar aslında çoktan bitmişti ama bayrak düşene kadar sürdü. Chess LUX ölü pozisyonu, tanıyabildiği ölçüde, oluştuğu hamlede ilan eder.

## Hakem nasıl karar verir

**Her hamleden sonra** oyunun bitip bitmediğine bakar. Mat, pat, ölü pozisyon (yetersiz materyal ya da kilitli yapı), beşli tekrar ve 75 hamle kuralı kendiliğinden uygulanır.

**Bayrak düştüğünde ya da bir oyuncu terk ettiğinde** tek bir soru sorar: rakip, iki taraf el ele verse bile, bu oyuncuyu mat edebilir mi? Cevabı üç adımda arar:

1. **Materyal:** Mat için yetersiz materyal doğrudan tanınır.
2. **Kilitli yapılar:** Piyon duvarları, hareket edemeyen taşlar ve fillerin ulaşamadığı kareler analiz edilir.
3. **Arama:** İki tarafın bütün hamleleri 99 yarım hamle derinliğe kadar denenir; arama en fazla 50.000 düğümde durur. Her varyantın bir kanıtla kapanması gerekir: materyal, kilitli yapı, pat, tekrar eden pozisyon ya da 75 hamle sınırı.

Kanıt bulunamazsa süresi kalan taraf kazanır. Öncelik hiçbir zaman yanlış beraberlik vermemek; bir beraberliği kaçırmak daha küçük bir kusur.

**Her eylemden önce saat kapatılır.** Hamle de, beraberlik talebi de, terk de önce geçen süreyi sırası gelen oyuncunun saatine yazar; bu saat sıfırlanıyorsa bayrak hükmü verilir ve eylem kabul edilmez. 99 ms'lik gösterim tiki bu yüzden yalnızca oyuncuların gördüğünü geciktirir, hakemin saydığını değil: geçen süre her zaman son okumadan itibaren ölçülür, iki tik arasında ne kaybolur ne de yanlış tarafa yazılır. Ölçüm `performance.now()` ile, yani monotonik bir kaynaktan yapılır; sistem saati düzeltilse de süre ne eksilir ne artar.

**Süre kontrolü.** İki alan da kesirli dahil her negatif olmayan sayıyı kabul eder: 2,5 dakika 150 saniyelik oyun, 2,5 saniye 2,5 saniyelik artırım demektir; hiçbir tarafta sessiz yuvarlama yoktur. Sonlu bir sayı olmayan her değer, 24 saat ve üzeri başlangıç süresi, 1 saat ve üzeri artırım oyun başlamadan reddedilir. Saat `DD:SS` okunur ve saat alanını yalnız gerektiğinde açar: `15:00` `15:00` kalır, 90 dakikalık kontrol `1:30:00` görünür.

**75 hamle kuralı bayrak hükmüne de girer.** Mat ancak 75 hamle sınırından sonra mümkünse bayrak düşmesi beraberlikle sonuçlanır:

```
4k1n1/8/8/8/8/8/8/4K1N1 w - - 146 1   beyazın süresi biterse beraberlik: dört yarım hamle sonra 75 hamle doluyor
4k1n1/8/8/8/8/8/8/4K1N1 w - - 0 1     aynı pozisyon, sayaç 0: siyah kazanır
```

**Katmanlar birbirini tamamlar.** Şu pozisyona bakalım:

```
1kb5/1p1p4/1P1P4/8/8/4p1p1/4P1P1/5BK1 w - - 0 1
```

İki fil de kendi piyonlarının arasında hapsolmuş; beyaz şah yalnızca g1 ile h1, siyah şah yalnızca b8 ile a8 arasında gidip gelebiliyor. Hiçbir taraf mat edemez, yani pozisyon FIDE'ye göre daha ilk hamleden ölü. Bugünkü kilit analizi hapsolmuş filleri henüz engel olarak saymadığı için ölü pozisyonu ilk hamlede ilan etmiyor. Yine de hakem doğru sonuca varıyor: bayrak düşerse arama dört yarım hamlede aynı pozisyona geri döndüğünü görüp beraberlik (`TM`) veriyor; şahlar gidip geldikçe oyun 16 yarım hamle sonra beşli tekrarla (`5R`) berabere bitiyor. Bu pozisyonu ilk hamlede `DP` olarak tanımak, fethedilecekler listesinde.

### Beraberlik talebi ve teklifi: yarı otomatik

Üçlü tekrar ve 50 hamle talepleri ne tamamen elle ne tamamen otomatik işler; ½ düğmesi FIDE Satranç Kuralları'ndaki masabaşı işleyişini izler:

- **½'ye doğrudan basmak:** Talep o anki pozisyon için değerlendirilir (9.2.2, 9.3.2).
- **½ işaretliyken hamle yapmak:** Talep, hamlenin ortaya çıkaracağı pozisyon için değerlendirilir (9.2.1, 9.3.1). Bu, masabaşında hamleyi yazıp niyetini hakeme bildirmenin karşılığıdır.
- **Önce tahta okunur:** Talep, ancak doğuracağı pozisyon incelendikten sonra değerlendirilir. O hamle mat ya da pat veriyorsa veya pozisyon ölüyse oyun orada biter, talep hiç doğmaz — mat 50 hamle kuralını da, üçlü tekrarı da, anlaşmalı beraberliği de ezer.
- **Talep yerinde değilse:** Rakibe beraberlik teklifi olarak gider (9.1.2.3). Hamleyle birlikte giden teklif, masabaşında hamleden sonra ve saate basmadan önce yapılan teklifin karşılığıdır (9.1.2.1).
- **Rakibin cevabı:** Rakip ½'ye basarsa oyun anlaşmalı beraberlikle biter; basmadan hamle yaparsa teklifi reddetmiş olur.

Tam manuel bir düzende oyuncu talep anını yakalamak zorunda kalırdı; hızlı oynayan bir rakip ya da ön hamle (premove) yüzünden talep hakkını kaçırabilirdi. Tam otomatik bir düzende ise oyun, oyuncunun istemediği bir anda beraberlikle biterdi. Yarı otomatik düzende işaret hamleden önce konur, hüküm hamleyle birlikte verilir ve talep etmek her zaman oyuncunun kararıdır.

½ düğmesi oyunun ilk hamlesinden itibaren kullanılabilir; böylece FEN'le başlatılan bir pozisyonda 50 hamle kuralı daha ilk hamlede talep edilebilir. Bu kadar erken üretilemeyen tek şey anlaşmalı beraberliktir: FIDE 5.2.3'e göre taraflar ancak ikisi de birer hamle yaptıktan sonra anlaşabilir, o ana kadar teklif kaydedilir ama karşı tarafa ulaşmaz. Düğmeyi ilk yarım hamlede kapatmak daha kolay bir çözüm olurdu, ama 50 hamle talebini de birlikte götürürdü.

### Sonuç kodları

| kod | anlamı |
|---|---|
| `W#` / `B#` | beyaz / siyah mat etti |
| `SM` | pat |
| `DP` | ölü pozisyon |
| `5R` / `75` | beşli tekrar / 75 hamle kuralı |
| `3R` / `50` | üçlü tekrar / 50 hamle talebi |
| `DA` | anlaşmalı beraberlik |
| `WT` / `BT` | bayrak düşmesiyle beyaz / siyah kazandı |
| `TM` | bayrak düştü ama rakip mat edemez: beraberlik |
| `WR` / `BR` | rakip terk etti: beyaz / siyah kazandı |
| `RM` | terk edildi ama rakip mat edemez: beraberlik (FIDE 5.1.2, 2023) |

## chasolver ve Chess LUX

[chasolver](https://github.com/miguel-ambrona/chasolver) (Miguel Ambrona), bayrak düşmesinin doğru hükme bağlanması için geliştirilmiş ve hakemli bir bilimsel makaleye ([FUN 2022](https://chasolver.org/FUN22-full.pdf)) dayanan bir çözücü. Hedefi, sunucuların haksız bayrak sonucu vermesini önlemek. Soruyu yalnızca pozisyon üzerinden ele alıyor; 75 hamle kuralı ve beşli tekrar gibi oyunun geçmişine bağlı kurallar kapsamı dışında.

Chess LUX'ın hedefi ise buna ek olarak oyunun bütün bitiş şekillerini otomatikleştirmek: ölü pozisyonlar uzamadan bitsin, beraberlik talepleri FIDE'deki gibi işlesin, oyuncular zaman kaybetmesin. FIDE uyumu konusunda titiz olduğum için 75 hamle kuralını ve beşli tekrarı da hakemliğe dahil ettim; 75 hamle kuralı bayrak hükmüne de giriyor. Chess960'ı da dışarıda bırakmadım: kilitli yapı analizi ve arama rok kuralına bağlı değil. Bu fonksiyonlar 960 destekli bir motora kolayca taşınabilir ya da bu motor 960'a uyarlanabilir; bugünkü `index.html` ise standart satranç için yazıldı.

Algoritmayı bağımsız olarak geliştirdim, ama Ambrona'nın çalışması olmasaydı bu kadar ilerletemezdim. Milyarlarca oyunu indirip analiz ederek süzdüğü veriyi ve topluluktan derlediği zorlu pozisyonları doğrudan kullandım; bu projenin doğruluk ölçütü onun çalışması.

### FideLite ile karşılaştırma

Chess LUX, [fidelite.art](https://fidelite.art)'taki FideLite motorunun en üst sürümü üzerine kurulu; kuralların ve fonksiyonların ayrıntılı anlatımı orada. FideLite'ın kilit dedektörü yalnızca şah ve piyonlardan oluşan kilitli yapıları tanıyor. Chess LUX, yaklaşık 1 KB'lık ek kodla bunu bütün taşlara genişletiyor; at, kale ve vezir ancak hareketsizken hesaba katılabildiği için bu genişletme pratikte fillerin eklenmesi demek. Kısacası bu repo, FideLite'ın bayt kaygısı hafifletilip kural doğruluğu artırılmış hâli. Adlandırma L3 sürümünü izler: L3'te zaten bulunan her ad korunur — `N` `indexOf`, `Q` `innerHTML`, kilit dedektörünün içindeki piyon kümeleri hâlâ bitişik bir çift (`m`/`n`; `l` artık dedektörün kendi adı) — ve tanımlar L3'teki sırayla gelir, böylece iki kaynak yan yana okunabilir. Yalnızca Chess LUX'ün eklediği şeyler yeni ad taşır.

Farkı en açık biçimde chasolver verisindeki kilitli yapı sınıfı ("Blocked", 50.526 oyun) gösteriyor:

| sürüm | yakalanamayan | kapsama |
|---|---|---|
| FideLite (fidelite.art, şah + piyon) | 3.567 | %92,94 |
| Chess LUX (+ filler, yaklaşık 1 KB) | 12 | %99,98 |
| fark | 3.555 | 7,04 puan |

## Kendin doğrula: BestArbiter.html

`BestArbiter.html`, iddiaları kendiniz sınayabilmeniz için hazırlanmış bir ölçüm sayfası; hükmü oyundaki motorun kendisi verir.

1. [BestArbiter.html](https://cuneytinann.github.io/Chess-LUX/BestArbiter.html) sayfasını çevrimiçi açın ya da dosyayı indirip tarayıcıda açın.
2. `chasolver-data-until-08-2026.csv`'yi (ya da iki `.txt` dosyasından birini) bu repodan indirip sayfaya sürükleyin.
3. **judge the files** düğmesine basın. **run both builds** ise iki motor sürümünü art arda çalıştırıp hükümlerin aynı olduğunu gösterir.

Varsayılan ayarlarla (99 yarım hamle, 50.000 düğüm) beklenen sonuçlar:

| dosya | beklenen |
|---|---|
| `chasolver-data-until-08-2026.csv` | 201.048 doğru beraberlik, 12 hükümsüz, 0 çelişki |
| `chasolver-blocked-until-08-2026.csv` | 50.514 doğru beraberlik, aynı 12 hükümsüz, 0 çelişki |
| `chasolver-positions.txt` | rakibin kazanabildiği 1.945 pozisyonda yanlış beraberlik: 0 |
| `chasolver-lichess.txt` | yanlış beraberlik: 0 |
| iki motor sürümü arasında | farklı hüküm: 0 |

Büyük dosyaların işlenmesi birkaç dakika sürebilir; varsayılanlar 99 yarım hamle ve 50.000 düğüme çıktığı için, çoğu pozisyonun kanıtlanamadığı iki `.txt` kümesi eskisinden belirgin biçimde uzun sürüyor. Tek bir pozisyonu denemek için FEN'i sayfadaki kutuya yapıştırmak yeterli.

## Dosyalar

| dosya | boyut | içerik | kaynak |
|---|---|---|---|
| `index.html` | 7.688 B | oyun ve hakem | bu proje |
| `BestArbiter.html` | 68.839 B | ölçüm sayfası; iki motor sürümü gömülü | bu proje |
| `chasolver-data-until-08-2026.csv` | 22,9 MB | chasolver'ın Lichess'te bulduğu 201.060 haksız bayrak sonucu (Ağustos 2026'ya kadar) | [chasolver.org](https://chasolver.org/unfair-games) |
| `chasolver-blocked-until-08-2026.csv` | 6,2 MB | aynı verinin chasolver'daki "Blocked" (kilitli yapı) sınıfı, 50.526 oyun | [chasolver.org](https://chasolver.org/unfair-games) |
| `chasolver-positions.txt` | 152 KB | chasolver'ın 3.414 etiketli zorlu test pozisyonu | [chasolver `tests/positions.txt`](https://github.com/miguel-ambrona/chasolver/blob/main/tests/positions.txt) (MIT) |
| `chasolver-lichess.txt` | 3,3 MB | 65.536 etiketli Lichess pozisyonu | [chasolver `tests/lichess.txt`](https://github.com/miguel-ambrona/chasolver/blob/main/tests/lichess.txt) (MIT) |
| `LICENSE` | | MIT lisansı | bu proje |

Dört `chasolver-*` veri dosyası Miguel Ambrona'nın çalışmasıdır. Ölçümler tekrarlanabilsin diye atıfla birlikte bu repoya eklendi; özgün kaynakları tablodaki bağlantılarda.

## Karşı örnek, katkı ve lisans

**Evrenin en adil satrancı LUX.** İddia ediyorum; daha iyisi varsa çıksın karşıma :)

Şaka bir yana, bu iddia sınamaya açık. Kazanılabilir bir pozisyona beraberlik veren ya da BestArbiter'de **FALSE TM** veya **found mate** sonucu çıkaran bir pozisyon bulursanız GitHub'da bir issue açarak paylaşın. Tek bir karşı örnek bile bizim için değerli.

Chess LUX, MIT lisansıyla yayımlanıyor; ayrıntılar `LICENSE` dosyasında.

## Teşekkür ve atıf

- **[chasolver](https://github.com/miguel-ambrona/chasolver), Miguel Ambrona:** kazanılamazlık analizi, test pozisyonları ve Lichess'teki haksız sonuç verisi ([chasolver.org](https://chasolver.org)).
- **[Lichess açık veritabanı](https://database.lichess.org/)** (CC0).
- **FideLite:** bu motorun temeli; kuralların ve fonksiyonların anlatımı [fidelite.art](https://fidelite.art)'ta.

---

## Teknik ek

Bu bölüm motoru değiştirmek ya da ölçümleri tekrarlamak isteyenler için.

<details>
<summary><b>Motor: tahta, durum ve fonksiyonlar</b></summary>

`b` 64 elemanlı bir dizidir: indeks 0 = a1, 63 = h8. Her kare `tür*2 + renk` değerini tutar; renk biti 1 beyaz, 0 siyahtır, boş kare 0'dır:

| tür | fil | kale | vezir | piyon | şah | at |
|---|---|---|---|---|---|---|
| kod (siyah/beyaz) | 2/3 | 4/5 | 6/7 | 8/9 | 10/11 | 12/13 |

Başlangıç dizilimi onaltılık bir metinden kurulur: ``5d37b3d5${10n**40n-10n**32n}888888884c26a2c4``.

| global | anlamı |
|---|---|
| `t` | hamle sırası (1 beyaz, 0 siyah) |
| `c` | rok hakları: 1 beyaz kısa, 2 beyaz uzun, 4 siyah kısa, 8 siyah uzun |
| `e` | geçerken alma hedef karesi; yalnız legal bir alım varsa kurulur, yoksa -1 |
| `n` | yarım hamle sayacı |
| `R`, `$` | tekrar tablosu (anahtar `b+t+e+c`) ve mevcut pozisyonun tekrar sayısı |
| `o` | beraberlik teklifi (1, 2 bitleri) ve hangi tarafın hamle yaptığı (4, 8 bitleri) |
| `z` | sonuç (0 = oyun sürüyor) |
| `N`, `Q` | `'indexOf'` ve `'innerHTML'` dizgileri, L3'teki gibi |
| `Y` | -1 sabiti: geçerken alma karesi yok, seçili kare yok, böyle bir taş yok |
| `Hn`, `Ht` | aramanın düğüm bütçesi ve transpozisyon tablosu |

| ad | görev | 1x | 4x |
|---|---|---|---|
| `G(i,f,T)` | hamle geometrisi ve rok; `T` doluysa saldırı modu | 263 | 264 |
| `V(s,u)` | `u` karesi `s`'nin rakibince saldırı altında mı; `u` yazılmazsa `s`'nin şahı şah altında mı | 50 | 53 |
| `L(i)` | taşın legal hedef kareleri | 104 | 105 |
| `C(i)` | karenin etkilediği rok hakları | 27 | 27 |
| `M(i,f,u)` | hamleyi tahtanın bir kopyasında yapar; 1x'te karelerin düşürdüğü rok haklarını da siler | 234 | 219 |
| `I(g)` | yetersiz materyal | 85 | 85 |
| `Im(g)` | bayrak hükmü için materyal kısa yolu | 75 | 75 |
| `Z()` | hamleden sonraki sonuç | 73 | 76 |
| `A(i,f,u)` | hamle oynatır | 60 | 75 |
| `D(g)` | beraberlik talebi ve teklifi | 43 | 43 |
| `H(g,d)` | yardım matı araması | 222 | 252 |
| `F(g,k)` | bayrak ya da terk hükmü | 43 | 43 |
| `l(a,G)` | ölü pozisyon ve kilit dedektörü | 1.213 | 1.326 |
| **motor dosyası** | | **2.644** | **2.795** |

`index.html`, 4x tanımlarını (`G V L C M I Im H l`) `engine_4x.js` ile bayt bayt aynı taşır; `Z D F A` ise sonucu metin koduyla yazan arayüz sürümleridir. Materyal testinin üstüne çoğu sitede olmayan iki katman ekleniyor: `l` (1.326 bayt) ve `H` (252 bayt), birlikte 1.578 bayt.

**Aramada rok hakları.** İki sürüm bunu farklı çözüyor. 4x `c`'yi `H`'nin içinde, `M`'den hemen önce güncelliyor; `M`, `L` ve `A` değişmiyor. 1x satırı `M`'in kendisine koyuyor, bu yüzden `L` `c`'yi kaydet/geri yaz çiftinde taşımak zorunda kalıyor ve `A` kendi kopyasını bırakıyor — on bayt ucuz, ama satır bu kez `L`'nin denediği her sözde-legal hamlede de çalışıyor, 833 pozisyonda ~%3 yavaş. İki durumda da transpozisyon anahtarı `b+t+e+c` oluyor ve aramanın anlık görüntüsü `c`'yi taşıyor.

**1x ve 4x.** Kullanıcıya sunulan her şey 4x yazılır: bayt önceliklidir, ama birkaç bayt karşılığında katlanarak hız kazanılıyorsa o bayt harcanır. 1x, aynı davranışın en kısa metnidir ve yalnızca ölçüm sayfasında deneysel seçenek olarak durur. İki sürüm her girdide aynı hükmü vermek zorundadır; gerçek oyunlarda 4x 100 ms'nin altında kalırken 1x saniyelere çıkabilir.

**Öncelikler.** (1) Soundness (sağlamlık): hakem asla yanlış beraberlik vermez. (2) Bayt. (3) Hız. %100 completeness (tamlık), yani hiçbir beraberliği kaçırmamak, hedeftir ama iddia edilmez.

</details>

<details>
<summary><b>Arama H: bayrak ve terk</b></summary>

`H(g,d)` doğru dönerse `g`'nin rakibi mat edemez. Bir düğüm şu sırayla kanıtlanır:

1. `Im(g)` materyal kısa yolu;
2. `Ht[b+t+e+c]>=d`, yani aynı pozisyonun en az bu derinlikte zaten incelenmiş olması;
3. `l(2-g)`, yalnız rakibin kazanma şansını soran tek taraflı kilit dedektörü.

Hiçbiri tutmazsa bütün legal hamleler ve terfi seçenekleri (`[3,2,1,6]`) denenir.

- **75 hamle:** Yarım hamle sayacı 149'u aşmış bir düğümde her hamle kanıtlanmış sayılır.
- **Hamlesiz uç:** Uçta mat olan `g` ise dal yanlış döner; pat ya da rakibin matıysa doğru döner.
- **Tekrar:** Aynı varyantta tekrar eden bir pozisyon ikinci kez görüldüğünde kesilir; mata giden en kısa yol hiçbir pozisyonu tekrar etmez.
- **Rok hakları:** Varyant boyunca güncellenir; `c` hem anlık görüntüde hem transpozisyon anahtarındadır, dolayısıyla arama hakkı düşmüş bir rok hamlesini hiç üretmez.
- **Bütçe ve derinlik:** Biri biterse dal yanlış döner ve hüküm galibiyete düşer. Bayrak aramasına oyun geçmişi (beşli tekrar) girmez. Derinlik sınırı 99 yarım hamleye çıktığı için pratikte tek sınır bütçedir: kanıt bulunamayan bir pozisyon 50.000 düğümün tamamını harcar, tarayıcıda kabaca bir saniye.

| yer | bütçe | derinlik |
|---|---|---|
| `index.html` | 50.000 düğüm | 99 ply |
| gömülü motorun `F`'si | 50.000 düğüm | 99 ply |
| ölçüm sayfası | ayarlanabilir, varsayılan 50.000 | ayarlanabilir, varsayılan 99 |

</details>

<details>
<summary><b>Kilit dedektörü l</b></summary>

**Arayüz.** `l(a=3, G=0)` global `b` ve `e` üzerinde çalışır; doğru dönmesi bir sertifikadır.
- **`a`:** Hangi tarafın kazanma şansı soruluyor: 2 beyazın, 1 siyahın, 3 ikisinin. Ölü pozisyon için 3 kullanılır; `H` içinden `l(2-g)` çağrılır.
- **`G`:** Donmuş şah seviyesi. 0 normal; 1'de siyah, 2'de beyaz şah hareketsiz varsayılır ve varsayım sonda doğrulanır.
- **Geçerken alma:** Oynanabilir bir geçerken alma varsa sertifika verilmez.

**Bitboard.** Her küme 64 bitlik bir `BigInt`; bit `i`, kare `i`'yi temsil eder.
- **Maskeler:** `f` bütün tahta, `y` a sütunu hariç, `x` h sütunu hariç, `d` açık renkli kareler.
- **Yardımcılar:** `S` yatay komşular, `N` karenin kendisi dahil 3×3 komşuluk, `O` ortogonal komşular, `T` çapraz komşular.
- **Taş kümeleri:** `M` bütün beyaz taşlar, `D` bütün siyah taşlar; `m`/`n` bir tur içinde her iki tarafın alınamayan piyonları, `I`/`J` ise iki tarafın kapalı kare kümeleri.

**Sabit nokta.** Her tur kümeleri büyütür ya da daraltır:
- **Tıkalı piyonlar:** `P`/`Q` yalnız küçülür; rakip şahın alamadığı ve önü kalıcı olarak kapalı piyonlar kalır.
- **Piyon konileri:** `C`/`Z`, piyonların ilerleyebileceği kareler.
- **Şah selleri:** `K`/`L`, şahların ulaşabileceği kareler; yalnız büyür. Selin değdiği rakip piyon `r`/`q` üzerinden alınabilir sayılır.
- **Fil selleri:** `R`/`c`, fillerin ulaşabileceği kareler; sabit karelerde durur.
- **Kutulu taşlar:** Komşuluğu açılan ya da rakibin ulaştığı at, kale veya vezir kaçışını `Y`'ye yazar; ilk kaçışta o seviye düşer.

4x'te `z` sağlaması (`C+Z+K+L+r+q-P-Q+R+c`) değişmeyince döngü durur (en fazla 769 tur; fuzz testlerinde 69'u geçmedi). 1x her zaman 768 tur döner.

**Kilit koşulları.**
- terfi yok;
- piyon alımı ve piyonla şah çekme yok;
- donmuş varsayılan şah gerçekten hareketsiz;
- hiçbir taş kutudan çıkmadı (`!Y`);
- fil varsa fil katmanı tutuyor.

**Fil katmanı.** Şah selleri kesişmemeli. Her taraf ve renk için en fazla bir fil olmalı ve ALIM testi tutmalı: fil hiçbir taşa ve piyon konisine değmemeli. Ardından iki yoldan biri gerekir: ya BÖLGE (fil savunan şahın seline hiç değmez) ya da savunanın bütün piyonları tıkalıyken RENK sertifikası.

**RENK sertifikası.** Fil ancak kendi rengindeki bir karede, çaprazdan şah çekerek mat edebilir. Sertifika, savunan şahın ulaşabildiği ve filin renginde olan her karede, şah çekilse bile bir kaçış karesi kaldığını gösterir.
- **Sert engelleyiciler:** Savunanın sabit taşları, saldıranın piyon saldırıları ve saldıran şahın komşu kareleri. Aynı anda istedikleri kadar kareyi kapatabilirler.
- **Yumuşak engelleyici:** Savunanın o renkteki tek fili; aynı anda en fazla bir kare kapatır. Bu yüzden iki sert olmayan kare ya da bir serbest kare, kaçışı garanti eder.

1x metni:

```js
|!(o?B^Q:W^P)&(!(a&1+o)|!(i=o?c:R,j=f^(p&~i|(o?S(C)<<8n|N(K):S(Z)>>8n|N(L))|t&~I),u=j&~i,s=j&I,A=u&I,m&I&~(E(9n)&E(7n)|O(u)|j>>8n&j<<8n|j>>1n&j<<1n&x&y|(j>>8n|j<<8n)&S(j))))
```

- `E=k=>s>>k&s<<k|A>>k|A<<k`: bir çapraz çiftte kaçış;
- `i`: savunanın fil seli;
- `j`: sert olmayan kareler;
- `u`: serbest kareler;
- `s`, `A`: filin rengine kısıtlanmış kopyalar.

**Değişmezler** (birini bozan bir değişiklik RENK sertifikasını geçersiz kılar):
1. her kombinasyonda en fazla bir fil;
2. savunanın piyonları kalıcı olarak tıkalı;
3. hiçbir kutulu taş çıkmadı;
4. terfi yok, dolayısıyla kaydırmalar 64 biti aşmaz; bu blok terfi kontrolünden önceye taşınmamalı;
5. şah çeken fil, şahın hattı dışındaki çapraz komşu kareleri kontrol edemez.

**Kapatılmış açıklar.** Aşağıdaki pozisyonlarda eski dedektör yanlış sertifika veriyordu; bugün `l(3)/l(2)/l(1)` şunları döndürür:

| açık | pozisyon | bugün |
|---|---|---|
| `T` yanlışlıkla XOR ile yazılmıştı | `6k1/8/8/p1p1p1p1/PpPpPpPp/1P1P1P1P/2B5/2B3K1 w - - 0 1` | 0/0/0 |
| alım testi ilerleyen piyonları görmüyordu | `4k3/7p/4p1pP/3pP1P1/2pPp3/1pP5/pP2P3/K5B1 w - - 0 1` | 0/0/0 |
| aynı aile | `4k3/7p/4p1pP/3pP1P1/2pP2p1/1pP5/pP4P1/K5B1 w - - 0 1` | 0/0/0 |
| RENK kenardaki şahı görmüyordu (…Bd1#) | `8/1k6/p1p1p1p1/P1p1P1P1/K1p1p1p1/P1P1P1P1/4b3/8 b - - 0 1` | 0/1/0 |
| aynı aile (…Bc6#) | `b7/k7/p3p3/P1p1p3/K1p1p1p1/P1P1P1P1/8/8 b - - 0 1` | 0/1/0 |
| aynı aile (Bd8#) | `8/4B3/p1p1p1p1/k1P1P1P1/p1P1p1p1/P1P1P1P1/1K6/8 w - - 0 1` | 0/0/1 |

**Köken.** `l`'nin çekirdeği FideLite'taki şah+piyon kilit dedektörü. Bu çekirdek 5,8 milyon benzersiz pozisyon ve 2.405.996 chasolver sorgusuyla fuzz testinden geçirildi; karşı örnek çıkmadı. Bilinen üç açığın üçü de fil katmanındaydı.

</details>

<details>
<summary><b>Doğrulama ayrıntıları ve ölçümler</b></summary>

| ölçüm | sonuç |
|---|---|
| chasolver verisi: kanıtlanan beraberlik | 201.048 / 201.060 (%99,9940) |
| kilitli yapı sınıfı: kanıtlanan beraberlik | 50.514 / 50.526 (%99,98) |
| hükümsüz | 12; hepsi kilitli yapı sınıfında ve hepsinde arama derinliği yetmiyor |
| aramanın bulduğu mat (referansla çelişki) | 0 |
| `chasolver-positions.txt`: rakibin kazanabildiği 1.945 pozisyonda yanlış beraberlik | 0 |
| `chasolver-positions.txt`: kanıtlanan / kaçan beraberlik (1.469 hedef) | 796 / 673 |
| `chasolver-lichess.txt`: yanlış beraberlik | 0 |
| 1x ve 4x arasında farklı hüküm | 0 |

**Bağımsız çapraz kontroller**
- **Perft:** chess.js 1.4.0 ile 19 zor pozisyonda, 3–4 derinlikte birebir aynı.
- **Rastgele oyunlar:** 450 oyun ve 123.453 yarım hamle boyunca legal hamle kümesi, tahta, rok hakları ve yarım hamle sayacı her adımda chess.js ile aynı.
- **FEN:** 14.005 FEN'de, `index.html`'in FEN yolundan kurulan pozisyonların legal hamleleri chess.js ile aynı.
- **`l`:** Mutasyonlu fuzz testinde yaklaşık 375 bin kilitli pozisyonda eski ve sadeleştirilmiş motor bit bit aynı hükmü verdi.

**Hız**
- **Gerçek oyunlar:** chasolver verisinin %10'luk örnekleminde (20.106 pozisyon) medyan 0,1 ms, en uzun 60 ms.
- **En zor pozisyon:** `B6b/pr6/8/8/8/4p1pp/P3Pp1p/2b2K1k b - -`. 20.107 düğüm kullanıyor ve doğru hükmü (WT) veriyor; orta sınıf bir dizüstü bilgisayarda, tarayıcıda yaklaşık 0,4 s. Hiç kanıt bulunamayan bir pozisyon ise artık 50.000 düğümlük bütçenin tamamını harcıyor, kabaca bir saniye.
- **Süreyi belirleyen:** Düğüm başına maliyet. Her düğümde legal hamle üretimi, tahta kopyası ve `l` çağrısı var.
- **Yerel kod:** Motor 64 bitlik tamsayılarla C++ ya da Rust'a çevrilir veya WebAssembly'ye derlenirse BigInt'in bellek ayırma yükü ortadan kalkar. Tipik pozisyonlar mikrosaniyeler, en zor pozisyon milisaniyeler düzeyine inebilir; pratikte buna gerek yok.

**FEN doğrulaması.** Şunlar kontrol edilir:
- 64 kare ve bilinen taş harfleri;
- her tarafta tek şah;
- birinci ve sekizinci sırada piyon olmaması;
- sırası gelmeyen tarafın şah altında olmaması;
- 0–255 aralığında yarım hamle sayacı.

Rok hakkı yalnız şah ve kale yerindeyse, geçerken alma alanı yalnız gerçekten oynanabilir bir alım varsa kurulur. Bilinen sınır: sıraların uzunluğu tek tek sayılmıyor.

</details>

<details>
<summary><b>Fethedilecek 12 pozisyon, yol haritası ve doğrulama kuyruğu</b></summary>

Bu 12 pozisyon yanlış hüküm değil, henüz yakalanamayan beraberlikler. Hepsinde arama 130–240 düğüm harcayıp derinlik sınırına takılıyor; varyant üzerindeki hiçbir pozisyon materyal ya da kilit analiziyle sertifikalanamıyor.

| gameId | bayrak | n | FEN |
|---|---|---|---|
| GyWj6Ymz | siyah | 8 | `8/4k1p1/4p1Pp/1p1pPp2/pP1P1P1P/P7/8/4K3 b - - 8 63` |
| 3KkCirHD | beyaz | 0 | `8/1p6/4k3/1P1p2p1/1p1P2P1/1P1K4/8/8 w - - 0 46` |
| chikSuUd | siyah | 11 | `4K3/8/6p1/5pP1/5P1k/5P1p/7P/8 b - - 11 59` |
| azjorRHB | siyah | 0 | `8/4k1p1/4p1Pp/1p1pPp2/pP1P1P1P/P4K2/8/8 b - - 0 48` |
| lBSDAx07 | beyaz | 0 | `8/4k3/1p4p1/pP1p1pP1/2pP1P2/P1P1KP2/8/8 w - - 0 49` |
| RwnQJq1k | beyaz | 11 | `8/7p/5p1P/5p1K/5Pp1/6P1/b3k3/8 w - - 11 51` |
| hPiwD75i | beyaz | 7 | `1n6/2Bp4/p1pPp1k1/PpP1Pp1p/1P3P1P/2K5/8/8 w - - 7 53` |
| vcaVIyhj | beyaz | 4 | `8/1p1k4/4p3/1P1pP1p1/1p1P2Pp/1P1K3P/8/8 w - - 4 47` |
| pw3hB0Tp | siyah | 0 | `8/1p2k3/8/1P1p2p1/1p1P2P1/1P3K2/8/8 b - - 0 44` |
| q2K0VFaV | beyaz | 0 | `8/6k1/6p1/p1p1p1P1/P1P1P1p1/6K1/6P1/8 w - a6 0 38` |
| FKr42ZRT | siyah | 13 | `8/8/7p/5p1P/5p1K/5Pp1/6P1/5kb1 b - - 13 63` |
| o2conOyc | beyaz | 1 | `8/2p2kp1/1pPp4/pP1Pp1P1/P3P1p1/6P1/8/5K2 w - - 1 44` |

**Yol haritası**
1. Kilit analizinin bu 12 yapıyı tanıması.
2. Hapsolmuş filleri engel olarak saymak: `1kb5/1p1p4/1P1P4/8/8/4p1p1/4P1P1/5BK1 w - - 0 1` ilk hamlede `DP` olmalı. Bugün bayrak hükmü doğru (`TM`) ve oyun beşli tekrarla berabere bitiyor, ama ölü pozisyon ilk hamlede ilan edilmiyor.
3. `chasolver-positions.txt`'te kaçan 673 beraberliği azaltmak.
4. Oyun geçmişini (beşli tekrar) bayrak aramasına katmak.
5. Chess960 uyarlaması.

**Doğrulama kuyruğu**
- "Kural doğruluğu en yüksek hakem" iddiasının bağımsız doğrulaması.
- Lichess veritabanında 75 hamle kuralı yüzünden berabere bitmesi gereken oyunları saymak.

</details>

<details>
<summary><b>Değişiklik yaparken: kontrol listesi, tuzaklar, Node ile ölçüm</b></summary>

**Kontrol listesi**
1. **Soundness gerekçesi:** Yeni bir sertifikanın matı neden imkânsız kıldığını yazın.
2. **4x ve 1x:** Kullanıcıya sunulan kod 4x yazılır; 1x, aynı davranışın en kısa metni olarak ayrıca güncellenir.
3. **Üç kopya:** Değişiklik `index.html`'e, `BestArbiter.html`'deki `src_x4`'e ve `src_nm`'ye uygulanır; `index.html` ile `src_x4` birebir aynı kalır. 1x'e taşırken `&` ile `&&` farkına dikkat edin.
4. **Sürüm denkliği:** İki sürüm her girdide aynı hükmü vermeli (run both builds).
5. **Regresyon:** Kapatılmış açıklar tablosundaki pozisyonlar yanlış yönde sertifika almamalı.
6. **Soundness ölçümü:** İki `.txt` dosyasında yanlış beraberlik 0, chasolver verisinde found mate 0 olmalı.
7. **Completeness gerilemesi yok:** chasolver verisinde doğru beraberlik ≥ 201.048; `chasolver-positions.txt`'te kaçan beraberlik ≤ 673 kalmalı.
8. **Yeni sertifika ailesi:** Hedefli pozisyonlar üretip chasolver'la çapraz kontrol edin; hazır test pozisyonları bir sertifikanın bütün köşe durumlarını göstermez.

**Tuzaklar**
- **Hazır test pozisyonları yetmez:** Bilinen açıkları hedefli fuzz testi buldu: şahı kenara taşımak, kilitli piyon çiftleri eklemek, iki renkte fil, tahtayı kaydırmak, renkleri çevirmek.
- **Saymayı unutmak:** "Bir taş bu kareye ulaşabilir" demek "bu kare kapatılabilir" demek değildir; tek bir taş aynı anda tek kare kapatır.
- **Negatif BigInt:** `~x` negatif bir sayı üretir; kaydırmadan önce `f&~(…)` ile 64 bite indirin.
- **Sütun taşması:** Yatay kaydırmada maske her zaman şart; tek renge kısıtlanmış kümede çapraz kaydırma maskesiz de doğrudur.
- **Operatör önceliği:** `&&` ve `||`, `|` ile `&`'den düşük önceliklidir; birini dönüştürünce komşularını da kontrol edin.
- **Erken çıkış sağlaması:** Büyüyen kümeler artı, küçülenler (`P`, `Q`) eksi işaretle girer.
- **1x'te `M` yan etkisiz değil:** orada rok haklarını da siliyor, bu yüzden `L` `c`'yi kaydet/geri yaz çiftinde taşımak zorunda. 4x'te `M` `c`'ye dokunmuyor, güncellemeyi arama yapıyor. Bu çiftin bir yarısını diğeri olmadan taşımayın.

**Node ile ölçüm**

```js
const fs = require('fs'), vm = require('vm');
const html = fs.readFileSync('BestArbiter.html', 'utf8');
const blocks = [...html.matchAll(/<script[^>]*>([\s\S]*?)<\/script>/g)].map((m) => m[1]);
const SRC = html.match(/id="src_x4">([\s\S]*?)<\/script>/)[1].trim();   // 1x için src_nm
const chunks = (t) => { const r = []; let d = 0, st = 0; for (let i = 0; i < t.length; i++) { const c = t[i]; if ('([{'.includes(c)) d++; else if (')]}'.includes(c)) d--; else if (c === ',' && d === 0) { r.push(t.slice(st, i)); st = i + 1; } } r.push(t.slice(st)); return r; };
const mirror = (x) => { const y = x.replace(/^H=/, 'Hd=').split('Ht[').join('Hs[').replace(',H(g,d-1)', ',Hd(g,d-1)'); const k = y.lastIndexOf('&&'); return y.slice(0, k + 2) + '(' + y.slice(k + 2, y.length - 1) + '||(HM=1,0)))'; };
const HD = chunks(SRC).filter((c) => c.startsWith('H=')).map(mirror)[0];
const ctx = { Math, BigInt, performance, console }; ctx.window = ctx; vm.createContext(ctx);
vm.runInContext(blocks[2], ctx);
vm.runInContext(SRC + ';' + HD + ';HM=0;Hs={};Ht={};Hn=0', ctx);
ctx.judge('8/8/4k3/3p2p1/1p1P1pP1/1P3P2/8/3K4 b - - 47 92', 99, 50000).code;   // 'DP'
```

`node --stack-size=4000` ile çalıştırın.

</details>
