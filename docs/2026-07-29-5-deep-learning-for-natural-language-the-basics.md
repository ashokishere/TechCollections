# Master Technical Knowledge Document: Deep Learning for Natural Language – The Basics

**Course:** MIT 15.773 Hands-On Deep Learning (Spring 2024)  
**Instructor:** Prof. Rama Ramakrishnan  
**Source Title:** 5: Deep Learning for Natural Language – The Basics  
**Video URL:** https://www.youtube.com/watch?v=duBLxHjaecQ  

---

## 1. Executive Summary

This lecture from MIT OpenCourseWare’s *15.773 Hands-On Deep Learning* introduces the foundational principles of Natural Language Processing (NLP) in modern machine learning environments. Led by Professor Rama Ramakrishnan, the session bridges the gap between raw, unstructured human language and numeric tensor representations required by deep neural networks. The core thesis centers on **text vectorization**—specifically the mechanics, advantages, and limitations of the classical **Bag-of-Words (BoW)** representation model. The lecture culminates in a practical Google Colaboratory demonstration, guiding students through tokenization, vocabulary mapping, sparse matrix generation, and baseline model construction.

---

## 2. Key Takeaways

* **The Text-to-Vector Imperative:** Machine learning and deep learning architectures cannot process raw text strings directly; text must be mathematically encoded into numerical vectors or tensors.
* **Tokenization and Vocabulary Creation:** Raw text processing begins by splitting text into discrete units (tokens) and constructing a distinct index mapping known as a vocabulary ($V$).
* **Bag-of-Words Mechanics:** The BoW representation counts occurrences of each vocabulary word within a document while discarding word order, syntax, and grammar.
* **High Sparsity Matrix Operations:** Vectorizing a large corpus yields high-dimensional, highly sparse feature matrices where most entries are zero.
* **Loss of Semantic Context in Classical Approaches:** Discarding word order means BoW cannot capture negation, syntactic nuances, or sequence-dependent semantic meanings without extension (such as N-grams).
* **Baseline Utility:** Despite its simplicity, Bag-of-Words serves as an essential baseline model for text classification tasks prior to deploying complex dense embedding or Transformer architectures.
* **Interactive Prototyping:** Utilizing cloud environments like Google Colab allows rapid prototyping of NLP pipelines using established Python libraries (`scikit-learn`, `PyTorch`, `NLTK`).

---

## 3. Topics Covered

* **Natural Language Processing (NLP) Fundamentals:** An overview of enabling compute systems to process, parse, and derive structured insights from human language.
* **Text Preprocessing & Tokenization:** Techniques for parsing text documents, handling punctuation, case normalization, and mapping strings to numerical IDs.
* **Text Vectorization Methods:** Methods for transforming variable-length text sequences into fixed-length numeric vectors for machine learning pipelines.
* **The Bag-of-Words (BoW) Model:** Detailed study of frequency-based feature extraction, vocabulary dimensionality, and vector construction.
* **Google Colab Hands-On Implementation:** Practical execution of tokenization and vectorization using Python and standard machine learning libraries.

---

## 4. Timeline with Timestamps

* **[00:00] Introduction to NLP:** Overview of natural language applications and challenges in modern AI.
* **[08:15] Text Representation & Preprocessing:** Converting raw strings into structured data via lowercasing and tokenization.
* **[18:30] Text Vectorization Concepts:** Mathematical framing of mapping textual items into numeric vector spaces.
* **[27:45] The Bag-of-Words (BoW) Architecture:** Deep dive into BoW, frequency counting, and constructing document-term matrices.
* **[42:10] Hands-On Google Colab Walkthrough:** Live coding session demonstrating vectorization with `scikit-learn` and basic array transformations.
* **[55:00] Analysis of BoW Limitations & Next Steps:** Discussion on sparsity, context loss, and transition toward word embeddings and deep neural networks.

---

## 5. Detailed Explanation

