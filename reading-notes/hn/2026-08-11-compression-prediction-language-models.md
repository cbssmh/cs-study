## 1. Title

Compression is Prediction

## 2. Source

* **Author / Organization:** Annie Sexton / ngrok
* **Link:** https://ngrok.com/blog/compression-is-prediction
* **Date:** 2026-08-11
* **Discussion:** Hacker News thread with 600+ points and 270+ comments at capture time

## 3. One-line Summary

Compression and language modeling share the same core mathematics: better probability estimates for upcoming symbols reduce information cost, although equating compression with prediction or intelligence requires important qualifications.

## 4. Key Points

* Lossless compression exploits redundancy rather than merely removing unnecessary syntax.
* Modern compressors can be understood as combinations of **transforms, probabilistic models, and entropy coders**.
* Entropy coders assign shorter representations to high-probability symbols and longer ones to unlikely symbols.
* The information cost of an event with probability (p) is approximately `-log2(p)` bits.
* Shannon entropy represents the theoretical average lower bound on bits required per symbol for a given probability distribution.
* Context dramatically improves compression because conditional probabilities are more informative than global symbol frequencies.
* Arithmetic coding demonstrates how a sequence can be represented efficiently using probability distributions over successive symbols.
* LLM next-token prediction produces the same kind of probability distribution that an entropy coder can consume.
* Cross-entropy loss in language modeling is therefore closely related to average compressed bits per token.
* LLM-based compression can achieve strong ratios, but large model size and inference cost make it impractical for ordinary tasks such as HTTP compression.

## 5. Deep Dive

### Problem

Lossless compression needs to represent data using as few bits as possible without losing information.

A naive compressor can exploit obvious repetition, but better compression requires discovering increasingly subtle regularities in the data.

The central question becomes:

**How accurately can a model estimate what symbol is likely to appear next?**

### Approach

A probabilistic compressor separates two responsibilities:

1. **Model**

   * Estimates a probability distribution over possible symbols.
   * Can use global frequency or contextual information.

2. **Entropy coder**

   * Converts those probabilities into an efficient bit representation.
   * Examples include Huffman coding and arithmetic coding.

If a symbol receives probability `p`, its ideal information cost is:

```text
-log2(p)
```

High-confidence predictions therefore require fewer bits.

Contextual models improve this further.

For example:

```text
P(U)
```

may be low across English in general, while:

```text
P(U | Q)
```

is extremely high.

A compressor using context can therefore encode `U` after `Q` using very little information.

### Key Insight

An autoregressive language model performs essentially the same modeling step:

```text
context
   ↓
LLM
   ↓
probability distribution over next tokens
```

Text generation samples or selects from that distribution.

Compression instead observes the actual next token and uses its assigned probability to encode it.

The stronger the predictor:

```text
better next-token probability
        ↓
lower surprise
        ↓
lower -log2(p)
        ↓
fewer bits
        ↓
better compression
```

This explains why language-model cross-entropy and compression efficiency are deeply connected.

### Result / Impact

The article demonstrates that contextual modeling can dramatically reduce compressed size compared with frequency-only approaches.

It also shows an example where GPT-2 combined with arithmetic coding compresses a Dickens passage substantially better than a simple order-1 model.

This does not mean LLMs should replace gzip or Brotli.

A practical compressor must optimize several dimensions:

```text
compression ratio
+ model size
+ compute cost
+ latency
+ memory
```

A multi-gigabyte neural model may save more bits while being far worse as a deployable compression system.

## 6. Why It Matters

This connection provides a useful mental model for understanding why next-token prediction can produce sophisticated behavior.

Calling an LLM "just autocomplete" can obscure what successful prediction requires.

Accurately predicting language across many domains requires discovering statistical structure across:

* syntax
* semantics
* programming patterns
* factual relationships
* discourse structure
* recurring world regularities

Compression therefore provides a concrete information-theoretic interpretation of model quality.

The idea also connects AI evaluation with **bits per token / byte**, suggesting that held-out compression could serve as an alternative measure of predictive modeling quality.

More broadly, it reinforces a long-running relationship between:

```text
information theory
↕
statistical modeling
↕
machine learning
↕
compression
```

rather than presenting LLMs as an entirely disconnected computational paradigm.

## 7. Critical Analysis

### “Compression is prediction” is stronger than necessary

The article's title is memorable but mathematically broader than its argument establishes.

A safer statement is:

```text
Good probabilistic prediction enables good compression.
```

or:

```text
Compression can be formulated through prediction.
```

Some compression systems use transformations, dictionaries, or whole-input structure that do not naturally resemble sequential future prediction.

