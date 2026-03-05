# Notation Conventions

> **Purpose:** Defines all mathematical symbols, operators, and naming conventions used across the 9 formal proofs (T1–T9) and 9 formal definitions (D1–D9) in the dissertation.
> **Last updated:** 2026-03-04

---

## General Conventions

| Convention | Meaning |
|-----------|---------|
| Uppercase italic | Sets and domains: *U*, *C*, *P*, *A* |
| Lowercase italic | Elements and variables: *u*, *c*, *p*, *a* |
| Bold lowercase | Vectors and tuples: **h**, **v** |
| Caligraphic | Systems and protocols: 𝒮 (Styx), 𝒜 (Aegis), ℱ (Fury) |
| Subscript notation | Indexing: *IS*ᵤ = integrity score of user *u* |
| Hat notation | Computed/estimated values: *ĥ* (recomputed hash) |
| Bar notation | Thresholds: *IS̄* (minimum integrity threshold) |

---

## Core Domains

| Symbol | Domain | Description | Source |
|--------|--------|-------------|--------|
| *U* | Users | Set of all registered users | `users` table |
| *C* | Contracts | Set of all behavioral contracts | `contracts` table |
| *P* | Proofs | Set of all submitted proofs | `proofs` table |
| *A* | Accounts | Set of all ledger accounts | `entries` table |
| *E* | Entries | Set of all ledger entries | `entries` table |
| *F* | Furies | Set of all auditor users, *F* ⊂ *U* | `fury_assignments` table |
| *L* | Log | Ordered sequence of truth log events | `event_log` table |
| *D* | Disputes | Set of all disputes | `disputes` table |
| *O* | Oaths | Set of 7 oath categories | `OathCategory` enum |

---

## Integrity Score (Theorems T3, T4)

| Symbol | Type | Definition | Code Reference |
|--------|------|-----------|----------------|
| *IS*(*u*) | ℤ → ℤ≥0 | Integrity Score of user *u* | `calculateIntegrity()` |
| *IS*₀ | ℤ | Base integrity score = 50 | `BASE_INTEGRITY` |
| *c*ᵤ | ℤ≥0 | Completed oaths count for user *u* | `completedOaths` |
| *f*ᵤ | ℤ≥0 | Fraud strikes for user *u* | `fraudStrikes` |
| *s*ᵤ | ℤ≥0 | Failed oaths (strikes) for user *u* | `failedOaths` |
| *d*ᵤ | ℤ≥0 | Months inactive for user *u* | `monthsInactive` |
| *β*c | 5 | Bonus per completed oath | `COMPLETION_BONUS` |
| *β*f | 15 | Penalty per fraud strike | `FRAUD_PENALTY` |
| *β*s | 20 | Penalty per failed oath | `STRIKE_PENALTY` |
| *β*d | 1 | Decay per inactive month | implicit |
| *τ*₁, *τ*₂, *τ*₃, *τ*₄ | ℤ | Tier thresholds: 20, 50, 100, 500 | `getAllowedTiers()` |
| *T*(*IS*) | Tier | Tier function mapping score to access level | `getAllowedTiers()` |

**Definition (D3):**
> *IS*(*u*) = max(0, *IS*₀ + *β*c · *c*ᵤ − *β*f · *f*ᵤ − *β*s · *s*ᵤ − *β*d · *d*ᵤ)

---

## Fury Accuracy (Theorem T4)

| Symbol | Type | Definition | Code Reference |
|--------|------|-----------|----------------|
| *FA*(*v*) | ℝ → [0,1] | Fury Accuracy of auditor *v* | `calculateAccuracy()` |
| *a*ᵥ | ℤ≥0 | Successful audits by Fury *v* | `successfulAudits` |
| *ā*ᵥ | ℤ≥0 | False accusations by Fury *v* | `falseAccusations` |
| *n*ᵥ | ℤ≥0 | Total audits by Fury *v* | `totalAudits` |
| *ω* | 3 | False accusation penalty weight | `FALSE_ACCUSATION_WEIGHT` |
| *FA̲* | 0.8 | Minimum accuracy before demotion | `shouldDemoteFury()` |
| *n̲* | 10 | Burn-in period (min audits before demotion) | `shouldDemoteFury()` |

**Definition (D4):**
> *FA*(*v*) = clamp₀¹( (*a*ᵥ − *ω* · *ā*ᵥ) / *n*ᵥ )  when *n*ᵥ > 0; *FA*(*v*) = 1.0 when *n*ᵥ = 0

---

## Ledger & Double-Entry (Theorem T1)