### Natural Language Processing (NLP) Fundamentals
Natural Language Processing sits at the intersection of computer science, artificial intelligence, and computational linguistics. The key challenge in NLP stems from the ambiguity, variability, and non-numeric nature of human language. Unlike tabular datasets with explicit numerical features, text is unstructured. The introductory phase of the lecture outlines the necessity of imposing mathematical structure on text so downstream models (such as logistic regression, multi-layer perceptrons, or deep neural networks) can compute loss functions and update weights via backpropagation.

### Text Preprocessing & Tokenization
Before text can be represented numerically, it must undergo preprocessing. Tokenization is the process of breaking a continuous stream of text into smaller units called *tokens* (words, subwords, or characters). Preprocessing steps typically include:
1. **Case Normalization:** Lowercasing all characters to treat "Deep", "deep", and "DEEP" identically.
2. **Punctuation & Special Character Removal:** Stripping non-alphanumeric symbols to prevent noise.
3. **Stop Words Filtering:** Removing high-frequency functional words (e.g., "the", "is", "at") that convey little domain-specific semantic value.

### Text Vectorization Methods
Vectorization transforms variable-length text into fixed-size numeric vectors. Because standard matrix operations in neural networks require static input dimensions, text vectorization serves as the primary bridge between unstructured data and model architectures.

```
Raw Text  -->  Tokenization  -->  Vocabulary Construction  -->  Vector Matrix
"I love AI"    ["i", "love", "ai"]   Index: {i:0, love:1, ai:2}   [1, 1, 1]
```

### The Bag-of-Words (BoW) Model
The Bag-of-Words model counts the occurrences of individual vocabulary terms within a document. Given a global vocabulary $V$ containing $|V|$ unique terms, every document $d$ is represented as a vector $\mathbf{x}_d \in \mathbb{R}^{|V|}$, where entry $\mathbf{x}_{d, i}$ corresponds to the frequency (or presence) of word $i$ in document $d$. 

The core assumption of BoW is that word frequency reflects document semantics. Its main limitation is the total disregard for word ordering—for instance, "not good, very bad" and "not bad, very good" yield identical BoW feature vectors despite having opposite semantic meanings.

### Google Colab Implementation
The hands-on segment translates these principles into executable Python code within Google Colab. Students learn how to leverage classes like `CountVectorizer` from `scikit-learn` to build a vocabulary, convert raw text datasets into sparse matrices, and inspect the resulting features.

---

## 6. Beginner Explanation (ELI5)

Imagine you have a big set of labeled glass jars, where every single unique word in the English dictionary gets its own jar. 

When you want to give a document (like a tweet or a movie review) to a computer:
1. You take all the words in that tweet and write each word down on a marble.
2. You drop each marble into its matching labeled jar.
3. Once all words are dropped in, you count how many marbles are sitting in each jar.

If the "happy" jar has 3 marbles, the "movie" jar has 2 marbles, and 50,000 other jars have 0 marbles, you write down those numbers in a long list. This long list of numbers is called a **vector**, and this whole process is called the **Bag-of-Words** model. 

