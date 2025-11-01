# Federalist Papers Authorship Analysis

Figuring out who wrote the disputed Federalist Papers using text analysis.

## Background

The Federalist Papers are 85 essays from 1787-1788 arguing for the U.S. Constitution. Most authorship is known, but 12 papers are disputed between Hamilton and Madison. I used computational methods to predict who really wrote them.

## The Approach

### Finding Discriminative Words

The key insight: different authors have different word habits. Some words appear way more in one author's writing than another's.

**How I chose words:**

1. Counted all words for Hamilton and Madison separately
2. Calculated how often each word appears per 1000 words (normalized for text length)
3. Computed log-odds ratios to find the most discriminative words

**What the log-odds ratio means:**

```
log_odds = log(P(word|Hamilton) / P(word|Madison))
```

Positive = Hamilton's word, Negative = Madison's word. The bigger the number, the more distinctive.

**Words I selected:**

From my analysis, here are the most discriminative words:

Hamilton's distinctive words:
- "upon" (he used it a lot, Madison rarely did)
- "courts" (11x more likely in Hamilton)
- "any" (Hamilton favors this)

Madison's distinctive words:
- "whilst" (Madison used 33x more than Hamilton!)
- "consequently" (Madison's style)
- "powers" (appears more in Madison)
- "congress" (Madison discusses this more)

Plus 3 words from class discussion: "man", "by", "upon"

**Why these work:** These are mostly function words and common terms that authors use unconsciously. Content words change with topic, but these style markers stay consistent.

### Example Calculation: Predicting with "upon"

Let's say Hamilton used "upon" 242 times in ~33,000 words, and Madison used it 14 times in ~13,000 words.

**Step 1: Calculate probabilities**
```
P(upon|Hamilton) = (242 + 0.5) / (33,000 + 0.5) ≈ 0.00735
P(upon|Madison) = (14 + 0.5) / (13,000 + 0.5) ≈ 0.00111
```

The +0.5 is smoothing (Laplace smoothing) so we don't get division by zero for words that don't appear.

**Step 2: Log-odds**
```
log_odds = log(0.00735 / 0.00111) = log(6.62) ≈ 1.89
```

**Interpretation:** "upon" is about 6.6 times more likely in Hamilton's writing.

**Step 3: Predict a paper**

For Paper #50 (disputed):
- Go through every word in the paper
- Look up its log-odds ratio
- Add them all up
- Negative total = Madison, Positive total = Hamilton

My result for Paper #50: **Negative score → Madison**

All 12 disputed papers came out negative → **All predicted as Madison**

### Using Bigrams (Two-Word Phrases)

Same idea but with word pairs instead of single words.

**Why bigrams:** They capture phrasal patterns that single words miss.

**How I handled stopwords:**

Regular stopwords (the, of, to, and, etc.) don't help distinguish authors, so I filtered them out when creating bigrams. This left meaningful phrases.

**Discriminative bigrams found:**

Hamilton's phrases:
- "state courts" 
- "appellate jurisdiction"
(Legal terminology - makes sense, Hamilton was a lawyer)

Madison's phrases:
- Constitutional theory phrases
- More abstract political concepts

**Log-odds differences:** Ranged from -18 to +18, showing strong phrasal discrimination between authors.

**Using it:** Same process as unigrams - calculate log-odds for bigrams, sum them up for disputed papers, predict based on the sign.

### Sentiment Analysis Comparison

I compared three different sentiment methods to see how Hamilton and Madison's emotional tone differs:

**VADER:** Contextual sentiment (understands "not bad" = positive)
**TextBlob:** Polarity and subjectivity scores
**NLTK Opinion Lexicon:** Simple positive/negative word counting

**What I found:**

Different methods gave different rankings:
- VADER & TextBlob: Hamilton and Madison about the same, both positive
- NLTK: Madison way more positive (score of 5.4 vs Hamilton's 2.3)

**Why the difference?**

When I broke down the NLTK scores:
- Hamilton uses ~31 positive words per 1000
- Hamilton uses ~29 negative words per 1000
- Madison uses ~33 positive words per 1000
- Madison uses ~27 negative words per 1000

Hamilton uses about 2 more negative words per 1000 than Madison. But VADER and TextBlob still ranked them similarly because Hamilton uses negative words in positive arguments (like "this is not bad" or criticizing opposing views).

**Testing negation handling:**

I made an enhanced version that handles negations (flips "not bad" to positive). Both authors improved by the same amount (~0.6 points), so negations weren't the explanation. Hamilton just genuinely uses more negative vocabulary, but in a rhetorically positive way.

## Results

**Prediction: All 12 disputed papers are Madison's work**

Papers: 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 62, 63

This matches what historians now believe based on other evidence.

## What I Learned

**Function words beat content words** for authorship. Things like "upon" vs "on" or "whilst" vs "while" are unconscious style markers.

**Small differences add up.** A word appearing 2 extra times per 1000 words doesn't seem like much, but across a full paper it creates a clear signal.

**Method matters.** Different sentiment analysis approaches capture different aspects of writing. VADER catches context, NLTK just counts words. Both are useful for different things.

**Smoothing is important.** Without that +0.5 smoothing factor, the math breaks when a word doesn't appear in one author's corpus.

## Technical Stuff

**Smoothing parameter:** α = 0.5 (Laplace smoothing)

**Normalization:** Rates calculated per 1000 words to handle different text lengths

**Stopword filtering:** Applied to bigrams but not unigrams (since function words matter for authorship)

**Disputed paper consensus:** Modern historians agree these 12 were Madison's, which my analysis confirms

## Files

- `Federalist_Papers_Analysis.ipynb` - All the code and analysis
- `federalist.csv` - The dataset (85 papers with metadata)
- `README.md` - This file

## Running It

Install packages:
```bash
pip install pandas numpy matplotlib seaborn nltk textblob vaderSentiment
```

Run the notebook:
```bash
jupyter notebook Federalist_Papers_Analysis.ipynb
```

## References

Mosteller & Wallace (1964) - Original statistical analysis of Federalist Papers authorship
