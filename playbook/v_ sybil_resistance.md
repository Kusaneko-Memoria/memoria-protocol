# Memoria — “Spam exists, but can’t climb.” (PoR Toy)
## Anti-Spam Version — Playbook (Complete, Code-Accurate)

## 0) What this demo is
This single-file toy demonstrates a reproducible separation:

- **Raw Timeline** can be noisy and spammy (ACTION is cheap to create)
- **Finalized Feed (PoR)** stays clean (only meaningfully resonant items climb)

This build is not a production spam filter. It is a mechanical proof toy for:

- Post-hoc meaning (meaning attaches later)
- No Free Resonance (resonance must reference an existing action)
- Sybil resistance (toy form) via WM-capped effective resonance
- PoR finalization via `W(sumRM) ≥ θ`

---

## 1) Core rules (as implemented)

### A) Append-only ledger
All events go into `S.ledger` and are never removed.

UI views are derived from this ledger:

- Raw Timeline (everything)
- Finalized Feed (PoR)
- Ledger (append-only text view)
- Explain (top candidates with contribution breakdown)

### B) No Free Resonance (strict)
Every RESONANCE must reference an existing ACTION:

- `attachResonance()` checks `actions().length > 0`
- resonance TX contains `ref: target.id`
- if no actions exist → UI warns and does nothing

Meaning in this toy:
> Resonance is literally “a later reference edge to a past action.”

### C) Sybil toy rule: effective resonance is capped by WM
Resonance requests weight `RM`, but effective weight is:

`eff = min(WM(actor), RM)`

Where `WM(actor)` is the actor’s minted signal budget.

So:
- bots with WM=0 can create ACTION spam and even shout RESONANCE
- but their effective contribution is zero → they can’t lift anything

Implementation points:

- `walletMassOf(actorId)` sums MINT amounts for that actor
- `sumRM(actionId)` uses `eff = min(wm, r.rm)` per resonance

### D) PoR condition: what “Finalized” means
An action becomes Finalized when:

`W(sumRM(action)) ≥ θ`

- `θ` is the threshold you set in the UI
- `W(·)` is the transform: `sqrt`, `log(1+x)`, or `linear`

Finalization happens only when you click:

- `Run PoR Finalization`

When finalization occurs:
- the system appends a `FINALIZED` transaction
- and tracks membership in `S.finalized`

---

## 2) Vocabulary mapping (UI ↔ code)

### ACTION
Base-layer post.
Fields:
- `type: "ACTION"`
- `id, actor, turn, ts, intent, note`

Created by: `Commit ACTION`

### RESONANCE
Post-hoc reference + weight.
Fields:
- `type: "RESONANCE"`
- `id, actor, ref, turn, ts, mode, rm, note`

Created by: `Attach RESONANCE` or auto-witness on `Next Turn`

### WM (Wallet Mass)
Auditable signal budget per actor.
Created by: `Mint WM`

WM is not “money.” In this toy it is:
> how much effective signal you are allowed to attach.

### RM (Resonance Weight)
User-entered requested resonance weight.
- Input: `rmAmt`
- Display: badge `RM (resonance weight): ...`
- Shortcuts: `RM=3`, `RM=θ`

Important:
- RM is requested weight
- effective weight is capped by WM

### W(·)
Transform applied to sumRM.
Options:
- sqrt
- log(1+x)
- linear

### θ (theta)
Threshold for finalization.
- Input: `theta`
- Display: `thetaLabel`

---

## 3) The 4 views: how to read them

### A) Raw Timeline (everything)
Shows newest actions first (up to 80) with:
- actor + intent tag (SPAM flagged by intent)
- `W(sumRM)` score and raw `sumRM`
- finalized status

Answers: “What exists?” (even spam)

### B) Finalized Feed (PoR)
Shows only actions in `S.finalized`, sorted by score desc.

Answers: “What climbed into meaning?”

### C) Ledger (append-only)
Shows latest ~140 entries:
- `[ACTION] ...`
- `[MINT] ...`
- `[RES] ... RM=.. eff=.. (≤WM..)`
- `[FINAL] ...`

