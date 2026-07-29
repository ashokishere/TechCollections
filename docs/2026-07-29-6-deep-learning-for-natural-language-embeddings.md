# Deep Learning for Natural Language – Embeddings

**Course**: MIT 15.773 Hands-On Deep Learning  
**Instructor**: Rama Ramakrishnan  
**Topic**: Stand-Alone vs. Contextual Word Embeddings, One-Hot Vector Limitations, and Co-occurrence Models  

---

## 1. Executive Summary

In this lecture from MIT 15.773 (*Hands-On Deep Learning*), instructor Rama Ramakrishnan details the evolution of text representation in Natural Language Processing (NLP). The presentation begins with an analysis of traditional one-hot encoding, identifying its primary operational bottlenecks: extreme vector sparsity, high dimensionality, and an absolute failure to encode semantic relationships due to spatial orthogonality.

To overcome these structural defects, the lecture introduces dense word embeddings. These representations map high-dimensional vocabulary spaces into low-dimensional, continuous vector spaces where geometric proximity corresponds to semantic similarity. The theoretical framework behind stand-alone embeddings is grounded in the distributional hypothesis and operationalized via word co-occurrence matrices. Ramakrishnan formulates a log-probabilistic co-occurrence model—decomposing pairwise word frequencies into independent token bias terms and vector dot products—which serves as the mathematical foundation for algorithms like GloVe. Finally, the lecture establishes a conceptual bridge from static, stand-alone embeddings to dynamic, contextual embeddings produced by transformer architectures.

---

## 2. Key Takeaways

- **One-Hot Encoding Deficits**: Represents words as mutually orthogonal, high-dimensional sparse vectors of size $|V|$, causing high computational overhead and eliminating semantic relationships.
- **Constant Distance Dilemma**: Under one-hot encoding, the Euclidean distance between any two distinct word vectors is invariant ($\sqrt{2}$), rendering "king" equidistant to "queen" and "banana".
- **Dense Word Embeddings**: Embeddings project tokens into a lower-dimensional continuous space ($d \in [50, 300]$), where semantics are distributed across vector components.
- **Distributional Hypothesis**: Words that occur in similar context windows share similar semantic profiles and are positioned close together in vector space.
- **Co-occurrence Matrix Optimization**: The frequency with which two words co-occur within a context window provides the empirical signal required to reverse-engineer latent embedding vectors.
- **Mathematical Decomposition**: Log co-occurrence probability is modeled as $\log P(i,j) \approx b_i + b_j + v_i \cdot v_j$, balancing individual word popularity (biases) with semantic affiliation (dot product).
- **Stand-Alone vs. Contextual Embeddings**: Static embeddings assign a single fixed vector per token regardless of context, whereas contextual embeddings (e.g., Transformers) dynamically compute vector representations based on surrounding sentence structure.

---

## 3. Topics Covered

1. **Limitations of One-Hot Encoding**  
   An analysis of traditional sparse token representation, highlighting memory inefficiencies and the total loss of semantic distance metrics.
2. **Foundations of Dense Word Embeddings**  
   An introduction to distributed representations in continuous vector spaces, explaining how dimensional compression captures latent semantic features.
3. **Stand-Alone vs. Contextual Embeddings**  
   A comparison between static token lookup tables (e.g., Word2Vec, GloVe) and dynamically generated context-aware embeddings (e.g., BERT, GPT).
4. **The Co-occurrence Matrix & Context Windows**  
   A study of empirical text statistics, defining context windows and constructing pairwise co-occurrence matrices from large text corpora.
5. **Mathematical Derivation of Embedding Proximity**  
   An exploration of the log-probabilistic relationship linking word biases and vector inner products to observed co-occurrence frequencies.
6. **Reverse-Engineering Latent Embeddings ("Mother Nature" Analogy)**  
   A conceptual view of representation learning as an optimization problem designed to extract hidden word parameters from observed corpus statistics.

---

## 4. Timeline with Timestamps

- **[00:00] Introduction and Course Context** — Overview of the session within MIT 15.773 and the scope of natural language representation.
- **[00:15] Review: One-Hot Encoding** — Step-by-step pipeline for tokenizing, indexing, and generating discrete sparse vectors.
- **[01:00] The Problem with One-Hot Vectors: Sparsity and Dimensionality** — Computational costs, memory footprint, and training inefficiencies associated with large vocabularies ($|V| \ge 100,000$).
- **[02:30] The Deeper Problem with One-Hot Vectors: Lack of Semantic Meaning** — Mathematical demonstration of vector orthogonality and invariant spatial distance ($\sqrt{2}$) between tokens.
- **[03:45] Introducing Word Embeddings: A Solution** — Definition of continuous vector spaces and the geometric formulation of word similarity.
- **[04:30] How Embeddings Work: Learning Distributed Representations** — Application of the distributional hypothesis to represent token meaning across dense vector dimensions.
- **[05:00] Stand-Alone Embeddings vs. Contextual Embeddings** — Comparing single-vector static lookup tables against transformer-based dynamic vector generation.
- **[05:45] The Co-occurrence Matrix: Building Intuition** — Quantifying word proximity using context sliding windows across a text corpus.
- **[06:30] Mathematical Model for Co-occurrence** — Formalizing $\log P(i,j) = b_i + b_j + v_i \cdot v_j$ to decouple word popularity from semantic alignment.
- **[07:45] The "Mother Nature" Analogy for Learning Embeddings** — Conceptualizing latent vectors as inherent parameters to be discovered via objective function optimization.
- **[09:00] Connecting Co-occurrence to Word Similarity** — Demonstrating how high dot products correlate directly with elevated pairwise co-occurrence statistics.
- **[10:15] Practical Implementation: Optimization and Loss Function** — Formulating mean squared error loss on log co-occurrence to fit model parameters (e.g., GloVe algorithm).
- **[11:00] Summary of Stand-Alone Embeddings and Future Topics** — Recap of static embedding mechanics and prelude to transformer self-attention mechanisms.

