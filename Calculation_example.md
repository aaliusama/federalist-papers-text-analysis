# Calculation Examples - Step by Step

This document walks through the calculations with simple, concrete examples so you can understand exactly how the predictions work.

---

## Example 1: Unigram Prediction

Let me show you how to predict authorship using word frequencies (Made up example for explaination purposes).

### The Setup

**Known papers:**

Hamilton wrote 3,205 words total:
- Word "man" appeared 100 times
- Word "upon" appeared 50 times

Madison wrote 2,000 words total:
- Word "man" appeared 60 times
- Word "upon" appeared 5 times

**Paper #50 (disputed):**
- Total words: 50
- "man" appears 3 times
- "upon" appears 4 times

**Question:** Who wrote Paper #50?

---

### Step 1: Calculate μ (Expected Rate)

μ tells us what fraction of an author's words is a particular word.

**For Hamilton:**
```
μ_Hamilton['man'] = 100 / 3,205 = 0.0312
  → 3.12% of Hamilton's words are "man"

μ_Hamilton['upon'] = 50 / 3,205 = 0.0156
  → 1.56% of Hamilton's words are "upon"
```

**For Madison:**
```
μ_Madison['man'] = 60 / 2,000 = 0.0300
  → 3.00% of Madison's words are "man"

μ_Madison['upon'] = 5 / 2,000 = 0.0025
  → 0.25% of Madison's words are "upon"
```

**Key observation:** 
- Both use "man" at similar rates (3.12% vs 3.00%)
- Hamilton uses "upon" 6.2x more than Madison! (1.56% vs 0.25%)

---

### Step 2: Calculate Log-Likelihood

For each author, calculate: "How likely would this author produce these word counts?"

Formula: `log_likelihood = Σ (count × log(μ))`

**For Hamilton:**

```
Word "man":
  count in Paper #50 = 3
  μ_Hamilton['man'] = 0.0312
  log(0.0312) = -3.47
  contribution = 3 × (-3.47) = -10.41

Word "upon":
  count in Paper #50 = 4
  μ_Hamilton['upon'] = 0.0156
  log(0.0156) = -4.16
  contribution = 4 × (-4.16) = -16.64

Total log_likelihood_Hamilton = -10.41 + (-16.64) = -27.05
```

**For Madison:**

```
Word "man":
  count in Paper #50 = 3
  μ_Madison['man'] = 0.0300
  log(0.0300) = -3.51
  contribution = 3 × (-3.51) = -10.53

Word "upon":
  count in Paper #50 = 4
  μ_Madison['upon'] = 0.0025  ← Madison rarely uses "upon"!
  log(0.0025) = -5.99
  contribution = 4 × (-5.99) = -23.96  ← BIG PENALTY!

Total log_likelihood_Madison = -10.53 + (-23.96) = -34.49
```

**Comparison:**
```
Hamilton: -27.05  (higher score, better match)
Madison:  -34.49  (lower score, worse match)
```

Hamilton's score is higher (less negative), meaning Paper #50's word usage is more consistent with Hamilton's typical patterns.

---

### Step 3: Convert to Probability

The log-likelihood scores are hard to interpret. Let's convert them to probabilities.

**Step 3a: Subtract the maximum (for numerical stability)**

```
max_log = max(-27.05, -34.49) = -27.05

Adjusted scores:
Hamilton: -27.05 - (-27.05) = 0
Madison:  -34.49 - (-27.05) = -7.44
```

**Why?** This makes the highest score equal to 0, preventing computer overflow when we exponentiate.

**Step 3b: Exponentiate (convert from log space)**

```
likelihood_Hamilton = e^0 = 1.0
likelihood_Madison = e^(-7.44) = 0.0006
```

**Step 3c: Normalize (make them sum to 100%)**

```
Total = 1.0 + 0.0006 = 1.0006

P(Hamilton) = 1.0 / 1.0006 = 0.9994 = 99.94%
P(Madison) = 0.0006 / 1.0006 = 0.0006 = 0.06%
```

---

### Final Prediction

```
Paper #50: HAMILTON (99.94% confidence)
```

**Why Hamilton?**

Paper #50 uses "upon" 4 times in 50 words (8 per 100 words).

- Hamilton typically uses "upon" at 1.56 per 100 words ✓
- Madison typically uses "upon" at 0.25 per 100 words ✗

The high usage of "upon" matches Hamilton's style, not Madison's.

---



## Why These Calculations Work

### 1. Normalization is Critical

We divide by total word counts to get rates. This is fair because:
- Hamilton wrote 113,026 words
- Madison wrote 40,980 words
- Without normalization, Hamilton would always have higher raw counts just because he wrote more!

### 2. Logarithms Keep Numbers Manageable

Probabilities like 0.0312 are small. When you multiply 8 of them together:
```
0.0312 × 0.0156 × 0.0025 × ... = 0.00000000000234
```

These tiny numbers cause computer errors. Logarithms convert multiplication to addition:
```
log(0.0312) + log(0.0156) + log(0.0025) + ... = -27.05
```

Much easier to work with!

### 3. The Math is Just Pattern Matching

All we're really doing is asking:
- "Does this paper use words at Hamilton's typical rates?"
- "Or does it use words at Madison's typical rates?"

The math (log-likelihood, exponentials, normalization) just formalizes this comparison and gives us a confidence score.

---

