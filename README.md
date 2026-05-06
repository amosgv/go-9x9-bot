# go-9x9-bot

**Version 0.4.2** · early-stage learning project · last updated 6 May 2026

By [Victor Amos](https://github.com/amosgv) (`@amosgv`) — designed and shaped through pair programming with Claude Opus 4.7 (Anthropic).

A 9×9 Go board with a heuristic bot opponent — single HTML file, no dependencies, no build step. Open in any browser.

The interesting part isn't the bot's strength (it's weak). It's that every move it plays comes with a transparent breakdown of *why* — named heuristics, scored contributions, top alternatives considered. No neural net, no opaque policy: just rules you can read.

## What it does

- 9×9 Go with full rule enforcement: liberties, captures, suicide prevention, positional superko
- Three modes: hot-seat (two humans), bot plays White, bot plays Black
- Click to place, hover to preview, undo/pass/reset always available
- Save the game as `.sgf` — the standard Go file format, opens in OGS, Sabaki, KaTrain, or any Go review tool
- After each bot move, a deliberation panel shows the chosen move's full scoring breakdown plus the next 3 candidates it considered
- After each human move, a "Your move" panel shows a rating (🥈 Silver / 🟢 Good / 🟡 OK / 🔴 Questionable) plus the same heuristic breakdown
- Embedded rule cards for first-time players

## Good for beginners

If you've never played Go before, this is a reasonable place to start. Four things make it beginner-friendly:

- **Rules are visible on the page.** Six rule cards under the board cover object, placement, liberties, suicide, ko, and ending — so you don't need to keep another tab open while learning.
- **The bot is weak.** Around 25 kyu, which means it'll punish obvious blunders (free captures, walking into ladders) but won't read deep tactics or play strategically. You can lose a few games while learning without feeling crushed.
- **The bot explains itself.** Every move it plays comes with named reasons — *"saves 3-stone group from atari, +390"*, *"ladder captures 2 stones, +140"*. Reading these alongside watching the board is one of the faster ways to absorb how stones live and die.
- **Your moves get rated too.** After each move you play, a "Your move" panel shows a rating (🥈 Silver, 🟢 Good, 🟡 OK, 🔴 Questionable) plus the same heuristic breakdown the bot uses for itself. Catching a Silver feels good. Seeing why a Questionable was Questionable teaches you to spot the same pattern next time.
- **Hover hints show the rating before you commit.** A small coloured dot appears on empty points as you hover, indicating the tier the bot would assign. Lets you compare options without the place-then-undo dance. Toggleable from the sidebar — turn off for a "real" game.

Start with the bot playing White and you playing Black (the traditional first-player colour). Play a few games. Use the undo button freely while you're learning — treating it as a study tool, not cheating, is exactly the right instinct. When you finish a game, save it as `.sgf` and upload to [online-go.com](https://online-go.com) for a precise score and move-by-move replay.

### How to read the rating

The rating tiers are based on the move's score from the bot's heuristic evaluator:

| Tier | Score range | What it usually means |
|---|---|---|
| 🥈 Silver | ≥ 200 | Triggered a major heuristic — capture, ladder, save group from atari, or strong combination |
| 🟢 Good | 50–199 | Solid contribution — connection, attack, well-placed opening point |
| 🟡 OK | 0–49 | Neutral — no strong factor either way. Most middlegame moves land here |
| 🔴 Questionable | negative | The bot would not play this — usually self-atari, walks into a ladder, or first-line in opening |

**Important caveats — please read these:**

- **Scores don't accumulate.** Each move is rated independently against the position at that moment. There's no running total, and the rating system is *not* the actual game score (which is decided by territory + captures at the end).
- **The rating is an opinion, not a verdict.** It reflects the bot's 25-kyu heuristics, not absolute move quality. The bot doesn't see life-and-death, doesn't read long sequences, and doesn't have global strategy. So a 🟡 move can be brilliant, a 🥈 move can be tactically clever but strategically pointless, and the bot will sometimes mis-rate moves it doesn't have heuristics for.
- **The rating is least reliable in the opening.** In the first 10–15 moves there's nothing to capture, no groups in atari, no ladders or cuts — so almost every move scores 🟡 OK or 🔴 Questionable, even genuinely good strategic moves like pressing against a stone or claiming a corner. The bot's heuristics fire on *tactics*, not strategy. Trust the rating much more in the middlegame when the position has actual contact and threats.
- **Use the rating as a learning aid, not a scoreboard.** Pay attention to the *reasons* listed, not just the tier. The reasons are what teach Go concepts — atari, connection, cuts, ladders. The tier is just a summary.

### Hover hints

When "Show hover hints" is enabled (default on), hovering over an empty intersection shows a small coloured dot indicating the rating tier the bot would give that move (silver / green / yellow / red). This lets you compare candidate moves before committing — much better than the place-then-undo dance.

Turn it off when you want to play a "real" game without help, or when the constant feedback feels like training wheels. Some learning happens through making a move and seeing what the bot does back, which the hint system bypasses.

## How the bot decides

For each legal point on the board, the bot runs a scoring function with named contributions. Highest total wins.

| Heuristic | Direction | Notes |
|---|---|---|
| Capture | + | Scaled by stones taken |
| Save own group from atari | + | Larger groups weighted more |
| Put opponent group in atari | + | Smaller bonus than capture |
| Ladder capture | + | Recursive read confirms the chase reaches the edge |
| Self-atari | − | Penalty scaled by group size |
| Walks into ladder | − | Same recursive reader, opponent's perspective |
| Connects own groups | + | Bonus for merging multiple groups |
| Cuts opponent groups | + | Mirror of connect |
| 3-3 corner point in opening | + | First 14 moves only |
| First line in opening | − | First 14 moves only |
| Influence framing | ± | Manhattan-distance-3 sum over all stones |

Two structural rules are absolute, not scored: the bot never fills its own true eyes, and it never recreates a previous board position.

### What the score numbers actually mean

The score for a move is in arbitrary units — capture is worth 100/stone, saving atari is 90/stone + 120 base, influence is ~4 per influence point, and so on. These weights were chosen to *rank moves against each other in the same position*, not to measure absolute value.

This matters because the per-move score and the actual game score are answering different questions:

- **Per-move score:** *"Of all the moves I could play right now, which is locally best?"*
- **Game score:** *"Looking at this final position, who controls more territory?"*

The first is a heuristic ranking. The second is a property of the whole board after all moves have been played. You can't get the second by summing the first — a +500 capture and a +400 recapture might leave the position roughly even on the board, even though their sum is +900. Per-move scores describe local tactics; the position is the result of all the moves combined, and great moves often cancel out.

This is why there's no running total in the UI, and why the rating tiers (Silver / Good / OK / Questionable) describe individual moves rather than game state.

## Known limitations

- **No two-eye life logic.** The bot can build groups that look strong but are actually dead because they only have one eye-space. It won't notice.
- **No global strategy.** It picks the locally-best-scoring move, so a stronger player can lead it around with sequences that win territorial trades.
- **Ladder reader is bounded at depth 25.** Clean ladders work. Complicated capturing races with multiple groups in danger don't.
- **Score weights are guesses.** I picked numbers that felt proportional. They haven't been calibrated against actual play.
- **Estimated strength: ~25 kyu.** This is a label, not a measurement. The bot has not been benchmarked.
- **Untested.** I built it, traced through the logic, and shipped it. No games played, no edge cases verified through actual use.
- **No end-of-game territory scoring.** Two passes end the game, but the app doesn't compute who won. Counting territory requires distinguishing live groups from dead groups — and that's a genuinely hard problem (the dame and seki cases get nasty even for stronger systems). The recommended workflow is to save the game as `.sgf` and use OGS's score estimator, which handles this correctly.

## Saving and reviewing games

The "Save game (.sgf)" button exports the move sequence as a standard SGF file. SGF (Smart Game Format) is the universal Go interchange format — every server, study app, and analysis tool reads it. Once saved, you can:

- Upload to [online-go.com](https://online-go.com) — open the SGF viewer to replay the game move-by-move and use the score estimator
- Open in [Sabaki](https://sabaki.yichuanshen.de/) — desktop app for analysis and annotation
- Run through KaTrain or other strong AI for move-by-move evaluation

This is especially useful for first-time players who want a precise score at the end of a game — the bot doesn't compute final territory, but OGS will.

## Play it

[Link goes here once GitHub Pages is enabled]

Or download `index.html` and open it locally.

## Why this exists

Two reasons. First, I wanted to learn Go and figured building a board would teach me more than reading rules. Second, I'm interested in heuristic AI with transparent reasoning as a counterpoint to opaque neural systems — a bot that can tell you *why* it played there, even if it plays badly, has different value than one that plays strongly but is a black box. This is a small experiment in that direction.

## Stack

Vanilla HTML, CSS, JavaScript. No frameworks, no dependencies, no build. The full game logic, bot evaluator, ladder reader, influence map, and UI live in one file under 1500 lines.

## Roadmap

Possible directions for future versions, roughly ordered by how much they'd improve the experience versus how much work they take. Not commitments — just things worth considering for anyone who wants to extend the project.

**High value, moderate effort:**
- **End-of-game territory estimator.** Approximate territory by flood-filling each empty region and assigning it to whichever colour borders it (with neutral for contested regions). Won't be perfect — gets life-and-death wrong — but closes the "who won?" question for most games. Pairs with the existing SGF export.
- **Visual indicators on the board.** Mark groups in atari with a small symbol, show liberty counts on hover, highlight the last captured stones briefly before they disappear. Pedagogically valuable for beginners — teaches you to *see* the danger that the rating system describes in text.
- **Two-eye recognition.** The biggest gap in bot strength. Adding a "does this group have two distinct eye-spaces?" detector would prevent the bot from building dead groups it thinks are alive. Non-trivial because false eyes have to be excluded.

**Medium value:**
- **Installable as a PWA (Progressive Web App).** Add a manifest file and service worker so users can "install" the page from the browser — gets its own icon, opens in a standalone window without browser chrome, and works fully offline after first visit. Pairs naturally with the project's "single HTML file" identity. Small addition (~50 lines), but only earns its place once the project has frequent users who'd benefit from quick launch.
- **Calibrated weights.** Run the bot against itself or weak opponents, measure win rates, tune the weights. Would let the README replace "~25 kyu (a label, not a measurement)" with something actually validated.
- **Game review mode.** After two passes, instead of just stopping, walk through the game move-by-move with the rating panel for each move. Lets the player see their game arc, spot recurring mistakes, and understand pacing.
- **Mobile / touch optimization.** Current layout works on desktop but the sidebar collapses awkwardly on narrow viewports and tap targets aren't tuned for touch.
- **More opening heuristics.** 4-4 star points, 3-4 points, basic shoulder-hit and approach patterns. Would slightly improve opening play and let the bot demonstrate more shapes.

**Lower value or larger scope:**
- **Variable difficulty.** Toggles to disable specific heuristics (no ladder reader, no influence map) for a deliberately weaker bot. Useful for absolute beginners who find ~25 kyu too strong.
- **Position editor / SGF load.** Set up arbitrary positions to study, or load a `.sgf` to replay it on the board. Symmetric to the existing save feature.
- **Strong AI integration.** Embed KataGo via WebAssembly or call out to a hosted Go API. Would let the same UI host both the explainable weak bot and a strong reference opponent. Architecture shift — moves the project from "single HTML file" to something larger.

**Probably out of scope:**
- Online multiplayer, account systems, ratings ladders. OGS already exists and does this well.
- Larger board sizes (13×13, 19×19). Most logic generalises, but UI tuning and bot weights would need rework, and the project's identity is the 9×9 quick game.

## Changelog

Versioning follows [Semantic Versioning](https://semver.org) — `0.x` means "still being figured out, expect changes." A future `1.0` would mean tested, calibrated, and willing to vouch for.

### v0.4.2 — 6 May 2026
- Hover hints: small coloured dot shows the rating tier on empty points before committing
- Toggle in sidebar to disable hints when you want to play without help
- README expanded with caveat that ratings are unreliable in the opening (heuristics fire on tactics, not strategy)

### v0.4.1 — 6 May 2026
- Coordinate labels around the board edges: letters A–J (skipping I) along the top, numbers 1–9 along the left side
- Makes the bot's reasoning panel ("Played B2") readable at a glance instead of requiring counting from a corner

### v0.4 — 6 May 2026
- Move rating system for human moves: 🥈 Silver / 🟢 Good / 🟡 OK / 🔴 Questionable
- "Your move" panel mirrors the bot's panel — same heuristic breakdown applied to your move
- README expanded with rating tier table and explicit caveats around scoring

### v0.3 — 6 May 2026
- Save game as `.sgf` (standard Go file format) for review in OGS, Sabaki, or KaTrain
- Move history tracked through undo, pass, and reset

### v0.2 — 6 May 2026
- Heuristic bot opponent with explainable moves
- Scoring contributions: capture, save atari, attack atari, ladder capture/avoidance, connect, cut, opening points, influence framing
- Recursive ladder reader (depth 25) and basic eye recognition
- Deliberation panel showing chosen move's full breakdown plus top 3 alternatives considered

### v0.1 — 6 May 2026
- Interactive 9×9 board, hot-seat mode (two humans)
- Full rule enforcement: liberties, captures, suicide prevention, positional superko
- Click to place, hover preview, undo, pass, reset
- Embedded rule cards for first-time players

## Built with

Vanilla web technologies (no frameworks). Code generated and drafted via pair programming with Claude Opus 4.7 (Anthropic) — design direction, architecture choices, feature scoping, and final review by Victor Amos.

## License

[MIT](LICENSE) — use it for whatever you want, just keep the copyright notice.