---

## 5. Detailed Explanation

### Limitations of One-Hot Encoding
In classic natural language processing pipelines, text preprocessing converts raw strings into discrete tokens. Given a vocabulary $V$ containing $|V|$ unique tokens, one-hot encoding assigns each token $w_i$ a binary vector $e_i \in \{0, 1\}^{|V|}$, where a single index contains $1$ and all other elements are $0$.

```
Vocabulary V = ["apple", "banana", "king", "queen"]  (|V| = 4)

"apple"  -> [1, 0, 0, 0]
"banana" -> [0, 1, 0, 0]
"king"   -> [0, 0, 1, 0]
"queen"  -> [0, 0, 0, 1]
```

This model suffers from two major structural issues:
1. **Extreme Sparsity and Dimensional Scaling**: Real-world corpora require vocabularies of $|V| \ge 100,000$. Input layers fed with one-hot vectors require weight matrices with hundreds of thousands of rows per token, leading to memory overhead and parameter inefficiency.
2. **Semantic Orthogonality**: The inner product (dot product) of any two distinct one-hot vectors $e_i$ and $e_j$ ($i \neq j$) is strictly zero:
   $$e_i \cdot e_j = 0$$
   The Euclidean distance between any pair is constant:
   $$\|e_i - e_j\|_2 = \sqrt{(1-0)^2 + (0-1)^2 + 0 + \dots} = \sqrt{2}$$
   Consequently, the representation asserts that the pair ("king", "queen") shares the exact same semantic distance as the pair ("king", "banana").

### Modern Dense Word Embeddings
Word embeddings resolve these structural defects by mapping discrete tokens into a lower-dimensional continuous vector space $\mathbb{R}^d$, where $d \ll |V|$ (typically $d \in [50, 300]$).

Instead of isolating a word's identity to a single dimension, an embedding uses a **distributed representation**. Every vector element represents a continuous scalar along an abstract latent semantic dimension (e.g., gender, animacy, royalty, syntactic role). Because vectors are dense, operations such as inner products, cosine similarity, and vector arithmetic yield semantic metrics:

$$\text{Cosine Similarity}(v_i, v_j) = \frac{v_i \cdot v_j}{\|v_i\|_2 \|v_j\|_2}$$

```
High Dimensional, Sparse (One-Hot)      Low Dimensional, Dense (Embedding)
|V| = 100,000                           d = 300

[0, 0, ..., 1, ..., 0, 0]        -->    [-0.24, 0.81, 0.05, ..., -0.12]
```

### Contextual vs. Stand-Alone Embeddings
The lecture differentiates static lookup-based embeddings from context-dependent representations:
- **Stand-Alone (Static) Embeddings**: Algorithms such as Word2Vec and GloVe construct a static mapping table $E \in \mathbb{R}^{|V| \times d}$. The word "bank" receives the exact same vector $v_{\text{bank}}$ whether it appears in *"river bank"* or *"investment bank"*.
- **Contextual Embeddings**: Modern neural architectures (such as Transformers) process full sequences to construct dynamic vector representations $v_{i,\text{context}} = f(w_i, w_1, w_2, \dots, w_N)$. Under this scheme, "bank" receives distinct vector configurations based on surrounding self-attention patterns.

```
Stand-Alone (Static) Model:
"river bank"      ---> [v_bank] (Fixed Vector)
"investment bank" ---> [v_bank] (Fixed Vector)

Contextual Model:
"river bank"      ---> [v_bank_nature] (Dynamic Vector)
"investment bank" ---> [v_bank_finance] (Dynamic Vector)
```

### The Co-occurrence Matrix and Mathematical Model
To learn static embeddings without manual annotation, systems leverage the **distributional hypothesis**: words appearing in similar contexts carry similar meanings.

A **co-occurrence matrix** $X$ is constructed by sliding a context window of size $c$ across a target corpus. The cell $X_{ij}$ records how many times word $j$ appears within a window of size $c$ centered on word $i$.

```
Corpus: "ice cream is good. cold ice cream."
Window Size c = 1

Co-occurrence Matrix X:
          ice   cream   cold   good
  ice      0      2      1      0
  cream    2      0      0      1
  cold     1      0      0      0
  good     0      1      0      0
```

Ramakrishnan models the log probability of word $i$ and word $j$ co-occurring as:

$$\log P(i,j) \approx b_i + b_j + v_i \cdot v_j$$