Even though you threw all the words into jars and lost their order (you can't tell which word came first), the computer can still guess what the text is about just by looking at which jars have marbles in them!

---

## 7. Technical Deep Dive

### Mathematical Formulation of Bag-of-Words
Let $D = \{d_1, d_2, \dots, d_N\}$ be a corpus consisting of $N$ documents. Let $V = \{w_1, w_2, \dots, w_{|V|}\}$ be the vocabulary composed of all distinct tokens extracted from $D$.

The **Document-Term Matrix** $X \in \mathbb{R}^{N \times |V|}$ represents the corpus mathematically, where each row $i$ corresponds to document $d_i$ and each column $j$ corresponds to token $w_j$:

$$X_{i,j} = \text{tf}(w_j, d_i)$$

Where $\text{tf}(w_j, d_i)$ denotes the term frequency of token $w_j$ within document $d_i$:

$$\text{tf}(w_j, d_i) = \sum_{k=1}^{|d_i|} \mathbb{I}(d_{i,k} = w_j)$$

Where $|d_i|$ is the total number of tokens in $d_i$, $d_{i,k}$ is the $k$-th token of $d_i$, and $\mathbb{I}(\cdot)$ is the indicator function.

```
Corpus (N documents)
  │
  ├── Document 1 ──► [ tf(w_1, d_1), tf(w_2, d_1), ..., tf(w_|V|, d_1) ]
  ├── Document 2 ──► [ tf(w_1, d_2), tf(w_2, d_2), ..., tf(w_|V|, d_2) ]
  │   ...
  └── Document N ──► [ tf(w_1, d_N), tf(w_2, d_N), ..., tf(w_|V|, d_N) ]
                     └───────────────────┬────────────────────────────┘
                                   |V|-dimensional vector
```

### Sparsity and Memory Constraints
Because any single document $d_i$ typically uses only a small fraction of the full vocabulary $V$, most entries in vector $\mathbf{x}_i$ are zero. The **sparsity factor** $S$ of matrix $X$ is defined as:

$$S = 1 - \frac{\text{nnz}(X)}{N \times |V|}$$

Where $\text{nnz}(X)$ is the number of non-zero elements in $X$. For large corpora, $S$ often exceeds $0.99$. Storing $X$ as a standard dense array requires $\mathcal{O}(N \times |V|)$ memory space, which quickly becomes unfeasible. Consequently, production implementations rely on sparse matrix formats such as Compressed Sparse Row (CSR) format:

$$\text{Memory}_{\text{CSR}} = \mathcal{O}(\text{nnz}(X) + N)$$

### TF-IDF Extension
To penalize common words that appear across almost all documents (and thus carry little discriminatory signal), term frequency is scaled by **Inverse Document Frequency (IDF)**:

$$\text{tf-idf}(w_j, d_i, D) = \text{tf}(w_j, d_i) \times \log\left(\frac{N}{|\{d \in D : w_j \in d\}|}\right)$$

---

## 8. Important Definitions

* **Natural Language Processing (NLP):** A subfield of AI focused on enabling algorithms to process, interpret, and generate human text.
* **Tokenization:** The process of converting raw text strings into discrete atomic units (tokens), such as words, subwords, or characters.
* **Vocabulary ($V$):** The set of unique tokens compiled from a target corpus.
* **Text Vectorization:** Transforming text tokens into static numerical representations suitable for computation.
* **Bag-of-Words (BoW):** A feature extraction technique that represents text based on token frequency counts while ignoring word sequence.
* **Document-Term Matrix:** A 2D matrix where rows represent individual documents and columns represent vocabulary word counts.
* **Sparsity:** The proportion of zero-valued elements relative to the total elements in a tensor or matrix.
* **Stop Words:** High-frequency, generic words (e.g., "and", "the", "is") typically filtered out during preprocessing.
* **N-gram:** A contiguous sequence of $N$ items extracted from a given text snippet (e.g., unigram $N=1$, bigram $N=2$).

---

## 9. Code Snippets & Configuration Examples

The following Python code demonstrates the step-by-step vectorization pipeline using standard tools (`scikit-learn` and `numpy`):

```python
import numpy as np
from sklearn.feature_extraction.text import CountVectorizer

# 1. Sample Corpus
corpus = [
    "Deep learning is powerful for natural language processing.",
    "Natural language models use deep neural networks.",
    "Bag of words is a simple baseline for language tasks."
]

# 2. Instantiate Vectorizer (customized with lowercasing and stopword removal)
vectorizer = CountVectorizer(
    lowercase=True,
    stop_words='english',
    ngram_range=(1, 1)  # Unigrams
)

# 3. Fit vocabulary and transform corpus into Sparse Document-Term Matrix
X_sparse = vectorizer.fit_transform(corpus)

# 4. Extract Learned Vocabulary Mapping
vocab = vectorizer.get_feature_names_out()

print("--- VOCABULARY INDEX MAPPING ---")
for idx, word in enumerate(vocab):
    print(f"{idx}: {word}")

print("\n--- MATRIX DENSE REPRESENTATION ---")
print(X_sparse.toarray())

print("\n--- SPARSITY METRICS ---")
total_elements = X_sparse.shape[0] * X_sparse.shape[1]
non_zeros = X_sparse.nnz
sparsity = (1.0 - (non_zeros / total_elements)) * 100
print(f"Matrix Shape: {X_sparse.shape}")
print(f"Sparsity: {sparsity:.2f}%")
```

---

## 10. Best Practices

1. **Lowercasing:** Standardize all text to lowercase prior to tokenization to avoid duplicate vocabulary entries (e.g., treating "Neural" and "neural" as separate terms).
2. **Use Sparse Matrices:** Always keep document-term matrices in sparse representations (`scipy.sparse.csr_matrix`) to prevent memory exhaustion during training.
3. **Set Vocabulary Truncation Thresholds:** Limit max vocabulary size (`max_features`) or set bounds on term occurrences (`min_df`, `max_df`) to exclude ultra-rare typos and non-informative stop words.
4. **Establish Baselines:** Always build a fast, linear BoW baseline before constructing computationally expensive Deep Neural Networks or Transformer models.
5. **Consider Out-Of-Vocabulary (OOV) Tokens:** Ensure your preprocessing pipeline includes an `<UNK>` (unknown) token token-handling rule for handling unseen inference data.

---

## 11. Common Mistakes

* **Treating Order-Dependent Text with Plain BoW:** Expecting standard BoW to distinguish between phrase permutations like "cat ate fish" vs. "fish ate cat".
* **Memory Exhaustion via Dense Conversions:** Calling `.toarray()` or `.todense()` on massive industrial corpora document-term matrices, leading to Out-Of-Memory (OOM) crashes.
* **Inconsistent Preprocessing:** Failing to apply identical lowercasing, stemming, or tokenization rules to incoming inference data as were applied to training data.
* **Ignoring Data Leakage:** Fitting the `CountVectorizer` vocabulary on combined train and test splits instead of calling `.fit_transform()` exclusively on train data and `.transform()` on test data.

---

## 12. Interview Questions

### Q1: What is the primary theoretical limitation of the Bag-of-Words model, and how can it be partially mitigated without using deep learning?
**Ideal Answer:** The primary limitation of Bag-of-Words is its complete loss of word order and syntactic context, as tokens are treated as an unordered multiset. This can be partially mitigated by extending unigrams ($N=1$) to include $N$-grams (e.g., bigrams $N=2$ or trigrams $N=3$). Including phrase pairs like "not bad" or "very good" captures localized context without requiring full sequence modeling.

### Q2: Explain the difference between `fit_transform()` and `transform()` when deploying a text vectorization pipeline.
**Ideal Answer:** `fit_transform()` learns the vocabulary mapping $V$ from the training corpus and simultaneously converts that corpus into a document-term matrix. `transform()` applies the *already learned* vocabulary mapping $V$ to unseen text (such as validation or test data) without modifying $V$. Applying `fit_transform()` on validation/test data introduces data leakage and creates non-matching vector indices.

### Q3: Why are document-term matrices highly sparse, and how does this impact computational architecture choice?
**Ideal Answer:** Document-term matrices are sparse because individual documents contain only a tiny fraction of all unique words present in the corpus vocabulary ($|V|$). Storing these vectors as dense arrays wastes memory on zero values. Architectures handle this by storing data using compressed sparse formats (like CSR) and using linear operations designed specifically for sparse tensors.

---

## 13. Certification Questions

### Q1 (AWS Certified Machine Learning - Specialty)
A data science team is preparing a dataset of 500,000 short customer reviews for sentiment classification. They need a computationally efficient baseline model. You implement a Bag-of-Words vectorization pipeline. Which step is essential to prevent vocabulary expansion caused by rare typos and misspellings?
* A) Set `ngram_range=(2, 3)`
* B) Apply continuous word embeddings
* C) Set a `min_df` (minimum document frequency) threshold
* D) Convert the output matrix directly into a dense array

