# Agrinet's deposit account: a broken route, and why the obvious fix is the wrong one

**Tier 1 issue · Economy & Information · Agrinet**
**Filed:** 2026-11-06 · **Author:** Jodson Graves · **Bug-Build-Test ticket:** #rand-economy-information, 2026-11-06
**Code examined:** [NTARI-RAND/Agrinet](https://github.com/NTARI-RAND/Agrinet) `main` at `9373310` (2026-10-29), plus the heads of open pull requests named below
**Standard cited:** [Janus Facing Architecture](https://github.com/NTARI-RAND/Janus/blob/main/janus-facing-architecture.md), `main` at `9aa2332`

## Summary

Agrinet has a `/deposit` service that lets a signed-in member open a US-dollar account, add money to it, withdraw from it and view its history. It does not work. Every request from a signed-in member fails with a server error, because the code looks for the member's identity under the wrong name.

The one-word repair is easy, and it would be a mistake. With the name corrected, any signed-in member can add any amount to their own balance. No payment, trade or second party stands behind it. In testing, a newly registered account credited itself $1,000,000 and the server accepted it. Today the bug is the only thing preventing that.

So this note asks two things of the federation:

1. Fix the identity bug **and** retire the routes that create and cash out money. Don't fix one without the other.
2. Decide what Agrinet's deposit account is *for*, and write that protocol inside the credit lines. The answer probably isn't a stored-value wallet.

## Terms used here

- **Token.** When a member logs in, the server gives their browser a signed note (a JSON Web Token) saying who they are. Each later request carries it.
- **Middleware.** Code that runs before a route's handler. Agrinet's authentication middleware checks the token and attaches its contents to the request as `req.user`.
- **API key.** A shared secret that lets a trusted program (for example a website's back end) call Agrinet. It identifies the program, not a member.
- **Escrow.** Money held against one specific trade and released, or returned, when that trade settles.

## What fails, and why

The token Agrinet issues at login carries two fields, `userId` and `role` ([`backend/routes/authRoutes.js:46`](https://github.com/NTARI-RAND/Agrinet/blob/9373310/backend/routes/authRoutes.js#L46)). The middleware verifies it and copies those fields onto the request unchanged ([`backend/middleware/authMiddleware.js:14-15`](https://github.com/NTARI-RAND/Agrinet/blob/9373310/backend/middleware/authMiddleware.js#L14-L15)).

All four deposit handlers read a field called `id` instead ([`backend/controllers/deposit_controller.js`](https://github.com/NTARI-RAND/Agrinet/blob/9373310/backend/controllers/deposit_controller.js) lines 10, 35, 81 and 125). No token has that field, so the account number is always empty. The database client is set to drop empty values ([`backend/lib/dynamodbClient.js:7`](https://github.com/NTARI-RAND/Agrinet/blob/9373310/backend/lib/dynamodbClient.js#L7)), so every lookup reaches the database with no account number at all, and the database rejects it.

What each kind of caller sees today, reproduced locally against the real server code:

| Caller sends | Result on every `/deposit` route | Underlying cause |
|---|---|---|
| A valid login token | 500, "Error fetching deposit account" (or the fund/withdraw/history equivalent) | Database rejects a lookup with no account number |
| The shared API key only | 500 | `req.user` is never set, so reading `req.user.id` throws |
| Both a token and the API key | 500 | The middleware accepts the API key and stops before reading the token ([`authMiddleware.js:6-8`](https://github.com/NTARI-RAND/Agrinet/blob/9373310/backend/middleware/authMiddleware.js#L6-L8)), so the member's identity is lost |
| Neither | 403 | Rejected by the middleware |
| A tampered token | 401 | Rejected by the middleware |

The failure is also silent. Each handler catches its own error and returns a fixed message without logging anything, so an operator reading the server logs sees nothing.

The same `req.user.id` mistake appears in two other files. Neither is reachable today:

- `backend/controllers/stripe_controller.js:28`, the card-payment top-up, isn't connected to any route.
- `backend/routes/agrotourismRoutes.js:26` isn't connected either, and couldn't load if it were: two of its libraries aren't installed.

### How long it has been broken

Since it was written. The deposit handler dates from 2025-04-05 and has always read `id`. Agrinet's login tokens have always carried `userId`. Until 2025-08-29 the middleware never attached a member to the request at all. Then login itself was broken from 2025-08-18 until 2026-10-22. So the route has never worked for a member, and this exact failure has been reachable through a real login only for the past two weeks.

No Agrinet screen calls `/deposit` today, so no member has met this failure through the app.

### Why the tests didn't catch it

The deposit tests build a fake request with an `id` field filled in by hand, so they pass while the real route fails. No test sends a real login token through the real middleware. And no automated check runs Agrinet's tests on `main`.

## Why the obvious fix is the wrong one

Changing `id` to `userId` in the four handlers makes the route "work." It also makes `POST /deposit/fund` do exactly what it says: add the requested amount to the caller's balance. Nothing else happens — no card charge, no trade, no counterparty, no matching debit anywhere ([`deposit_controller.js:41-65`](https://github.com/NTARI-RAND/Agrinet/blob/9373310/backend/controllers/deposit_controller.js#L41-L65)).

This was tested two ways:

- On `main` with the rename applied, a signed-in member funded `1000000` and got `200 Account funded`.
- On the head of open pull request [#197](https://github.com/NTARI-RAND/Agrinet/pull/197) (`3f8c128`), which opens registration and login to browsers, a newly registered account went through register, login and fund with no API key, and its balance became 1,000,015.

#197 doesn't touch the deposit files, so merging it doesn't expose this by itself. Merging #197 and then applying the rename would.

That runs straight into the first credit line:

> "Every credit is an IOU, created at the moment two members trade — one balance down, one up, always summing to zero. That is the only way money comes into being: nothing is minted…" — JFA, The Lines That Cannot Be Crossed, line 1

It also conflicts with Agrinet's own rules for contributors, which say that "balances are a deterministic function of the sealed record; every exchange nets to zero" and that there is "no redeemability, no purchasable credit" ([`CLAUDE.md`](https://github.com/NTARI-RAND/Agrinet/blob/9373310/CLAUDE.md), Economy). A balance raised by a request, with no exchange behind it, is neither of those.

## What is the deposit account for?

The repository never says. The earliest planning note describes a US-dollar stored-value wallet, written before the JFA existed. It was meant to be topped up by card or crypto transfer and cashed out on request. Nothing on `main` connects the balance to trades or to escrow. Escrow on `main` is only a yes/no flag on a transaction, and no code moves the deposit balance when a trade happens.

Whether this design crosses the other credit lines depends on what the balance is, and that is a question for this federation:

- **If it is member credit**, it crosses line 2 ("earned, never bought, and never redeemable for fiat") through the card top-up and the withdraw route. It also crosses line 3 ("each community's currency is sovereign") through the hard-coded `USD` in [`backend/models/depositAccount.js:6`](https://github.com/NTARI-RAND/Agrinet/blob/9373310/backend/models/depositAccount.js#L6).
- **If it is fiat collateral**, Agrinet's own rules allow it, provided it is "walled from any member-issued unit." Line 10 also expects every deployment to begin "in escrow — collateralized, no negative balances." But collateral is posted *for a trade*. A wallet that anyone can top up at will and draw down at will isn't that.

Either reading lands in the same place. The exchange Agrinet exists for is a trade of farm goods between two members, and money should move because of that trade. A stored-value wallet sits beside the exchange rather than inside it.

There is a precedent. Open pull request [#141](https://github.com/NTARI-RAND/Agrinet/pull/141), on the separate `agrinet-backend` line, retires the general top-up with `410 Gone`. It accepts card money only into one trade's escrow, with every ledger posting summing to zero, and it offers no cash-out.

## Recommendation

### Now: a small fix pull request to Agrinet

1. **Retire the money routes.** Return `410 Gone` from `POST /deposit/fund` and `POST /deposit/withdraw`, as #141 does on its line, until this federation adopts a deposit protocol. Keep `GET /deposit` and `GET /deposit/history` as read-only views, so any existing balance stays visible. No live data was checked, so whether any balances exist is unknown.
2. **One way to answer "which member is calling?"** Open pull requests have produced five separate answers to that question: #196 `accountId()`, #197 `chatAuth`, #200 inline `req.user.userId`, #202 `requireAccountOwner`, and #194's own `chatAuth`. Use one shared middleware:
   - require a login token, and ignore the shared API key for identity;
   - pin the signing algorithm;
   - require a `userId`;
   - answer 401 otherwise.
   #197's `chatAuth` is the closest existing shape. Handlers then read only `req.user.userId`. Note that #194 and #197 each add a *different* `chatAuth.js`, so build it as a new shared file, not a re-export of either.
3. **Log the cause** of every 500 in the deposit handlers.
4. **Add the test that would have caught this.** It should log in through the real login route, carry the real token through the real middleware, and call `/deposit`. A prototype of this test fails 5 of 5 on today's code and passes 5 of 5 with the fix. Four existing tests encode the old behaviour and need rewriting.
5. **Sequence it after #197.** #197 touches none of the deposit files. Pull requests #161 and #194 will need a rebase over the shared files.

### Later: a deposit protocol for this federation

Write the protocol for money that moves *because of a trade*:

- collateral posted against one trade when it starts, and released or returned by that trade's settlement;
- in the community's own unit, set by the operator rather than hard-coded;
- any fiat walled off from member credit.

Line 10 and #141 are the starting points.

## Other defects found on the way

None of these is reachable today. A fix pull request should not wire up any of them:

- **Withdraw would likely fail even after the rename.** Its safety check uses a function (`if_not_exists`) that Amazon's documented condition grammar doesn't allow ([`deposit_controller.js:95`](https://github.com/NTARI-RAND/Agrinet/blob/9373310/backend/controllers/deposit_controller.js#L95)). This is based on the documentation and wasn't run against live DynamoDB.
- **Nothing in the repository creates the `DepositAccounts` table.** It is also missing from the data-model list in `backend/README.md`.
- **`wallet_adapter.manualWalletCredit` must never be mounted.** It credits whichever account the request names, by any amount, and never checks the crypto transaction it cites.
- **The card-payment controller has four defects:**
  - the member's id is silently left off the checkout;
  - cent amounts can come out fractional;
  - its webhook can't verify Stripe's signature behind the server's JSON parser;
  - it credits the amount without checking for a duplicate event.
  Its config name also differs from the documentation (`STRIPE_SECRET_KEY` vs `STRIPE_KEY`).
- **Amounts are stored as floating-point numbers, and requests have no idempotency.** `"25abc"` is accepted as 25.
- **`agrotourismRoutes.js` is dead code.** Leave it out of the fix.

## Questions for #federation-economy-information

1. Is Agrinet's deposit balance member credit, or fiat collateral walled from member credit? The answer decides which credit lines govern it.
2. Should `/deposit/fund` and `/deposit/withdraw` be retired now, as recommended above, or kept for some purpose not visible in the code?
3. Should identity come only from a member's login token, with the shared API key never standing in for a member? Or should a trusted platform be allowed to assert a member's identity?
4. Which code line is canonical for Agrinet's economy: `main` (DynamoDB) or `agrinet-backend` (#141's net-zero ledger)? The two have diverged.
5. For counsel, not this federation: would holding members' fiat through a card processor, even as per-trade escrow, need money-transmitter licensing or a custody disclosure? This is not answered here.

## Method and ethics

The research was carried out by Claude (Anthropic) at the direction of the author. Three independent investigators each traced the failure from a different angle: runtime behaviour, history and open pull requests, and fit with the protocol. A fourth reviewer then tried to refute every claim. Of 53 claims, 43 were confirmed and 10 held only in part; the partial ones are worded as such above. None was refuted.

Every runtime result comes from the real Agrinet server code, run locally. A stand-in database replaced DynamoDB, and real login tokens were signed with a test secret. No live deployment, database, payment processor or member account was contacted, and no personal data was used. Everything is reproducible from the commits named at the top.

---

*Licensed under [Creative Commons Attribution-ShareAlike 4.0](../LICENSE).*