Where:
- $P(i,j) = \frac{X_{ij}}{N}$: The joint empirical probability of observing words $i$ and $j$ together.
- $b_i$: The unigram log-popularity bias of word $i$ (how frequently word $i$ appears across the overall corpus).
- $b_j$: The unigram log-popularity bias of word $j$.
- $v_i \cdot v_j$: The dot product of the embedding vectors $v_i, v_j \in \mathbb{R}^d$.

If two words appear together more often than expected based purely on their individual usage frequencies ($b_i + b_j$), the model forces their embedding dot product $v_i \cdot v_j$ to be positive and large. Geometric proximity in the vector space directly reflects semantic affinity in the raw text.

---

## 6. Beginner Explanation (ELI5)

Imagine you are organizing a giant library with 100,000 distinct books.

### The Old Way: Catalog ID Numbers (One-Hot Encoding)
If you give every book a completely unique serial number from 1 to 100,000, you can look up any book instantly. However, the serial numbers tell you nothing about what is inside the books:
- Book #5,000 might be *The History of Science*.
- Book #5,001 might be *Baking French Pastries*.
- Book #5,002 might be *Advanced Physics*.

In this system, Book #5,000 is no closer in meaning to Book #5,002 than it is to Book #5,001, even though two of them are physics books. The numbers carry no semantic meaning.

### The New Way: The Map of Flavors (Word Embeddings)
Instead of assigning a arbitrary serial number, imagine describing every food item using a set of 300 continuous flavor scores between -1.0 and +1.0 (such as Sweetness, Saltiness, Crunchiness, Warmth):

- **Ice Cream**: `[Sweet: +0.9, Cold: +0.9, Savory: -0.8]`
- **Gelato**: `[Sweet: +0.88, Cold: +0.89, Savory: -0.79]`
- **Hot Soup**: `[Sweet: -0.6, Cold: -0.9, Savory: +0.8]`

Because "Ice Cream" and "Gelato" have nearly identical scores across all flavor traits, their points sit right next to each other on a 300-dimensional map. "Hot Soup" sits on the far opposite side of the map. 

### How the Computer Learns: The "Neighborhood Watch"
How does the computer figure out these scores without a human telling it? It uses the **Neighborhood Rule**:
1. It reads millions of sentences and tracks which words hang out together.
2. It notices that "ice" and "cream" are frequently seen together in the same sentence.
3. It concludes: *"Because these two words share the same neighborhoods, their points on our 300-dimensional map must be placed close together."*

---

## 7. Technical Deep Dive

### Formal Orthogonality Proof of One-Hot Vectors
Let $V$ represent a dictionary indexed by $i \in \{1, \dots, |V|\}$. The canonical one-hot basis vector $e_i \in \mathbb{R}^{|V|}$ is defined as:

$$(e_i)_k = \delta_{ik} = \begin{cases} 1 & \text{if } k = i \\ 0 & \text{if } k \neq i \end{cases}$$

The inner product between any distinct pair $e_i, e_j$ ($i \neq j$) is:

$$\langle e_i, e_j \rangle = \sum_{k=1}^{|V|} (e_i)_k (e_j)_k = \sum_{k=1}^{|V|} \delta_{ik} \delta_{jk} = 0$$

The $L_2$ norm metric distance is:

$$d_{L_2}(e_i, e_j) = \sqrt{\langle e_i - e_j, e_i - e_j \rangle} = \sqrt{\|e_i\|^2_2 + \|e_j\|^2_2 - 2\langle e_i, e_j \rangle} = \sqrt{1 + 1 - 0} = \sqrt{2}$$

Because inner products evaluate to $0$ and distances evaluate to $\sqrt{2}$, gradient descent algorithms operating directly on raw one-hot indices cannot infer feature correlations.

### GloVe Formulation and Loss Optimization
The co-occurrence model discussed in the lecture mirrors the Global Vectors for Word Representation (GloVe) objective. Let $X_{ij}$ denote the co-occurrence count between word $i$ and word $j$. The model defines:

$$w_i^T \tilde{w}_j + b_i + \tilde{b}_j = \log X_{ij}$$

Where $w_i \in \mathbb{R}^d$ is the main word vector, $\tilde{w}_j \in \mathbb{R}^d$ is the context word vector, $b_i$ is the main word bias, and $\tilde{b}_j$ is the context word bias.

To address heteroscedasticity and avoid taking $\log(0)$, GloVe minimizes a weighted least-squares loss function $J$:

$$J = \sum_{i,j=1}^{|V|} f(X_{ij}) \left( w_i^T \tilde{w}_j + b_i + \tilde{b}_j - \log X_{ij} \right)^2$$

Where $f(X_{ij})$ is a continuous weighting function designed to suppress noise from extremely rare co-occurrences while preventing frequent stop words from dominating parameter updates:

$$f(x) = \begin{cases} \left( \frac{x}{x_{\max}} \right)^\alpha & \text{if } x < x_{\max} \\ 1 & \text{otherwise} \end{cases}$$

Typically, parameters are set to $x_{\max} = 100$ and $\alpha = 0.75$.

```
Co-occurrence Matrix (X) ---> Log Transformation (log X)
                                     |
                                     v
                       Weighted MSE Loss Optimization
         J = SUM f(X_ij) * (w_i^T * w_j + b_i + b_j - log X_ij)^2
                                     |
                                     v
                       Dense Embedding Weights (W)
```