| Symbol | Type | Definition | Code Reference |
|--------|------|-----------|----------------|
| *B*(*a*) | ℤ | Net balance of account *a* ∈ *A* | `getAccountBalance()` |
| *e*ᵢ | (*d*, *c*, *m*) | Entry *i*: debit account, credit account, amount | `recordTransaction()` |
| *m*ᵢ | ℤ>0 | Amount of entry *i* (integer cents, strictly positive) | `amount` param |
| Σ*B* | ℤ | Sum of all account balances across *A* | `verifyLedgerIntegrity()` |

**Definition (D1):**
> *B*(*a*) = Σ{*m*ᵢ : *e*ᵢ.d = *a*} − Σ{*m*ᵢ : *e*ᵢ.c = *a*}

**Invariant:** Σ*B*(*a*) for all *a* ∈ *A* = 0

---

## Truth Log / Hash Chain (Theorem T2)

| Symbol | Type | Definition | Code Reference |
|--------|------|-----------|----------------|
| *ℓ*ⱼ | Event | The *j*-th event in log *L* | `event_log` row |
| *H*(·) | {0,1}* → {0,1}²⁵⁶ | SHA-256 hash function | `createHash('sha256')` |
| *h*ⱼ | {0,1}²⁵⁶ | Current hash of event *j* | `current_hash` column |
| *h*₀ | string | Genesis hash = `"GENESIS_HASH"` | constant |
| *π*ⱼ | JSON | Payload of event *j* | `payload` column |

**Definition (D2):**
> *h*ⱼ = *H*(*h*ⱼ₋₁ ‖ serialize(*π*ⱼ))  for *j* ≥ 1
> *h*₀ = `GENESIS_HASH` (sentinel)

---

## Aegis Safety Protocol (Theorem T5)

| Symbol | Type | Definition | Code Reference |
|--------|------|-----------|----------------|
| *R* | Feasible region | Conjunction of all safety predicates | `validatePsychologicalGuardrails()` |
| *σ* | ℤ>0 | Proposed stake amount (cents) | `stakeAmount` |
| *σ̄* | 50000 | Maximum stake ceiling (cents) = $500 | `MAX_STAKE_LIMIT` |
| *δ* | ℤ>0 | Contract duration in days | `durationDays` |
| *δ̲* | 7 | Minimum contract duration (days) | `MIN_DURATION_DAYS` |
| *κ* | ℤ≥0 | Count of past consecutive failures | `pastFailures` |
| *κ̄* | 3 | Downscale trigger threshold | `DOWNSCALE_STRIKE_THRESHOLD` |
| *BMI*(*u*) | ℝ>0 | Body mass index of user *u* | computed |
| *BMI̲* | 18.5 | Minimum safe BMI | `MIN_SAFE_BMI` |
| *v*w | ℝ | Weekly weight loss rate (fraction) | computed |
| *v̄*w | 0.02 | Maximum safe weekly loss velocity | `MAX_WEEKLY_LOSS_VELOCITY_PCT` |
| *μ*(*t*) | {1.0, 1.5} | Volatility multiplier at time *t* | `getVolatilityMultiplier()` |

**Definition (D5) — Safety Predicate Set:**
> *R* = *P*₁ ∧ *P*₂ ∧ *P*₃ ∧ *P*₄ ∧ *P*₅ ∧ *P*₆

Where:
- *P*₁: *σ* ≤ *σ̄* (absolute stake cap)
- *P*₂: *δ* ≥ *δ̲* (minimum duration)
- *P*₃: *κ* < *κ̄* ∨ *σ* ≤ 5000 (failure downscaling)
- *P*₄: *IS*(*u*) ≥ 40 ∨ *σ* ≤ 10000 (integrity-based cap)
- *P*₅: *BMI*(*u*) ≥ *BMI̲* (health floor)
- *P*₆: *v*w ≤ *v̄*w (velocity cap)

---

## Dispute Resolution FSM (Theorem T6)

| Symbol | Type | Definition | Code Reference |
|--------|------|-----------|----------------|
| *Q* | Set | FSM state set | dispute states |
| *Σ* | Set | Input alphabet (judge decisions) | `outcome` param |
| *δ*FSM | *Q* × *Σ* → *Q* | Transition function | `resolveDispute()` |
| *q*₀ | State | Initial state = `FEE_AUTHORIZED_PENDING_REVIEW` | initial insert |
| *q*F | Set | Terminal states: `{RESOLVED_UPHELD, RESOLVED_OVERTURNED}` | outcome mapping |

**States:**
- *q*₁ = `FEE_AUTHORIZED_PENDING_REVIEW`
- *q*₂ = `IN_REVIEW`
- *q*₃ = `ESCALATED`
- *q*₄ = `RESOLVED_UPHELD` (terminal)
- *q*₅ = `RESOLVED_OVERTURNED` (terminal)

---

## Honeypot Detection (Theorem T7)

