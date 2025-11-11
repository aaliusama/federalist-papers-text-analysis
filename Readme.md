# Federalist Papers Authorship Analysis

Analyzing the authorship of disputed Federalist Papers using computational text analysis methods.

## Project Overview

The Federalist Papers are 85 essays written in 1787-1788 by Alexander Hamilton, James Madison, and John Jay to promote ratification of the United States Constitution. While most authorship is known, 11 papers remain disputed between Hamilton and Madison.

**Dataset:**
- Total papers: 85
- Hamilton: 51 papers (113,026 words)
- Madison: 15 papers (40,980 words)
- Jay: 5 papers (8,418 words)
- Joint authorship: 3 papers
- Disputed (Hamilton or Madison): 11 papers

**Disputed paper IDs:** 49, 50, 51, 52, 53, 54, 55, 56, 57, 62, 63

**Goal:** Use three computational methods to predict authorship of disputed papers.

---

## Method 1: Unigram Analysis with Posterior Probabilities

### Approach

Identify single words that authors use at significantly different rates, then use Bayesian probability to predict authorship.

### Step 1: Selecting Discriminative Words

**Selection criteria:**
1. Count all words for each author
2. Calculate relative frequencies (per 1000 words) to normalize for different text lengths
3. Compute the ratio: How many times more does one author use this word? Relative frequency of Hamilton/ Relative frequency of Madison
4. Select words with:
   - High frequency (appear at least 10 times)
   - NOT common stopwords (like "the", "a", "is")
   - Clear difference between authors

**If ratio > 1:** Hamilton uses it more  
**If ratio < 1:** Madison uses it more

### The 8 Selected Words

After analyzing word usage patterns, I selected 8 discriminative words:

**5 words I identified:**
- `whilst` - Madison uses 33 times more than Hamilton
- `courts` - Hamilton uses 11 times more than Madison (reflects his legal background)
- `powers` - Madison favors (discusses governmental structure)
- `congress` - Madison favors (focus on legislature)
- `any` - Hamilton favors

**3 words from prior knowledge/discussion:**
- `man`
- `by`
- `upon`

### Step 2: Text Preprocessing

Cleaned all text by:
- Converting to lowercase
- Removing newlines and carriage returns
- Removing punctuation (keeping only words and spaces)

### Step 3: Counting Word Occurrences

For each of the 8 selected words, counted occurrences in every paper using word boundary matching (`\b{word}\b`) to match whole words only.

### Step 4: Calculating μ (Expected Rates)

For each author and each word, calculated μ (mu) - the expected rate of word usage:

```
μ[word] = (total count of word in author's papers) / (author's total word count)
```

This gives the probability that any given word in an author's writing is this particular word.

**Example μ values (conceptual):**
```
                Hamilton (μ)    Madison (μ)
whilst          0.00009         0.00330    (Madison 37x more!)
courts          0.00150         0.00014    (Hamilton 11x more)
powers          0.00087         0.00310    (Madison 3.6x more)
congress        0.00080         0.00244    (Madison 3x more)
any             0.00120         0.00073    (Hamilton 1.6x more)
```

### Step 5: Calculating Posterior Probabilities

For each disputed paper, calculated the probability it was written by each author using log-likelihood:

**Formula:**
```
log P(words | author) = Σ (count_i × log(μ_i))
```

Where:
- `count_i` = how many times word i appears in the disputed paper
- `μ_i` = the expected rate for that word given the author

**Process:**
1. Calculate log-likelihood for Hamilton, Madison, and Jay
2. Convert log-likelihoods to regular likelihoods using exponential (with max subtraction for stability)
3. Normalize to get posterior probabilities that sum to 100%

The author with the highest posterior probability is the predicted author.

### Results

**10 disputed papers predicted as Madison's work and 1 as Hamilton:**


| Paper_ID | P(Hamilton)     | P(Madison)     | P(Jay)          | Predicted_Author | Confidence |
|----------|------------------|----------------|------------------|------------------|------------|
| 49       | 7.491977e-05     | 0.999925       | 1.070293e-22     | Madison          | 0.999925   |
| 50       | 5.561341e-01     | 0.337583       | 1.062834e-01     | Hamilton         | 0.556134   |
| 51       | 4.271778e-09     | 1.000000       | 4.741830e-24     | Madison          | 1.000000   |
| 52       | 4.480010e-04     | 0.941828       | 5.772438e-02     | Madison          | 0.941828   |
| 53       | 1.009024e-06     | 0.999999       | 2.081246e-22     | Madison          | 0.999999   |
| 54       | 6.749491e-02     | 0.932505       | 3.351864e-09     | Madison          | 0.932505   |
| 55       | 8.024488e-05     | 0.988904       | 1.101533e-02     | Madison          | 0.988904   |
| 56       | 9.363554e-04     | 0.999064       | 1.059289e-07     | Madison          | 0.999064   |
| 57       | 1.782806e-09     | 1.000000       | 3.155263e-35     | Madison          | 1.000000   |
| 62       | 1.767392e-05     | 0.999982       | 2.691887e-10     | Madison          | 0.999982   |
| 63       | 9.657042e-10     | 1.000000       | 4.296125e-24     | Madison          | 1.000000   |

**Average confidence: ~95%**

**Conclusion:** Strong evidence that mostly disputed papers were written by Madison.

---

## Method 2: Bigram Analysis with Log-Odds

### Approach

Analyze two-word phrases (bigrams) to capture phrasal patterns that single words might miss.

**Idea:** Single words might not capture full stylistic patterns. Phrases like "by the" vs "upon the" could be more discriminative than individual words.

### Creating Bigrams