### Geometric Vector Arithmetic
Because objective functions preserve linear co-occurrence ratios, latent semantic relations are encoded as linear vector translations:

$$v_{\text{king}} - v_{\text{man}} + v_{\text{woman}} \approx v_{\text{queen}}$$

```
  Vector Space Geometry:
  
  [king]  + (woman - man) ---------> [queen]
    |                                   ^
    |                                   |
    +---- - (man) ----> + (woman) ------+
```

---

## 8. Important Definitions

- **One-Hot Vector**: A high-dimensional sparse vector representation where a single coordinate is $1$ and all other coordinates are $0$.
- **Dense Embedding**: A low-dimensional continuous vector space representation ($\mathbb{R}^d$) where scalar values are learned via gradient optimization.
- **Distributional Hypothesis**: The linguistic concept stating that words occurring in similar context windows exhibit similar semantic meanings.
- **Co-occurrence Matrix**: A square matrix $X \in \mathbb{R}^{|V| \times |V|}$ where element $X_{ij}$ records the count of occurrences of word $j$ within a context window surrounding word $i$.
- **Stand-Alone Embedding**: Static vector representation that maps each vocabulary index to a single fixed continuous vector regardless of sentence context.
- **Contextual Embedding**: Dynamic vector representation that computes token representations as a function of the entire context sequence.
- **Dot Product ($v_i \cdot v_j$)**: A vector multiplication operation calculating the algebraic sum of coordinate-wise products, serving as an unnormalized measure of vector alignment.
- **Cosine Similarity**: A normalized metric measuring the cosine of the angle between two vectors, ranging from $-1.0$ (opposite) to $+1.0$ (identical direction).

---

## 9. Code Snippets & Configuration Examples

### 1. Constructing a Co-occurrence Matrix in Python

```python
import numpy as np
from collections import defaultdict

def build_cooccurrence_matrix(corpus: list[str], window_size: int = 2):
    # Step 1: Build Vocabulary
    tokens = [word.lower() for sentence in corpus for word in sentence.split()]
    vocab = sorted(list(set(tokens)))
    word_to_idx = {word: idx for idx, word in enumerate(vocab)}
    idx_to_word = {idx: word for idx, word in enumerate(vocab)}
    vocab_size = len(vocab)
    
    # Step 2: Populate Matrix
    co_matrix = np.zeros((vocab_size, vocab_size), dtype=np.float32)
    
    for sentence in corpus:
        words = sentence.lower().split()
        for i, target_word in enumerate(words):
            target_idx = word_to_idx[target_word]
            start = max(0, i - window_size)
            end = min(len(words), i + window_size + 1)
            
            for j in range(start, end):
                if i != j:
                    context_word = words[j]
                    context_idx = word_to_idx[context_word]
                    co_matrix[target_idx, context_idx] += 1.0
                    
    return co_matrix, word_to_idx, idx_to_word

# Demo execution
corpus = [
    "deep learning for natural language processing",
    "natural language embeddings capture deep semantic meaning"
]
X, w2i, i2w = build_cooccurrence_matrix(corpus, window_size=1)
print(f"Vocabulary Size: {len(w2i)}")
print("Co-occurrence Matrix Shape:", X.shape)
```

### 2. PyTorch Stand-Alone Embedding Lookup Layer

```python
import torch
import torch.nn as nn

class StandAloneEmbeddingModel(nn.Module):
    def __init__(self, vocab_size: int, embedding_dim: int):
        super().__init__()
        # PyTorch Embedding layer operates as a dense lookup table E in R^(vocab_size x embedding_dim)
        self.embeddings = nn.Embedding(
            num_embeddings=vocab_size, 
            embedding_dim=embedding_dim
        )
        
    def forward(self, input_indices: torch.Tensor) -> torch.Tensor:
        # Input shape: (batch_size, sequence_length)
        # Output shape: (batch_size, sequence_length, embedding_dim)
        return self.embeddings(input_indices)

# Example usage
VOCAB_SIZE = 100000
EMBEDDING_DIM = 300

model = StandAloneEmbeddingModel(vocab_size=VOCAB_SIZE, embedding_dim=EMBEDDING_DIM)
token_ids = torch.tensor([[4, 982, 12, 4410]]) # Batch size 1, Sequence length 4
dense_vectors = model(token_ids)

print("Output Tensor Shape:", dense_vectors.shape)
# Output: torch.Size([1, 4, 300])
```

### 3. Measuring Semantic Proximity via Cosine Similarity

```python
import torch
import torch.nn.functional as F

def compute_word_similarity(vec1: torch.Tensor, vec2: torch.Tensor) -> float:
    """
    Computes Cosine Similarity = (v1 . v2) / (||v1|| * ||v2||)
    """
    sim = F.cosine_similarity(vec1.unsqueeze(0), vec2.unsqueeze(0))
    return sim.item()

# Mock embedding vectors for 'king', 'queen', 'apple'
torch.manual_seed(42)
king_vec = torch.randn(300)
queen_vec = king_vec + torch.randn(300) * 0.2  # Close to king
apple_vec = torch.randn(300)                  # Unrelated

print(f"Similarity (king, queen): {compute_word_similarity(king_vec, queen_vec):.4f}")
print(f"Similarity (king, apple): {compute_word_similarity(king_vec, apple_vec):.4f}")
```