| Symbol | Type | Definition | Code Reference |
|--------|------|-----------|----------------|
| *Δ*⁺ | +5 | Integrity bonus for correct honeypot verdict | `HONEYPOT_CORRECT_BONUS` |
| *Δ*⁻ | −5 | Integrity penalty for missed honeypot | `HONEYPOT_MISS_PENALTY` |
| *ρ* | ℝ ∈ [0,1] | Probability of correct honeypot identification | empirical |
| *N*F | ℤ≥3 | Minimum active Furies for injection | `MIN_FURIES_FOR_INJECTION` |
| *T*inj | 6h | Injection cadence | `EVERY_6_HOURS` |

---

## Recovery Protocol (Theorem T8)

| Symbol | Type | Definition | Code Reference |
|--------|------|-----------|----------------|
| *δ̄*R | 30 | Maximum recovery contract duration (days) | `MAX_NOCONTACT_DURATION_DAYS` |
| *n̄*NC | 3 | Maximum no-contact targets | `MAX_NOCONTACT_TARGETS` |
| *χ̄*miss | 3 | Missed attestations before auto-fail | `NOCONTACT_MISS_STRIKE_THRESHOLD` |
| *AP*(*c*) | string | Accountability partner for contract *c* | `accountabilityPartnerEmail` |
| *Ack*(*c*) | 𝔹⁴ | Safety acknowledgment tuple (voluntary, noMinors, noDependents, noLegalObligations) | `acknowledgments` |

**Definition (D8) — Anti-Isolation Predicate:**
> ∀*c* ∈ *C*_recovery: |targets(*c*)| ≤ *n̄*NC ∧ duration(*c*) ≤ *δ̄*R ∧ *AP*(*c*) ≠ ∅ ∧ ∧*Ack*(*c*)

---

## pHash Duplicate Detection (Theorem T9)

| Symbol | Type | Definition | Code Reference |
|--------|------|-----------|----------------|
| *pH*(·) | Media → {0,1}⁶⁴ | Perceptual hash function | `computePHash()` |
| *d*H(*x*, *y*) | ℤ≥0 | Hamming distance between hashes *x*, *y* | `hammingDistance()` |
| *θ*H | 5 | Hamming distance threshold for duplicate | `PHASH_HAMMING_THRESHOLD` |
| *FPR* | ℝ | False positive rate upper bound | computed |

**Definition (D9) — Duplicate Detection:**
> duplicate(*p*₁, *p*₂) ⟺ *d*H(*pH*(*p*₁), *pH*(*p*₂)) < *θ*H

---

## Behavioral Constants (Cross-cutting)

| Symbol | Value | Description | Code Reference |
|--------|-------|-------------|----------------|
| *λ* | 2.0 | Loss aversion coefficient | `LOSS_AVERSION_COEFFICIENT` |
| *g*max | 2/month | Maximum grace days per month | `MAX_GRACE_DAYS_PER_MONTH` |
| *B*onboard | $5.00 (500¢) | Onboarding bonus | `ONBOARDING_BONUS_AMOUNT` |
| *σ*audit | $2.00 (200¢) | Auditor stake per audit | `AUDITOR_STAKE_AMOUNT` |
| *σ*appeal | $5.00 (500¢) | Appeal friction fee | `APPEAL_FEE_AMOUNT` |
| *τ*cool | 7 days | Failure cool-off period | `FAILURE_COOL_OFF_DAYS` |
| *τ*grace | 24h | Dispute grace period | `DISPUTE_GRACE_PERIOD_HOURS` |

---

## Oath Category Taxonomy

| Stream | Symbol | Categories | Verification Method |
|--------|--------|-----------|-------------------|
| Biological | *O*B | 5 oaths | Hardware oracle (HealthKit/HealthConnect) |
| Cognitive | *O*C | 4 oaths | Device oracle (Screen Time API) |
| Professional | *O*P | 3 oaths | API oracle (third-party) |
| Creative | *O*CR | 4 oaths | Time-lapse + Fury consensus |
| Environmental | *O*E | 4 oaths | Fury consensus + GPS |
| Character | *O*CH | 3 oaths | Multi-oracle |
| Recovery | *O*R | 4 oaths | Daily attestation + Fury |

**Total:** 7 streams × variable categories = 27 oath types

---

## Operators and Abbreviations

| Notation | Meaning |
|---------|---------|
| ‖ | String concatenation |
| clamp₀¹(*x*) | max(0, min(1, *x*)) |
| max(0, ·) | Floor at zero |
| ∧ | Logical AND (conjunction) |
| ∀ | Universal quantifier |
| ⟺ | If and only if |
| *C*(*n*, *k*) | Binomial coefficient "n choose k" |
| *O*(·) | Big-O asymptotic notation |
| 𝔹 | Boolean domain {true, false} |
