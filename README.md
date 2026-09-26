# Chess LUX

**English** · [Türkçe](#chess-lux-türkçe)

▶ **[Play in your browser](https://cuneytinann.github.io/Chess-LUX/)** · [Benchmarking page](https://cuneytinann.github.io/Chess-LUX/BestArbiter.html)

**The world's most rule-accurate chess arbiter, in a single 7,984-byte HTML file.**

Chess LUX is [FideLite](https://www.fidelite.art)'s full FIDE arbiter with its lock detector rewritten. FideLite's detector, `l`, recognizes locked structures made of kings and pawns only; a wider version is [on hold](https://www.fidelite.art/#bishops) there, for its bytes, its speed cost and the risk it poses to the engine's guarantee. Chess LUX builds that wider version: with a looser budget, `l` also handles bishops and boxed-in knights, rooks and queens.

Everything else is FideLite's: how each FIDE article is read and applied, the board representation, move generation, the flag-fall search, claims and offers, the fifteen result codes. All of it is documented at **[fidelite.art](https://www.fidelite.art)**. This README covers only what Chess LUX changes.

## At a glance

| | |
|---|---|
| **Base** | FideLite's full arbiter (`L3` rule level), `engine_4x.js` build |
| **What changes** | the lock detector `l`, and how the flag-fall search consults it |
| **Size** | one file, 7,984 bytes: game, clock, interface and arbiter |
| **On Lichess data** | the correct verdict in 201,049 of the 201,060 games chasolver found wrongly decided on time (99.9945%); FideLite: 197,493 (98.2%) |
| **In locked positions** | 51,047 of 51,058 (99.98%); FideLite: 47,491 (93.01%) |
| **False draws** | 0 on the Lichess test positions; 3 in chasolver's constructed positions, all of them already checkmate and impossible to reach in a game |
| **Speed** | flag verdict in a median of 0.1 ms on real games; about 0.4 s on the hardest constructed position |
| **Installation** | none: play online, or download the file and play offline |

## Try it now

1. Open [cuneytinann.github.io/Chess-LUX](https://cuneytinann.github.io/Chess-LUX/), or download `index.html` and open it locally; the tab will read "FideLite.Art".
2. In the dialog, keep the starting position and the time (10 min + 5 s, Fischer increment), or enter your own FEN and choose Fischer increment, Bronstein or simple delay, then press **Play**.
3. Play by clicking or dragging the pieces. Right-drag across the board to mark every square in between, or right-click a single square to mark just that one; a left click clears the marks. ½ claims or offers a draw and ⚐ resigns; when the game ends, ↺ starts a new one.

Any current browser will do: Chrome/Edge 85+, Firefox 98+, Safari 15.4+ (2022 or later). How claims, offers, the counters and the result codes work is explained on [fidelite.art](https://www.fidelite.art/#gameplay).

## What Chess LUX changes

Compared definition by definition with FideLite's [`builds/engine.js`](https://github.com/cuneytinann/FideLite/blob/main/builds/engine.js) and [`builds/engine_4x.js`](https://github.com/cuneytinann/FideLite/blob/main/builds/engine_4x.js):

| definition | FideLite | Chess LUX |
|---|---|---|
| `l` | kings and pawns only; returns at once when any other piece is on the board | also bishops (bishop floods and a bishop layer) and boxed-in knights, rooks and queens; can be asked about one side only (`a`) and can assume a frozen king (`G`) |
| `H` | asks `l()`: is the position dead for both sides? | asks `l(2-g)`: can the opponent still win? |
| `F` | search limits of 20,000 nodes and 15 plies | 50,000 nodes and 99 plies |
| `A`, `D` | an agreement needs both offer bits | `A` also records that each side has moved (`o\|=4<<t`); `D` accepts an agreement only once both have (`o>14`, FIDE 5.2.3), and reads the board (`Z()`) before any claim |
| header | carries a default clock, `U=[600,600]` | no clock; the driver owns it |
| rewrites | | `G` and `M` in both builds, `L` and `H` in 4x: the board copy moves into `M`'s parameter list and the 4x pawn-capture test becomes `d<2&v`; 9 bytes shorter in 4x, the same size in 1x; same perft, same search nodes |

Everything not in the table is FideLite's, byte for byte.

**One example.** In the Lichess game [ijyj0mHa](https://lichess.org/ijyj0mHa#120), White ran out of time in this position and lost:

```
5b2/p7/Pp3k2/1Pp1pBp1/2P1P1P1/5K2/8/8 w - -
```

The pawn wall is permanently locked; Black cannot mate even with White's help, so the correct result is a draw. With a bishop on the board FideLite's `l` does not run, its search reaches the 15-ply limit after 17 nodes, and the verdict is a win. Chess LUX's `l(1)` certifies at the root that Black cannot win: a draw, without any search. Had Black's flag fallen instead, both engines would rightly award White the win, since White does have a mating line.

**The driver.** The interface in `index.html` is Chess LUX's own: a FEN and time-control dialog with Fischer, Bronstein and simple-delay modes; a clock read from `performance.now()` and settled before every action, claims and resignation included; drag and drop, right-click marks, captured material, last-move and check highlights. It runs the 4x build. The rule layer does not depend on it: without an interface the engine is `engine_4x.js`, embedded in `BestArbiter.html`.

**Naming.** Chess LUX follows the naming of FideLite's `L3`: every name `L3` already has keeps it — `N` is `indexOf`, `Q` is `innerHTML`, the pawn sets inside the lock detector are still an adjacent pair (`m`/`n`, since `l` names the detector itself) — and the declarations appear in `L3`'s order, so the two sources can be read side by side. Only what Chess LUX adds carries new names.

## Results on chasolver's data

[chasolver](https://github.com/miguel-ambrona/chasolver) (Miguel Ambrona, [FUN 2022](https://chasolver.org/FUN22-full.pdf)) scanned the Lichess database and found 201,060 games wrongly decided on time; its data and test positions are the yardstick for this project. The two engines differ in the locked-structure class ("Blocked", 51,058 games):

| engine | search | undetected | coverage |
|---|---|---|---|
| FideLite (`l`: kings and pawns) | 20,000 nodes, 15 plies | 3,567 | 93.01% |
| Chess LUX (`l`: + bishops, boxed pieces) | 50,000 nodes, 99 plies | 11 | 99.98% |

The rows compare the two engines as shipped. Their search limits differ too, so the table does not split the gain between `l` and the larger search. Why these games matter under FIDE 6.9 and 5.2.2, and why most servers get them wrong, is explained on fidelite.art: [flag fall](https://www.fidelite.art/#flag), [dead position](https://www.fidelite.art/#dead), [chasolver](https://www.fidelite.art/#chasolver).

## The lock detector

`l` returns a certificate: true means that the side or sides it is asked about cannot mate by any series of legal moves. FideLite's version grows the king floods and pawn cones of a kings-and-pawns board to a fixpoint, then asks whether a pawn can still promote, capture or give check ([line by line](https://www.fidelite.art/#lscan)). Chess LUX keeps that core, which was fuzzed with 5.8 million unique positions and 2,405,996 chasolver queries without a counterexample, and adds three things:

- **Bishops.** Bishop floods grow alongside the king floods and stop at fixed squares. A bishop layer then decides whether a bishop could still take part in a mate: at most one bishop per side and square color, a capture test, and either a region test or a color certificate.
- **Boxed-in knights, rooks and queens.** A piece that cannot leave its box counts as a fixed blocker; its first escape fails the certificate. A boxed piece the enemy king or bishop can reach is dropped instead: it can never capture anything, so all that can happen to it is being captured.
- **One-sided and frozen-king queries.** `l(a,G)` can ask about one side's winning chances only, which is how the search uses it (`l(2-g)`), and it can assume that a king cannot move, verifying the assumption at the end.

FideLite keeps this extension on hold partly because bishops put its one proven property at risk: every position it calls dead must really be dead. Chess LUX answers that with measurement rather than proof: zero false draws on chasolver's data, targeted fuzzing of the bishop layer, and a regression table of the three holes it found and closed (see the appendix). Neither `l` nor the search depends on the castling rules, so both carry over to Chess960.

## Verify it yourself: BestArbiter.html

`BestArbiter.html` is a benchmarking page that lets you put the claims to the test yourself; the verdicts come from the very same engine that runs the game.

1. Open [BestArbiter.html](https://cuneytinann.github.io/Chess-LUX/BestArbiter.html) online, or download it and open it locally.
2. Download `chasolver-data-until-08-2026.csv` (or one of the two `.txt` files) from this repository and drag it onto the page.
3. Press **judge the files**. **run both builds** runs the two builds of the selected engine back to back and shows that their verdicts match.

The engine menu also offers FideLite's `L3`, fast and normal. Selecting it sets the ply and node fields to `L3`'s own limits (15 and 20,000) unless you have changed them, so both engines can be judged on the same files.

Expected results with the default settings (99 plies, 50,000 nodes):

| file | expected |
|---|---|
| `chasolver-data-until-08-2026.csv` | 201,049 correct draws, 11 without a verdict, 0 contradictions |
| `chasolver-missed-draws-08-2026.csv` | the same 11 positions, all without a verdict |
| `chasolver-positions.txt` | 3 false draws across the 1,945 positions the opponent can win, all of them already checkmate (see the verification details); 867 of the 1,469 draws proven |
| `chasolver-lichess.txt` | 0 false draws |
| between the two engine builds | 0 differing verdicts |

Large files can take a few minutes; the two `.txt` sets, where most positions admit no proof, take the longest. To try a single position, paste its FEN into the box on the page.

## Files

| file | size | contents | source |
|---|---|---|---|
| `index.html` | 7,984 B | game and arbiter | this project |
| `BestArbiter.html` | 81,562 B | benchmarking page with both builds of Chess LUX and of FideLite `L3` embedded | this project |
| `chasolver-data-until-08-2026.csv` | 22.3 MB | the 201,060 wrongly decided timeouts chasolver found on Lichess (through August 2026) | [chasolver.org](https://chasolver.org/unfair-games) |
| `chasolver-positions.txt` | 152 KB | chasolver's 3,414 labeled challenging test positions | [chasolver `tests/positions.txt`](https://github.com/miguel-ambrona/chasolver/blob/main/tests/positions.txt) (MIT) |
| `chasolver-lichess.txt` | 3.3 MB | 65,536 labeled Lichess positions | [chasolver `tests/lichess.txt`](https://github.com/miguel-ambrona/chasolver/blob/main/tests/lichess.txt) (MIT) |
| `chasolver-missed-draws-08-2026.csv` | 2 KB | the 11 games of `chasolver-data-until-08-2026.csv` the arbiter leaves without a verdict at 99 plies and 50,000 nodes, as BestArbiter exports them | this project |
| `chasolver-missed-draw-positions.csv` | 75 KB | the 602 draws in `chasolver-positions.txt` the arbiter misses at the same settings, as BestArbiter exports them | this project |
| `LICENSE` | | MIT License | this project |

The three chasolver source files (`chasolver-data-until-08-2026.csv`, `chasolver-positions.txt`, `chasolver-lichess.txt`) are Miguel Ambrona's work; the two `chasolver-missed-*` files are this project's exports from them. They are included here with attribution so that the measurements can be reproduced; the originals are linked in the source column.

## Counterexamples, contributions and license

**LUX: the fairest chess in the universe.** I stand by that claim, and if there's anything better out there, bring it on :)

Jokes aside, the claim is open to challenge. If you find a winnable position where Chess LUX awards a draw, or one that makes BestArbiter report **FALSE TM** or **found mate**, please share it by opening an issue on GitHub. Even a single counterexample is valuable to us.

Chess LUX is released under the MIT License; see the `LICENSE` file for details.

## Acknowledgments and credits

- **[FideLite](https://github.com/cuneytinann/FideLite):** the engine Chess LUX is built on; its rules, functions and design are documented at [fidelite.art](https://www.fidelite.art).
- **[chasolver](https://github.com/miguel-ambrona/chasolver), Miguel Ambrona:** unwinnability analysis, test positions and data on wrongly decided Lichess games ([chasolver.org](https://chasolver.org)). I developed the algorithm independently, but without his work I could never have taken it this far.
- **[Lichess open database](https://database.lichess.org/)** (CC0).

---

## Technical appendix

This section is for anyone who wants to modify `l` or reproduce the measurements. How the rest of the engine works is on [fidelite.art](https://www.fidelite.art/#engine).

<details>
<summary><b>Sizes, builds and priorities</b></summary>

Bytes per definition, FideLite against Chess LUX. Definitions not listed are identical; the header is 9 bytes shorter in both builds (no default clock).

| definition | FideLite 1x | Chess LUX 1x | FideLite 4x | Chess LUX 4x |
|---|---|---|---|---|
| `l` | 444 | 1,204 | 477 | 1,321 |
| `H` | 219 | 222 | 258 | 252 |
| `F` | 45 | 45 | 45 | 45 |
| `A` | 52 | 60 | 67 | 75 |
| `D` | 36 | 42 | 36 | 42 |
| `G` | 249 | 249 | 252 | 251 |
| `L` | 104 | 104 | 113 | 104 |
| `M` | 234 | 234 | 210 | 219 |
| **engine file** | **1,847** | **2,615** | **1,945** | **2,787** |

In 1x, `G` and `M` keep their size: `G` only swaps the operands of one `&`, and `M` makes its board copy in the parameter list instead of the body. `F` differs only in its two limits.

`index.html` carries the 4x definitions (`G V L C M I Im H l`) byte-for-byte identical to `engine_4x.js` in `BestArbiter.html`; `Z D F A` are interface versions that report the result as a text code.

**1x and 4x.** As in FideLite, everything shipped to users is written in 4x style: bytes come first, but a few extra bytes are spent whenever they buy a multiplicative speedup. 1x is the shortest source with identical behavior and is offered only as an experimental option on the benchmarking page. The two builds must return the same verdict on every input; on real games 4x stays under 100 ms, whereas 1x can take seconds. The optimizations the two projects share are described under [FideLite's `engine_4x`](https://www.fidelite.art/#speed).

**Priorities.** (1) Soundness: the arbiter never awards a false draw. (2) Bytes. (3) Speed. 100% completeness, meaning never missing a draw, is the goal but is not claimed.

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
- **Occupancy:** `M` all White men, `D` all Black men, `F`/`g` the boxed-in knights, rooks and queens of each side; `m`/`n` are the uncapturable pawns of each side within one iteration, and `I`/`J` the two sides' blocked-square sets.

**Fixed point.** Each iteration grows or shrinks the sets:
- **Blocked pawns:** `P`/`Q` only ever shrink; what remains are the pawns the enemy king can never capture and whose way forward is permanently blocked.
- **Pawn cones:** `C`/`Z`, the squares the pawns can advance to.
- **King floods:** `K`/`L`, the squares the kings can reach; they only grow. An enemy pawn touched by the flood counts as capturable via `r`/`q`.
- **Bishop floods:** `R`/`c`, the squares the bishops can reach; they stop at fixed squares.
- **Boxed-in pieces:** A knight, rook or queen whose neighborhood opens up records its escape in `Y`; the first escape fails that level. A boxed piece the enemy king or bishop can reach is not an escape: it can never capture anything, so all that can happen to it is being captured. Its square is dropped from `F`/`g` and treated as empty from then on. A pawn that could take it fails the level through the pawn-capture condition.

In 4x the loop stops as soon as the `z` checksum (`C+Z+K+L+r+q-P-Q+R+c-F-g`) stops changing. The twelve sets in it only ever move one way — `P`, `Q`, `F` and `g` shrink and enter with a minus sign, the rest grow — so the checksum moves on every pass that changes anything, and the loop cannot stop early. The ten flood and pawn sets can change at most 640 bits, `F` and `g` only the squares of the knights, rooks and queens, so fewer than 768 passes ever change anything; 1x always runs 768. In practice 4x needs a median of 11 passes on the chasolver data and never needed more than 69 in fuzzing. The bound relies on `&f` keeping `C` and `q` inside 64 bits: without it, bits pushed past the eighth rank keep the checksum moving and the recursion never ends. 1x needs no such mask, since it stops after 768 passes anyway and those bits change no verdict.

**Lock conditions.**
- no promotion;
- no pawn captures and no pawn checks;
- the king assumed frozen really is immobile;
- no piece has escaped its box (`!Y`);
- if bishops are present, the bishop layer holds.

**Bishop layer.** The king floods must not intersect. For each side and color there may be at most one bishop, and the CAPTURE test must pass: the bishop must not touch an enemy pawn, bishop or pawn cone (a boxed piece it can reach is dropped instead, see above). Beyond that, one of two conditions is required: either REGION (the bishop never touches the defending king's flood) or, with all of the defender's pawns blocked, the COLOR certificate.

**COLOR certificate.** A bishop can deliver mate only by checking along a diagonal on a square of its own color. The certificate shows that on every such square within the defending king's reach, an escape square remains even when the king is in check.
- **Hard blockers:** The defender's fixed pieces (a dropped boxed piece included: while it stands there it seals its square), the attacker's pawn attacks and the squares adjacent to the attacking king. They can seal any number of squares at once.
- **Soft blocker:** The defender's single bishop of that color, which can seal at most one square at a time. That is why two non-hard squares, or one free square, guarantee an escape.

1x source:

```js
|!(o?B^Q:W^P)&(!(a&1+o)|!(i=o?c:R,j=f^((o?D^v^_:M^V^$)|(o?S(C)<<8n|N(K):S(Z)>>8n|N(L))|t&~I),u=j&~i,s=j&I,A=u&I,m&I&~(E(9n)&E(7n)|O(u)|j>>8n&j<<8n|j>>1n&j<<1n&x&y|(j>>8n|j<<8n)&S(j))))
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

**Speed build.** Two of `l`'s 4x optimizations come from FideLite's `engine_4x`: the checksum stop, here over twelve sets instead of eight, and `&f` on `C` and `q`. The rest belong to the parts Chess LUX adds. Before the fixpoint, two cheap checks fail fast: a boxed-in knight, rook or queen with a step square its own pawns and boxed pieces do not hold, and, at the frozen-king levels, a king with a neighboring square that is neither held by its own men nor covered by an enemy pawn. `T` opens with `s&&`, so a side without a bishop returns at once; `S` shifts where FideLite adds; and in the bishop layer `&`/`|` become `&&`/`||`, so the rest is skipped once the outcome is decided. One cost has no counterpart: FideLite's `l` returns at the first piece that is neither a king nor a pawn, whereas Chess LUX's `l` has to examine such positions; the fast-fail checks keep that cost down.

**Regression positions.** An earlier version of the bishop layer certified the positions below wrongly; `l(3)/l(2)/l(1)` must return:

| hole | position | expected |
|---|---|---|
| `T` was mistakenly written with XOR | `6k1/8/8/p1p1p1p1/PpPpPpPp/1P1P1P1P/2B5/2B3K1 w - - 0 1` | 0/0/0 |
| the capture test ignored advancing pawns | `4k3/7p/4p1pP/3pP1P1/2pPp3/1pP5/pP2P3/K5B1 w - - 0 1` | 0/0/0 |
| same family | `4k3/7p/4p1pP/3pP1P1/2pP2p1/1pP5/pP4P1/K5B1 w - - 0 1` | 0/0/0 |
| COLOR overlooked a king on the edge (…Bd1#) | `8/1k6/p1p1p1p1/P1p1P1P1/K1p1p1p1/P1P1P1P1/4b3/8 b - - 0 1` | 0/1/0 |
| same family (…Bc6#) | `b7/k7/p3p3/P1p1p3/K1p1p1p1/P1P1P1P1/8/8 b - - 0 1` | 0/1/0 |
| same family (Bd8#) | `8/4B3/p1p1p1p1/k1P1P1P1/p1P1p1p1/P1P1P1P1/1K6/8 w - - 0 1` | 0/0/1 |

All three lay in the bishop layer, none in FideLite's kings-and-pawns core.

</details>

<details>
<summary><b>Search: what changes in H and F</b></summary>

How `H` works — the order in which a node is proven, the 75-move cutoff, the repetition cut, castling rights along the line — is FideLite's and is explained on fidelite.art ([line by line](https://www.fidelite.art/#flow), [the budget](https://www.fidelite.art/#hbudget)). Chess LUX changes two things.

**The lock gate is one-sided.** FideLite's `H` asks `l()` whether the position is dead for both sides. Chess LUX's `H` asks `l(2-g)` whether `g`'s opponent can still win, which certifies more nodes: in the ijyj0mHa example the whole verdict comes from this gate at the root.

**The limits.** If either limit runs out, the branch returns false and the verdict defaults to a win. Because every line must be proven, the first line that fails ends the whole search: most positions without a proof stop the first time a line reaches 99 plies, usually after a few hundred nodes (the 11 missed chasolver draws stop after 130–240). The 50,000-node budget runs out only rarely: of every seventh position in `chasolver-positions.txt` (488 positions), 315 stopped at the depth limit and 13 at the budget.

| where | budget | depth |
|---|---|---|
| FideLite (`F`) | 20,000 nodes | 15 plies |
| `index.html` | 50,000 nodes | 99 plies |
| `F` in the embedded engine | 50,000 nodes | 99 plies |
| benchmarking page | adjustable, default 50,000 | adjustable, default 99 |

</details>

<details>
<summary><b>Verification details and measurements</b></summary>

| measurement | result |
|---|---|
| chasolver data: proven draws | 201,049 / 201,060 (99.9945%) |
| locked-structure class: proven draws | 51,047 / 51,058 (99.98%) |
| no verdict | 11; all in the locked-structure class, and in each the search runs out of depth |
| mates found by the search (contradicting the reference) | 0 |
| `chasolver-positions.txt`: false draws across the 1,945 positions the opponent can win | 3, all positions that are already checkmate by a double check from two same-coloured bishops, which no legal move can produce; `Im` answers before the mate test. In a game the arbiter ends such a position as checkmate before any flag can fall |
| `chasolver-positions.txt`: proven / missed draws (1,469 targets) | 867 / 602 |
| `chasolver-lichess.txt`: false draws | 0 |
| differing verdicts between 1x and 4x | 0 |

**Independent cross-checks**
- **Perft:** Identical to chess.js 1.4.0 on 19 tricky positions at depths 3–4, and identical to FideLite on the standard positions.
- **Random games:** Across 450 games and 123,453 plies, the legal move set, board, castling rights and halfmove clock matched chess.js at every step.
- **FEN:** On 14,005 FENs, positions set up through `index.html`'s FEN path produced the same legal moves as chess.js.
- **`l`:** Mutation-based fuzzing on about 375,000 locked positions yielded bit-for-bit identical verdicts from the original and the streamlined detector.

**Speed**
- **Real games:** On a 10% sample of the chasolver data (20,106 positions), median 0.1 ms, maximum 60 ms.
- **Hardest position:** `B6b/pr6/8/8/8/4p1pp/P3Pp1p/2b2K1k b - -`. It uses 20,107 nodes and returns the correct verdict (WT); about 0.4 s in a browser on a mid-range laptop. A position without a proof usually stops at the depth limit after a few hundred nodes; the full 50,000 nodes, roughly a second, are spent only when the budget itself runs out.
- **What drives the time:** The cost per node. Each node involves legal move generation, a board copy and a call to `l`.
- **Native code:** Ported to C++ or Rust with 64-bit integers, or compiled to WebAssembly, the engine would shed the memory-allocation overhead of BigInt. Typical positions could drop to microseconds and the hardest one to milliseconds; in practice, though, there is no need.

**FEN validation.** The dialog checks:
- 64 squares and valid piece letters;
- exactly one king per side;
- no pawns on the first or eighth rank;
- the side not to move is not in check;
- a halfmove clock between 0 and 255.

Castling rights are set only when the king and rook stand on their original squares, and the en passant square only when a capture is actually possible. Known limitation: rank lengths are not checked individually.

</details>

<details>
<summary><b>Eleven positions left to conquer, roadmap and verification queue</b></summary>

These 11 positions (also in `chasolver-missed-draws-08-2026.csv`) are not wrong verdicts but draws that have yet to be detected. In each of them the search spends 130–240 nodes before hitting the depth limit; no position along the line can be certified by the material or lock analysis.

| gameId | flagged | n | FEN |
|---|---|---|---|
| GyWj6Ymz | Black | 8 | `8/4k1p1/4p1Pp/1p1pPp2/pP1P1P1P/P7/8/4K3 b - - 8 63` |
| 3KkCirHD | White | 0 | `8/1p6/4k3/1P1p2p1/1p1P2P1/1P1K4/8/8 w - - 0 46` |
| chikSuUd | Black | 11 | `4K3/8/6p1/5pP1/5P1k/5P1p/7P/8 b - - 11 59` |
| azjorRHB | Black | 0 | `8/4k1p1/4p1Pp/1p1pPp2/pP1P1P1P/P4K2/8/8 b - - 0 48` |
| lBSDAx07 | White | 0 | `8/4k3/1p4p1/pP1p1pP1/2pP1P2/P1P1KP2/8/8 w - - 0 49` |
| RwnQJq1k | White | 11 | `8/7p/5p1P/5p1K/5Pp1/6P1/b3k3/8 w - - 11 51` |
| vcaVIyhj | White | 4 | `8/1p1k4/4p3/1P1pP1p1/1p1P2Pp/1P1K3P/8/8 w - - 4 47` |
| pw3hB0Tp | Black | 0 | `8/1p2k3/8/1P1p2p1/1p1P2P1/1P3K2/8/8 b - - 0 44` |
| q2K0VFaV | White | 0 | `8/6k1/6p1/p1p1p1P1/P1P1P1p1/6K1/6P1/8 w - a6 0 38` |
| FKr42ZRT | Black | 13 | `8/8/7p/5p1P/5p1K/5Pp1/6P1/5kb1 b - - 13 63` |
| o2conOyc | White | 1 | `8/2p2kp1/1pPp4/pP1Pp1P1/P3P1p1/6P1/8/5K2 w - - 1 44` |

**Roadmap**
1. Teach the lock analysis to recognize these 11 structures. (A twelfth, hPiwD75i, is solved: a boxed knight the bishop can capture does not break the lock.)
2. Treat trapped bishops as blockers: `1kb5/1p1p4/1P1P4/8/8/4p1p1/4P1P1/5BK1 w - - 0 1` should be `DP` on the first move. Today the flag verdict is correct (`TM`) and the game ends by fivefold repetition, but the dead position is not declared on move one.
3. Reduce the 602 missed draws in `chasolver-positions.txt` (listed in `chasolver-missed-draw-positions.csv`).
4. Bring the game history (fivefold repetition) into the flag search.
5. Adapt the engine to Chess960.

**Verification queue**
- Independent verification of the "most rule-accurate arbiter" claim.
- Counting the Lichess games that should have been drawn under the 75-move rule.

</details>

<details>
<summary><b>Making changes: checklist, pitfalls, measuring with Node</b></summary>

**Checklist**
1. **FideLite first:** Outside the definitions in [What Chess LUX changes](#what-chess-lux-changes) the engine is FideLite's; a change there belongs in FideLite and is carried over from it, so that table stays true.
2. **Soundness rationale:** Write down why a new certificate makes mate impossible.
3. **4x and 1x:** Code shipped to users is written in 4x style; 1x is updated separately as the shortest source with identical behavior.
4. **Three copies:** Apply every change to `index.html` and to both `src_x4` and `src_nm` in `BestArbiter.html`; `index.html` and `src_x4` must stay identical. When porting to 1x, mind the difference between `&` and `&&`.
5. **Build equivalence:** The two builds must return the same verdict on every input (run both builds).
6. **Regression:** The positions in the regression table must never be certified in the wrong direction.
7. **Soundness check:** 0 false draws on `chasolver-lichess.txt`, none on `chasolver-positions.txt` beyond the three already-checkmated positions, and 0 found mates on the chasolver data.
8. **No completeness regression:** At least 201,049 correct draws on the chasolver data and at most 602 missed draws on `chasolver-positions.txt`.
9. **New certificate families:** Generate targeted positions and cross-check them against chasolver; ready-made test sets do not cover every corner case of a certificate.

**Pitfalls**
- **Ready-made test sets are not enough:** The known holes were found by targeted fuzzing: moving the king to the edge, adding locked pawn pairs, bishops on both colors, shifting the board, swapping colors.
- **Forgetting to count:** "A piece can reach this square" does not mean "this square can be sealed"; a single piece seals only one square at a time.
- **Negative BigInt:** `~x` yields a negative number; mask it down to 64 bits with `f&~(…)` before shifting.
- **File wraparound:** Horizontal shifts always need a mask; on a set restricted to one color, diagonal shifts are correct even without one.
- **Operator precedence:** `&&` and `||` bind more loosely than `|` and `&`; when you convert one, check its neighbors too.
- **Early-exit checksum:** Growing sets enter with a plus sign, shrinking ones (`P`, `Q`, `F`, `g`) with a minus sign.
- **`M` is not side-effect-free in 1x:** there it also clears castling rights, so `L` must carry `c` through its save/restore pair. In 4x `M` leaves `c` alone and the search updates it instead. Never port one half of that pair without the other.

**Measuring with Node**

```js
const fs = require('fs'), vm = require('vm');
const html = fs.readFileSync('BestArbiter.html', 'utf8');
const blocks = [...html.matchAll(/<script[^>]*>([\s\S]*?)<\/script>/g)].map((m) => m[1]);
const SRC = html.match(/id="src_x4">([\s\S]*?)<\/script>/)[1].trim();   // src_nm for 1x
const chunks = (t) => { const r = []; let d = 0, st = 0; for (let i = 0; i < t.length; i++) { const c = t[i]; if ('([{'.includes(c)) d++; else if (')]}'.includes(c)) d--; else if (c === ',' && d === 0) { r.push(t.slice(st, i)); st = i + 1; } } r.push(t.slice(st)); return r; };
const mirror = (x) => { const y = x.replace(/^H=/, 'Hd=').split('Ht[').join('Hs[').replace(',H(g,d-1)', ',Hd(g,d-1)'); if (y.endsWith('&v)')) return y.slice(0, -3) + '&&(v||(HM=1,0)))'; const k = y.lastIndexOf('&&(v||'); return y.slice(0, k + 2) + '(' + y.slice(k + 2, y.length - 1) + '||(HM=1,0)))'; };
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

**7.984 baytlık tek bir HTML dosyasında, dünyanın kural doğruluğu en yüksek satranç hakemi.**

Chess LUX, [FideLite](https://www.fidelite.art/tr)'ın tam FIDE hakeminin kilit dedektörü yeniden yazılmış hâli. FideLite'ın dedektörü `l` yalnızca şah ve piyonlardan oluşan kilitli yapıları tanıyor; daha geniş bir sürümü orada baytı, hız maliyeti ve motorun güvencesine getirdiği risk yüzünden [askıda](https://www.fidelite.art/tr#bishops) duruyor. Chess LUX o sürümü kuruyor: daha gevşek bir bütçeyle `l`, filleri ve kutulu at, kale ve vezirleri de hesaba katıyor.

Geri kalan her şey FideLite'ın: her FIDE maddesinin nasıl okunup uygulandığı, tahta temsili, hamle üretimi, bayrak araması, talep ve teklifler, on beş sonuç kodu. Hepsinin anlatımı **[fidelite.art](https://www.fidelite.art/tr)**'ta. Bu README yalnızca Chess LUX'ın değiştirdiklerini anlatıyor.

## Bir bakışta

| | |
|---|---|
| **Temel** | FideLite'ın tam hakemi (`L3` kural seviyesi), `engine_4x.js` sürümü |
| **Değişen** | kilit dedektörü `l` ve bayrak aramasının ona nasıl danıştığı |
| **Boyut** | tek dosya, 7.984 bayt: oyun, saat, arayüz ve hakem bir arada |
| **Lichess verisiyle** | chasolver'ın bulduğu, süreden haksız sonuçlanmış 201.060 oyunun 201.049'unda doğru hüküm (%99,9945); FideLite: 197.493 (%98,2) |
| **Kilitli yapılarda** | 51.058 oyunun 51.047'sinde (%99,98); FideLite: 47.491 (%93,01) |
| **Yanlış beraberlik** | Lichess test pozisyonlarında 0; chasolver'ın kurgu pozisyonlarında 3, üçü de zaten mat ve oyunda oluşamayan pozisyonlar |
| **Hız** | bayrak hükmü gerçek oyunlarda medyan 0,1 ms; en zor kurgulanmış pozisyonda yaklaşık 0,4 s |
| **Kurulum** | yok: çevrimiçi oynayın ya da dosyayı indirip internetsiz oynayın |

## Hemen dene

1. [cuneytinann.github.io/Chess-LUX](https://cuneytinann.github.io/Chess-LUX/) adresini açın ya da `index.html`'i indirip tarayıcıda açın. Sekmede "FideLite.Art" başlığı görünür.
2. Açılan pencerede başlangıç pozisyonunu ve süreyi (10 dk + 5 sn, Fischer artırımı) olduğu gibi bırakın ya da kendi FEN'inizi girip Fischer artırımı, Bronstein ya da simple delay seçin, ardından **Play**'e basın.
3. Taşları tıklayarak ya da sürükleyerek oynayın. Sağ tuşla tahtada sürüklerseniz aradaki bütün kareler işaretlenir, tek kareye sağ tıklarsanız yalnız o kare işaretlenir; sol tıklama işaretleri siler. ½ beraberlik talebi ve teklifi, ⚐ terk içindir; oyun bitince ↺ yeni bir oyun açar.

Güncel bir tarayıcı yeterli: Chrome/Edge 85+, Firefox 98+, Safari 15.4+ (2022 ve sonrası). Talep ve tekliflerin, sayaçların ve sonuç kodlarının nasıl işlediği [fidelite.art](https://www.fidelite.art/tr#gameplay)'ta anlatılıyor.

## Chess LUX'ın değiştirdikleri

FideLite'ın [`builds/engine.js`](https://github.com/cuneytinann/FideLite/blob/main/builds/engine.js) ve [`builds/engine_4x.js`](https://github.com/cuneytinann/FideLite/blob/main/builds/engine_4x.js) dosyalarıyla tanım tanım karşılaştırınca:

| tanım | FideLite | Chess LUX |
|---|---|---|
| `l` | yalnız şah ve piyon; tahtada başka bir taş varsa hemen döner | filleri (fil selleri ve fil katmanı) ve kutulu at, kale ve vezirleri de kapsar; tek bir taraf için sorulabilir (`a`) ve donmuş bir şah varsayabilir (`G`) |
| `H` | `l()`'ye sorar: pozisyon iki taraf için de ölü mü? | `l(2-g)`'ye sorar: rakip hâlâ kazanabilir mi? |
| `F` | 20.000 düğüm ve 15 yarım hamle arama sınırı | 50.000 düğüm ve 99 yarım hamle |
| `A`, `D` | anlaşma için iki teklif biti yeter | `A` her tarafın hamle yaptığını da kaydeder (`o\|=4<<t`); `D` anlaşmayı ancak ikisi de hamle yaptıktan sonra kabul eder (`o>14`, FIDE 5.2.3) ve herhangi bir talepten önce tahtayı okur (`Z()`) |
| başlık | varsayılan bir saat taşır, `U=[600,600]` | saat yok; saat sürücünün |
| yeniden yazımlar | | iki sürümde `G` ve `M`, 4x'te `L` ve `H`: tahta kopyası `M`'nin parametre listesine geçer, 4x'te piyon alım testi `d<2&v` olur; 4x'te 9 bayt kısa, 1x'te boy aynı; perft ve arama düğümleri aynı |

Tabloda olmayan her şey bayt bayt FideLite'ın.

**Bir örnek.** Lichess'te oynanan [ijyj0mHa](https://lichess.org/ijyj0mHa#120) oyununda beyazın bu pozisyonda süresi bitti ve oyun beyazın yenilgisiyle sonuçlandı:

```
5b2/p7/Pp3k2/1Pp1pBp1/2P1P1P1/5K2/8/8 w - -
```

Piyon duvarı kalıcı olarak kilitli; siyah, beyaz yardım etse bile mat edemez, doğru sonuç beraberlik. Tahtada fil olduğu için FideLite'ın `l`'si çalışmıyor, araması 17 düğümde 15 yarım hamle sınırına dayanıyor ve hüküm galibiyet oluyor. Chess LUX'ın `l(1)`'i siyahın kazanamayacağını daha kökte belgeliyor: arama hiç yapılmadan beraberlik. Aynı pozisyonda süresi biten siyah olsaydı iki motor da haklı olarak beyazın galibiyetini verirdi, çünkü beyazın mat edebileceği bir yol var.

**Sürücü.** `index.html`'in arayüzü Chess LUX'ın kendisinin: Fischer, Bronstein ve simple delay modlarıyla bir FEN ve süre penceresi; `performance.now()` ile okunan ve talep ile terk dahil her eylemden önce kapatılan bir saat; sürükle-bırak, sağ tık işaretleri, alınan taşlar, son hamle ve şah vurguları. 4x sürümüyle çalışıyor. Kural katmanı ona bağlı değil: arayüzsüz hâli, `BestArbiter.html` içinde `engine_4x.js` adıyla duruyor.

**Adlandırma.** Chess LUX, FideLite'ın `L3` adlandırmasını izler: `L3`'te zaten bulunan her ad korunur — `N` `indexOf`, `Q` `innerHTML`, kilit dedektörünün içindeki piyon kümeleri hâlâ bitişik bir çift (`m`/`n`; `l` dedektörün kendi adı) — ve tanımlar `L3`'teki sırayla gelir, böylece iki kaynak yan yana okunabilir. Yalnızca Chess LUX'ın eklediği şeyler yeni ad taşır.

## chasolver verisinde sonuçlar

[chasolver](https://github.com/miguel-ambrona/chasolver) (Miguel Ambrona, [FUN 2022](https://chasolver.org/FUN22-full.pdf)) Lichess veritabanını tarayıp süreden haksız sonuçlanmış 201.060 oyun buldu; verisi ve test pozisyonları bu projenin doğruluk ölçütü. İki motor kilitli yapı sınıfında ("Blocked", 51.058 oyun) ayrışıyor:

| motor | arama | yakalanamayan | kapsama |
|---|---|---|---|
| FideLite (`l`: şah ve piyon) | 20.000 düğüm, 15 yarım hamle | 3.567 | %93,01 |
| Chess LUX (`l`: + filler, kutulu taşlar) | 50.000 düğüm, 99 yarım hamle | 11 | %99,98 |

Satırlar iki motoru yayımlandıkları hâliyle karşılaştırıyor. Arama sınırları da farklı olduğu için tablo kazancı `l` ile daha geniş arama arasında bölüştürmüyor. Bu oyunların FIDE 6.9 ve 5.2.2 açısından neden önemli olduğu ve çoğu sunucunun onları neden yanlış sonuçlandırdığı fidelite.art'ta anlatılıyor: [süre bitimi](https://www.fidelite.art/tr#flag), [ölü pozisyon](https://www.fidelite.art/tr#dead), [chasolver](https://www.fidelite.art/tr#chasolver).

## Kilit dedektörü

`l` bir sertifika döndürür: doğru dönmesi, sorulan tarafın ya da tarafların hiçbir legal hamle dizisiyle mat edemeyeceği anlamına gelir. FideLite'ın sürümü şah ve piyonlardan oluşan bir tahtada şah sellerini ve piyon konilerini sabit noktaya kadar büyütür, ardından bir piyonun hâlâ terfi edip edemeyeceğini, alım yapıp yapamayacağını ya da şah çekip çekemeyeceğini sorar ([satır satır](https://www.fidelite.art/tr#lscan)). Chess LUX bu çekirdeği korur — 5,8 milyon benzersiz pozisyon ve 2.405.996 chasolver sorgusuyla fuzz testinden geçti, karşı örnek çıkmadı — ve üç şey ekler:

- **Filler.** Fil selleri şah selleriyle birlikte büyür ve sabit karelerde durur. Ardından bir fil katmanı, bir filin hâlâ bir mata katılıp katılamayacağına karar verir: her taraf ve kare rengi için en fazla bir fil, bir alım testi ve bir bölge testi ya da renk sertifikası.
- **Kutulu at, kale ve vezirler.** Kutusundan çıkamayan bir taş sabit bir engel sayılır; ilk kaçışı sertifikayı düşürür. Rakip şahın ya da filin ulaşabildiği kutulu taş ise hesaptan çıkar: hiçbir şey alamaz, başına gelebilecek tek şey alınmaktır.
- **Tek taraflı ve donmuş şahlı sorular.** `l(a,G)` yalnız bir tarafın kazanma şansını sorabilir — arama onu böyle kullanır (`l(2-g)`) — ve bir şahın hareket edemediğini varsayıp bu varsayımı sonda doğrulayabilir.

FideLite bu genişletmeyi kısmen, fillerin motorun tek ispatlı özelliğini riske atmasından dolayı askıda tutuyor: ölü denen her pozisyon gerçekten ölü olmalı. Chess LUX buna ispat yerine ölçümle cevap veriyor: chasolver verisinde sıfır yanlış beraberlik, fil katmanına yönelik hedefli fuzz testi ve bulup kapattığı üç açığın regresyon tablosu (bkz. teknik ek). Ne `l` ne de arama rok kuralına bağlı; ikisi de Chess960'a taşınabilir.

## Kendin doğrula: BestArbiter.html

`BestArbiter.html`, iddiaları kendiniz sınayabilmeniz için hazırlanmış bir ölçüm sayfası; hükmü oyundaki motorun kendisi verir.

1. [BestArbiter.html](https://cuneytinann.github.io/Chess-LUX/BestArbiter.html) sayfasını çevrimiçi açın ya da dosyayı indirip tarayıcıda açın.
2. Bu depodan `chasolver-data-until-08-2026.csv`'yi (ya da iki `.txt` dosyasından birini) indirip sayfaya sürükleyin.
3. **dosyalara hüküm ver** düğmesine basın (İngilizce arayüzde **judge the files**). **iki sürümü de koştur** (**run both builds**) seçili motorun iki sürümünü art arda çalıştırıp hükümlerin aynı olduğunu gösterir.

Motor menüsünde FideLite'ın `L3`'ü de hızlı ve normal olarak var. Seçildiğinde ply ve düğüm alanları, siz değiştirmediyseniz, `L3`'ün kendi sınırlarına (15 ve 20.000) geçer; böylece iki motor aynı dosyalarla yargılanabilir.

Varsayılan ayarlarla (99 yarım hamle, 50.000 düğüm) beklenen sonuçlar:

| dosya | beklenen |
|---|---|
| `chasolver-data-until-08-2026.csv` | 201.049 doğru beraberlik, 11 hükümsüz, 0 çelişki |
| `chasolver-missed-draws-08-2026.csv` | aynı 11 pozisyon, hepsi hükümsüz |
| `chasolver-positions.txt` | rakibin kazanabildiği 1.945 pozisyonda 3 yanlış beraberlik, üçü de zaten mat (bkz. doğrulama ayrıntıları); 1.469 beraberliğin 867'si kanıtlandı |
| `chasolver-lichess.txt` | 0 yanlış beraberlik |
| iki motor sürümü arasında | 0 farklı hüküm |

Büyük dosyalar birkaç dakika sürebilir; pozisyonların çoğunun kanıt taşımadığı iki `.txt` kümesi en uzun sürenler. Tek bir pozisyonu denemek için FEN'ini sayfadaki kutuya yapıştırmanız yeterli.

## Dosyalar

| dosya | boyut | içerik | kaynak |
|---|---|---|---|
| `index.html` | 7.984 B | oyun ve hakem | bu proje |
| `BestArbiter.html` | 81.562 B | Chess LUX'ın ve FideLite `L3`'ün ikişer sürümünü gömülü taşıyan ölçüm sayfası | bu proje |
| `chasolver-data-until-08-2026.csv` | 22,3 MB | chasolver'ın Lichess'te bulduğu, süreden haksız sonuçlanmış 201.060 oyun (Ağustos 2026'ya kadar) | [chasolver.org](https://chasolver.org/unfair-games) |
| `chasolver-positions.txt` | 152 KB | chasolver'ın etiketli 3.414 zorlu test pozisyonu | [chasolver `tests/positions.txt`](https://github.com/miguel-ambrona/chasolver/blob/main/tests/positions.txt) (MIT) |
| `chasolver-lichess.txt` | 3,3 MB | etiketli 65.536 Lichess pozisyonu | [chasolver `tests/lichess.txt`](https://github.com/miguel-ambrona/chasolver/blob/main/tests/lichess.txt) (MIT) |
| `chasolver-missed-draws-08-2026.csv` | 2 KB | `chasolver-data-until-08-2026.csv`'deki, hakemin 99 yarım hamle ve 50.000 düğümde hükümsüz bıraktığı 11 oyun; BestArbiter'ın dışa aktardığı biçimde | bu proje |
| `chasolver-missed-draw-positions.csv` | 75 KB | `chasolver-positions.txt`'te hakemin aynı ayarlarla kaçırdığı 602 beraberlik; BestArbiter'ın dışa aktardığı biçimde | bu proje |
| `LICENSE` | | MIT Lisansı | bu proje |

Üç chasolver kaynak dosyası (`chasolver-data-until-08-2026.csv`, `chasolver-positions.txt`, `chasolver-lichess.txt`) Miguel Ambrona'nın emeği; iki `chasolver-missed-*` dosyası bu projenin onlardan aldığı çıktılar. Ölçümler tekrarlanabilsin diye atıfla burada duruyorlar; asılları kaynak sütununda bağlantılı.

## Karşı örnek, katkı ve lisans

**LUX: evrenin en adil satrancı.** Bu iddianın arkasındayım; daha iyisi varsa buyursun gelsin :)

Şaka bir yana, iddia sınanmaya açık. Chess LUX'ın beraberlik verdiği kazanılabilir bir pozisyon ya da BestArbiter'da **FALSE TM** veya **found mate** çıkaran bir pozisyon bulursanız GitHub'da bir issue açarak paylaşın. Tek bir karşı örnek bile bizim için değerli.

Chess LUX MIT Lisansı ile yayımlanıyor; ayrıntılar için `LICENSE` dosyasına bakın.

## Teşekkür ve atıf

- **[FideLite](https://github.com/cuneytinann/FideLite):** Chess LUX'ın üzerine kurulu olduğu motor; kuralların, fonksiyonların ve tasarımın anlatımı [fidelite.art](https://www.fidelite.art/tr)'ta.
- **[chasolver](https://github.com/miguel-ambrona/chasolver), Miguel Ambrona:** mat edilemezlik analizi, test pozisyonları ve haksız sonuçlanmış Lichess oyunlarının verisi ([chasolver.org](https://chasolver.org)). Algoritmayı bağımsız olarak geliştirdim, ama onun çalışması olmasaydı bu kadar ilerletemezdim.
- **[Lichess açık veritabanı](https://database.lichess.org/)** (CC0).

---

## Teknik ek

Bu bölüm `l`'yi değiştirmek ya da ölçümleri tekrarlamak isteyenler için. Motorun geri kalanının nasıl çalıştığı [fidelite.art](https://www.fidelite.art/tr#engine)'ta.

<details>
<summary><b>Boyutlar, sürümler ve öncelikler</b></summary>

Tanım başına bayt, FideLite ve Chess LUX. Listede olmayan tanımlar aynı; başlık iki sürümde de 9 bayt kısa (varsayılan saat yok).

| tanım | FideLite 1x | Chess LUX 1x | FideLite 4x | Chess LUX 4x |
|---|---|---|---|---|
| `l` | 444 | 1.204 | 477 | 1.321 |
| `H` | 219 | 222 | 258 | 252 |
| `F` | 45 | 45 | 45 | 45 |
| `A` | 52 | 60 | 67 | 75 |
| `D` | 36 | 42 | 36 | 42 |
| `G` | 249 | 249 | 252 | 251 |
| `L` | 104 | 104 | 113 | 104 |
| `M` | 234 | 234 | 210 | 219 |
| **motor dosyası** | **1.847** | **2.615** | **1.945** | **2.787** |

1x'te `G` ile `M`'nin boyu değişmiyor: `G`'de yalnızca bir `&`'nin iki tarafının yeri değişiyor, `M` tahta kopyasını gövdede değil parametre listesinde alıyor. `F`'nin tek farkı iki sınırı.

`index.html`, 4x tanımlarını (`G V L C M I Im H l`) `BestArbiter.html`'deki `engine_4x.js` ile bayt bayt aynı taşır; `Z D F A` ise sonucu metin koduyla yazan arayüz sürümleridir.

**1x ve 4x.** FideLite'ta olduğu gibi, kullanıcıya sunulan her şey 4x yazılır: bayt önceliklidir, ama birkaç bayt karşılığında katlanarak hız kazanılıyorsa o bayt harcanır. 1x, aynı davranışın en kısa metnidir ve yalnızca ölçüm sayfasında deneysel seçenek olarak durur. İki sürüm her girdide aynı hükmü vermek zorundadır; gerçek oyunlarda 4x 100 ms'nin altında kalırken 1x saniyelere çıkabilir. İki projenin ortak optimizasyonları [FideLite'ın `engine_4x` bölümünde](https://www.fidelite.art/tr#speed) anlatılıyor.

**Öncelikler.** (1) Soundness (sağlamlık): hakem asla yanlış beraberlik vermez. (2) Bayt. (3) Hız. %100 completeness (tamlık), yani hiçbir beraberliği kaçırmamak, hedeftir ama iddia edilmez.

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
- **Taş kümeleri:** `M` bütün beyaz taşlar, `D` bütün siyah taşlar, `F`/`g` iki tarafın kutulu at, kale ve vezirleri; `m`/`n` bir tur içinde her iki tarafın alınamayan piyonları, `I`/`J` ise iki tarafın kapalı kare kümeleri.

**Sabit nokta.** Her tur kümeleri büyütür ya da daraltır:
- **Tıkalı piyonlar:** `P`/`Q` yalnız küçülür; rakip şahın alamadığı ve önü kalıcı olarak kapalı piyonlar kalır.
- **Piyon konileri:** `C`/`Z`, piyonların ilerleyebileceği kareler.
- **Şah selleri:** `K`/`L`, şahların ulaşabileceği kareler; yalnız büyür. Selin değdiği rakip piyon `r`/`q` üzerinden alınabilir sayılır.
- **Fil selleri:** `R`/`c`, fillerin ulaşabileceği kareler; sabit karelerde durur.
- **Kutulu taşlar:** Komşuluğu açılan at, kale veya vezir kaçışını `Y`'ye yazar; ilk kaçışta o seviye düşer. Rakip şahın ya da filin ulaşabildiği kutulu taş kaçış sayılmaz: hiçbir şey alamaz, başına gelebilecek tek şey alınmaktır. Karesi `F`/`g`'den düşer ve o andan sonra boş sayılır. Onu alabilecek bir piyon, piyon alımı koşuluyla seviyeyi düşürür.

4x'te `z` sağlaması (`C+Z+K+L+r+q-P-Q+R+c-F-g`) değişmeyince döngü durur. İçindeki on iki küme yalnız tek yönde hareket eder — `P`, `Q`, `F` ve `g` küçülür ve eksi işaretle girer, gerisi büyür — bu yüzden sağlama bir şeyi değiştiren her turda değişir, döngü erken duramaz. On sel ve piyon kümesi toplamda en fazla 640 bit, `F` ile `g` yalnız at, kale ve vezir karelerini değiştirebildiği için 768'den az tur bir şey değiştirir; 1x her zaman 768 tur döner. Pratikte 4x'e chasolver verisinde medyan 11 tur yetiyor, fuzz testlerinde 69'u hiç geçmedi. Bu sınır, `C` ve `q`'yu 64 bitte tutan `&f`'ye dayanır: o olmadan sekizinci sıranın ötesine itilen bitler sağlamayı oynatmayı sürdürür ve özyineleme hiç bitmez. 1x'in böyle bir maskeye ihtiyacı yok; zaten 768 turdan sonra duruyor ve o bitler hiçbir hükmü değiştirmiyor.

**Kilit koşulları.**
- terfi yok;
- piyon alımı ve piyonla şah çekme yok;
- donmuş varsayılan şah gerçekten hareketsiz;
- hiçbir taş kutudan çıkmadı (`!Y`);
- fil varsa fil katmanı tutuyor.

**Fil katmanı.** Şah selleri kesişmemeli. Her taraf ve renk için en fazla bir fil olmalı ve ALIM testi tutmalı: fil rakibin hiçbir piyonuna, filine ve piyon konisine değmemeli (ulaştığı kutulu taş bunun yerine düşer, bkz. yukarı). Ardından iki yoldan biri gerekir: ya BÖLGE (fil savunan şahın seline hiç değmez) ya da savunanın bütün piyonları tıkalıyken RENK sertifikası.

**RENK sertifikası.** Fil ancak kendi rengindeki bir karede, çaprazdan şah çekerek mat edebilir. Sertifika, savunan şahın ulaşabildiği ve filin renginde olan her karede, şah çekilse bile bir kaçış karesi kaldığını gösterir.
- **Sert engelleyiciler:** Savunanın sabit taşları (düşürülmüş kutulu taş dahil: yerinde durduğu sürece karesini kapatır), saldıranın piyon saldırıları ve saldıran şahın komşu kareleri. Aynı anda istedikleri kadar kareyi kapatabilirler.
- **Yumuşak engelleyici:** Savunanın o renkteki tek fili; aynı anda en fazla bir kare kapatır. Bu yüzden iki sert olmayan kare ya da bir serbest kare, kaçışı garanti eder.

1x metni:

```js
|!(o?B^Q:W^P)&(!(a&1+o)|!(i=o?c:R,j=f^((o?D^v^_:M^V^$)|(o?S(C)<<8n|N(K):S(Z)>>8n|N(L))|t&~I),u=j&~i,s=j&I,A=u&I,m&I&~(E(9n)&E(7n)|O(u)|j>>8n&j<<8n|j>>1n&j<<1n&x&y|(j>>8n|j<<8n)&S(j))))
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

**Hız sürümü.** `l`'nin 4x optimizasyonlarından ikisi FideLite'ın `engine_4x`'inden geliyor: sağlamayla durma (burada sekiz yerine on iki küme üzerinden) ve `C` ile `q`'daki `&f`. Geri kalanlar Chess LUX'ın eklediği kısımlara ait. Sabit noktadan önce iki ucuz kontrol hızla eliyor: kendi piyonlarının ve kutulu taşlarının tutmadığı bir adım karesi olan kutulu at, kale ya da vezir ve donmuş şah seviyelerinde, ne kendi taşlarınca tutulan ne de rakip piyonca korunan bir komşu karesi olan şah. `T` `s&&` ile açılıyor, böylece fili olmayan taraf hemen dönüyor; `S`, FideLite'ın topladığı yerde kaydırıyor; fil katmanında da `&`/`|` yerine `&&`/`||` geliyor, sonuç belli olunca geri kalanı atlanıyor. Karşılığı olmayan bir maliyet de var: FideLite'ın `l`'si şah ya da piyon olmayan ilk taşta dönüyor, Chess LUX'ınki ise bu pozisyonlara bakmak zorunda; hızlı eleyen kontroller bu maliyeti düşük tutuyor.

**Regresyon pozisyonları.** Fil katmanının önceki bir sürümü aşağıdaki pozisyonlara yanlış sertifika veriyordu; `l(3)/l(2)/l(1)` şunları döndürmeli:

| açık | pozisyon | beklenen |
|---|---|---|
| `T` yanlışlıkla XOR ile yazılmıştı | `6k1/8/8/p1p1p1p1/PpPpPpPp/1P1P1P1P/2B5/2B3K1 w - - 0 1` | 0/0/0 |
| alım testi ilerleyen piyonları görmüyordu | `4k3/7p/4p1pP/3pP1P1/2pPp3/1pP5/pP2P3/K5B1 w - - 0 1` | 0/0/0 |
| aynı aile | `4k3/7p/4p1pP/3pP1P1/2pP2p1/1pP5/pP4P1/K5B1 w - - 0 1` | 0/0/0 |
| RENK kenardaki şahı görmüyordu (…Bd1#) | `8/1k6/p1p1p1p1/P1p1P1P1/K1p1p1p1/P1P1P1P1/4b3/8 b - - 0 1` | 0/1/0 |
| aynı aile (…Bc6#) | `b7/k7/p3p3/P1p1p3/K1p1p1p1/P1P1P1P1/8/8 b - - 0 1` | 0/1/0 |
| aynı aile (Bd8#) | `8/4B3/p1p1p1p1/k1P1P1P1/p1P1p1p1/P1P1P1P1/1K6/8 w - - 0 1` | 0/0/1 |

Üçü de fil katmanındaydı; FideLite'ın şah+piyon çekirdeğinde hiçbiri yok.

</details>

<details>
<summary><b>Arama: H ve F'de değişenler</b></summary>

`H`'nin nasıl çalıştığı — bir düğümün hangi sırayla kanıtlandığı, 75 hamle kesmesi, tekrar kesmesi, varyant boyunca rok hakları — FideLite'ın ve fidelite.art'ta anlatılıyor ([satır satır](https://www.fidelite.art/tr#flow), [bütçe](https://www.fidelite.art/tr#hbudget)). Chess LUX iki şeyi değiştiriyor.

**Kilit kapısı tek taraflı.** FideLite'ın `H`'si `l()`'ye pozisyonun iki taraf için de ölü olup olmadığını soruyor. Chess LUX'ın `H`'si `l(2-g)`'ye `g`'nin rakibinin hâlâ kazanıp kazanamayacağını soruyor; bu daha çok düğümü belgeliyor: ijyj0mHa örneğinde hükmün tamamı kökte bu kapıdan geliyor.

**Sınırlar.** Biri biterse dal yanlış döner ve hüküm galibiyete düşer. Her varyantın kanıtlanması gerektiği için kanıtlanamayan ilk varyant bütün aramayı bitirir: kanıtı olmayan pozisyonların çoğu, bir varyant ilk kez 99 yarım hamleye ulaştığında, genellikle birkaç yüz düğüm sonra durur (kaçan 11 chasolver beraberliği 130–240 düğümde duruyor). 50.000 düğümlük bütçe nadiren tükenir: `chasolver-positions.txt`'teki her yedinci pozisyonda (488 pozisyon) 315'i derinlik sınırında, 13'ü bütçede durdu.

| yer | bütçe | derinlik |
|---|---|---|
| FideLite (`F`) | 20.000 düğüm | 15 yarım hamle |
| `index.html` | 50.000 düğüm | 99 yarım hamle |
| gömülü motorun `F`'si | 50.000 düğüm | 99 yarım hamle |
| ölçüm sayfası | ayarlanabilir, varsayılan 50.000 | ayarlanabilir, varsayılan 99 |

</details>

<details>
<summary><b>Doğrulama ayrıntıları ve ölçümler</b></summary>

| ölçüm | sonuç |
|---|---|
| chasolver verisi: kanıtlanan beraberlik | 201.049 / 201.060 (%99,9945) |
| kilitli yapı sınıfı: kanıtlanan beraberlik | 51.047 / 51.058 (%99,98) |
| hükümsüz | 11; hepsi kilitli yapı sınıfında ve hepsinde arama derinliği yetmiyor |
| aramanın bulduğu mat (referansla çelişen) | 0 |
| `chasolver-positions.txt`: rakibin kazanabildiği 1.945 pozisyonda yanlış beraberlik | 3; üçü de iki aynı renkli filin çifte şahıyla zaten mat olmuş, hiçbir legal hamlenin üretemeyeceği pozisyonlar; `Im` mat testinden önce cevap veriyor. Oyunda hakem böyle bir pozisyonu, bayrak düşmeden önce mat olarak bitirir |
| `chasolver-positions.txt`: kanıtlanan / kaçan beraberlik (1.469 hedef) | 867 / 602 |
| `chasolver-lichess.txt`: yanlış beraberlik | 0 |
| 1x ile 4x arasında farklı hüküm | 0 |

**Bağımsız çapraz kontroller**
- **Perft:** chess.js 1.4.0 ile 19 zor pozisyonda, 3–4 derinlikte birebir aynı; standart pozisyonlarda FideLite ile de aynı.
- **Rastgele oyunlar:** 450 oyun ve 123.453 yarım hamle boyunca legal hamle kümesi, tahta, rok hakları ve yarım hamle sayacı her adımda chess.js ile aynı.
- **FEN:** 14.005 FEN'de, `index.html`'in FEN yolundan kurulan pozisyonların legal hamleleri chess.js ile aynı.
- **`l`:** Mutasyonlu fuzz testinde yaklaşık 375 bin kilitli pozisyonda ilk ve sadeleştirilmiş dedektör bit bit aynı hükmü verdi.

**Hız**
- **Gerçek oyunlar:** chasolver verisinin %10'luk örnekleminde (20.106 pozisyon) medyan 0,1 ms, en uzun 60 ms.
- **En zor pozisyon:** `B6b/pr6/8/8/8/4p1pp/P3Pp1p/2b2K1k b - -`. 20.107 düğüm kullanıyor ve doğru hükmü (WT) veriyor; orta sınıf bir dizüstü bilgisayarda, tarayıcıda yaklaşık 0,4 s. Kanıtı olmayan bir pozisyon genellikle birkaç yüz düğüm sonra derinlik sınırında duruyor; 50.000 düğümün tamamı, kabaca bir saniye, yalnız bütçenin kendisi tükendiğinde harcanıyor.
- **Süreyi belirleyen:** Düğüm başına maliyet. Her düğümde legal hamle üretimi, tahta kopyası ve `l` çağrısı var.
- **Yerel kod:** Motor 64 bitlik tamsayılarla C++ ya da Rust'a çevrilir veya WebAssembly'ye derlenirse BigInt'in bellek ayırma yükü ortadan kalkar. Tipik pozisyonlar mikrosaniyeler, en zor pozisyon milisaniyeler düzeyine inebilir; pratikte buna gerek yok.

**FEN doğrulaması.** Pencere şunları kontrol eder:
- 64 kare ve bilinen taş harfleri;
- her tarafta tek şah;
- birinci ve sekizinci sırada piyon olmaması;
- sırası gelmeyen tarafın şah altında olmaması;
- 0–255 aralığında yarım hamle sayacı.

Rok hakkı yalnız şah ve kale yerindeyse, geçerken alma alanı yalnız gerçekten oynanabilir bir alım varsa kurulur. Bilinen sınır: sıraların uzunluğu tek tek sayılmıyor.

</details>

<details>
<summary><b>Fethedilecek 11 pozisyon, yol haritası ve doğrulama kuyruğu</b></summary>

Bu 11 pozisyon (`chasolver-missed-draws-08-2026.csv`'de de var) yanlış hüküm değil, henüz yakalanamayan beraberlikler. Hepsinde arama 130–240 düğüm harcayıp derinlik sınırına takılıyor; varyant üzerindeki hiçbir pozisyon materyal ya da kilit analiziyle sertifikalanamıyor.

| gameId | bayrak | n | FEN |
|---|---|---|---|
| GyWj6Ymz | siyah | 8 | `8/4k1p1/4p1Pp/1p1pPp2/pP1P1P1P/P7/8/4K3 b - - 8 63` |
| 3KkCirHD | beyaz | 0 | `8/1p6/4k3/1P1p2p1/1p1P2P1/1P1K4/8/8 w - - 0 46` |
| chikSuUd | siyah | 11 | `4K3/8/6p1/5pP1/5P1k/5P1p/7P/8 b - - 11 59` |
| azjorRHB | siyah | 0 | `8/4k1p1/4p1Pp/1p1pPp2/pP1P1P1P/P4K2/8/8 b - - 0 48` |
| lBSDAx07 | beyaz | 0 | `8/4k3/1p4p1/pP1p1pP1/2pP1P2/P1P1KP2/8/8 w - - 0 49` |
| RwnQJq1k | beyaz | 11 | `8/7p/5p1P/5p1K/5Pp1/6P1/b3k3/8 w - - 11 51` |
| vcaVIyhj | beyaz | 4 | `8/1p1k4/4p3/1P1pP1p1/1p1P2Pp/1P1K3P/8/8 w - - 4 47` |
| pw3hB0Tp | siyah | 0 | `8/1p2k3/8/1P1p2p1/1p1P2P1/1P3K2/8/8 b - - 0 44` |
| q2K0VFaV | beyaz | 0 | `8/6k1/6p1/p1p1p1P1/P1P1P1p1/6K1/6P1/8 w - a6 0 38` |
| FKr42ZRT | siyah | 13 | `8/8/7p/5p1P/5p1K/5Pp1/6P1/5kb1 b - - 13 63` |
| o2conOyc | beyaz | 1 | `8/2p2kp1/1pPp4/pP1Pp1P1/P3P1p1/6P1/8/5K2 w - - 1 44` |

**Yol haritası**
1. Kilit analizinin bu 11 yapıyı tanıması. (On ikincisi hPiwD75i çözüldü: filin alabildiği kutulu at kilidi bozmuyor.)
2. Hapsolmuş filleri engel olarak saymak: `1kb5/1p1p4/1P1P4/8/8/4p1p1/4P1P1/5BK1 w - - 0 1` ilk hamlede `DP` olmalı. Bugün bayrak hükmü doğru (`TM`) ve oyun beşli tekrarla berabere bitiyor, ama ölü pozisyon ilk hamlede ilan edilmiyor.
3. `chasolver-positions.txt`'te kaçan 602 beraberliği azaltmak (listesi `chasolver-missed-draw-positions.csv`'de).
4. Oyun geçmişini (beşli tekrar) bayrak aramasına katmak.
5. Chess960 uyarlaması.

**Doğrulama kuyruğu**
- "Kural doğruluğu en yüksek hakem" iddiasının bağımsız doğrulaması.
- Lichess veritabanında 75 hamle kuralı yüzünden berabere bitmesi gereken oyunları saymak.

</details>

<details>
<summary><b>Değişiklik yaparken: kontrol listesi, tuzaklar, Node ile ölçüm</b></summary>

**Kontrol listesi**
1. **Önce FideLite:** [Chess LUX'ın değiştirdikleri](#chess-luxın-değiştirdikleri) tablosundaki tanımların dışında motor FideLite'ın; oradaki bir değişiklik FideLite'ta yapılır ve oradan taşınır, böylece tablo doğru kalır.
2. **Soundness gerekçesi:** Yeni bir sertifikanın matı neden imkânsız kıldığını yazın.
3. **4x ve 1x:** Kullanıcıya sunulan kod 4x yazılır; 1x, aynı davranışın en kısa metni olarak ayrıca güncellenir.
4. **Üç kopya:** Değişiklik `index.html`'e, `BestArbiter.html`'deki `src_x4`'e ve `src_nm`'ye uygulanır; `index.html` ile `src_x4` birebir aynı kalır. 1x'e taşırken `&` ile `&&` farkına dikkat edin.
5. **Sürüm denkliği:** İki sürüm her girdide aynı hükmü vermeli (iki sürümü de koştur).
6. **Regresyon:** Regresyon tablosundaki pozisyonlar yanlış yönde sertifika almamalı.
7. **Soundness ölçümü:** `chasolver-lichess.txt`'te yanlış beraberlik 0, `chasolver-positions.txt`'te zaten mat olan üç pozisyon dışında 0, chasolver verisinde found mate 0 olmalı.
8. **Completeness gerilemesi yok:** chasolver verisinde doğru beraberlik ≥ 201.049; `chasolver-positions.txt`'te kaçan beraberlik ≤ 602 kalmalı.
9. **Yeni sertifika ailesi:** Hedefli pozisyonlar üretip chasolver'la çapraz kontrol edin; hazır test pozisyonları bir sertifikanın bütün köşe durumlarını göstermez.

**Tuzaklar**
- **Hazır test pozisyonları yetmez:** Bilinen açıkları hedefli fuzz testi buldu: şahı kenara taşımak, kilitli piyon çiftleri eklemek, iki renkte fil, tahtayı kaydırmak, renkleri çevirmek.
- **Saymayı unutmak:** "Bir taş bu kareye ulaşabilir" demek "bu kare kapatılabilir" demek değildir; tek bir taş aynı anda tek kare kapatır.
- **Negatif BigInt:** `~x` negatif bir sayı üretir; kaydırmadan önce `f&~(…)` ile 64 bite indirin.
- **Sütun taşması:** Yatay kaydırmada maske her zaman şart; tek renge kısıtlanmış kümede çapraz kaydırma maskesiz de doğrudur.
- **Operatör önceliği:** `&&` ve `||`, `|` ile `&`'den düşük önceliklidir; birini dönüştürünce komşularını da kontrol edin.
- **Erken çıkış sağlaması:** Büyüyen kümeler artı, küçülenler (`P`, `Q`, `F`, `g`) eksi işaretle girer.
- **1x'te `M` yan etkisiz değil:** orada rok haklarını da siliyor, bu yüzden `L` `c`'yi kaydet/geri yaz çiftinde taşımak zorunda. 4x'te `M` `c`'ye dokunmuyor, güncellemeyi arama yapıyor. Bu çiftin bir yarısını diğeri olmadan taşımayın.

**Node ile ölçüm**

```js
const fs = require('fs'), vm = require('vm');
const html = fs.readFileSync('BestArbiter.html', 'utf8');
const blocks = [...html.matchAll(/<script[^>]*>([\s\S]*?)<\/script>/g)].map((m) => m[1]);
const SRC = html.match(/id="src_x4">([\s\S]*?)<\/script>/)[1].trim();   // 1x için src_nm
const chunks = (t) => { const r = []; let d = 0, st = 0; for (let i = 0; i < t.length; i++) { const c = t[i]; if ('([{'.includes(c)) d++; else if (')]}'.includes(c)) d--; else if (c === ',' && d === 0) { r.push(t.slice(st, i)); st = i + 1; } } r.push(t.slice(st)); return r; };
const mirror = (x) => { const y = x.replace(/^H=/, 'Hd=').split('Ht[').join('Hs[').replace(',H(g,d-1)', ',Hd(g,d-1)'); if (y.endsWith('&v)')) return y.slice(0, -3) + '&&(v||(HM=1,0)))'; const k = y.lastIndexOf('&&(v||'); return y.slice(0, k + 2) + '(' + y.slice(k + 2, y.length - 1) + '||(HM=1,0)))'; };
const HD = chunks(SRC).filter((c) => c.startsWith('H=')).map(mirror)[0];
const ctx = { Math, BigInt, performance, console }; ctx.window = ctx; vm.createContext(ctx);
vm.runInContext(blocks[2], ctx);
vm.runInContext(SRC + ';' + HD + ';HM=0;Hs={};Ht={};Hn=0', ctx);
ctx.judge('8/8/4k3/3p2p1/1p1P1pP1/1P3P2/8/3K4 b - - 47 92', 99, 50000).code;   // 'DP'
```

`node --stack-size=4000` ile çalıştırın.

</details>