---

## 10. Best Practices

1. **Dimensionality Selection**: Use empirical heuristic rules for embedding size ($d$). For standard tasks, $d \in [100, 300]$ balances capacity and computational efficiency. A common heuristic is $d \approx \sqrt[4]{|V|}$.
2. **Subword Tokenization Handling**: Pair embeddings with subword tokenizers (e.g., Byte-Pair Encoding or WordPiece) to eliminate Out-Of-Vocabulary (OOV) tokens.
3. **Vector Normalization**: Normalize learned embedding vectors to unit length ($\|v\|_2 = 1$). This converts dot products directly into cosine similarity scores, speeding up vector search operations.
4. **Pretrained Vector Initialization**: Prefer initializing downstream models with robust pretrained embeddings (e.g., GloVe or FastText) when training on small datasets ($<10^6$ tokens).
5. **Context Window Optimization**: Set narrow context window sizes ($c \in [2, 5]$) to capture syntactic and functional word similarities. Expand to larger windows ($c \in [8, 15]$) to capture broad topical relationships.

---

## 11. Common Mistakes

- **Using Euclidean Distance on Unnormalized Vectors**: Evaluating vector proximity with $L_2$ distance without normalizing for magnitude leads to skewed similarity metrics dominated by word frequency.
- **Treating Static Embeddings as Context-Aware**: Expecting static lookup tables to differentiate polysemous words (e.g., "bank", "apple", "python") based on sentence context.
- **Overfitting Embedding Weights**: Fine-tuning an entire $E \in \mathbb{R}^{|V| \times d}$ lookup table on small target datasets without freezing weights or applying weight decay.
- **Ignoring Stop-Word Frequency Dominance**: Building raw co-occurrence matrices without applying smoothing, term-frequency weighting, or sub-sampling (such as GloVe weighting $f(X_{ij})$) causes function words ("the", "is") to distort semantic relationships.

---

## 12. Interview Questions

### Q1: Prove mathematically why the Euclidean distance between any two distinct one-hot vectors is always equal to $\sqrt{2}$.
**Answer:**  
Let $e_i, e_j \in \mathbb{R}^{|V|}$ be two distinct standard basis vectors where $i \neq j$. By definition, $(e_i)_k = 1$ if $k=i$ and $0$ otherwise.  
The squared Euclidean distance is given by:

$$\|e_i - e_j\|_2^2 = \sum_{k=1}^{|V|} ((e_i)_k - (e_j)_k)^2$$

Evaluating coordinate by coordinate:
- At $k = i$: $((1) - (0))^2 = 1$
- At $k = j$: $((0) - (1))^2 = 1$
- At $k \neq i, j$: $((0) - (0))^2 = 0$

Summing over all dimensions yields:

$$\|e_i - e_j\|_2^2 = 1 + 1 + 0 = 2 \implies \|e_i - e_j\|_2 = \sqrt{2}$$

Because this value is constant for any pair $i \neq j$, one-hot spaces cannot encode relative semantic distance.

### Q2: What is the distributional hypothesis, and how does it enable unsupervised embedding learning?
**Answer:**  
The distributional hypothesis states that words occurring in similar surrounding contexts share similar semantic meanings. This hypothesis enables unsupervised learning because raw, unannotated text corpora serve as their own training labels. By defining a moving context window, we collect target-context word pairs $(w_i, w_c)$. The learning objective optimizes continuous vector parameters to maximize the probability (or dot product correlation) of observed word co-occurrences while minimizing it for unobserved pairs.

### Q3: How does the co-occurrence model $\log P(i,j) \approx b_i + b_j + v_i \cdot v_j$ isolate semantic similarity from word popularity?
**Answer:**  
In natural text, frequent words (e.g., "the", "and") co-occur with almost all words simply due to high baseline frequencies. If co-occurrence were modeled using only the inner product $v_i \cdot v_j$, embedding space would be dominated by word frequency rather than meaning. 

By introducing independent scalar bias terms $b_i$ and $b_j$, $b_i$ absorbs the baseline log-frequency of word $i$, and $b_j$ absorbs the baseline log-frequency of word $j$. The remaining dot product term $v_i \cdot v_j$ isolates the residual association beyond random baseline co-occurrence, isolating pure semantic similarity.

---

## 13. Certification Questions

### Q1: In a vocabulary with $|V| = 50,000$ unique tokens, what are the dimensions of the embedding lookup matrix $E$ if the embedding size is $d = 300$?
- A) $300 \times 300$
- B) $50,000 \times 50,000$
- C) $50,000 \times 300$
- D) $300 \times 1$

**Correct Answer:** **C**  
**Explanation:** An embedding matrix maps each of the $|V|$ discrete vocabulary tokens to a dense $d$-dimensional continuous vector. Thus, the embedding lookup table has shape $|V| \times d = 50,000 \times 300$.

### Q2: What primary limitation of stand-alone embeddings (e.g., GloVe, Word2Vec) led to the development of contextual embeddings (e.g., BERT)?
- A) Inability to run on GPU hardware.
- B) Higher memory consumption than one-hot encoding.
- C) Static representations that assign the exact same vector to a word regardless of polysemy or context.
- D) Inability to compute cosine similarities.