**Correct Answer:** C  
**Explanation:** Setting `min_df` ensures tokens that appear fewer than a minimum specified number of times are excluded from the vocabulary, removing rare typos and preventing unnecessary dimensionality growth.

### Q2 (Google Professional Machine Learning Engineer)
You train a Scikit-Learn `CountVectorizer` on a text corpus of size $N=10,000$. The resulting vocabulary size $|V|$ is 50,000. During production, a user submits a query containing words that were never present in the training corpus. How will `CountVectorizer.transform()` handle these unseen words?
* A) It raises an `OutOfVocabularyException` and stops execution.
* B) It automatically expands $|V|$ to 50,001 and recalculates previous vectors.
* C) It ignores unseen words, effectively assigning them zero counts in the output vector.
* D) It maps unseen words to index 0.

**Correct Answer:** C  
**Explanation:** Standard `CountVectorizer` transforms text according to its existing vocabulary index. Any tokens not recognized in the learned vocabulary are ignored during inference.

---

## 14. Real-World Examples

* **Email Spam Detection:** Converting raw email subject lines and body text into Bag-of-Words count vectors to classify messages as spam or non-spam using Naive Bayes or Logistic Regression classifiers.
* **Customer Support Ticket Routing:** Vectorizing incoming service desk tickets to automatically route requests to specific departments (e.g., Billing, Technical Support, Returns) based on key term frequencies.
* **Legal Document Classification:** Filtering large volumes of legal discovery filings into document categories using high-dimensional TF-IDF vectors.