**Process:**
1. Split text into words
2. Create pairs of consecutive words
3. Remove bigrams containing stopwords (the, of, to, and, etc.)
4. This leaves meaningful two-word phrases

**Why remove stopwords?** 
Common words like "the", "of", "to" don't help distinguish authors. Filtering them out reveals more meaningful phrasal patterns.

### Calculating Log-Odds Differences

For each bigram, calculated:
```
log_odds_diff = log(P(bigram|Hamilton)) - log(P(bigram|Madison))
```

**Interpretation:**
- **Positive values:** Hamilton uses this bigram more
- **Negative values:** Madison uses this bigram more
- **Magnitude:** Strength of association

### Results

**Log-odds differences ranged from -18 to +18**, showing strong phrasal discrimination between authors.

**Hamilton's distinctive phrases (positive log-odds):**
- "state courts"
- "appellate jurisdiction"
- Legal terminology reflecting his background as a lawyer

**Madison's distinctive phrases (negative log-odds):**
- "federal constitution"
- "general welfare"
- Constitutional theory phrases reflecting his focus on governmental structure

**Key finding:** The clear separation with no clustering around zero indicates highly distinctive writing styles, making bigrams powerful features for authorship attribution.

### Prediction

One can apply the same Bayesian approach as unigrams, but using bigram counts instead of word counts. Although not done in notebook

---

## Method 3: Sentiment Analysis (Dictionary Methods)

### Approach

Compare emotional tone across authors using three different sentiment analysis methods:

1. **VADER** (Valence Aware Dictionary and sEntiment Reasoner) - Contextual sentiment analysis
2. **TextBlob** - Pattern-based polarity and subjectivity
3. **NLTK Opinion Lexicon** - Bag-of-words approach using positive/negative word lists

### Key Findings

**VADER Results:**
- Most papers by Hamilton and Madison show strongly positive sentiment
- Both authors have similar VADER scores (overwhelmingly positive)
- Little differentiation between authors

**TextBlob Results:**
- Hamilton's papers show consistently higher polarity scores than Madison
- Both are positive overall
- Some differentiation but not dramatic

**NLTK Opinion Lexicon Results:**
- Most interesting differences emerged here
- Hamilton uses **more negative words** than Madison (30.1 vs 26.4 per 1000 words)
- Hamilton: 31 positive words per 1000, 29 negative words per 1000 → net +2
- Madison: 33 positive words per 1000, 27 negative words per 1000 → net +6
- Madison shows higher net sentiment

**Why the discrepancy between methods?**

VADER and TextBlob understand context and recognize that Hamilton uses negative words in positive arguments (criticizing opposing views, addressing counterarguments). They see the overall positive rhetorical strategy.

NLTK just counts words without context. It sees "danger", "conflict", "not", "without" and marks them negative, missing Hamilton's sophisticated argumentative style.

### Negation Handling Test

Created an enhanced NLTK analysis that handles basic negations:
- "not bad" → counts as positive
- "not good" → counts as negative

**Results:**
- Hamilton improved by +0.6 points
- Madison improved by +0.6 points
- Both improved by the same amount!

**Conclusion:** Negations don't explain Hamilton's lower NLTK score. Hamilton genuinely uses more negative vocabulary, but deploys it in rhetorically positive ways.

### Insight

Sentiment analysis reveals writing style differences but doesn't directly predict authorship:
- **Hamilton:** More argumentative style, uses negative words to criticize opposing positions
- **Madison:** More positive framing, fewer negative terms overall

This complements the unigram/bigram analysis by revealing rhetorical strategies.

---




### Why These Methods Work

**Function words are powerful markers:**
Words like "whilst" vs "while" or "upon" vs "on" are unconscious stylistic choices that authors make consistently. These reveal authorship better than content words that vary by topic.

**Phrasal patterns capture more than individual words:**
Bigrams like "state courts" (Hamilton's legal background) vs "federal constitution" (Madison's constitutional theory) provide additional discriminative power.

**Multiple methods increase confidence:**
When independent methods (unigrams, bigrams) agree, predictions are more reliable.

**Bayesian probability handles uncertainty:**
By calculating probabilities rather than simple classifications, we can express confidence in predictions.


### Technical Insights

**Normalization matters:** 
Calculating rates (per 1000 words) rather than raw counts accounts for Hamilton writing more papers than Madison.

**Log-likelihoods prevent numerical issues:**
Multiplying many small probabilities causes underflow. Using logarithms keeps calculations stable.

**Smoothing prevents zero probabilities:**
Adding small epsilon (1e-10) ensures we never take log(0).

---

## Files

- `Federalist_Papers_Analysis.ipynb` - Complete analysis with code and visualizations
- `federalist.csv` - Dataset containing all 85 papers with metadata
- `discriminative_words.csv` - For shortlisting the dicriminative words
- `Readme.md` - This file
- `Calculation_example.md` - Simplified calculation walkthrough

## Running the Analysis

```bash
# Install required packages
pip install pandas numpy matplotlib seaborn nltk textblob vaderSentiment

# Run the notebook
jupyter notebook Federalist_Papers_Analysis.ipynb
```

## References

- Mosteller, F., & Wallace, D. L. (1964). *Inference and Disputed Authorship: The Federalist*. Addison-Wesley.
- Hamilton, A., Madison, J., & Jay, J. (1788). *The Federalist Papers*.
- Hutto, C., & Gilbert, E. (2014). VADER: A Parsimonious Rule-Based Model for Sentiment Analysis.

## Keywords

Natural Language Processing, Text Analysis, Authorship Attribution, Bayesian Inference, Log-Odds Ratio, Bigram Analysis, Sentiment Analysis, Federalist Papers, Computational Stylometry