**Correct Answer:** **C**  
**Explanation:** Stand-alone embeddings map each vocabulary word to a single fixed vector. Consequently, polysemous words (e.g., "bank" in financial vs. river contexts) receive an identical static vector representation, failing to capture context-specific meaning.

---

## 14. Real-World Examples

### 1. Enterprise Semantic Search Engines
Traditional keyword search systems (e.g., TF-IDF) fail when query keywords do not match document terms exact-string for exact-string. Modern search engines map incoming user queries and target document chunks into dense vector spaces. Performing approximate nearest neighbor (ANN) vector searches (e.g., via FAISS or Pinecone) retrieves relevant documents based on semantic content even if no exact keywords overlap.

### 2. E-Commerce Item-to-Item Recommendations (Item2Vec)
By treating customer purchasing sessions as "sentences" and product IDs as "words", recommendation systems train stand-alone embeddings on product co-occurrences using skip-gram or GloVe objectives. Products frequently purchased together within the same session end up adjacent in vector space, driving real-time product recommendations.

---

## 15. Analogies

### 1. The GPS Navigation Map
- **One-Hot Encoding**: Like identifying cities solely by arbitrary zip codes. You know that 90210 is Beverly Hills and 10001 is New York, but the numbers give you no information about geographic distance or direction.
- **Word Embeddings**: Like mapping cities using continuous (Latitude, Longitude) coordinates. Calculating the mathematical distance between coordinates directly reveals spatial distance.

### 2. Modern Decryption and Machine Translation
- **Mother Nature's Hidden Code**: Imagine two alien civilisations writing independent books about astronomy. Even though the written symbols differ, the underlying physical laws they describe are identical. The co-occurrence statistics of their words produce matching vector cluster geometries. Discovering word embeddings is like discovering the underlying constellation map of human thought.

---

## 16. Frequently Asked Questions

### 1. Why is the embedding dimension usually chosen between 50 and 300?
Dimensions below 50 lack sufficient capacity to resolve complex semantic relationships across large vocabularies. Dimensions above 300 encounter diminishing returns, increased computational costs, and higher risk of overfitting without noticeable gains in downstream model accuracy.

### 2. What is the difference between Word2Vec and GloVe?
Word2Vec is a predictive neural network architecture that updates vectors using local context sliding windows via Skip-gram or Continuous Bag-of-Words (CBOW) objectives. GloVe is a matrix-factorization-based method that constructs a global co-occurrence matrix across the entire corpus and optimizes a weighted log-least-squares objective.

### 3. How do continuous embeddings handle Out-Of-Vocabulary (OOV) tokens?
Standard stand-alone word-level lookup tables map unseen words to a single generic `<UNK>` token. Modern implementations resolve this issue by using subword models (such as FastText or BPE) that construct unseen word vectors by summing character $n$-gram embeddings.

### 4. Why are static embeddings still used if contextual embeddings (BERT/GPT) exist?
Static embeddings are computationally efficient, require zero GPU infrastructure for inference, and run quickly on CPU environments. They remain useful for low-latency text classification, fast baseline clustering, and lightweight retrieval pipelines.

---

## 17. Related Technologies

- **Word2Vec (Skip-Gram / CBOW)**: Predictive neural frameworks for learning static continuous representations.
- **GloVe (Global Vectors)**: Global log-bilinear matrix factorization model for learning stand-alone embeddings.
- **FastText**: Extension of Word2Vec leveraging character $n$-grams to represent subwords and out-of-vocabulary tokens.
- **PyTorch `nn.Embedding` / TensorFlow `tf.keras.layers.Embedding`**: Hardware-accelerated dense matrix lookup layers.
- **BERT / RoBERTa**: Bidirectional transformer architectures generating contextual embeddings.
- **FAISS (Facebook AI Similarity Search)**: Vector database engine designed for ultra-fast vector indexing and similarity search.

---

## 18. Important Quotes

> *"The distance between any two one-hot vectors is always the same... 'king' and 'queen' are as far apart as 'king' and 'banana'."* — **Rama Ramakrishnan**

> *"Wouldn't it be nice if words that are similar in meaning were close to each other in vector space?"* — **Rama Ramakrishnan**

> *"The co-occurrence count between two words is a function of their individual popularity (biases) and the connection between their embedding vectors (dot product)."* — **Rama Ramakrishnan**

---

## 19. Glossary

| Term | Category | Definition |
| :--- | :--- | :--- |
| **Co-occurrence Matrix** | Natural Language Processing | A matrix $X$ where entry $X_{ij}$ stores the empirical frequency with which token $j$ appears within a context window of token $i$. |
| **Cosine Similarity** | Linear Algebra | A metric evaluating the inner product of two normalized vectors, reflecting directional alignment. |
| **Distributional Hypothesis** | Linguistics | The hypothesis asserting that words occurring in similar contextual distribution share similar semantics. |
| **Dot Product** | Linear Algebra | Algebraic operation taking two equal-length sequences of vectors and returning a single scalar sum of elementwise products. |
| **One-Hot Vector** | Machine Learning | Sparse binary vector of dimension $|V|$ with a single coordinate set to $1$ and all others set to $0$. |
| **Polysemy** | Linguistics | The capacity for a single word or phrase to hold multiple distinct meanings depending on context. |
| **Sparsity** | Data Structures | The proportion of zero-valued elements relative to total elements within a matrix or vector tensor. |
| **Stand-Alone Embedding** | Deep Learning | Fixed embedding lookup table mapping vocabulary token indices to static dense continuous vectors. |

