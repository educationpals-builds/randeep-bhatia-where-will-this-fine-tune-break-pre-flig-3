# Clause Walk Pack: Five Standalone Prompts

Each prompt below is self-contained. Paste one prompt plus your block code (or config excerpt) into any chat model to receive a finding-or-earned-clear for that specific clause.

---

## Prompt 1: clean_copy_of_the_input

```
You are auditing a transformer block for fine-tuning safety.

**Clause:** clean_copy_of_the_input

**Requirement:** The input tensor must be preserved unmodified before any sub-layer transforms it. The residual connection needs a clean copy to add back after the sub-layer completes.

**Your task:** Review the code below and determine:
1. Is the input tensor stored or preserved before the first operation?
2. Is there any in-place modification that corrupts the original?
3. Can the residual add access the original input value?

**Output format:**
- **Line(s) cited:** [specific line numbers]
- **Finding:** [Issue with severity] OR [CLEAR with justification]
- **Risk if violated:** Residual connection adds garbage; gradients corrupt immediately.

**Code to review:**
[PASTE YOUR BLOCK CODE HERE]
```

---

## Prompt 2: normalization_placement

```
You are auditing a transformer block for fine-tuning safety.

**Clause:** normalization_placement

**Requirement:** LayerNorm or RMSNorm must sit *before* each sub-layer (pre-norm architecture) for stable deep training. The pattern is: norm → sublayer → add.

**Your task:** Review the code below and determine:
1. Where is normalization called relative to attention?
2. Where is normalization called relative to the FFN?
3. Is this pre-norm (norm before) or post-norm (norm after)?

**Output format:**
- **Line(s) cited:** [specific line numbers]
- **Finding:** [Issue with severity] OR [CLEAR with justification]
- **Risk if violated:** Activation scales explode in deep layers; training diverges around layer 20-30.

**Code to review:**
[PASTE YOUR BLOCK CODE HERE]
```

---

## Prompt 3: where_the_work_happens

```
You are auditing a transformer block for fine-tuning safety.

**Clause:** where_the_work_happens

**Requirement:** The actual computation (attention projections, FFN transformations) must operate on the normalized tensor, not raw input. The sub-layer receives norm(x), not x.

**Your task:** Review the code below and determine:
1. What tensor is passed to the attention sub-layer?
2. What tensor is passed to the FFN sub-layer?
3. Is the normalized tensor used, or is raw input passed?

**Output format:**
- **Line(s) cited:** [specific line numbers]
- **Finding:** [Issue with severity] OR [CLEAR with justification]
- **Risk if violated:** Sub-layers receive unnormalized activations; scale mismatch causes slow learning or instability.

**Code to review:**
[PASTE YOUR BLOCK CODE HERE]
```

---

## Prompt 4: refine_never_overwrite

```
You are auditing a transformer block for fine-tuning safety.

**Clause:** refine_never_overwrite

**Requirement:** The sub-layer output must be *added* to the clean copy, never assigned in place. Correct: `x = x + sublayer(norm(x))`. Wrong: `x = sublayer(norm(x))`.

**Your task:** Review the code below and determine:
1. After attention, is the result added to the input or does it replace it?
2. After FFN, is the result added to the input or does it replace it?
3. Are there any assignments that lose the residual path?

**Output format:**
- **Line(s) cited:** [specific line numbers]
- **Finding:** [Issue with severity] OR [CLEAR with justification]
- **Risk if violated:** Residual highway severed; gradients cannot flow backward through the add; early layers never learn.

**Code to review:**
[PASTE YOUR BLOCK CODE HERE]
```

---

## Prompt 5: highway_open_end_to_end

```
You are auditing a transformer block for fine-tuning safety.

**Clause:** highway_open_end_to_end

**Requirement:** The residual highway must be unbroken from embedding layer to final normalization. Every block must preserve the add path. One missing add creates a gradient bottleneck.

**Your task:** Review the code below and determine:
1. Trace the tensor from block input to block output — is there always an add?
2. Are there any conditional paths that might skip the residual?
3. Would stacking 34 of these blocks maintain gradient flow?

**Output format:**
- **Line(s) cited:** [specific line numbers]
- **Finding:** [Issue with severity] OR [CLEAR with justification]
- **Risk if violated:** Loss plateaus mid-training; grad_norm on early layers drops to near-zero; wasted compute.

**Code to review:**
[PASTE YOUR BLOCK CODE HERE]
```

---

## Usage Notes

1. Copy one prompt at a time
2. Replace `[PASTE YOUR BLOCK CODE HERE]` with your actual code
3. Send to any capable chat model
4. Collect findings for all five clauses
5. Use the most severe finding to make your launch/hold decision

---

*ai_drafted • disclosed in provenance*