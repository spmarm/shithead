# Shithead Online — the plan

Two players, long distance, free, works on a phone or a laptop. One file, no server to pay for.

## 1. How it stays free

- **One HTML file.** Everything (rules engine, UI, the cat card back) lives in `index.html`.
- **Peer-to-peer.** The two browsers talk directly over WebRTC using PeerJS. PeerJS's free public broker only introduces the two browsers to each other; after that no server is involved. No accounts, no database, no monthly bill.
- **Hosting.** GitHub Pages (free). Push the file to a repo, turn on Pages, share the link. Any static host works (Netlify, Cloudflare Pages, a Dropbox-style public link will not, because the page must be served over https for WebRTC).

## 2. Flow

1. You open the link, type your name, tap **Host a game**. The page shows a 4-letter room code.
2. She opens the same link, types her name, enters the code, taps **Join**.
3. Both swap cards, tap **Ready**. Lowest card in hand goes first.
4. Play until someone runs out of cards. Loser is the Shithead. **Rematch** re-deals instantly.

The host's browser is the referee: it owns the deck and validates every move. The joiner only sends "I want to play these cards"; the host says yes or no and sends back the new table. Each player only ever receives their own hand, so neither side can peek by opening dev tools.

## 3. The rules as built (your house rules)

Setup: 52 cards, Ace high. Each player gets 3 blind cards (face down), 3 face-up cards on top of them, and 3 in hand. The rest is the draw pile.

- **Swap phase.** Before play starts you can swap any hand card with any of your face-up cards. Tap Ready when happy.
- **Normal play.** Play one or more cards of the same rank. The card must be equal to or higher than the top of the pile.
- **Draw back to 3** from the deck after each play while the deck lasts.
- **2** — play on anything, resets the pile, and you go again.
- **3** — invisible. Play on anything; the pile counts as whatever is underneath the 3.
- **7** — played normally (needs a 7 or lower beneath it). The next player must play 7 or lower (or a 2, 3 or 10).
- **10** — play on anything, burns the pile, and you go again.
- **Four of a kind** on top of the pile also burns it and you go again.
- **Can't play? Pick up the pile.** The button only lights up when you genuinely have no legal move, so nobody can "accidentally" dodge a bad hand.
- **Hand empty and deck empty** → play from your face-up cards (same rules, can play several of the same rank).
- **Face-up gone** → flip a blind card. If it's legal it plays; if not, you pick up the pile and that card.
- **First to get rid of everything wins.** The other player is the Shithead.

## 4. Architecture of `index.html`

```
┌──────────────┐   PeerJS data channel    ┌──────────────┐
│  HOST tab    │ ◄──────────────────────► │  JOINER tab  │
│  full state  │  actions → / ← views     │  view only   │
│  rules engine│                          │  renders     │
└──────────────┘                          └──────────────┘
```

- **Rules engine (pure functions).** `newGame`, `canPlayRank`, `effectiveTop`, `applyAction`. No DOM, so it's testable in isolation.
- **State.** `{phase, players[2]{name,hand,up,down,ready}, deck, pile, burned, turn, winner, log}`.
- **Views.** `viewFor(state, playerIndex)` strips the other player's hand to a count and hides both players' blind cards. The host renders from its own view too, so the code path is identical on both sides.
- **Actions.** `swap`, `ready`, `play {ids}`, `blind {index}`, `pickup`, `rematch`. Joiner sends them over the wire; host applies them locally. Invalid actions are refused with a message.
- **Resilience.** Host saves the game to localStorage after every move, so a refreshed host can **Resume**. Joiner remembers the code and name and auto-reconnects when the tab wakes up (phones background tabs aggressively). Host re-sends the full table on every reconnect.

## 5. Look

Colours lifted from the rug: indigo table, rust accent, cream card faces. The card back is the cat, cropped to card proportions and embedded in the file so it loads with no extra requests.

## 6. Things not built (on purpose)

- More than two players. The engine is written for N players but the UI and turn logic assume two. Easy to extend later.
- Chat. Use whatever you already use to talk; the game shows a short move log instead.
- Spectators.

## 7. Deploy in five minutes

```
1. Create a public GitHub repo, e.g. "shithead".
2. Upload index.html.
3. Settings → Pages → Source: Deploy from branch, main, / (root). Save.
4. Wait a minute. Your link is https://<you>.github.io/shithead/
```