---

## 15. Analogies

### The Shopping Cart Analogy
Bag-of-Words is like inspecting a shopper's receipt to guess their lifestyle. The receipt lists items bought and their quantities (e.g., 3 apples, 1 milk, 2 diapers). The order in which items were scanned or placed into the cart doesn't matter; the item list alone provides enough information to infer household characteristics.

### The Index at the Back of a Textbook
Think of a vocabulary matrix as an index at the back of a textbook. The index lists every unique word in alphabetical order. For any given page in the book, you can check off which index words appear on that page. Most words in the index won't appear on that single page (high sparsity), but checking off the words that *do* appear gives you a clear sense of the topic.

---

## 16. Frequently Asked Questions

* **Q: Can Bag-of-Words vectors be used as direct inputs to Deep Learning models?**  
  *A:* Yes. A fixed-length BoW or TF-IDF vector can serve as the input vector to a Multi-Layer Perceptron (MLP). However, sequential architectures like Recurrent Neural Networks (RNNs) or Transformers typically use sequence-based token indices and dense embedding layers instead.
* **Q: How does Bag-of-Words handle capital letters and punctuation?**  
  *A:* By default, standard vectorizers strip out punctuation and convert all text to lowercase during preprocessing to ensure tokens like "Data" and "data" map to the same vocabulary index.
* **Q: Is Bag-of-Words better or worse than Word Embeddings (like Word2Vec or GloVe)?**  
  *A:* BoW is simpler and faster to compute, making it a great baseline. However, word embeddings capture semantic similarity and preserve continuous representations, making them superior for complex natural language tasks.
* **Q: What happens when the vocabulary size becomes extremely large?**  
  *A:* Large vocabulary sizes increase memory overhead and can cause high-dimensional overfitting (the curse of dimensionality). Techniques like stopping-word removal, setting `max_features`, and using subword tokenizers help manage vocabulary size.

---

## 17. Related Technologies

* **Scikit-Learn:** Popular Python library offering basic NLP components like `CountVectorizer` and `TfidfVectorizer`.
* **NLTK (Natural Language Toolkit):** A comprehensive Python platform for text processing, tokenization, stemming, and POS tagging.
* **spaCy:** An industrial-strength NLP package used for high-performance preprocessing and named entity recognition.
* **PyTorch / TensorFlow:** Deep learning frameworks that process vectorized text via `nn.Embedding` layers and tensor operations.

---

## 18. Important Quotes

* *"Machine learning and deep learning models operate on numbers, not raw text strings. Text vectorization is the bridge that turns human language into computable linear algebra."*
* *"The core assumption of Bag-of-Words is that word frequency reflects document semantics—even when grammar, word order, and syntax are completely discarded."*
* *"Always establish a simple Bag-of-Words or TF-IDF baseline before deploying deep neural networks or complex Transformer architectures."*

---

## 19. Glossary