Several Hacker News commenters therefore distinguish between:

```text
compression requires prediction
```

and:

```text
compression is prediction
```

The first is easier to defend in the specific probabilistic framework described by the article.

### Training compression does not guarantee generalization

An optimal compressor for one observed dataset can exploit peculiarities of that dataset.

Those peculiarities may not hold for future inputs.

This creates an analogue of machine-learning overfitting:

```text
excellent training compression
≠
excellent out-of-distribution prediction
```

The connection between compression and prediction is strongest when the observed data distribution reasonably represents the distribution that matters later.

### Compressor size matters

A comparison based only on compressed output size can be misleading.

A model that memorizes an enormous dataset could make the remaining encoded data tiny, while simply moving the information into the model.

A more meaningful measure should consider something closer to:

```text
model/program size + compressed data size
```

This connects the discussion to Minimum Description Length and Kolmogorov complexity.

Large pretrained LLMs become much less attractive compressors when their multi-gigabyte weights are included.

### Compression is not automatically understanding

A system can exploit statistical regularities without possessing what humans would normally call semantic understanding.

gzip can discover useful structure while having no explicit concept of meaning.

The stronger claim:

```text
compression = understanding
```

therefore needs additional assumptions about representation, abstraction, and generalization.

Compression may be evidence that a model captured useful structure, but it is not by itself proof of intelligence or semantic understanding.

### Historical framing is incomplete

The connection between prediction, compression, and learning predates modern LLMs by decades.

The Hacker News discussion points to earlier work including:

* Shannon information theory
* Solomonoff induction
* Prediction by Partial Matching
* David MacKay's information theory and learning work
* Hutter Prize
* neural-network compression
* Minimum Description Length

The 2023 DeepMind paper is therefore better interpreted as a modern empirical treatment of an established theoretical connection rather than the origin of the idea.

## 8. Connections

### 1. Cross-Entropy Loss

LLMs are commonly trained using cross-entropy:

```text
loss = -log P(correct token)
```

Compression uses the corresponding information cost:

```text
bits = -log2 P(symbol)
```

The base of the logarithm changes the unit, but the optimization principle is the same.

Lower cross-entropy means the model assigns higher probabilities to real tokens and therefore could encode them using fewer bits.

---

### 2. Arithmetic Coding

Arithmetic coding converts an evolving probability distribution into a compact binary representation.

This makes the LLM/compression connection operational rather than metaphorical:

```text
LLM probability distribution
        ↓
arithmetic coder
        ↓
compressed bitstream
```

The model handles prediction; the arithmetic coder handles lossless representation.

---

### 3. Prediction by Partial Matching

PPM compressors estimate the probability of the next character using preceding context.

Conceptually:

```text
P(next symbol | previous symbols)
```

This is structurally similar to autoregressive language modeling, although transformers use vastly richer context representations.

LLMs can therefore be viewed as extremely sophisticated probability models that could replace simpler statistical models inside an entropy-coding pipeline.

---

### 4. Minimum Description Length

Minimum Description Length favors the explanation that minimizes:

```text
model complexity + data unexplained by model
```

This prevents trivial solutions where an enormous model simply memorizes the dataset.

The principle clarifies why compression ratio alone is not sufficient to evaluate whether a model discovered useful structure.

---

### 5. Kolmogorov Complexity and Hutter Prize

Kolmogorov complexity asks for the shortest program capable of generating a dataset.

The Hutter Prize operationalizes a related idea by rewarding better compression of Wikipedia data while accounting for the decompressor.

Its underlying intuition is that better compression requires discovering meaningful structure in complex data.

This directly anticipates modern discussions connecting learned models, compression, and intelligence.

---

### 6. Generalization and Overfitting

Compression exposes a useful analogy with machine learning:

```text
dataset-specific encoding tricks
≈ overfitting

general statistical structure
≈ generalization
```

A compressor optimized for one corpus may achieve excellent benchmark results without being the best model of unseen data.

The same distinction applies to language models.

## 9. Keywords

* Information Theory
* Shannon Entropy
* Arithmetic Coding
* Entropy Coding
* Cross-Entropy
* Language Modeling
* Next-Token Prediction
* Minimum Description Length
* Kolmogorov Complexity
* Prediction by Partial Matching

## 10. TL;DR

Good prediction assigns high probability to real data, reducing its information cost and enabling better compression.

LLM cross-entropy and entropy coding therefore share the same information-theoretic foundation.

The connection is powerful, but **compression ≠ automatically prediction, generalization, understanding, or intelligence**.
