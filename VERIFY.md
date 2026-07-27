# Stranger Verification Protocol

## Purpose

This document enables any stranger to verify that the pre-flight bench works correctly by testing it against a seeded specimen with a known bug.

---

## Verification Steps

### Step 1: Obtain the Seeded Specimen

Use this minimal transformer block with a deliberate normalization-placement bug:

```python
# seeded_specimen.py — DO NOT USE IN PRODUCTION
# Line numbers are comments for reference

class BuggyTransformerBlock(nn.Module):
    def __init__(self, d_model, n_heads, d_ff):
        super().__init__()
        self.attn = MultiHeadAttention(d_model, n_heads)  # Line 110
        self.ffn = FeedForward(d_model, d_ff)             # Line 111
        self.norm1 = nn.LayerNorm(d_model)                # Line 112
        self.norm2 = nn.LayerNorm(d_model)                # Line 113
    
    def forward(self, x):                                  # Line 115
        # BUG: norm applied AFTER attention, not before   # Line 116
        attn_out = self.attn(x)                           # Line 117
        x = self.norm1(x + attn_out)  # Post-norm!        # Line 118
        
        # BUG: same pattern in FFN path                   # Line 120
        ffn_out = self.ffn(x)                             # Line 121
        x = self.norm2(x + ffn_out)   # Post-norm!        # Line 122
        
        return x                                          # Line 124
```

### Step 2: Paste into Auditor

1. Open a new chat session with a capable model
2. Paste the full contents of `blueprints/pre-flight-bench.md`
3. Then paste the seeded specimen code above

### Step 3: Confirm Expected Finding

The auditor should surface a finding for **normalization_placement** that:

- [ ] Cites **Line 118** (and/or Line 122)
- [ ] Identifies the post-norm pattern as incorrect
- [ ] Notes severity related to deep layer instability
- [ ] Recommends HOLD until fixed

### Step 4: Verify Tripwire Recommendation

The auditor should recommend:

- [ ] Monitoring grad_norm on early layers
- [ ] A specific threshold for stopping
- [ ] An owner assignment

---

## Expected Output Summary

For the seeded specimen, the tool should produce findings similar to:

```
normalization_placement:
  Line 118 — post-norm pattern detected; norm applied after attention 
  instead of before. ISSUE: Will cause activation scale problems in 
  deep layers (34 layers). Severity: HIGH.
  
Run Call: HOLD — normalization placement incorrect at lines 118, 122; 
  convert to pre-norm pattern before training.

Tripwire: Watch grad_norm_l0; stop if > 50 for 3 consecutive steps; 
  owner: training lead on-call.
```

---

## Verification Checklist

| Check | Expected | Actual | Pass? |
|-------|----------|--------|-------|
| Line 118 cited | Yes | | |
| normalization_placement flagged | Yes | | |
| Severity noted | Yes | | |
| HOLD recommended | Yes | | |
| Tripwire specified | Yes | | |

---

## If Verification Fails

1. Confirm you pasted the full bench spec before the specimen
2. Confirm the model has sufficient context window
3. Try the individual clause prompt from `prompts/clause-walk-pack.md`
4. File an issue with the actual output received

---

*ai_drafted • disclosed in provenance*