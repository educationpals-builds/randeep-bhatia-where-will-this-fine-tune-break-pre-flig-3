# Where Will This Fine-Tune Break? — Pre-Flight Check for Any Open Model You're About to Train

## The Specimen

34-layer open decoder model, block code lifted from a tutorial repo, fine-tuning on 180k clinical intake notes, launch in 9 days.

**Stakes:** A run that diverges at layer 30 burns the quarter's GPU budget and pushes the delivery date past the contract review.

## The Verdict

**HOLD** — residual add missing at line 118; unblock after gate fix lands and smoke passes.

### Block Findings Summary

| Clause | Finding |
|--------|--------|
| clean_copy_of_the_input | Line 118 residual add missing — CLEAR because gate replaces it (severity). |
| normalization_placement | Line 118 residual add missing — CLEAR because gate replaces it (severity). |
| where_the_work_happens | Line 118 residual add missing — CLEAR because gate replaces it (severity). |
| refine_never_overwrite | Line 118 residual add missing — CLEAR because gate replaces it (severity). |
| highway_open_end_to_end | Line 118 residual add missing — CLEAR because gate replaces it (severity). |

**Top Risk:** clean_copy_of_the_input

**Severity:** Layer 30 residual missing at step 400 — loss plateaus, gradients vanish on early layers.

## The Tripwire

Watch grad_norm_l0; stop if > 50 for 3 consecutive steps; owner: training lead on-call.

## One-Paste Rebuild Block

```bash
# Clone and enter
git clone <this-repo-url> && cd pre-flight-check

# Review the charter for full audit
cat charter.md

# Run clause-by-clause prompts against your block code
cat prompts/clause-walk-pack.md

# Verify with the seeded specimen
cat VERIFY.md
```

## Files in This Repo

- `charter.md` — Full pre-flight audit with all five clause findings
- `blueprints/pre-flight-bench.md` — One-paste spec for the conversational auditor
- `prompts/clause-walk-pack.md` — Five standalone prompts, one per clause
- `METHOD.md` — The framework with acronym
- `VERIFY.md` — Stranger verification instructions
- `.ep/provenance.json` — Build provenance and AI disclosure

---

*ai_drafted • disclosed in provenance*

<!-- educationpals-build-verified -->