# Chess LUX

**English** · [Türkçe](#chess-lux-türkçe)

▶ **[Play in your browser](https://cuneytinann.github.io/Chess-LUX/)**

**FideLite's FIDE arbiter as a complete game, in a single 9,526-byte HTML file.**

Chess LUX is a branch of [FideLite](https://www.fidelite.art). The rules engine is FideLite's fast build, `engine_4x.js`. It keeps FideLite's reading of every FIDE article, its board, move generation, flag-fall search, claims and offers, and its fifteen result codes; all of that is documented at **[fidelite.art](https://www.fidelite.art)**.

On top of that core, Chess LUX adds:
- a full game page: game setup, three clock types, time periods, a draw-offer interval and Chess960;
- three engine extensions: boxed-in knights, rooks and queens in the lock detector, castling by moving the king onto its own rook, and a short search for dead positions after every move;
- a much larger flag-fall search.

This README covers only those additions.

## At a glance

| | FideLite | Chess LUX |
|---|---|---|
| **What it is** | the rules engine (`engine_4x.js`, 1,978 B) | the same engine inside a complete game page (`index.html`, 9,526 B) |
| **Game setup** | — | any FEN; Fischer, Bronstein or simple delay; separate times for White and Black; two extra periods; draw-offer interval; automatic 3R/50; board orientation |
| **Chess960** | — | all 960 starting positions by number or at random; standard chess is number 518 |
| **Castling** | the king moves two squares | the king moves onto its own rook, in every game; in standard chess a click on the king's destination still works |
| **Lock detector** | kings, pawns and bishops | also boxed-in knights, rooks and queens |
| **Dead position (5.2.2)** | the lock detector | the lock detector, then a 9-ply search for each side after every move |
| **Flag-fall search** | 9 plies, 2,000 nodes | 450 plies, 600,000 nodes |
| **chasolver's 201,060 wrongly decided Lichess timeouts** | 201,050 correct | 201,058 correct |
| **… of which already dead when the flag fell** | 50,955 | 116,010 |

Any current browser runs it: Chrome/Edge 85+, Firefox 98+, Safari 15.4+. There is nothing to install; download `index.html` to play offline. The tab reads "FideLite.Art".

**The file.** `index.html` has five lines: the page (HTML and CSS), a blank line, the rules engine, a blank line, and the interface. The engine line opens the way FideLite's engines do, with `b=[c=un=15,e=y=t=8,n=o=0]`, and ends with `D`, the claims and offers.

## Try it now

1. **Open the game.** Open [cuneytinann.github.io/Chess-LUX](https://cuneytinann.github.io/Chess-LUX/), or open `index.html` locally.
2. **Set up the game in the dialog.** You can keep the starting position and the time (10 min + 5 s, Fischer increment), or change any of these:
   - **FEN and clock type.** Enter your own FEN, and choose Fischer increment, Bronstein or simple delay.
   - **Separate times.** Untick **⚖** to give Black a different time. A second minutes box appears, and the first becomes White's.
   - **Periods.** **per2** and **per3** add up to two further periods. Each one has a move (**#**, 40 and 60 by default), the minutes added when that move is completed, and the seconds added to the increment or delay from the next move on (negative values reduce it). Left at 0 they change nothing.
   - **½ Offer.** Sets how many moves must pass between a player's draw offers (1 by default).
   - **Auto 3R/50.** Makes threefold repetition and the 50-move rule end the game automatically instead of by claim.
   - **960.** Holds the number of the starting position: 518 is the standard one. Type another number or press 🎲 for a random one. The arrangement appears beside it, and the FEN field fills in with that start; you can still edit it.
   - **Orientation.** Beside **Play**, choose whether the board turns after every move (**Flip**) or stays with **White** or **Black** at the bottom.
   - **Reset.** **⌫** restores every setting.
3. **Play.**
   - **Moving.** Click or drag the pieces. To castle, move the king onto its own rook; in standard chess you can also click the king's destination.
   - **Marks.** Right-drag across the board to mark every square in between, or right-click a single square to mark just that one. A left click clears the marks.
   - **½ and ⚐.** ½ claims, offers or accepts a draw; with Auto 3R/50 it only offers and accepts. ⚐ resigns.
   - **Ending.** When the game ends, ↺ starts a new one.

The dialog checks a FEN you enter for basic validity only:
- the board must have eight ranks of eight squares, exactly one king per side and no pawn on the first or last rank;
- every piece beyond the starting set (a second queen, a third rook or knight, a second bishop on the same square colour) must be a promotion paid for by a missing pawn: for each side, pawns plus such extra pieces are at most 8. This also keeps each side to at most 8 pawns and 16 men;
- the side not to move must not be in check;
- the counters must be consistent;
- castling rights are kept only where the king and rook stand on the arrangement's files, and an en passant square only when the capture is legal.

Some positions no game can reach still pass these checks; for those the arbiter gives no guarantee.

**Double check is not tested.** The dialog does not ask whether a double check on the side to move could have arisen. The rules tab at [fidelite.art](https://www.fidelite.art) ("Not every double check can happen") works out which pairs of checkers are possible: at most two, at least one of them along a ray, never two diagonal ones, and two straight ones only after a capturing promotion. These rules are necessary but not sufficient:
- **A counterexample.** In `4k3/4R3/3N4/8/8/8/8/4K3 b - - 0 1` the knight and the rook belong to different families, yet the rook stands next to the king. No square lies between them from which the knight could have come and opened the rook's file, and a knight's check cannot be opened.
- **The missing condition.** One checker must have come from an empty square on the other's line, or an en passant capture must have opened two lines at once.
- **How much it matters.** Among 8,503 random double checks that no last move can produce, the pair rules catch fewer than half (3,874).

A test that takes back the last move closes this gap:
- **How it works.** It puts one of the opponent's pieces back on an empty square it could have come from: as a pawn if the piece stands on the promotion rank, and with the captured pawn restored for en passant. The side to move must then not be in check.
- **Results.** It rejected all 8,503, and none of 6,926 double checks that a last move can produce. Those include 411 from random games, standard and 960, and the 17 in chasolver's data.
- **Size.** It would add 237 bytes, including a check that an en passant square does not come with a check that was already there before the two-square advance. It is not in `index.html`.

Even that test looks only one move back, so it still lets through positions ruled out by something else:
- **The halfmove clock.** A clock above 0 rules out a capture or a pawn move. `4kR2/8/8/8/4R3/8/8/K7 b - - 5 50` passes, though only the capturing promotion exf8=R can give that double check.
- **Castling rights.** They can claim that the rook or king that has just moved never moved.
- **Earlier positions.** The position before the last move may itself be unreachable.

## Armageddon

Armageddon is a decisive game in which a draw counts as a win for Black; in return, Black starts with less time. Chess LUX needs no separate mode for it, because the dialog can set it up:

1. Tick **Auto 3R/50**, so that threefold repetition and the 50-move rule end the game on their own instead of waiting for a claim.
2. Untick **⚖** and give Black less time, for example 5 minutes for White and 4 for Black.
3. Optionally, set **½ Offer** to a very large number, such as 99999, so that no draw can be offered. Unlike the first two steps this is not required: an agreed draw counts as a win for Black anyway.
4. If the format adds an increment later in the game, use a period. For example, 2 seconds per move from move 61 on is **per2** `# 60 +0 min +2 s`, with the main increment at 0.

The page reports a drawn game as ½-½, like any other draw. This includes a flag fall after which White cannot mate (FIDE 6.9). Under Armageddon rules, that result means Black wins.

## What Chess LUX adds

### The game page

The page shows the following:
- **Clocks.** Each clock is read from `performance.now()` and settled before every action, claims and resignation included. It turns red in its last 30 seconds.
- **Captured material.** Beside each clock: the pieces that side has captured, and the material balance.
- **Board highlights.** Legal targets of the selected piece, the last move, and a king in check.
- **Draw light.** ½ lights up green when a 50-move or threefold claim is available, or the opponent has offered a draw.
- **Status line.** The halfmove count (`M`) and how many times the current position has occurred (`R`).
- **Promotion.** A choice of queen, rook, bishop or knight.

### Time controls

**Periods.** Every game has three periods; the second and third change nothing until they are given values.
- **Move counter.** It is the FEN's fullmove number (its sixth field). The move a player is making carries that number, and it goes up after Black's move, exactly as on a scoresheet: `b … 0 1` starts with `1...`, and White's first move is then move 2.
- **What a period adds.** When a player completes move #, that period's minutes are added. From the next move on, its seconds change the increment (Fischer or Bronstein) or the delay. The two thresholds are independent, so their order does not matter.
- **Bronstein.** The refund is capped at the time used, before the period's minutes are added, so the minutes are never cut.

**Starting past a threshold.** The time entered in the dialog is the time left on each clock. A game set up past a threshold (say, from move 65 with # 40 and 60) gets no minutes for it, but plays at the rate that applies from its move number.

**Separate times.** Unticking ⚖ gives Black its own starting time. The increment, the delay and the periods stay common to both players.

### Draw offers and claims

**Offer interval.** A player may offer a draw only when at least ½ Offer moves (1 by default) have passed since their own last offer, counted with the same move numbers.
- **First offer.** It becomes possible once both players have completed that many moves: Black may offer with move ½ Offer, White with the move after it.
- **When an offer counts.** When the move carrying it is made, so taking it back before moving costs nothing.
- **½ is never blocked.** Until the interval has passed, a move made with ½ only claims a draw and sends no offer. Accepting an offer and claiming a draw are never restricted.

**FIDE 5.2.3.** With the default of 1, the interval is exactly FIDE 5.2.3: an agreement needs both players to have moved. White's offer with its first move could never be accepted, while Black may offer with its first move and White may accept.

**Auto 3R/50.**
- **On:** threefold repetition and the 50-move rule end the game automatically, as on online servers, and ½ only offers and accepts. The flag-fall search then also ends its lines at 50 moves instead of 75.
- **Off:** the game follows FIDE's over-the-board rules. Those two are claimed, while fivefold repetition and the 75-move rule end the game automatically.

### Chess960

**Numbering.** The starting position is generated from its number with Scharnagl's formula, the official Chess960 numbering:
- light-squared bishop, dark-squared bishop, queen, the two knights;
- then rook, king and rook in the three squares left.

No table of positions is embedded, and all 960 numbers match the official list. Standard chess is simply number 518, so it is one of the 960 starts rather than a separate mode.

**Castling.** In every game, castling is played by moving the king onto its own rook.
- **Where the pieces end up.** The king lands on the g- or c-file and the rook on the f- or d-file.
- **What it requires.** The castling right; empty squares on the paths of king and rook, apart from those two pieces; and no attacked square on the king's path, start and destination included.
- **Adjacent king and rook.** If they already stand next to each other, castling can simply swap them; 162 of the 960 starts allow a castle on the very first move. If the king already stands on its destination, only the rook moves.
- **Standard chess.** Clicking the king's destination (g1 or c1, or b1 for the long castle) works too.
- **Castling rights in a FEN.** The `KQkq` rights of an edited FEN refer to the rook files of the selected arrangement.

**Tested.** Move generation matches python-chess on all 960 starting positions, on random middlegames and on 1,500 positions built to stress castling. Board, castling rights, en passant square and 50-move counter also match after every move of 400 random games, which include 322 castles, 90 of them king–rook swaps.

### The engine

**Boxed-in knights, rooks and queens.** FideLite's lock detector certifies locked structures of kings, pawns and bishops. Any knight, rook or queen on the board switches it off. Chess LUX lets such a piece stay when it is boxed in.
- **When a piece is boxed in.** Every square it could step to holds one of its own fixed men:
  - for a knight, its knight squares;
  - for a rook, its four orthogonal neighbours;
  - for a queen, all eight neighbours.

  A fixed man is either a permanently blocked pawn that the enemy king cannot take, or another boxed-in piece. A boxed-in piece can never move, so it counts as a wall like the pawns around it.
- **A boxed piece can block pawns too.** A pawn with a boxed-in piece of either colour directly in front of it can never advance. This is how a rook on a8 and its own pawn on a7 hold each other.
- **A fallen piece.** A boxed-in piece the enemy king or a bishop can reach may be captured. It still never moves, but it stops counting as a wall for anything.
- **A pawn capture.** If an enemy pawn could take a boxed-in piece, there is no certificate.

Example: in the Lichess game [hPiwD75i](https://lichess.org/hPiwD75i) White lost on time here:

```
1n6/2Bp4/p1pPp1k1/PpP1Pp1p/1P3P1P/2K5/8/8 w - - 7 53
```

The black knight on b8 is boxed in by its pawns on a6, c6 and d7. The white bishop can take it, but taking it opens nothing, and Black cannot mate by any series of moves. The correct result is a draw. Chess LUX gives it without a search.

**Chess960 does not disturb the lock.** The lock detector never reads castling rights, and it does not need to:
- **Every rook is boxed.** A certificate exists only when every rook on the board is boxed in, or there is none.
- **A boxed rook cannot castle.** A castle always needs one of the rook's two neighbours on its rank to be empty or to be the king itself: the rook moves through one of them, or the king passes or lands next to it. A boxed-in rook has fixed men on both sides.

In a test, castling rights were added to every position of chasolver's test sets where king and rooks allowed it. No lock certificate came out for a position in which a castle was legal.

**Dead positions found by search.** After every move, besides the lock detector, Chess LUX asks the search two short questions, one for each side: can this side still be mated? Each search runs to 9 plies and 2,000 nodes and plays every legal move of both sides. A side in check plays only the moves that get it out of check; a side that is already mated has ended the game as checkmate before this test is reached; stalemate counts as no mate. When both searches prove that no line leads to mate, the game ends as a dead position (FIDE 5.2.2).
- **What it adds.** Positions the lock detector cannot certify but that are dead within a few moves: a piece that must be captured, a king whose only move takes the last pawn, lines that all end in stalemate. Of chasolver's 201,060 timeouts, 116,010 (58%) are already dead when the flag falls, against 50,956 with the lock detector alone. In those games Chess LUX would have ended the game as a dead position no later than that move. Deep locks with bishops still need the flag-fall search.
- **Soundness.** A line that reaches the ply or node limit counts as unproven, so a position is called dead only when every line ends without mate: in stalemate, in a repetition, or at a position the lock detector certifies. The 50- and 75-move rules do not end a line here: a position is never called dead just because the move counter is close to its limit. On chasolver's labelled test sets it called no position dead that one side can still win.
- **What it costs.** About 0.1 ms per move on average, a few tens of milliseconds at most.

**The flag-fall search.** When a flag falls, the search checks whether the opponent can still mate. It runs to 450 plies and 600,000 nodes, where FideLite stops at 9 plies and 2,000 nodes.
- **Why 450.** A line ends by itself at the 75-move rule, so 450 plies leave room for three full 75-move stretches between pawn moves and captures.
- **What it adds.** On chasolver's data it proves the draw in games that need lines of up to about 150 plies, and in constructed locked positions that need up to about 450.
- **What it costs.** Most flag verdicts take well under a millisecond. In the hardest positions the page can pause for a few seconds (about 5 s at most in the tests) while the search runs.

**Two accelerators FideLite dropped.** FideLite's `engine_4x.js` once had two more shortcuts and left them out, because in its 9-ply, 2,000-node search they did not pay for their bytes; measured again with the current build, they still do not. Chess LUX keeps both, because its searches are far longer:
- **A king that can step out.** Before the lock detector runs its full analysis for a king, it asks whether that king can simply step to a square no pawn holds or attacks; if it can, that case is settled at once.
- **Asking the lock detector at the right time.** Inside the flag-fall search, the lock detector is asked only in the two plies that follow a capture or a pawn move, and only while no en passant capture is open: a new lock cannot appear anywhere else.

Neither changes a verdict: on all 201,060 positions of chasolver's data the build without them gives the same result for every position. It is only slower: the flag-fall search took about 45% longer in total and about twice as long on the heaviest position, and the dead-position search after every move about 15% longer.

**Three of FideLite's new move-generation shortcuts.** The current `engine_4x.js` (1,978 B) speeds up move generation with four shortcuts that change no result. Chess LUX carries three of them:
- **Check test.** A pawn, knight or king that stands too far from the king to give check is skipped before its move is traced.
- **Rook and queen.** A target that lies on none of the piece's lines is rejected at once, instead of after a walk along the rank.
- **Pawn advance.** The squares of a pawn advance are walked only when the target square is empty and in reach.

The fourth limits the targets of pawns, knights and kings to the 5×5 box around them. Chess LUX leaves it out, because here the king castles onto its own rook, which can stand up to six files away.

Nothing is decided differently: the engine gives the same perft counts as before on the Chess Programming Wiki test positions, and the same legal moves at every ply of 60 random Chess960 games.

## Counterexamples, credits and license

**LUX: the fairest chess in the universe.** I stand by that claim, and if there's anything better out there, bring it on :)

Jokes aside, the claim is open to challenge. If you find a winnable position where Chess LUX awards a draw, please share it by opening an issue on GitHub. Even a single counterexample is valuable.

- **[FideLite](https://github.com/cuneytinann/FideLite):** the engine Chess LUX is built on; its rules, functions and design are documented at [fidelite.art](https://www.fidelite.art).
- **[chasolver](https://github.com/miguel-ambrona/chasolver), Miguel Ambrona:** unwinnability analysis, test positions and the data on wrongly decided Lichess games ([chasolver.org](https://chasolver.org)) that the measurements above use.
- **[Lichess open database](https://database.lichess.org/)** (CC0).

Chess LUX is released under the MIT License.

---

# Chess LUX (Türkçe)

[English](#chess-lux) · **Türkçe**

▶ **[Tarayıcıda oyna](https://cuneytinann.github.io/Chess-LUX/)**

**FideLite'ın FIDE hakemi, eksiksiz bir oyun olarak, tek bir 9.526 baytlık HTML dosyasında.**

Chess LUX, [FideLite](https://www.fidelite.art)'ın bir koludur. Kural motoru FideLite'ın hızlı sürümü `engine_4x.js`. Her FIDE maddesinin nasıl okunduğu, tahta, hamle üretimi, bayrak düşme araması, talepler ve teklifler, on beş sonuç kodu FideLite'ınkiyle aynı; hepsi **[fidelite.art](https://www.fidelite.art)**'ta anlatılıyor.

Chess LUX bu çekirdeğin üstüne şunları ekliyor:
- eksiksiz bir oyun sayfası: oyun kurma, üç saat türü, zaman periyotları, beraberlik teklifi aralığı ve Chess960;
- motorda üç ek: kilit dedektöründe gömülü at, kale ve vezir; şahın kendi kalesinin üstüne giderek yaptığı rok; ve her hamleden sonra ölü pozisyon için kısa bir arama;
- çok daha geniş bir bayrak düşme araması.

Bu README yalnızca bu eklemeleri anlatıyor.

## Bir bakışta

| | FideLite | Chess LUX |
|---|---|---|
| **Ne olduğu** | kural motoru (`engine_4x.js`, 1.978 B) | aynı motor, eksiksiz bir oyun sayfasının içinde (`index.html`, 9.526 B) |
| **Oyun kurma** | — | istenen FEN; Fischer, Bronstein ya da simple delay; beyaz ve siyah için ayrı süre; iki ek periyot; beraberlik teklifi aralığı; otomatik 3R/50; tahta yönü |
| **Chess960** | — | 960 başlangıç pozisyonunun hepsi, numarasıyla ya da rastgele; standart satranç 518 numara |
| **Rok** | şah iki kare gider | her oyunda şah kendi kalesinin üstüne gider; standart satrançta şahın varış karesine tıklamak da çalışır |
| **Kilit dedektörü** | şahlar, piyonlar ve filler | ayrıca gömülü at, kale ve vezir |
| **Ölü pozisyon (5.2.2)** | kilit dedektörü | kilit dedektörü, ardından her hamleden sonra iki taraf için 9 ply'lık arama |
| **Bayrak düşme araması** | 9 ply, 2.000 düğüm | 450 ply, 600.000 düğüm |
| **chasolver'ın süreden yanlış sonuçlanan 201.060 Lichess oyunu** | 201.050 doğru | 201.058 doğru |
| **… bunlardan bayrak düştüğünde zaten ölü olan** | 50.955 | 116.010 |

Güncel her tarayıcıda çalışır: Chrome/Edge 85+, Firefox 98+, Safari 15.4+. Kurulum gerekmez; çevrimdışı oynamak için `index.html`'i indirmen yeterli. Sekmede "FideLite.Art" yazar.

**Dosya.** `index.html` beş satırdır: sayfa (HTML ve CSS), boş bir satır, kural motoru, boş bir satır ve arayüz. Motor satırı FideLite'ın motorları gibi `b=[c=un=15,e=y=t=8,n=o=0]` ile açılır ve talepleri ve teklifleri karşılayan `D` ile biter.

## Hemen dene

1. **Oyunu aç.** [cuneytinann.github.io/Chess-LUX](https://cuneytinann.github.io/Chess-LUX/) adresini ya da `index.html`'i yerelde aç.
2. **Oyunu açılan pencerede kur.** Başlangıç pozisyonunu ve süreyi (10 dk + 5 sn, Fischer artırımı) olduğu gibi bırakabilir ya da bunlardan istediğini değiştirebilirsin:
   - **FEN ve saat türü.** Kendi FEN'ini gir; Fischer artırımı, Bronstein ya da simple delay seç.
   - **Ayrı süreler.** Siyaha farklı bir süre vermek için **⚖**'nin işaretini kaldır. İkinci bir dakika kutusu belirir, ilk kutu beyazın süresi olur.
   - **Periyotlar.** **per2** ve **per3** iki ek periyot tanımlar. Her birinde bir hamle (**#**, varsayılan 40 ve 60), o hamle tamamlanınca eklenecek dakika ve sonraki hamleden itibaren artırıma ya da gecikmeye eklenecek saniye var (negatif değer azaltır). 0 bırakılırsa hiçbir şey değişmez.
   - **½ Offer.** Bir oyuncunun beraberlik teklifleri arasında kaç hamle geçmesi gerektiğini belirler (varsayılan 1).
   - **Auto 3R/50.** Üçlü tekrarı ve 50 hamle kuralını, talep yerine oyunu kendiliğinden bitiren kurallar yapar.
   - **960.** Başlangıç pozisyonunun numarası; standart pozisyon 518. Başka bir numara yaz ya da rastgele biri için 🎲'ya bas. Dizilim yanında görünür, FEN alanı da o başlangıçla dolar; FEN'i yine de düzenleyebilirsin.
   - **Tahta yönü.** **Play**'in yanında tahtanın her hamlede dönmesini (**Flip**) ya da **White** veya **Black** altta sabit kalmasını seç.
   - **Sıfırlama.** **⌫** bütün ayarları geri yükler.
3. **Oyna.**
   - **Hamle.** Taşları tıklayarak ya da sürükleyerek oyna. Rok için şahı kendi kalesinin üstüne götür; standart satrançta şahın varış karesine tıklamak da olur.
   - **İşaretler.** Sağ tuşla tahtada sürüklersen aradaki bütün kareler işaretlenir; tek kareye sağ tıklarsan yalnız o kare işaretlenir. Sol tıklama işaretleri siler.
   - **½ ve ⚐.** ½ beraberlik talebi, teklifi ve kabulü içindir; Auto 3R/50 açıkken yalnız teklif ve kabul. ⚐ terk içindir.
   - **Bitiş.** Oyun bitince ↺ yeni bir oyun açar.

Pencere, girilen FEN'i yalnızca temel geçerlilik açısından denetler:
- tahta sekizer karelik sekiz sıra olmalı, iki tarafın da tek şahı olmalı, ilk ve son sırada piyon olmamalı;
- başlangıç kadrosunu aşan her taş (ikinci vezir, üçüncü kale ya da at, aynı renk karede ikinci fil) eksik bir piyonla ödenmiş bir terfi olmalı: her taraf için piyonlar ile bu fazla taşların toplamı en çok 8. Bu, her tarafı en çok 8 piyon ve 16 taşla da sınırlar;
- sırası gelmeyen taraf şahta olmamalı;
- sayaçlar birbiriyle tutarlı olmalı;
- rok hakları yalnız şah ve kale dizilimin sütunlarında duruyorsa, en passant karesi de yalnız alış yasalsa korunur.

Hiçbir oyunun ulaşamayacağı bazı pozisyonlar bu denetimlerden geçebilir; onlar için hakem bir şey garanti etmez.

**Çifte şah denetlenmiyor.** Pencere, sıradaki tarafa çekilen bir çifte şahın oluşup oluşamayacağına bakmaz. [fidelite.art](https://www.fidelite.art)'taki kurallar sekmesi ("Çifte şah her biçimde olmaz") hangi şah çeken ikililerin mümkün olduğunu çıkarıyor: en çok iki taş, en az biri ışın boyunca, hiçbir zaman iki çapraz, iki düz ise yalnızca alarak terfiden sonra. Bu kurallar gerekli ama yeterli değil:
- **Bir karşı örnek.** `4k3/4R3/3N4/8/8/8/8/4K3 b - - 0 1` konumunda at ile kale farklı ailelerden, ama kale şahın yanında duruyor. Aralarında, atın gelip kalenin dikeyini açabileceği bir kare yok; atın şahı da açılamaz.
- **Eksik koşul.** Şah çeken taşlardan biri, öbürünün hattı üstündeki boş bir kareden gelmiş olmalı; ya da bir geçerken alma iki hattı birden açmış olmalı.
- **Ne kadar fark ediyor.** Hiçbir son hamlenin oluşturamayacağı 8.503 rastgele çifte şahın yarısından azını (3.874) ikili kuralları yakalıyor.

Son hamleyi geri alan bir test bu açığı kapatıyor:
- **Nasıl çalışıyor.** Rakibin taşlarından birini gelmiş olabileceği boş bir kareye geri koyar: taş terfi yatayında duruyorsa piyon olarak, geçerken almada da alınan piyonu yerine koyarak. Bunun ardından sıradaki taraf şahta olmamalı.
- **Sonuçlar.** 8.503 konumun hepsini reddetti; bir son hamlenin oluşturabileceği 6.926 çifte şahın hiçbirini reddetmedi. Bunların 411'i standart ve 960 rastgele oyunlardan, 17'si chasolver'ın verisinden.
- **Boyut.** 237 bayt eklerdi; buna, en passant karesinin iki kare ilerleyişten önce de var olan bir şahla gelmediğinin denetimi dahil. `index.html`'de yok.

O test de yalnızca bir hamle geriye bakar; başka bir nedenle dışlanan konumları yine geçirir:
- **Yarım hamle sayacı.** Sayaç 0'dan büyükse son hamle alış ya da piyon hamlesi olamaz. `4kR2/8/8/8/4R3/8/8/K7 b - - 5 50` geçiyor, oysa bu çifte şahı ancak alarak terfi exf8=R verebilir.
- **Rok hakları.** Az önce oynamış olması gereken kalenin ya da şahın hiç oynamadığını söyleyebilir.
- **Önceki konumlar.** Son hamleden önceki konum da ulaşılamaz olabilir.

## Armageddon

Armageddon, beraberliğin siyahın galibiyeti sayıldığı, buna karşılık siyahın daha az süreyle başladığı kesin sonuçlu bir oyundur. Chess LUX'ın bunun için ayrı bir moda ihtiyacı yok, açılan pencere kurabiliyor:

1. **Auto 3R/50**'yi işaretle; üçlü tekrar ve 50 hamle kuralı talep beklemeden oyunu bitirsin.
2. **⚖**'nin işaretini kaldır ve siyaha daha az süre ver; örneğin beyaza 5, siyaha 4 dakika.
3. İstersen **½ Offer**'ı 99999 gibi çok büyük bir sayı yap; böylece beraberlik teklif edilemez. İlk iki adımın aksine bu şart değil: anlaşmalı beraberlik zaten siyahın galibiyeti sayılır.
4. Formatta oyunun ilerisinde artırım varsa bir periyot kullan. Örneğin 61. hamleden itibaren hamle başına 2 saniye, ana artırım 0'ken **per2** `# 60 +0 dk +2 sn` olur.

Sayfa beraberliği her beraberlik gibi ½-½ olarak bildirir. Buna, bayrak düştükten sonra beyazın mat edemediği durum da dahil (FIDE 6.9). Armageddon kurallarında bu sonuç siyahın kazandığı anlamına gelir.

## Chess LUX'ın ekledikleri

### Oyun sayfası

Sayfada şunlar görünür:
- **Saatler.** Her saat `performance.now()` ile okunur ve talep ile terk dahil her eylemden önce kapatılır. Son 30 saniyesinde kırmızıya döner.
- **Alınan taşlar.** Her saatin yanında o tarafın aldığı taşlar ve malzeme farkı.
- **Tahta vurguları.** Seçilen taşın gidebileceği kareler, son hamle ve şah altındaki şah.
- **Beraberlik ışığı.** 50 hamle ya da üçlü tekrar talebi mümkün olduğunda veya rakip beraberlik teklif ettiğinde ½ yeşil yanar.
- **Durum satırı.** Yarım hamle sayacı (`M`) ve mevcut pozisyonun kaçıncı kez geldiği (`R`).
- **Terfi.** Vezir, kale, fil ya da at seçimi.

### Zaman kontrolü

**Periyotlar.** Her oyunun üç periyodu var; ikinci ve üçüncüye değer verilmedikçe hiçbir şey değiştirmezler.
- **Hamle sayacı.** FEN'in tam hamle numarası (altıncı alan). Oyuncunun yaptığı hamle o numarayı taşır ve numara siyahın hamlesinden sonra artar, tıpkı bir hamle kâğıdındaki gibi: `b … 0 1` `1...` ile başlar, beyazın ilk hamlesi de 2. hamle olur.
- **Periyodun ekledikleri.** Oyuncu # hamlesini tamamlayınca o periyodun dakikaları eklenir. Sonraki hamleden itibaren saniyeleri artırımı (Fischer ya da Bronstein) ya da gecikmeyi değiştirir. İki eşik birbirinden bağımsızdır, sıraları önemli değil.
- **Bronstein.** İade, periyodun dakikaları eklenmeden önce harcanan süreyle sınırlıdır; böylece dakikalar asla kırpılmaz.

**Eşik geçildikten sonra başlamak.** Pencereye girilen süre her saatte kalan süredir. Bir eşiği geçmiş olarak kurulan oyun (örneğin # 40 ve 60 ile 65. hamleden başlayan) o eşiğin dakikalarını almaz, ama kendi hamle numarasından itibaren geçerli tempoyla oynanır.

**Ayrı süreler.** ⚖'nin işaretini kaldırmak siyaha kendi başlangıç süresini verir. Artırım, gecikme ve periyotlar iki oyuncu için ortak kalır.

### Beraberlik teklifi ve talebi

**Teklif aralığı.** Bir oyuncu, kendi son teklifinden bu yana en az ½ Offer kadar hamle geçmişse beraberlik teklif edebilir (varsayılan 1); hamleler aynı numaralarla sayılır.
- **İlk teklif.** İki oyuncu da o kadar hamleyi tamamladığında mümkün olur: siyah ½ Offer numaralı hamlesiyle, beyaz ondan sonraki hamlesiyle teklif edebilir.
- **Teklifin geçerli olduğu an.** Onu taşıyan hamle yapıldığında; hamle yapmadan geri almak bir şey kaybettirmez.
- **½ hiç kilitlenmez.** Aralık dolana kadar ½ ile yapılan hamle yalnızca beraberlik talep eder, teklif göndermez. Teklifi kabul etmek ve beraberlik talep etmek hiçbir zaman kısıtlanmaz.

**FIDE 5.2.3.** Varsayılan 1 değeriyle aralık FIDE 5.2.3'ün kendisidir: anlaşma için iki oyuncunun da hamle yapmış olması gerekir. Beyazın ilk hamlesiyle yaptığı teklif hiçbir zaman kabul edilemez; siyah ise ilk hamlesiyle teklif edebilir ve beyaz kabul edebilir.

**Auto 3R/50.**
- **Açıkken:** üçlü tekrar ve 50 hamle kuralı, çevrim içi sunuculardaki gibi oyunu kendiliğinden bitirir; ½ yalnızca teklif ve kabul için kalır. Bayrak düşünce yapılan arama da varyantları 75 yerine 50 hamlede keser.
- **Kapalıyken:** oyun FIDE'nin tahta başı kurallarını izler. Bu ikisi talep edilir; beşli tekrar ve 75 hamle kuralı ise oyunu kendiliğinden bitirir.

### Chess960

**Numaralandırma.** Başlangıç pozisyonu numarasından Scharnagl formülüyle üretiliyor; bu, Chess960'ın resmî numaralandırması:
- açık kareli fil, koyu kareli fil, vezir, iki at;
- sonra kalan üç kareye kale, şah ve kale.

Pozisyon tablosu gömülü değil ve 960 numaranın hepsi resmî listeyle aynı. Standart satranç yalnızca 518 numara; ayrı bir mod değil, 960 başlangıçtan biri.

**Rok.** Her oyunda rok, şahı kendi kalesinin üstüne götürerek yapılır.
- **Taşların vardığı yer.** Şah g ya da c sütununa, kale f ya da d sütununa gider.
- **Gerekenler.** Rok hakkı; şah ile kalenin yolundaki karelerin bu iki taş dışında boş olması; şahın yolundaki hiçbir karenin (başlangıç ve varış dahil) tehdit altında olmaması.
- **Yan yana şah ve kale.** Zaten yan yanaysalar rok ikisinin yerini değiştirir; 960 başlangıcın 162'sinde rok daha ilk hamlede yapılabilir. Şah zaten varış karesindeyse yalnız kale oynar.
- **Standart satranç.** Şahın varış karesine (g1 ya da c1, uzun rok için b1) tıklamak da çalışır.
- **FEN'deki rok hakları.** Düzenlenmiş bir FEN'deki `KQkq` hakları seçilen dizilimin kale sütunlarını gösterir.

**Test.** Hamle üretimi python-chess ile aynı: 960 başlangıç pozisyonunun hepsinde, rastgele oyun ortalarında ve roku zorlamak için kurulmuş 1.500 konumda. 400 rastgele oyunun her hamlesinden sonra tahta, rok hakları, en passant karesi ve 50 hamle sayacı da aynı; bu oyunlarda 322 rok var, 90'ı şah ile kalenin yer değiştirmesi.

### Motor

**Gömülü at, kale ve vezir.** FideLite'ın kilit dedektörü şahlardan, piyonlardan ve fillerden oluşan kilitli yapıları onaylar. Tahtada bir at, kale ya da vezir olunca devre dışı kalır. Chess LUX böyle bir taş gömülüyse onun kalmasına izin veriyor.
- **Bir taş ne zaman gömülü sayılır.** Gidebileceği her karede kendi sabit taşlarından biri durmalı:
  - at için kendi at kareleri;
  - kale için dört dik komşusu;
  - vezir için sekiz komşusu.

  Sabit taş ya önü kalıcı olarak tıkalı olan ve rakip şahın alamadığı bir piyon, ya da gömülü başka bir taştır. Gömülü taş hiç kıpırdayamaz, bu yüzden etrafındaki piyonlar gibi duvar sayılır.
- **Gömülü taş piyonu da kilitler.** Önünde iki renkten herhangi birinin gömülü taşı duran piyon hiç ilerleyemez. a8'deki kale ile a7'deki kendi piyonu birbirini böyle tutar.
- **Düşmüş taş.** Rakip şahın ya da bir filin ulaşabildiği gömülü taş alınabilir. Yine hiç kıpırdamaz, ama artık hiçbir taşa duvar sayılmaz.
- **Piyonla alış.** Rakip bir piyon gömülü bir taşı alabiliyorsa onay verilmez.

Örnek: Lichess oyunu [hPiwD75i](https://lichess.org/hPiwD75i)'de beyaz bu pozisyonda süreden kaybetti:

```
1n6/2Bp4/p1pPp1k1/PpP1Pp1p/1P3P1P/2K5/8/8 w - - 7 53
```

b8'deki siyah at a6, c6 ve d7'deki piyonlarıyla gömülü. Beyaz fil onu alabilir, ama almak hiçbir şey açmaz ve siyah hiçbir hamle dizisiyle mat edemez. Doğru sonuç beraberlik; Chess LUX bunu aramaya girmeden veriyor.

**Chess960 kilidi bozmaz.** Kilit dedektörü rok haklarına hiç bakmaz ve bakması da gerekmez:
- **Bütün kaleler gömülü.** Onay ancak tahtadaki her kale gömülüyse (ya da hiç kale yoksa) çıkar.
- **Gömülü kale rok yapamaz.** Rok her zaman kalenin sıradaki iki komşusundan birinin boş olmasını ya da şahın kendisi olmasını ister: kale onlardan birinin üstünden geçer, ya da şah yanından geçer veya yanına varır. Gömülü bir kalenin iki yanında da sabit taş durur.

Bir testte chasolver'ın test setlerinde şahın ve kalelerin izin verdiği her pozisyona rok hakları eklendi. Rokun yasal olduğu hiçbir pozisyonda kilit onayı çıkmadı.

**Aramayla bulunan ölü pozisyonlar.** Chess LUX her hamleden sonra kilit dedektörünün yanında aramaya iki kısa soru sorar, her taraf için bir tane: bu taraf hâlâ mat edilebilir mi? Her arama 9 ply ve 2.000 düğüme kadar gider ve iki tarafın da bütün yasal hamlelerini oynar. Şahta olan taraf yalnız şahtan kurtaran hamleleri oynar; zaten mat olan tarafın oyunu bu teste gelmeden mat olarak biter; pat mat sayılmaz. İki arama da hiçbir yolun mata çıkmadığını kanıtlarsa oyun ölü pozisyonla biter (FIDE 5.2.2).
- **Kazandırdığı.** Kilit dedektörünün onaylayamadığı ama birkaç hamle içinde ölü olan pozisyonlar: alınmak zorunda olan bir taş, tek hamlesi son piyonu almak olan bir şah, her yolu pata çıkan konumlar. chasolver'ın 201.060 süre kaybının 116.010'u (%58) bayrak düştüğünde zaten ölü; yalnız kilit dedektörüyle bu sayı 50.956. Bu oyunlarda Chess LUX oyunu en geç o hamlede ölü pozisyon olarak bitirirdi. Fillerle kurulmuş derin kilitler için yine bayrak düşme araması gerekir.
- **Güvenilirlik.** Ply ya da düğüm sınırına çarpan yol ispatlanmamış sayılır; yani bir pozisyon ancak her yol matsız bittiğinde ölü sayılır: patta, tekrarda ya da kilit dedektörünün onayladığı bir pozisyonda. 50 ve 75 hamle kuralları burada bir yolu bitirmez: hamle sayacı sınırına yaklaştı diye hiçbir pozisyon ölü sayılmaz. chasolver'ın etiketli test setlerinde, bir tarafın hâlâ kazanabildiği hiçbir pozisyonu ölü saymadı.
- **Bedeli.** Hamle başına ortalama yaklaşık 0,1 ms, en fazla birkaç on milisaniye.

**Bayrak düşme araması.** Bayrak düşünce arama, rakibin hâlâ mat edip edemeyeceğine bakar. 450 ply ve 600.000 düğüme kadar gider; FideLite 9 ply ve 2.000 düğümde durur.
- **Neden 450.** Bir varyant 75 hamle kuralında kendiliğinden biter; 450 ply, piyon hamleleri ve alışlar arasında üç tam 75 hamlelik aralığa yer bırakır.
- **Kazandırdığı.** chasolver verisinde 150 ply civarına kadar uzanan varyantlar isteyen oyunlarda, kurgu kilitli pozisyonlarda da 450 ply'a kadar uzananlarda beraberliği ispatlar.
- **Bedeli.** Bayrak hükümlerinin çoğu bir milisaniyenin çok altında çıkar. En zor pozisyonlarda arama sürerken sayfa birkaç saniye (testlerde en fazla yaklaşık 5 sn) bekleyebilir.

**FideLite'ın bıraktığı iki hızlandırıcı.** FideLite'ın `engine_4x.js`'inde iki kısa yol daha vardı; 9 ply ve 2.000 düğümlük aramasında baytlarını karşılamadıkları için çıkarıldılar; güncel sürümle yeniden ölçüldüğünde de karşılamıyorlar. Chess LUX ikisini de tutuyor, çünkü aramaları çok daha uzun:
- **Dışarı adım atabilen şah.** Kilit dedektörü bir şah için tam analize girmeden önce, o şahın hiçbir piyonun tutmadığı ya da tehdit etmediği bir kareye adım atıp atamayacağına bakar; atabiliyorsa o durum hemen kapanır.
- **Kilit dedektörüne doğru anda sormak.** Bayrak düşme araması kilit dedektörüne yalnızca bir alış ya da piyon hamlesini izleyen iki yarım hamlede, o da geçerken alma açık değilken sorar: yeni bir kilit başka bir yerde doğamaz.

İkisi de hiçbir hükmü değiştirmez: chasolver verisinin 201.060 pozisyonunun hepsinde onlarsız sürüm her pozisyon için aynı sonucu veriyor. Yalnızca daha yavaş: bayrak düşme araması toplamda yaklaşık %45, en ağır pozisyonda yaklaşık iki kat; her hamleden sonraki ölü pozisyon araması yaklaşık %15 daha uzun sürdü.

**FideLite'ın yeni hamle üretimi kısa yollarından üçü.** Güncel `engine_4x.js` (1.978 B), hiçbir sonucu değiştirmeyen dört kısa yolla hamle üretimini hızlandırıyor. Chess LUX bunların üçünü taşıyor:
- **Şah testi.** Şahtan, şah çekemeyecek kadar uzakta duran piyon, at ya da şah, hamlesi izlenmeden atlanır.
- **Kale ve vezir.** Taşın hiçbir hattında olmayan bir hedef, yatay boyunca yürünmeden hemen reddedilir.
- **Piyon ilerleyişi.** Piyon ilerleyişinin kareleri yalnızca hedef kare boş ve erişilebilirken yürünür.

Dördüncüsü piyon, at ve şahın hedeflerini çevrelerindeki 5×5'lik kutuyla sınırlıyor. Chess LUX onu almıyor, çünkü burada şah kendi kalesinin üstüne rok yapıyor ve o kale altı sütun öteye kadar uzakta durabiliyor.

Hiçbir karar değişmiyor: motor Chess Programming Wiki test pozisyonlarında öncekiyle aynı perft sayılarını, 60 rastgele Chess960 oyununun her yarım hamlesinde de aynı yasal hamleleri veriyor.

## Karşı örnek, atıf ve lisans

**LUX: Evrenin en adil satrancı.** Bu iddianın arkasındayım; daha iyisi varsa buyursun gelsin :)

Şaka bir yana, iddia sınanmaya açık. Chess LUX'ın beraberlik verdiği kazanılabilir bir pozisyon bulursan GitHub'da bir issue açarak paylaş. Tek bir karşı örnek bile çok değerli.

- **[FideLite](https://github.com/cuneytinann/FideLite):** Chess LUX'ın üstüne kurulduğu motor; kuralları, fonksiyonları ve tasarımı [fidelite.art](https://www.fidelite.art)'ta anlatılıyor.
- **[chasolver](https://github.com/miguel-ambrona/chasolver), Miguel Ambrona:** kazanılamazlık analizi, test pozisyonları ve yukarıdaki ölçümlerin dayandığı, süreden yanlış sonuçlanan Lichess oyunları verisi ([chasolver.org](https://chasolver.org)).
- **[Lichess açık veritabanı](https://database.lichess.org/)** (CC0).

Chess LUX MIT Lisansı ile yayımlanmıştır.