---

## 20. One-Page Cheat Sheet

```
=================================================================================================
                            ONE-HOT VS. STAND-ALONE VS. CONTEXTUAL
=================================================================================================
METRIC                ONE-HOT ENCODING              STAND-ALONE EMBEDDING      CONTEXTUAL EMBEDDING
-------------------------------------------------------------------------------------------------
Representation Space  Sparse Discrete ({0,1}^|V|)   Dense Continuous (R^d)     Dynamic Continuous (R^d)
Dimension Size        High (|V| >= 100,000)        Low (d in [50, 300])       Model Dependent (d=768+)
Inner Product (v1.v2) Always 0                      Proportional to Similarity Dynamic per Context
Euclidean Distance    Always sqrt(2)                Variable (Reflects Sim)    Dynamic per Context
Handles Polysemy?     No                            No                         Yes
Lookup Mechanism      Index Access                  Matrix Row Lookup E[i]     Transformer Forward Pass

=================================================================================================
                            MATHEMATICAL FORMULATIONS
=================================================================================================
1. One-Hot Vector Distance:
   ||e_i - e_j||_2 = sqrt(2)  (for all i != j)

2. Continuous Vector Cosine Similarity:
   CosineSim(v_i, v_j) = (v_i . v_j) / (||v_i||_2 * ||v_j||_2)

3. Log Co-occurrence Model (GloVe Formative):
   log P(i, j) = b_i + b_j + (v_i . v_j)
   where:
     P(i, j) = Joint Co-occurrence Probability
     b_i     = Log Unigram Frequency Bias of Target Word
     b_j     = Log Unigram Frequency Bias of Context Word
     v_i.v_j = Vector Dot Product (Semantic Affinity Metric)

4. Objective Loss Minimization:
   J = SUM f(X_ij) * (w_i^T * w_j + b_i + b_j - log X_ij)^2
=================================================================================================
```

---

## 21. Flash Cards

- **Card 1 | NLP Limitations**
  - **Q:** Why does one-hot encoding fail to capture semantic relationships?
  - **A:** Because all one-hot vectors are mutually orthogonal. Their dot product is always $0$ and their Euclidean distance is always $\sqrt{2}$.

- **Card 2 | Fundamentals**
  - **Q:** What is the Distributional Hypothesis?
  - **A:** The concept that words appearing in similar contexts share similar semantic meanings.

- **Card 3 | Mathematical Formulations**
  - **Q:** In the model $\log P(i,j) \approx b_i + b_j + v_i \cdot v_j$, what role do $b_i$ and $b_j$ play?
  - **A:** They represent single-word popularity biases, absorbing baseline word usage frequencies so the dot product isolates pure semantic affinity.

- **Card 4 | System Architecture**
  - **Q:** What is the primary difference between stand-alone embeddings and contextual embeddings?
  - **A:** Stand-alone embeddings assign a single static vector per token, whereas contextual embeddings dynamically compute vectors based on full surrounding sentence contexts.

- **Card 5 | Vector Metrics**
  - **Q:** Why is Cosine Similarity preferred over Euclidean Distance for unnormalized vectors?
  - **A:** Cosine Similarity measures directional alignment independent of magnitude, preventing word frequency scale variations from skewing similarity metrics.

---

## 22. Quiz

### Q1: What is the primary mathematical issue with using one-hot representations in deep learning models with large vocabularies?
- A) They cause gradient clipping errors.
- B) They result in extremely high-dimensional, sparse vectors, causing memory inefficiencies and orthogonal distance metrics.
- C) They cannot be stored in standard GPU memory tensors.
- D) They require non-linear activations to process.
**Correct Answer:** **B**  
**Explanation:** One-hot vectors scale with $|V|$. For large vocabularies, vectors become sparse and memory-inefficient, and all word pairs are equidistant ($\sqrt{2}$).

### Q2: What is the Euclidean distance between any two distinct one-hot vectors $e_i$ and $e_j$?
- A) $0$
- B) $1$
- C) $\sqrt{2}$
- D) $|V|$
**Correct Answer:** **C**  
**Explanation:** Evaluating $\|e_i - e_j\|_2 = \sqrt{(1)^2 + (-1)^2} = \sqrt{2}$.

### Q3: Stand-alone embedding models map vocabulary tokens into continuous vector spaces of typically what dimension size ($d$)?
- A) $2 \text{ to } 5$
- B) $50 \text{ to } 300$
- C) $10,000 \text{ to } 50,000$
- D) Equal to vocabulary size $|V|$
**Correct Answer:** **B**  
**Explanation:** Standard continuous word vector embeddings use dimensionality between $50$ and $300$ to balance model capacity and computational efficiency.