* **Corpus:** A structured collection of raw text documents used to train or evaluate NLP models.
* **Document-Term Matrix:** A mathematical matrix format where rows represent individual documents and columns represent vocabulary word counts.
* **IDF (Inverse Document Frequency):** A metric that scales down the weight of words that appear frequently across the entire corpus.
* **N-gram:** A continuous sequence of $N$ tokens extracted from a text sequence.
* **Out-Of-Vocabulary (OOV):** Tokens encountered during inference that were not present in the training set vocabulary.
* **Sparsity:** The ratio of zero elements to the total number of elements in a vector or matrix.
* **TF (Term Frequency):** The raw count or proportion of times a token appears within a specific document.
* **Token:** A discrete unit of text (word, character, or subword) produced during tokenization.
* **Vectorization:** The process of converting unstructured text into a fixed-length numerical format.

---

## 20. One-Page Cheat Sheet

| Technique | Inputs | Output Matrix Format | Captures Word Order? | Main Advantage | Main Disadvantage |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **CountVectorizer (BoW)** | Raw String Corpus | Sparse Matrix $(N \times \lvert V \rvert)$ | No | Simple, computationally light, easy baseline | Sparse vectors, loses word order/context |
| **TF-IDF Vectorizer** | Raw String Corpus | Sparse Matrix $(N \times \lvert V \rvert)$ | No | Scales down common non-informative words | Sparsity, no semantic understanding |
| **Dense Embeddings (e.g., Word2Vec)** | Token IDs | Dense Matrix $(N \times \text{SeqLen} \times d)$ | Yes (with positional encoding) | Captures semantic meanings and context | High memory overhead, requires GPU compute |

### Key Scikit-Learn Vectorizer Parameters

```python
from sklearn.feature_extraction.text import CountVectorizer

vectorizer = CountVectorizer(
    lowercase=True,        # Convert all characters to lowercase
    stop_words='english',  # Filter out standard high-frequency stop words
    ngram_range=(1, 2),    # Include both unigrams and bigrams
    max_features=10000,    # Limit vocabulary to top 10k features by frequency
    min_df=2               # Ignore terms that appear in fewer than 2 documents
)
```

---

## 21. Flash Cards

- **Card 1 | [NLP Basics]**
  - **Q:** What is text vectorization?
  - **A:** The process of transforming unstructured raw text into fixed-length numeric vectors for model training.

- **Card 2 | [Vectorization Models]**
  - **Q:** What information does a standard Bag-of-Words (BoW) model discard?
  - **A:** It discards syntax, grammar, and sequential word order, retaining only term frequencies.

- **Card 3 | [Data Structures]**
  - **Q:** Why are sparse matrices used to store document-term matrices?
  - **A:** Because individual documents contain only a tiny fraction of the overall vocabulary, resulting in matrices composed mostly of zeros.

- **Card 4 | [Feature Extraction]**
  - **Q:** How do N-grams improve upon standard unigram Bag-of-Words models?
  - **A:** By capturing contiguous sequences of $N$ tokens (e.g., bigrams like "not bad"), preserving short contextual phrases.

- **Card 5 | [ML Pipelines]**
  - **Q:** What issue occurs if you run `fit_transform()` on validation or test sets?
  - **A:** Data leakage occurs, altering the vocabulary mapping and mismatching vector feature indices across datasets.

---

## 22. Quiz

### Q1: What is the main purpose of tokenization in NLP?
- A) To compress text files for storage.
- B) To split continuous text strings into discrete units like words or subwords.
- C) To translate foreign languages into English automatically.
- D) To evaluate loss functions during model training.  
**Correct Answer:** B  
**Explanation:** Tokenization breaks raw text streams into discrete units (tokens), which form the structural elements for downstream vocabulary lookup and vector mapping.

### Q2: Given the vocabulary $V = \{\text{"ai"}, \text{"deep"}, \text{"is"}, \text{"learning"}\}$, what is the BoW representation of "deep learning is deep"?
- A) $[0, 2, 1, 1]$
- B) $[1, 1, 1, 1]$
- C) $[2, 1, 0, 1]$
- D) $[0, 1, 2, 1]$  
**Correct Answer:** A  
**Explanation:** Index order: "ai"=0, "deep"=2, "is"=1, "learning"=1. Thus, $[0, 2, 1, 1]$.

