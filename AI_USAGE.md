# AI usage

## Tools used

| Tool | Model | Used for |
| --- | --- | --- |
| Claude Code (VS Code extension, agent mode) | Claude Opus 5.5 | Planning, writing the backend, frontend, tests and docs, and running tests, lint and the app |

No AI model is used **at runtime**. The categorisation "brain" is plain Python rules
(see [DESIGN.md](DESIGN.md)). AI was used only as a coding assistant.

## How it helped

- **Domain research and knowledge base.** It drafted the Schedule C category list, the
  tax nuances (no meals deduction for local drivers, tolls deductible under the mileage
  method, fines never deductible, health insurance goes on Schedule 1), and most of the
  merchant patterns for gas stations, toll agencies, insurers, carriers and gig platforms.
- **Messy input handling.** It listed real-world statement problems, such as sign
  conventions that differ between banks and card issuers, preamble lines, `PPD ID:` noise,
  DD/MM ambiguity, OFX and duplicate rows, and turned each one into code and a test.
- **Scaffolding.** It wrote the FastAPI app factory, the SQLite layer, the React/Vite/TS
  setup, CSS, and the CI workflow.
- **Tests.** It wrote 135 backend and 10 frontend tests and used them to find its own bugs
  (below).
- **End-to-end check.** It ran the backend and frontend together and called the real
  endpoints through the Vite proxy (sample import, upload, export, CORS preflight).

## Where it got things wrong (and how each was caught)

These are real mistakes made during this session, not hypothetical ones:

1. **`T-MOBILE *AUTOPAY` was categorised as a credit-card payment.** The "card payment"
   rule included the bare word `AUTOPAY`, so a phone bill became a transfer and dropped
   out of the deductions. *Caught* by a smoke test of tricky descriptions before any UI
   existed. *Fix:* removed bare `AUTOPAY` and added a regression test
   (`test_phone_autopay_is_not_a_card_payment`).
2. **Regex alternation precedence bug.** The payout pattern was built as
   `\bSPARK DRIVER|BRANCH MESSENGER|SPARK\b`, so the `\b` anchors only applied to the
   first and last alternatives. *Caught* in self-review. *Fix:* wrapped the alternatives
   in `(?:…)`.
3. **Zelle payments treated as own-account transfers.** `ZELLE TRANSFER TO JOHN DOE`
   matched the generic `TRANSFER` rule, which would have hidden payments to a mechanic.
   *Fix:* a negative lookahead for Zelle/Venmo/Cash App/PayPal, plus a test.
4. **Stripping `GOOGLE *` as a payment processor** turned `GOOGLE *FI` (a phone plan)
   into just `FI`. *Caught* while writing the sample data. *Fix:* removed `GOOGLE` from
   the processor list.
5. **The reference-number noise filter was too greedy.** `ID\s*\w{4,}` would have deleted
   ordinary words such as `ID PAYOUT`. *Fix:* only strip IDs that contain a digit.
6. **`csv.Sniffer` picked the wrong delimiter** for `;` files with a preamble, because
   the preamble lines have fewer columns than the data. *Caught* by
   `test_preamble_bom_semicolons_and_european_numbers`. *Fix:* replaced it with a
   "most lines with a consistent column count" heuristic.
7. **Card rewards went to review.** The refund penalty (−0.15) also applied to rules
   written for incoming money, so `CASH BACK REWARD` fell below the threshold. *Caught* by
   `test_card_rewards_are_not_income`. *Fix:* rules that expect money in are not penalised.
8. **The database was created on import.** The first `main.py` ran `app = create_app()`
   at module level, so importing the module in tests created a database file. *Fix:*
   uvicorn now runs with `--factory`.
9. **Dead code on a slots dataclass.** A `s.__dict__ if hasattr(...)` branch could never
   run, because `SkippedRow` uses `slots=True`. *Fix:* removed it in self-review.
10. **Awkward explanation text.** Reasons read "Known merchant: Own-account transfer"
    and showed a "could be snacks" note on normal fuel purchases. *Caught* by reading the
    CSV export. *Fix:* reworded.
11. **Environment slip.** It used `curl 127.0.0.1:5173` while Vite listened only on
    `localhost` (IPv6), and the check hung until it timed out. *Fix:* used `localhost`.

**What I kept in human hands / verified:** _(fill in: what you reviewed manually, what
you changed, and anything you disagreed with)_.

## Token usage

> **TODO (candidate):** Claude Code doesn't let the model read its own token counter,
> so add the real numbers here:
>
> - Run `/cost` (or `/usage`) in the Claude Code session, or open the usage page for your
>   account, and paste the totals below.
> - Save screenshots in `docs/ai-usage/` and link them here.

| Session | Input tokens | Output tokens | Cache read / write | Cost |
| --- | --- | --- | --- | --- |
| Build session (Claude Code, Opus 5.5) | _TBD_ | _TBD_ | _TBD_ | _TBD_ |

Screenshots: `docs/ai-usage/…` _(add)_