### Q4: The statement "Words that occur in similar contexts tend to have similar meanings" refers to which concept?
- A) The Contextual Self-Attention Theorem
- B) The Universal Approximation Theorem
- C) The Distributional Hypothesis
- D) The Orthogonal Mapping Postulate
**Correct Answer:** **C**  
**Explanation:** The distributional hypothesis forms the foundational linguistic basis for unsupervised vector representation models.

### Q5: How is a co-occurrence matrix populated during corpus text preprocessing?
- A) By taking the derivative of word frequency distributions.
- B) By counting how often token pairs appear together within a specified moving context window size.
- C) By computing inverse document frequency scores across text files.
- D) By assigning random normal float values to token intersections.
**Correct Answer:** **B**  
**Explanation:** Co-occurrence matrices track local context co-occurrences within a fixed sliding window $c$.

### Q6: In the co-occurrence equation $\log P(i,j) \approx b_i + b_j + v_i \cdot v_j$, what does $v_i \cdot v_j$ compute?
- A) The Euclidean distance between word indices.
- B) The unigram baseline popularity of both words combined.
- C) The vector dot product measuring semantic connection independent of baseline usage frequencies.
- D) The learning rate scaling factor for stochastic gradient descent.
**Correct Answer:** **C**  
**Explanation:** $v_i \cdot v_j$ isolates the vector dot product, measuring true semantic alignment after individual popularity biases ($b_i, b_j$) are subtracted.

### Q7: Why do static stand-alone word embeddings struggle with polysemous words such as "bank"?
- A) Because they use floating-point precision numbers.
- B) Because they assign every word index a single, invariant vector regardless of surrounding sentence context.
- C) Because co-occurrence matrices cannot store decimal numbers.
- D) Because dot products cannot operate on negative vector numbers.
**Correct Answer:** **B**  
**Explanation:** Static stand-alone embeddings map each token index to a single row in an embedding table, producing the same vector regardless of context.

### Q8: How does modern transformer-based contextual representation differ from static GloVe embeddings?
- A) Transformers construct one-hot sparse matrices.
- B) Transformers process full input sequences dynamically to generate context-dependent representations.
- C) Transformers eliminate continuous vector spaces.
- D) Transformers do not use tokenizers.
**Correct Answer:** **B**  
**Explanation:** Transformers run self-attention across entire sequences, producing dynamic embeddings tailored to the surrounding sentence structure.

### Q9: Which weight function modification prevents frequent stop words from dominating the GloVe embedding loss function?
- A) Using a clipped weighting function $f(X_{ij}) = \min(1, (X_{ij}/x_{\max})^\alpha)$.
- B) Setting all stop word vectors to zero.
- C) Squaring the dot product $v_i \cdot v_j$.
- D) Removing bias terms $b_i$ and $b_j$ from model parameterization.
**Correct Answer:** **A**  
**Explanation:** GloVe uses a bounded weighting function $f(X_{ij})$ to cap the impact of high-frequency co-occurrences.

### Q10: What vector operation yields semantic analogy results like $v_{\text{king}} - v_{\text{man}} + v_{\text{woman}} \approx v_{\text{queen}}$?
- A) Linear Vector Arithmetic
- B) Matrix Inversion
- C) Cross-Entropy Reduction
- D) Singular Value Decomposition
**Correct Answer:** **A**  
**Explanation:** Linear vector arithmetic over dense continuous embedding spaces exposes relational semantic sub-spaces.

---

## 23. Action Items

- [ ] **Construct a Co-occurrence Matrix**: Implement a pure Python script using `Numpy` to generate co-occurrence matrices from sample text data across varying context window sizes ($c \in \{1, 3, 5\}$).
- [ ] **Implement PyTorch Embeddings**: Build an `nn.Embedding` layer in PyTorch. Verify forward pass shape mechanics mapping `(batch_size, seq_len)` index tensors to `(batch_size, seq_len, d)` continuous vector representations.
- [ ] **Analyze Vector Geometry**: Load pretrained GloVe vectors using `gensim.downloader`. Write script functions evaluating top-$K$ cosine similarities and verify vector math analogies (e.g., $v_{\text{paris}} - v_{\text{france}} + v_{\text{germany}} \approx v_{\text{berlin}}$).
- [ ] **Compare Static vs. Contextual Vector Similarity**: Use Hugging Face `transformers` to extract token representations from BERT for polysemous words ("bank" in two distinct sentences). Compare cosine similarity scores against static GloVe vector metrics.

---

## 24. Recommended Further Reading

1. **Pennington, J., Socher, R., & Manning, C. D. (2014)**. *GloVe: Global Vectors for Word Representation*. Proceedings of the 2014 Conference on Empirical Methods in Natural Language Processing (EMNLP), 1532–1543.  
2. **Mikolov, T., Chen, K., Corrado, G., & Dean, J. (2013)**. *Efficient Estimation of Word Representations in Vector Space*. arXiv preprint arXiv:1301.3781.  
3. **Firth, J. R. (1957)**. *A synopsis of linguistic theory, 1930-1955*. Studies in linguistic analysis, 1-32. (Origin of the Distributional Hypothesis).  
4. **PyTorch Documentation**: *nn.Embedding Module Reference*. https://pytorch.org/docs/stable/generated/torch.nn.Embedding.html