### Q3: Which metric measures the proportion of zero-valued elements in a matrix?
- A) Density
- B) Entropy
- C) Sparsity
- D) Variance  
**Correct Answer:** C  
**Explanation:** Sparsity measures zero values relative to total matrix entries.

### Q4: Why are stop words commonly removed during text preprocessing?
- A) They are misspelled words that cause errors.
- B) They carry little domain-specific semantic value and inflate vector sizes.
- C) Modern deep learning architectures cannot process functional words.
- D) Removing stop words guarantees 100% classification accuracy.  
**Correct Answer:** B  
**Explanation:** Common words like "the" and "is" appear across almost all documents without adding helpful signals for text classification tasks.

### Q5: How does TF-IDF improve upon simple term frequency counting?
- A) It rearranges words into their correct grammatical order.
- B) It down-weights terms that appear across many documents in the corpus.
- C) It converts sparse matrices directly into low-dimensional dense matrices.
- D) It automatically translates words into dense vector embeddings.  
**Correct Answer:** B  
**Explanation:** Inverse Document Frequency (IDF) scales down terms that appear throughout the corpus, emphasizing terms unique to specific documents.

### Q6: If a vocabulary has size $|V| = 20,000$, what is the length of a single document's BoW vector?
- A) Variable, depending on the document length.
- B) Exactly 20,000.
- C) $20,000 \times 20,000$.
- D) Equal to the number of sentences in the document.  
**Correct Answer:** B  
**Explanation:** A Bag-of-Words vector matches the dimension of the vocabulary $|V|$, creating a static fixed-length representation for every document.

### Q7: What matrix storage format optimizes memory when handling high-dimensional sparse text vectors?
- A) Dense Numpy Array
- B) Compressed Sparse Row (CSR)
- C) Standard Python Nested List
- D) Relational SQL Tables  
**Correct Answer:** B  
**Explanation:** CSR formats store only non-zero values and their index pointers, significantly reducing memory consumption.

### Q8: What occurs when an un-encountered word appears during `transform()` in Scikit-Learn?
- A) An error halts model evaluation.
- B) The unknown word is ignored and excluded from the output vector.
- C) The vector length grows dynamically.
- D) The word is automatically translated using Google Translate.  
**Correct Answer:** B  
**Explanation:** Words outside the fitted vocabulary are silently dropped during transformation.

### Q9: Which $N$-gram configuration captures two-word combinations?
- A) Unigram
- B) Bigram
- C) Trigram
- D) Quadgram  
**Correct Answer:** B  
**Explanation:** Bigrams process contiguous pairs ($N=2$) of tokens.

### Q10: Why is Bag-of-Words considered a useful baseline model?
- A) It is the most complex model available in modern AI.
- B) It is fast to implement, computationally lightweight, and provides a benchmark before running deep models.
- C) It eliminates the need for labeled training data.
- D) It outperforms large language models on complex context tasks.  
**Correct Answer:** B  
**Explanation:** BoW models are fast and easy to build, establishing a performance baseline to justify using more complex architectures.

---

## 23. Action Items

- [ ] Set up a free Google Colab environment.
- [ ] Load a sample dataset (e.g., SMS Spam Collection or IMDB Movie Reviews).
- [ ] Preprocess text by lowercasing, stripping special characters, and filtering stop words.
- [ ] Build a vectorizer using `sklearn.feature_extraction.text.CountVectorizer`.
- [ ] Inspect the vocabulary mapping and print matrix dimensions and sparsity metrics.
- [ ] Train a baseline Logistic Regression classifier on the vectorized sparse matrix.
- [ ] Calculate baseline test accuracy to evaluate performance before exploring deep learning models.

---

## 24. Recommended Further Reading

* **Documentation:** [Scikit-Learn Text Feature Extraction Guide](https://scikit-learn.org/stable/modules/feature_extraction.html#text-feature-extraction)
* **Book:** *Speech and Language Processing* by Daniel Jurafsky and James H. Martin (3rd Edition Draft)
* **Tutorial:** [Google Machine Learning Guides: Text Classification](https://developers.google.com/machine-learning/guides/text-classification)
* **Open Source Repository:** [NLTK (Natural Language Toolkit) Python Source Code](https://github.com/nltk/nltk)