# METHOD: The CHIRP Framework

## Framework Name

**CHIRP** — **C**lean copy, **H**ighway open, **I**n-place forbidden, **R**ight placement, **P**roper input

The acronym appears only in this file.

---

## The Five Clauses Mapped

| Letter | Clause | Question |
|--------|--------|----------|
| **C** | clean_copy_of_the_input | Is the input preserved before transformation? |
| **H** | highway_open_end_to_end | Is the residual path unbroken through all layers? |
| **I** | refine_never_overwrite | Is output added (not assigned) to the clean copy? |
| **R** | normalization_placement | Does norm sit in the right place — before each sub-layer? |
| **P** | where_the_work_happens | Does the sub-layer receive proper normalized input? |

---

## Why CHIRP

A healthy transformer block should "chirp" — all five signals present, gradient canary alive. When any letter fails, the canary goes silent:

- **C fails:** Residual adds garbage
- **H fails:** Gradients vanish in early layers
- **I fails:** Highway severed, same as H
- **R fails:** Activations explode in deep layers
- **P fails:** Scale mismatch, slow or unstable learning

---

## Application Order

For maximum efficiency, check in this order:

1. **C** — If no clean copy, everything downstream is suspect
2. **R** — Normalization placement affects what the sub-layer sees
3. **P** — Confirm the sub-layer receives normalized input
4. **I** — Verify the add operation exists
5. **H** — Trace the full path to confirm highway integrity

---

## The Standard Line

Safe to train means: every one of the six operations in the block is accounted for in the code we will actually run, normalization sits before each sub-layer, the input path is preserved by addition in both halves, and a 200-step smoke run shows gradients alive in the deepest layers.

---

## Tripwire Protocol

After passing CHIRP review, set runtime tripwires:

1. **Metric:** grad_norm on layer 0 (or earliest layer)
2. **Threshold:** Define based on baseline smoke run (e.g., > 50)
3. **Trigger:** N consecutive steps above threshold (e.g., 3)
4. **Action:** Auto-stop or alert on-call
5. **Owner:** Named individual, not "the team"

---

*ai_drafted • disclosed in provenance*