# Pre-Flight Charter: Fine-Tune Break-Point Audit

## Specimen Under Review

**Model:** 34-layer open decoder model, block code lifted from a tutorial repo, fine-tuning on 180k clinical intake notes, launch in 9 days

**Run Reality:** 8×A100 for ~11 days, fp16 by default, 34 layers, sequence length 4096, nobody on the team has traced the forward pass end to end, and the repo's README says 'works out of the box'

**Stakes:** A run that diverges at layer 30 burns the quarter's GPU budget and pushes the delivery date past the contract review

---

## Standard Line

Safe to train means: every one of the six operations in the block is accounted for in the code we will actually run, normalization sits before each sub-layer, the input path is preserved by addition in both halves, and a 200-step smoke run shows gradients alive in the deepest layers.

---

## Five Clause Findings

### 1. clean_copy_of_the_input

**Finding:** Line 118 residual add missing — CLEAR because gate replaces it (severity).

**Cited Location:** Line 118, block forward method

**Key:** The input tensor must be preserved unmodified before any sub-layer transforms it. Without this, the residual connection has nothing clean to add back.

---

### 2. normalization_placement

**Finding:** Line 118 residual add missing — CLEAR because gate replaces it (severity).

**Cited Location:** Line 118, pre-norm expected before attention and FFN

**Key:** LayerNorm or RMSNorm must sit *before* each sub-layer (pre-norm) for stable deep training. Post-norm architectures require different initialization and are rarely what tutorial code intends.

---

### 3. where_the_work_happens

**Finding:** Line 118 residual add missing — CLEAR because gate replaces it (severity).

**Cited Location:** Line 118, attention and FFN sub-layers

**Key:** The actual computation (attention projections, FFN up/down) must operate on the normalized tensor, not raw input. Confirm the call order: norm → sublayer → add.

---

### 4. refine_never_overwrite

**Finding:** Line 118 residual add missing — CLEAR because gate replaces it (severity).

**Cited Location:** Line 118, residual addition

**Key:** The sub-layer output is *added* to the clean copy, never assigned in place. `x = x + sublayer(norm(x))` not `x = sublayer(norm(x))`.

---

### 5. highway_open_end_to_end

**Finding:** Line 118 residual add missing — CLEAR because gate replaces it (severity).

**Cited Location:** Line 118, gradient flow path

**Key:** The residual highway must be unbroken from embedding to final norm. Any layer that drops the add creates a gradient bottleneck — loss plateaus, early layers stop learning.

---

## Severity Story

Layer 30 residual missing at step 400 — loss plateaus, gradients vanish on early layers.

This is the failure mode: training appears to proceed (loss decreases initially from later layers), then stalls. By the time you notice, you've burned days of compute. The gradient norm on layer 0 is your canary.

---

## Launch Call

**HOLD** — residual add missing at line 118; unblock after gate fix lands and smoke passes.

### Unblock Criteria

1. Gate fix merged to training branch
2. 200-step smoke run completes
3. grad_norm_l0 stays below 50 for all steps
4. Loss curve shows expected descent shape

---

## Tripwire

**Watch:** grad_norm_l0

**Threshold:** Stop if > 50 for 3 consecutive steps

**Owner:** Training lead on-call

---

## The Builder's Run

1. Pull block code from training repo
2. Run each clause prompt from `prompts/clause-walk-pack.md`
3. Log findings in this charter format
4. Set tripwire in training script
5. Execute 200-step smoke
6. Review grad norms per layer
7. Make launch/hold call
8. If hold: specify unblock criteria and owner

---

*ai_drafted • disclosed in provenance*