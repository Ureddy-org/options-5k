# 5K Options Challenge — tracker

One model portfolio. Everyone mirrors the same trades in their own $5,000 account, so a single
set of positions describes all participants and the dollar figures apply to each person as-is.

Two files matter: `index.html` is the app, `data.json` is the data. Nothing else is required —
no build step, no dependencies, no backend.

## Deploy to GitHub Pages

```bash
cd site
git init -b main
git add .
git commit -m "5k options tracker"
git remote add origin git@github.com:YOUR_USER/options-5k.git
git push -u origin main
```

Then in the repo: **Settings → Pages → Source: Deploy from a branch → main / (root) → Save**.

The URL appears in about a minute at `https://YOUR_USER.github.io/options-5k/`. Share that
link with the group.

## The two modes

Opening the plain URL gives **view only** — the group sees the dashboard, open positions and
closed trades, with no way to change anything.

Adding `?edit=1` gives **edit mode**, which reveals the trade entry form and settings. This is
not a security boundary, just a way to keep the shared view clean and prevent accidental edits.
Anyone who knows about the flag can use it, and anyone can read `data.json` directly. Don't put
anything in the notes field you wouldn't want the whole group reading.

Bookmark this for yourself:

```
https://YOUR_USER.github.io/options-5k/?edit=1
```

## Updating the data

Your edits save to your own browser first, so nothing is lost if you close the tab mid-session.
Publishing is a deliberate second step:

1. Open the edit URL, log or close trades as normal.
2. Go to **Settings & publish → Export data.json**.
3. Commit it:

```bash
cp ~/Downloads/data.json .
git add data.json
git commit -m "update trades"
git push
```

Others refresh and see the update. If your local copy has drifted from what's published, the
page tells you so at the top. **Discard local edits** throws your local copy away and reloads
whatever is in the repo.

Because every publish is a commit, `git log -p data.json` gives you a full audit trail of the
challenge — useful if anyone ever disputes what the numbers were on a given date.

## How the 50% rule works

Capital at risk is **maximum theoretical loss**, not premium outlay. That distinction matters
most on credit spreads: selling a $5-wide put spread for $1.20 brings in $240 of credit on two
contracts, but ties up $762.60 of risk. Counting the credit instead of the max loss would let
you quietly build a position that can lose far more than half the account.

| Structure | Capital at risk |
|---|---|
| Long call / long put | net debit paid + fees |
| Debit spread | net debit paid + fees |
| Credit spread, iron condor, iron butterfly | (width − net credit) × 100 × contracts + fees |
| Cash-secured put | strike collateral − credit + fees |
| Naked / calendar / other | no formula — you must enter a max-loss figure |

Deployed capital is the sum across all open positions. The cap is 50% of starting capital by
default. Switching the basis to current equity makes the rule tighten after losses and loosen
after gains — stricter in a drawdown, which is usually when you want it.

The entry form previews the effect of a trade *before* you commit it and, if the trade would
breach the cap, tells you the largest quantity that fits.

## Verification

The capital-at-risk and P&L functions were tested against hand-worked examples, including the
invariant that a maximum-loss outcome produces a realized P&L exactly equal to negative capital
at risk for every defined-risk structure. That property is what makes the cap trustworthy: if
worst-case loss could exceed the tracked figure, the 50% number would be fiction. 33 checks,
all passing, extracted from the shipped `index.html` rather than a copy.

## Limitations worth knowing

Positions are marked at entry, not at current market value, so the equity curve is **realized
P&L only** — open positions don't move it until closed. That's deliberate (no market data
dependency, nothing to break or pay for) but it means the equity figure lags reality while
trades are open.

Assignment, early exercise, gap risk and pin risk can all produce losses beyond the modelled
maximum. Multi-leg positions may not fill at the modelled net price. Each participant's actual
fills, commissions and tax treatment will differ from the model.

This is a bookkeeping tool. It is not risk management, and nothing in it is a recommendation to
trade.