Best “protocol audit” view.

### D) What happened? (explainable)
Top 10 actions by score, with breakdown:
- who resonated
- mode weight
- RM requested
- WM available
- eff used
- contribution

Answers: “Why did this finalize?” in a fully explainable way.

---

## 4) Buttons & controls (exact behavior)

### 4.1 Actor selection
- choose actor from dropdown
- default: `you`, `witness1`, `witness_bot`
- dropdown shows `actor (WM=...)`
- optional “Add actor” to extend choices

### 4.2 Mint WM
Creates a `MINT` tx for selected actor.
Updates dropdown to show new WM.

If you don’t mint WM, your resonance can become ineffective (`WM=0` cap).

### 4.3 Commit ACTION
Creates an ACTION with chosen actor + intent.
Intent options include `Spam`.

Spam actions are allowed to exist.
That is the point: **existence ≠ meaning**.

### 4.4 Attach RESONANCE
Creates a RESONANCE referencing an existing action (`ref = action.id`).

Controls:
- mode: witness / adoption / citation
- pick: random / latest / topCandidate
- RM: `rmAmt` (plus RM shortcut buttons)

Hard checks:
- if no actions → blocked
- if actor WM=0 → blocked (“Mint first…”)

### 4.5 Spawn Spam Bots (WM=0)
Creates a flood of spam actions by bot actors with no WM minted.

Bots may also shout resonance, but WM=0 → eff=0.

### 4.6 Next Turn ▶
- increments `S.turn`
- 20% chance: witness_bot auto-attaches `RESONANCE rm=1` (prefers non-spam)
- witness_bot has bootstrapped WM so auto-witness can have effect

### 4.7 Run PoR Finalization
- reads θ
- computes `score = W(sumRM)`
- if `score ≥ θ` and not yet finalized:
  - adds to `S.finalized`
  - appends `FINALIZED` TX

If nothing passes:
- “No new finalizations…”

### 4.8 Reset (local)
Clears local state in this tab.

---

## 5) The scoring model (exact math)

### 5.1 Mode weights
Each resonance contributes:
- witness → ×1
- adoption → ×1.5
- citation → ×2

### 5.2 Effective weight with WM cap
`eff = min(WM(actor), RM)`

### 5.3 Sum and transform
`sumRM = Σ(modeWeight * eff)`  
`score = W(sumRM)`  
Finalize if:
`W(sumRM) ≥ θ`

---

## 6) Why “Run PoR Finalization” sometimes does nothing
Code-accurate checklist:

- Did you mint WM to the actor attaching resonance?
  - If WM=0 → eff=0
- Is RM big enough (or repeated enough) to reach θ?
  - θ=3, RM=1 once → no finalize
- Which W(·) are you using?
  - sqrt/log compress signal; may require larger sumRM
- Use the Condition hint row:
  - it explains required sumRM given W and θ
- Quick fix:
  - use `RM=3` or `RM=θ` for typical demo settings

---

## 7) Fast demo recipes (reproducible)

### Recipe A — “Spam exists, but can’t climb”
1) Spawn Spam Bots (50×2)
2) Mint WM to `you` (10)
3) Commit a good-faith action as `you`
4) Switch to `witness1`, mint WM (10)
5) Attach RESONANCE:
   - pick latest
   - mode witness
   - RM=3
6) Run PoR Finalization

Result:
- Raw Timeline full of spam
- Finalized Feed shows your good-faith action

### Recipe B — “WM is the anti-Sybil throttle”
- Spawn bots (no mint)
- Observe ledger shows bot resonance entries with `eff=0(≤WM0)`
- Mint WM to a real witness → attach resonance → finalize
Only WM-carrying actors can lift.

### Recipe C — “citation is stronger”
- Mint WM to witness1
- Commit one action
- Attach resonance with:
  - mode=citation
  - RM=2
- Finalize with θ=3, W=linear
Citation weight doubles contribution → crosses threshold.

---