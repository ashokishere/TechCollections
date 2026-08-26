# Deeplearning/How Transformer LLMs Work: Master Knowledge Document

---

## 1. Executive Summary

"How Transformer LLMs Work," hosted by Andrew Ng alongside Jay Alammar and Maarten Grootendorst (co-authors of *Hands-On Large Language Models*), provides a foundational exploration of modern Large Language Model (LLM) architectures. The course traces the historical evolution of numerical language representations—from early static sparse vectors (Bag-of-Words) and dense static representations (Word2Vec) to contextual sequence modeling (Recurrent Neural Networks and Attention mechanisms)—culminating in the modern Transformer architecture introduced by Vaswani et al. (2017).

The material details the mechanics of tokenization, vector embeddings, self-attention, feed-forward neural networks (FFNN), and language modeling heads. It contrasts encoder-only (e.g., BERT), decoder-only (e.g., GPT, Llama), and encoder-decoder paradigms. Key operational concepts are explained, including autoregressive token generation, decoding strategies (greedy vs. stochastic sampling), KV caching, grouped-query attention (GQA), rotary positional embeddings (RoPE), and Mixture of Experts (MoE) architectures like Mixtral 8x7B.

---

## 2. Key Takeaways

- **Evolution of Language Representation**: Text evolved from non-contextual static representations (Bag-of-Words word counts, Word2Vec static dense vectors) to dynamic, contextualized vector representations powered by self-attention mechanisms.
- **Autoregressive Token-by-Token Generation**: Decoder-only LLMs generate text sequentially—one token per step. Each generated token is appended to the input sequence (context window) to compute the next token distribution.
- **Self-Attention Mechanics**: Self-attention computes dynamic weight distributions using Query ($Q$), Key ($K$), and Value ($V$) matrix projections, allowing tokens to selectively pull relevant information from preceding context.
- **Dual Architecture Components**: A standard Transformer block relies on two main mechanisms:
  1. *Self-Attention*: Computes contextual relevance across tokens.
  2. *Feed-Forward Neural Network (FFNN)*: Acts as a statistical memory bank storing world knowledge and syntactic/semantic relationships.
- **Attention Efficiency Scalability**: Scalability optimizations address quadratic attention computational bottlenecks ($\mathcal{O}(N^2)$):
  - *Grouped-Query Attention (GQA)*: Groups query heads to share key-value heads, reducing memory bandwidth during inference.
  - *KV Caching*: Preserves key and value vectors of historical tokens across generation steps to prevent redundant matrix operations.
- **Modern Positional Encodings**: Dynamic positional methods, such as Rotary Position Embeddings (RoPE), insert sequence-order information directly into $Q$ and $K$ representations inside attention heads, enabling document packing and context extension.
- **Sparse Execution via Mixture of Experts (MoE)**: MoE models (e.g., Mixtral 8x7B) replace monolithic FFNN layers with multiple specialized "expert" networks. A lightweight router assigns tokens to specific experts, decoupling total loaded parameter size from active per-token compute footprint.

---

## 3. Topics Covered

- **History of NLP Vectorization**: 
  Overview of Bag-of-Words (sparse token counts) and Word2Vec (dense static embeddings learned via continuous bag-of-words/skip-gram context prediction).
- **Recurrent Neural Networks & Early Attention**: 
  Sequential context encoding using hidden state vectors, and the introduction of additive/dot-product attention (Bahdanau/Luong) to solve long-sequence bottlenecking.
- **Original Transformer & Architecture Variants**: 
  The original Encoder-Decoder setup (Vaswani et al. 2017) and its divergence into Encoder-only (BERT), Decoder-only (GPT/Llama), and Encoder-Decoder architectures.
- **Tokenization & Embedding Matrices**: 
  Deconstruction of raw text into subword tokens using fixed-vocabulary tokenizers, and mapping token IDs to initial continuous embedding vectors.
- **Decoding Strategies & Generation Loop**: 
  Probabilistic output scoring through the Language Modeling Head; comparison between Greedy Decoding (temperature = 0) and Top-$P$/Nucleus sampling.
- **Transformer Block Internal Mechanics**: 
  Step-by-step breakdown of Self-Attention ($Q, K, V$ projections, relevance scoring, weighted sum) paired with Feed-Forward Neural Networks (expansion, activation, projection down).
- **Advanced Attention Efficiency Patterns**: 
  Mechanics of Multi-Head Attention (MHA), Multi-Query Attention (MQA), Grouped-Query Attention (GQA), Sparse Attention, and Ring Attention.
- **Modern Decoder Block Innovations**: 
  Pre-Layer Normalization (Pre-LN), Rotary Position Embeddings (RoPE), and document packing during base model training.
- **Mixture of Experts (MoE) Architecture**: 
  Router networks, top-$k$ expert selection, sparse parameters vs. active parameter calculations, and Mixtral 8x7B performance characteristics.
- **Practical Code Implementation**: 
  Inspecting transformer architecture hierarchies, layer dimensions, tensor shapes, and manual generation loops using Hugging Face `transformers`.

---

## 4. Timeline with Timestamps

- **[00:00:00] Course Overview & Introduction**: Andrew Ng introduces instructors Jay Alammar and Maarten Grootendorst, detailing course objectives and the shift from 2017's *Attention is All You Need* to modern LLMs.
- **[00:00:05] Evolution of Language Representations**: Maarten Grootendorst breaks down numerical language modeling from early non-transformer baselines to transformer dense vectors.
- **[00:02:40] Bag-of-Words (BoW) Deep Dive**: Detailed walk-through of tokenization, vocabulary creation, word count vectors, and limitations of order-agnostic sparse representations.
- **[00:05:32] Word2Vec & Static Semantic Embeddings**: Explanation of neural embedding generation, vector space dimensions, static semantic properties, subword tokenization, and vector averaging.
- **[00:08:46] Sequential Modeling with RNNs & Attention**: Analysis of Recurrent Neural Networks, context vector bottlenecks, auto-regressive decoding, and the introduction of Attention (2014).
- **[00:14:25] The Transformer Architecture**: Transition from RNNs to non-recurrent parallelizable attention, breakdown of the Vaswani et al. Encoder-Decoder model.
- **[00:17:44] Encoder-Only (BERT) vs. Decoder-Only (GPT)**: Overview of BERT (Masked Language Modeling, CLS token fine-tuning) vs. GPT (Autoregressive causal decoders, masked self-attention).
- **[00:21:57] Tokenization, Vocabulary, and Generation Loop**: Jay Alammar presents the three core components: Tokenizer, Transformer Block Stack, and Language Modeling Head.
- **[00:25:00] Decoding Strategies & KV Caching**: Explanation of logit probabilities, greedy decoding, temperature control, Top-$P$ sampling, and key-value (KV) cache computation reuse.
- **[00:28:17] Inside the Transformer Block**: Dual-layer architecture: Self-Attention for contextual mixing vs. Feed-Forward Neural Networks (FFNN) for statistical factual storage.
- **[00:32:00] Self-Attention Mechanics ($Q, K, V$)**: Detailed operational walk-through of Queries, Keys, Values, matrix multiplication for relevance scoring, and value aggregation.
- **[00:36:00] Scaled Attention Innovations (MQA, GQA, Sparse, Ring)**: Scaling self-attention computational bottlenecks via Multi-Query Attention, Grouped-Query Attention, Sparse windowing, and Ring Attention.
- **[00:41:40] Deconstructing Modern LLM Papers**: Mapping mathematical parameters from Meta's Llama 3.1 8B architecture specification tables to visual component diagrams.
- **[00:43:00] Hands-on HuggingFace Code Walkthrough**: Python code demonstration downloading `Phi-3-mini`, inspecting structural layers, tensor shapes (`[batch, seq_len, hidden_dim]`), and logit extraction.
- **[00:52:50] Modern Transformer Architectural Enhancements**: Analysis of Pre-Layer Normalization, Rotary Position Embeddings (RoPE), and efficient sequence document packing.
- **[00:58:30] Mixture of Experts (MoE) Layer Breakdown**: Detailed inspection of MoE sub-networks: Router top-$k$ classification, expert selection, sparse vs. active parameters, and Mixtral 8x7B resource analysis.
- **[01:07:44] Conclusion & Final Remarks**: Summary of architectural intuitions and course wrap-up.

---

## 5. Detailed Explanation

### Evolution of Text Vectorization

#### Bag-of-Words (BoW)
Bag-of-Words splits text into discrete sub-units called tokens using whitespace or punctuation tokenization. Given a corpus of documents, a unified vocabulary $V$ containing all unique tokens is created. A document $D$ is represented as a sparse vector $\mathbf{v} \in \mathbb{R}^{|V|}$, where each entry $v_i$ represents the raw count of vocabulary token $i$ within document $D$:

$$v_i = \text{count}(w_i, D)$$

*Limitations*: BoW completely ignores word order ($\text{count}("dog\ bites\ man") = \text{count}("man\ bites\ dog")$) and produces large, sparse vectors that scale linearly with $|V|$ without capturing semantic similarity.

```
Document 1: "That is a cute dog"
Document 2: "My cat is cute"

Vocabulary V = ["That", "is", "a", "cute", "dog", "My", "cat"] (|V| = 7)

BoW Vector (Document 2): [0, 1, 0, 1, 0, 1, 1]
```

#### Word2Vec (Static Embeddings)
Introduced by Mikolov et al. (2013), Word2Vec learns dense, low-dimensional vector representations $\mathbf{e}_w \in \mathbb{R}^d$ ($d \ll |V|$, typically $d \in [300, 1024]$) by training shallow neural networks to predict target words from context words (Continuous Bag-of-Words) or context words from target words (Skip-Gram). The objective function maximizes the dot-product similarity between co-occurring word vectors:

$$\mathcal{L} = \sum_{t=1}^{T} \sum_{-c \le j \le c, j \neq 0} \log P(w_{t+j} | w_t) \quad \text{where} \quad P(w_o | w_i) = \frac{\exp(\mathbf{v}'_{w_o}{^\top} \mathbf{v}_{w_i})}{\sum_{w=1}^{|V|} \exp(\mathbf{v}'_{w}{^\top} \mathbf{v}_{w_i})}$$

*Limitations*: Embeddings are static. The word `"bank"` receives the exact same vector representation whether it occurs in *"river bank"* or *"financial bank"*.

```
Static Word Vector properties (Word2Vec):
Word: "cats"  -> Vector: [-0.2,  0.8, -0.9,  0.1,  0.95] (High animal/plural, Low human)
Word: "puppy" -> Vector: [-0.1,  0.75,-0.85, 0.88, -0.4 ] (High animal/newborn, Low plural)
```

#### Contextualized Embeddings via Recurrent Neural Networks & Attention
Recurrent Neural Networks (RNNs) process token sequences sequentially, maintaining a hidden state vector $\mathbf{h}_t = f(\mathbf{h}_{t-1}, \mathbf{x}_t)$. In sequence-to-sequence translation, an Encoder compresses the entire input sequence into a single fixed-size context vector $\mathbf{c} = \mathbf{h}_T$. A Decoder then autoregressively generates target tokens conditioned on $\mathbf{c}$.

*The Bottleneck*: A single vector $\mathbf{c}$ cannot retain information for long sequences. 

*Bahdanau Attention (2014)* solved this by computing dynamic scalar weights $\alpha_{t,s}$ between decoder state $\mathbf{s}_t$ and all encoder hidden states $\mathbf{h}_s$:

$$\alpha_{t,s} = \frac{\exp(\text{score}(\mathbf{s}_{t-1}, \mathbf{h}_s))}{\sum_{k=1}^{S} \exp(\text{score}(\mathbf{s}_{t-1}, \mathbf{h}_k))}, \quad \mathbf{c}_t = \sum_{s=1}^{S} \alpha_{t,s} \mathbf{h}_s$$

---

### The Transformer Block Architecture

Vaswani et al. (2017) discarded recurrence entirely, relying exclusively on self-attention to model global dependencies in parallel. Modern generative models exclusively use the *Decoder-only* variant.

```
                 [ Output Token Distribution ]
                              ^
                    [ Language Model Head ]
                              ^
             +----------------------------------+
             | Layer Normalization (Pre-LN)    |
             |----------------------------------|
             | Feed-Forward Neural Network      |
             | (SwiGLU / MLP Expansion)         |
             +----------------------------------+
                              ^ (Residual Connection +)
                              |
             +----------------------------------+
             | Layer Normalization (Pre-LN)    |
             |----------------------------------|
             | Masked Self-Attention            |
             | (Grouped-Query Attention + RoPE) |
             +----------------------------------+
                              ^ (Residual Connection +)
                              |
                     [ Input Embeddings ]
                              ^
                    [ Subword Tokenizer ]
```

#### 1. Tokenization & Initial Embedding
Input text is split into subword token IDs $t_1, t_2, \dots, t_N$ using algorithms such as Byte-Pair Encoding (BPE) or WordPiece. Each token ID indexes an embedding matrix $W_E \in \mathbb{R}^{|V| \times d_{\text{model}}}$, producing initial hidden representation vectors $\mathbf{X}^{(0)} \in \mathbb{R}^{N \times d_{\text{model}}}$.

#### 2. Scaled Dot-Product Self-Attention
For a given layer $\ell$, input representations $\mathbf{X}$ are linearly projected into Query ($Q$), Key ($K$), and Value ($V$) space using learned projection matrices $W_Q, W_K, W_V \in \mathbb{R}^{d_{\text{model}} \times d_{\text{head}}}$:

$$Q = \mathbf{X} W_Q, \quad K = \mathbf{X} W_K, \quad V = \mathbf{X} W_V$$

The similarity between all Query vectors and Key vectors is computed via dot products, scaled by the square root of the head dimension $\sqrt{d_k}$ to stabilize gradients during softmax activation:

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{Q K^\top}{\sqrt{d_k}} + M\right) V$$

Where $M$ is the causal attention mask preventing position $i$ from attending to future positions $j > i$:

$$M_{i,j} = \begin{cases} 0 & \text{if } i \ge j \\ -\infty & \text{if } i < j \end{cases}$$

```
Self-Attention Calculation Steps:
1. Matrix Multiply Q and K^T       ---> Raw Attention Scores Matrix (N x N)
2. Scale by 1 / sqrt(d_k)         ---> Stabilized Logits
3. Apply Causal Mask M            ---> Set Upper-Triangular values to -infinity
4. Softmax per row                ---> Relevance Weights (Sum to 1.0 per row)
5. Matrix Multiply by V           ---> Enriched Contextualized Output Vectors
```

#### 3. Feed-Forward Neural Networks (FFNN)
Following self-attention, each token vector passes independently through a two-layer multi-layer perceptron (MLP). The FFNN expands the inner dimensionality (typically $d_{\text{ff}} \approx 4 \times d_{\text{model}}$ or $\frac{8}{3} d_{\text{model}}$ for SwiGLU variants) to compute non-linear feature interactions before projecting back down:

$$\text{FFN}(\mathbf{x}) = \text{Activation}(\mathbf{x} W_1 + \mathbf{b}_1) W_2 + \mathbf{b}_2$$

While Self-Attention routes information *across* tokens in a sequence, the FFNN acts as a static key-value memory layer that processes factual correlations *within* single vector dimensions.

---

### Decoding & Probabilistic Sampling Mechanics

At layer $L$, the final sequence representations $\mathbf{X}^{(L)} \in \mathbb{R}^{N \times d_{\text{model}}}$ enter the **Language Modeling Head**. The last token's output vector $\mathbf{x}_N^{(L)}$ is multiplied by the transposed embedding matrix $W_E^\top \in \mathbb{R}^{d_{\text{model}} \times |V|}$ (weight tying) or a distinct classification weight matrix $W_{\text{lm}} \in \mathbb{R}^{d_{\text{model}} \times |V|}$ to derive raw vocabulary unnormalized probability scores (logits $\mathbf{z}$):

$$\mathbf{z} = \mathbf{x}_N^{(L)} W_{\text{lm}}$$

Softmax normalizes logits into discrete probability distributions:

$$P(w_i | w_{<N}) = \frac{\exp(z_i / T)}{\sum_{j=1}^{|V|} \exp(z_j / T)}$$

Where $T$ is the temperature hyperparameter:
- $T = 0$ (**Greedy Decoding**): Always selects $\arg\max_{i} (z_i)$. Output is deterministic.
- $T > 0$ (**Stochastic Sampling**): Flattens distributions to increase response output diversity.
- **Top-$P$ (Nucleus) Sampling**: Filters candidates to the smallest cumulative set of tokens whose summed probability exceeds threshold $P$:

$$\sum_{w \in V^{(P)}} P(w | w_{<N}) \ge P$$

---

## 6. Beginner Explanation (ELI5)

Imagine you are writing a story with a group of friends, one word at a time.

1. **Tokens (Building Blocks)**: Computers cannot read words like "dog" or "llama." Instead, a machine cuts words into small puzzle pieces called **Tokens**. Short words are single tokens; long or complicated words get sliced into tiny pieces (like "vocal-ization").
2. **Embeddings (Map Coordinates)**: Every token gets mapped to a specific address in a giant multi-dimensional map. Similar concepts are placed close together (e.g., "king" and "queen" sit near each other; "apple" and "banana" sit in a different district).
3. **The Transformer (The Story Team)**: 
   - **Self-Attention (The Group Chat)**: When deciding the next word after *"The dog chased the llama because it..."*, the word *"it"* needs to figure out who it is talking about. Self-attention acts like a group chat where the word *"it"* asks all previous words: *"Who sent this message?"* The word *"llama"* shouts back with high confidence, so *"it"* attaches extra context from *"llama"*.
   - **Feed-Forward Neural Network (The Library)**: Once a word gathers information from its neighbors, it passes through a personal library (the Feed-Forward layer) to look up general world facts (like remembering that the capital of France is Paris).
4. **Greedy vs. Creative Generation**:
   - Setting **Temperature = 0 (Greedy)** means picking the absolute most predictable next word every time.
   - Setting a higher **Temperature** allows the computer to occasionally roll a die and select slightly unexpected words, making generated text sound more natural and creative.
5. **Mixture of Experts (The Specialist Committee)**: Imagine instead of one smart person answering every question, you have a room with 8 specialized consultants (a mathematician, a historian, a coder, etc.). A receptionist (**The Router**) looks at your question and hands it to only the 2 best experts for that task. You get the knowledge of a massive team while only paying for the time of two specialists!

---

## 7. Technical Deep Dive

### Scaled Attention Variants: MHA vs. MQA vs. GQA

Standard Multi-Head Attention (MHA) instantiates independent Query, Key, and Value heads, creating heavy memory memory overhead during autoregressive decoding. 

Let $h_q$ be the number of query heads, $h_{kv}$ be the number of key-value heads, $b$ be batch size, $s$ be sequence length, and $d_h$ be head dimension size.

| Attention Variant | Key-Value Heads ($h_{kv}$) | Memory Footprint per Token | Scalability / Throughput Impact |
| :--- | :--- | :--- | :--- |
| **Multi-Head Attention (MHA)** | $h_{kv} = h_q$ | $2 \times b \times s \times h_q \times d_h$ | Memory bandwidth bound at high batch sizes/long context. |
| **Multi-Query Attention (MQA)** | $h_{kv} = 1$ | $2 \times b \times s \times 1 \times d_h$ | Maximal memory compression; minor capacity loss. |
| **Grouped-Query Attention (GQA)** | $1 < h_{kv} < h_q$ | $2 \times b \times s \times g \times d_h$ | Optimal trade-off; preserves MHA quality with MQA speed. |

```
MHA (Multi-Head Attention):
Query Heads:    [ Q1 ] [ Q2 ] [ Q3 ] [ Q4 ] [ Q5 ] [ Q6 ] [ Q7 ] [ Q8 ]
Key/Value Heads:[ K1 ] [ K2 ] [ K3 ] [ K4 ] [ K5 ] [ K6 ] [ K7 ] [ K8 ]

MQA (Multi-Query Attention):
Query Heads:    [ Q1 ] [ Q2 ] [ Q3 ] [ Q4 ] [ Q5 ] [ Q6 ] [ Q7 ] [ Q8 ]
Key/Value Heads:[                   K1 / V1                   ]

GQA (Grouped-Query Attention - e.g., 2 Groups):
Query Heads:    [ Q1  Q2  Q3  Q4 ]  [ Q5  Q6  Q7  Q8 ]
Key/Value Heads:[     K1 / V1    ]  [     K2 / V2    ]
```

#### Key-Value (KV) Caching Formula
During generation step $t$, recalculating key and value vectors for historical tokens $1 \dots t-1$ requires redundant compute. KV caching stores historical vectors in GPU SRAM/VRAM. The total memory allocated for the KV Cache across an $L$-layer model is computed as:

$$\text{Memory}_{\text{KVCache}} = 2 \times b \times L \times h_{kv} \times d_h \times s \times p_{\text{bytes}}$$

Where $p_{\text{bytes}}$ represents parameter precision bytes (e.g., $2$ bytes for FP16/BF16, $1$ byte for INT8).

---

### Positional Encodings: Static vs. Rotary (RoPE)

Transformers are inherently permutation-invariant. To introduce token sequence ordering:

#### Absolute Static Positional Encoding (Vaswani 2017)
Static vectors calculated via sinusoidal functions are directly added to initial input token embeddings:

$$PE_{(pos, 2i)} = \sin\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right), \quad PE_{(pos, 2i+1)} = \cos\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)$$

$$\mathbf{X}_{\text{input}} = \mathbf{X}_{\text{embedding}} + PE$$

#### Rotary Position Embedding (RoPE - Su et al. 2021)
Modern decoders (Llama, Phi) incorporate relative positioning dynamically within self-attention by rotating Query and Key vectors in 2D vector subspaces by an angle proportional to sequence position $m$:

$$\mathbf{R}_{\Theta, m}^{d} = \text{diag}\left( R_{\theta_1, m}, R_{\theta_2, m}, \dots, R_{\theta_{d/2}, m} \right)$$

Where $R_{\theta_i, m}$ is a 2D rotation matrix:

$$R_{\theta_i, m} = \begin{pmatrix} \cos m\theta_i & -\sin m\theta_i \\ \sin m\theta_i & \cos m\theta_i \end{pmatrix}$$

The inner product between rotated Query at position $m$ and Key at position $n$ explicitly incorporates relative displacement $(m - n)$:

$$\langle \mathbf{R}_{\Theta, m}^{d} \mathbf{q}_m, \mathbf{R}_{\Theta, n}^{d} \mathbf{k}_n \rangle = \mathbf{q}_m^\top \mathbf{R}_{\Theta, n-m}^{d} \mathbf{k}_n$$

This formulation enables document packing during training and facilitates context length extension during inference.

---

### Mixture of Experts (MoE) Architecture & Parameter Sizing

An MoE layer replaces the standard dense FFNN layer with $E$ discrete expert feed-forward networks $\{E_1, E_2, \dots, E_n\}$ alongside a gating router network $G(\mathbf{x})$.

```
                    [ Aggregated Weighted Vector ]
                                  ^
                               (+ Sum)
                +-----------------+-----------------+
                | (Weight g_1)                      | (Weight g_2)
          [ Expert 1 (FFN) ]                 [ Expert 3 (FFN) ]
                ^                                   ^
                +-----------------+-----------------+
                                  |
                   [ Router Classifier G(x) ]
                                  ^
                     [ Input Vector from LayerNorm ]
```

#### Routing Formulation
Given input vector $\mathbf{x}$, the router computes softmax probabilities over linear transformation weights $W_g \in \mathbb{R}^{d_{\text{model}} \times E}$:

$$G(\mathbf{x}) = \text{Softmax}\left( \text{TopK}( \mathbf{x} W_g, k ) \right)$$

Where $\text{TopK}(, k)$ sets all logits outside the top-$k$ values to $-\infty$. The final output vector $y$ is the weighted sum of selected experts:

$$y = \sum_{i \in \text{TopK}} G(\mathbf{x})_i \cdot E_i(\mathbf{x})$$

#### Parameter Calculations: Mixtral 8x7B Case Study
Mixtral 8x7B features $E = 8$ experts per layer with $k = 2$ active routing.

- **Shared Non-Expert Parameters ($P_{\text{shared}}$)**: Token embeddings, normalization parameters, self-attention layer parameters ($Q, K, V, O$ projections), and Language Modeling Head.
- **Expert Parameters per Layer ($P_{\text{expert\_ffn}}$)**: $5.6 \text{ Billion per expert}$.
- **Total Sparse Parameters ($P_{\text{sparse}}$)**: Total memory footprint required to load model weights into VRAM.

$$P_{\text{sparse}} = P_{\text{shared}} + \sum_{\ell=1}^{L} \left( P_{\text{router}}^{(\ell)} + E \times P_{\text{expert\_ffn}}^{(\ell)} \right) \approx 46.7 \text{ Billion}$$

- **Active Compute Parameters ($P_{\text{active}}$)**: Parameters executed during a forward pass for a single token.

$$P_{\text{active}} = P_{\text{shared}} + \sum_{\ell=1}^{L} \left( P_{\text{router}}^{(\ell)} + k \times P_{\text{expert\_ffn}}^{(\ell)} \right) \approx 12.9 \text{ Billion}$$

---

## 8. Important Definitions

- **Autoregressive Generation**: Sequential decoding pattern where an LLM generates text one token at a time, using previously output tokens as input context for subsequent predictions.
- **Token**: The primary atomic structural sub-unit of text processed by an LLM, representing a word, subword segment, character, or symbol.
- **Vocabulary**: The pre-defined set of all unique tokens supported by an LLM's tokenizer (e.g., 32,000 for Llama-2; 128,000 for Llama-3).
- **Self-Attention**: An internal sequence-mixing mechanism computing pairwise relevance scalar weights across all tokens in a prompt, enriching isolated token representations with contextual dependencies.
- **Language Modeling Head**: The terminal output linear classification layer converting the final transformer block vector into unnormalized logit scores across the entire vocabulary.
- **Greedy Decoding**: A deterministic inference strategy selecting the single token index with the highest probability score ($T=0$).
- **KV Caching**: An optimization technique storing computed Key and Value vectors in GPU memory to eliminate redundant matrix operations during sequential generation.
- **Grouped-Query Attention (GQA)**: An attention architecture variant partitioning Query heads into distinct groups sharing single Key/Value head pairs to optimize VRAM bandwidth.
- **Rotary Position Embedding (RoPE)**: A dynamic positional encoding technique rotating Query and Key vectors in complex space to represent sequence positioning based on relative distance.
- **Mixture of Experts (MoE)**: A sparse network architecture replacing standard dense Feed-Forward layers with multiple parallel expert sub-networks routed by a learnable classifier.
- **Sparse Parameters**: The absolute total count of model weights that must be loaded into memory to support an MoE architecture.
- **Active Parameters**: The exact subset of parameters involved in matrix multiplication operations for a single token forward pass.

---

## 9. Code Snippets & Configuration Examples

### Inspecting LLM Layer Architecture with PyTorch & Transformers

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

# 1. Load Model and Tokenizer (Phi-3-Mini execution example)
model_id = "microsoft/Phi-3-mini-4k-instruct"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(
    model_id, 
    torch_dtype=torch.float16, 
    device_map="auto"
)

# 2. Inspect Structural Hierarchy
print("=== Model Architecture Hierarchy ===")
print(model)

# 3. Tokenize Input Prompt
prompt = "The capital of France is"
inputs = tokenizer(prompt, return_tensors="pt").to("cuda")

print("\n=== Tokenized Sequence Details ===")
print(f"Token IDs Shape : {inputs['input_ids'].shape}")
print(f"Tokenized IDs   : {inputs['input_ids'][0].tolist()}")
print(f"Decoded Tokens  : {[tokenizer.decode([t]) for t in inputs['input_ids'][0]]}")

# 4. Step-by-Step Forward Pass (Extracting Hidden States & Logits)
with torch.no_grad():
    # Pass through Transformer Blocks (Excluding LM Head)
    transformer_outputs = model.model(inputs['input_ids'])
    last_hidden_state = transformer_outputs.last_hidden_state
    
    # Pass Final Vector through LM Head manually
    logits = model.lm_head(last_hidden_state)

print("\n=== Tensor Output Dimensions ===")
print(f"Last Hidden State Shape [Batch, Seq, Hidden_Dim] : {last_hidden_state.shape}")
print(f"Logits Matrix Shape     [Batch, Seq, Vocab_Size]  : {logits.shape}")

# 5. Extract Next Token Logits & Compute Greedy Prediction
next_token_logits = logits[0, -1, :]
top_token_id = torch.argmax(next_token_logits).item()
predicted_token = tokenizer.decode([top_token_id])

print("\n=== Generation Logit Selection ===")
print(f"Predicted Token ID   : {top_token_id}")
print(f"Predicted Text Token : '{predicted_token}'")
```

---

### Custom PyTorch Implementation of Scaled Dot-Product Attention with GQA

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class GroupedQueryAttention(nn.Module):
    def __init__(self, d_model: int, n_heads: int, n_kv_groups: int):
        super().__init__()
        self.d_model = d_model
        self.n_heads = n_heads
        self.n_kv_groups = n_kv_groups
        self.n_kv_heads = n_heads // n_kv_groups
        self.head_dim = d_model // n_heads

        self.q_proj = nn.Linear(d_model, n_heads * self.head_dim, bias=False)
        self.k_proj = nn.Linear(d_model, self.n_kv_heads * self.head_dim, bias=False)
        self.v_proj = nn.Linear(d_model, self.n_kv_heads * self.head_dim, bias=False)
        self.out_proj = nn.Linear(n_heads * self.head_dim, d_model, bias=False)

    def forward(self, x: torch.Tensor, mask: torch.Tensor = None) -> torch.Tensor:
        batch_size, seq_len, _ = x.shape

        # Project representations
        q = self.q_proj(x).view(batch_size, seq_len, self.n_heads, self.head_dim).transpose(1, 2)
        k = self.k_proj(x).view(batch_size, seq_len, self.n_kv_heads, self.head_dim).transpose(1, 2)
        v = self.v_proj(x).view(batch_size, seq_len, self.n_kv_heads, self.head_dim).transpose(1, 2)

        # Expand Key/Value heads to match Query groups
        if self.n_kv_groups > 1:
            k = k.repeat_interleave(self.n_kv_groups, dim=1)
            v = v.repeat_interleave(self.n_kv_groups, dim=1)

        # Scaled Dot-Product Attention
        scores = torch.matmul(q, k.transpose(-2, -1)) / (self.head_dim ** 0.5)

        if mask is not None:
            scores = scores.masked_fill(mask == 0, float('-inf'))

        attn_weights = F.softmax(scores, dim=-1)
        output = torch.matmul(attn_weights, v) # [batch, n_heads, seq_len, head_dim]
        
        output = output.transpose(1, 2).contiguous().view(batch_size, seq_len, -1)
        return self.out_proj(output)
```

---

## 10. Best Practices

1. **Decoding Parameter Calibration**:
   - Use **Greedy Decoding** ($T=0$) for deterministic tasks like code generation, structured JSON extraction, and mathematical reasoning.
   - Use **Top-$P$ (Nucleus) Sampling** ($P \in [0.85, 0.95]$, $T \in [0.6, 0.8]$) for creative writing, translation, and open-ended dialogue to prevent repetitive loops while filtering out improbable tails.
2. **Context Window & KV Cache Optimization**:
   - Enable **PagedAttention** (vLLM) and **Grouped-Query Attention (GQA)** models in production to minimize VRAM fragmentation during multi-turn chats.
   - Utilize 8-bit or 4-bit KV cache quantization when context lengths exceed 32,000 tokens to prevent out-of-memory errors during generation.
3. **Training Data Document Packing**:
   - Avoid padding sequences naively during base model training or fine-tuning. Concatenate multiple documents into single rows separated by `<eos>` tokens, applying custom block diagonal attention masks to maximize GPU throughput.
4. **Mixture of Experts Deployment**:
   - Allocate GPU host memory based on **Total Sparse Parameters** ($P_{\text{sparse}}$), but size generation batch limits based on **Active Compute Parameters** ($P_{\text{active}}$). Ensure auxiliary load-balancing losses are monitored to prevent expert collapse during training.

---

## 11. Common Mistakes

- **Confusing MoE Experts with Domain Specialists**:
  *Misconception*: Assuming individual experts in models like Mixtral become dedicated subject experts (e.g., Expert 1 = Biology, Expert 2 = Law).
  *Reality*: Router networks route tokens based on low-level sub-token syntactic properties, part-of-speech structure, and positioning—not broad domain topics.
- **Ignoring KV Cache Memory Expansion**:
  Failing to account for linear KV cache VRAM growth as token sequence lengths expand. At long context lengths (e.g., 128k tokens), KV cache memory demands can surpass the static weight footprint of the model itself.
- **Assuming Word2Vec Handles Polysemy**:
  Believing static word embeddings can differentiate between multiple word meanings. Static vectors generate identical representations for homonyms regardless of surrounding context.
- **Mistaking Model Context Length for Generation Capacity**:
  Equating an LLM's maximal input context window (e.g., 128,000 context limit) with its maximum single-step output generation limit (typically capped at 4,096 tokens due to decoder output settings).

---

## 12. Interview Questions

### Q1: Explain the functional difference between the Self-Attention layer and the Feed-Forward Neural Network (FFNN) layer within a Transformer block.
**Answer:**
Self-Attention is a sequence-mixing layer. It computes dynamic relational scores across *all* sequence positions, enabling individual tokens to pull context from preceding or surrounding tokens (e.g., resolving pronoun references). 

Conversely, the Feed-Forward Neural Network (FFNN) is a position-wise layer. It operates on each token vector independently, projecting features into higher dimensions. It functions as a static memory store that encodes non-contextual world knowledge, syntax rules, and associations learned during pretraining.

---

### Q2: How does Grouped-Query Attention (GQA) improve LLM inference efficiency compared to Multi-Head Attention (MHA)?
**Answer:**
Multi-Head Attention maintains an equal number of Query, Key, and Value heads ($h_q = h_{kv}$). During autoregressive inference, every key and value head matrix must be fetched from VRAM at every generation step, bound by memory bandwidth. 

GQA groups query heads together so that multiple Query heads share a single Key/Value head pair ($h_{kv} < h_q$). This reduces key-value memory read operations by a factor of $h_q / h_{kv}$, reducing KV cache memory footprints and enabling larger batch sizes during inference without degrading output quality.

---

### Q3: Why are static absolute positional encodings inadequate for modern dynamic context LLMs, and how do Rotary Position Embeddings (RoPE) resolve this?
**Answer:**
Static absolute encodings add fixed positional vectors to token embeddings prior to the initial transformer layer. This approach struggles with sequence length extrapolation beyond pretraining boundaries and fails when documents are packed dynamically into single context windows. 

RoPE resolves this by applying relative rotational transformations directly to Query and Key vectors inside self-attention heads at each layer. Rotating vectors by angles proportional to their relative distance preserves positional relationships regardless of absolute token indices, enabling smooth context extrapolation and efficient sequence packing.

---

### Q4: In a Mixture of Experts (MoE) model like Mixtral 8x7B, why is it incorrect to calculate total computational cost as $8 \times 7\text{B}$ operations per forward pass?
**Answer:**
While Mixtral load-allocates 8 distinct expert networks into VRAM (totaling ~46.7 billion sparse parameters), its top-$k$ router selects only 2 active experts per token for execution. Thus, a forward pass executes matrix multiplications across shared parameters plus only 2 experts (totaling ~12.9 billion active compute parameters). Computational throughput matches a 13B dense model, despite delivering the capacity of a significantly larger parameter network.

---

## 13. Certification Questions

### Q1: What is the primary operational characteristic of an autoregressive LLM during text generation?
- A) It processes the entire output prompt in a single parallel step.
- B) It generates outputs token-by-token, appending each new token to the context window for subsequent predictions.
- C) It leverages encoder-only modules to refine global sentence representations dynamically.
- D) It uses bidirectional attention to modify previously generated tokens.

**Correct Answer:** B  
**Explanation:** Autoregressive generative decoders process context sequentially, predicting one token per forward pass. Each generated token is appended to the input sequence to generate the next token logit distribution.

---

### Q2: How does setting Temperature to 0 ($T=0$) affect the Language Modeling Head's sampling process?
- A) It introduces maximum randomness, sampling uniformly across all tokens.
- B) It enforces Nucleus (Top-$P$) sampling with a threshold of $0.5$.
- C) It converts decoding to Greedy Decoding, deterministically picking the token with the highest logit probability score.
- D) It bypasses softmax calculations and picks random subword tokens.

**Correct Answer:** C  
**Explanation:** A temperature of zero eliminates logit scaling variance, turning the softmax output into a point distribution where the highest probability token is selected every time (Greedy Decoding).

---

### Q3: In an MoE architecture, what is the core responsibility of the Router network?
- A) To split incoming text into subword tokens.
- B) To compute multi-head cross-attention between queries and key values.
- C) To calculate token probability scores for the top 50,000 vocabulary words.
- D) To calculate probability scores across available expert networks and route tokens to the top-$k$ experts.

**Correct Answer:** D  
**Explanation:** The router is a lightweight linear neural network within the MoE layer that scores token suitability across expert sub-networks and assigns execution to the top-$k$ experts.

---

## 14. Real-World Examples

### 1. Enterprise RAG Systems (Encoder vs. Decoder Hybrid Pipelines)
Production Retrieval-Augmented Generation (RAG) deployments combine encoder-only models (e.g., `BAAI/bge-large-en-v1.5`) to encode document chunks into dense vector embeddings for vector databases, paired with fast decoder-only LLMs (e.g., Llama-3-8B using GQA) to process retrieved context and generate accurate answers.

### 2. Commercial Chat API Infrastructure (KV Cache & PagedAttention)
High-throughput LLM platforms like OpenAI or Anthropic rely on KV caching and memory paging (vLLM) to serve millions of simultaneous requests. Caching multi-turn conversation histories in memory allows servers to calculate self-attention only for newly added user messages, keeping response times fast and consistent.

### 3. Edge LLM Execution (Mixture of Experts on Consumer Hardware)
Deploying Mixtral 8x7B on localized workstations highlights the power of sparse activation. By offloading sparse parameters to system RAM and caching the 12.9B active expert paths on local GPU VRAM, edge devices can achieve high-capacity inference without requiring multi-GPU server clusters.

---

## 15. Analogies

### 1. The Symphony Orchestra (Multi-Head Self-Attention)
Imagine an orchestra performance where each musician represents a word in a sentence. 
- **Non-attention baseline**: Every musician plays their isolated sheet music without listening to others.
- **Self-Attention**: The conductor forces musicians to adjust their volume and timing based on neighboring instruments. The violinist (Query) listens closely to the cellist (Key) and blends their tones together (Value), creating a unified harmonic sound (Contextual Representation).

### 2. The Hospital Triage Desk (MoE Router & Experts)
Imagine a busy hospital emergency department:
- **Dense Model**: Every patient is examined by every single doctor in the building sequentially, regardless of their ailment.
- **MoE Model**: A triage nurse (**The Router**) evaluates incoming patients (**Tokens**) and directs someone with a broken bone to the Orthopedist and Radiologist (**Top-2 Experts**), bypassing the Cardiologist and Neurologist (**Unactivated Experts**).

### 3. The Sticky-Note Reader (KV Cache)
Imagine reading a 500-page mystery book to write a summary:
- **Without KV Cache**: Every time you turn to a new page, you are forced to re-read the entire book from page 1 to refresh your memory before reading the new page.
- **With KV Cache**: As you read each page, you write key summary notes on sticky notes (**Keys and Values**). When you turn to page 501, you simply read page 501 and reference your existing stack of sticky notes.

---

## 16. Frequently Asked Questions

### 1. What is the main difference between an Encoder-only and a Decoder-only Transformer?
Encoder-only models (e.g., BERT) use bidirectional attention to process all tokens simultaneously, making them ideal for text classification and embedding extraction. Decoder-only models (e.g., GPT, Llama) use causal masked self-attention to restrict tokens from attending to future context, making them optimal for autoregressive text generation.

### 2. Why do modern transformers use subword tokenization instead of whole words?
Whole-word tokenization requires massive vocabularies and struggles with out-of-vocabulary (OOV) words. Subword tokenization (e.g., Byte-Pair Encoding) breaks rare or complex words into smaller components (e.g., `"un-believ-able"`), allowing models to process any arbitrary string using compact vocabularies (e.g., 32,000 to 128,000 tokens).

### 3. How does setting a non-zero Temperature alter LLM outputs?
Temperature scales raw logit scores before the softmax function ($z_i / T$). Lower temperatures ($T < 1$) sharpen the probability distribution, making high-probability tokens more dominant. Higher temperatures ($T > 1$) flatten the distribution, increasing output variety and creativity by giving lower-ranked tokens a better chance of selection.

### 4. What is the primary computational bottleneck during LLM inference?
The primary bottleneck during autoregressive generation is **memory bandwidth**, not pure compute capability. Transferring massive weight matrices and historical KV cache tensors from GPU VRAM to high-speed SRAM at every generation step limits throughput. Techniques like GQA and KV cache quantization directly mitigate this bottleneck.

### 5. Why are Mixture of Experts (MoE) models harder to train than standard dense models?
MoE models suffer from load-balancing challenges where the router network defaults to sending all tokens to a few favorite experts (expert collapse). Training requires specialized loss functions to balance load across experts, along with distributed computing frameworks to manage parallel routing across GPUs.

---

## 17. Related Technologies

- **vLLM / PagedAttention**: An open-source LLM serving engine that optimizes KV cache memory management using virtual memory paging techniques to boost inference throughput.
- **FlashAttention (Dao et al.)**: An exact self-attention algorithm that reorganizes GPU memory reads/writes to compute attention in fast SRAM, reducing memory usage from $\mathcal{O}(N^2)$ to linear $\mathcal{O}(N)$.
- **Hugging Face Transformers**: The standard Python framework for loading, fine-tuning, inspecting, and running open-source LLM architectures.
- **State Space Models (SSMs - Mamba/Jamba)**: Non-transformer neural architectures that replace self-attention with linear state transitions, offering continuous $\mathcal{O}(N)$ sequence scaling while integrating MoE layers.
- **TensorRT-LLM**: NVIDIA's industrial optimization library designed for compiling, quantizing (FP8, INT4), and executing high-throughput LLM deployments on Tensor Core GPUs.

---

## 18. Important Quotes

> *"The original transformer architecture consisted of two main parts: an encoder and decoder... The encoder model provides rich context-sensitive representations... and the decoder model performs text generation tasks... and is the basis for most popular LLMs."* — **Andrew Ng**

> *"Transformers might seem a little bit like magic to some people... But the magic of LLMs actually comes from two parts: One, the transformer architecture... and second, all the incredibly rich data that the models learn from."* — **Andrew Ng**

> *"Self-attention allows the model to attend to previous tokens and incorporate context into its understanding of the token it's currently looking at... It boils down to two things: relevance scoring, and then combining the relevant information."* — **Jay Alammar**

> *"Although having multiple experts rather than a single expert seems like it would only increase computational requirements, it is actually a bit more nuanced... parameter loading requires full memory footprint, but inference executes on a fraction of active parameters."* — **Maarten Grootendorst**

---

## 19. Glossary

- **Autoregressive**: A model property where outputs are generated sequentially, feeding past outputs back into the model as context for future predictions.
- **BPE (Byte-Pair Encoding)**: A subword tokenization algorithm that iteratively merges the most frequent pairs of characters or bytes into unified vocabulary units.
- **Causal Attention Mask**: A triangular matrix mask applied within decoder self-attention to block tokens from attending to future sequence positions.
- **Embedding**: A continuous dense vector representation mapping categorical tokens into a high-dimensional mathematical space.
- **Feed-Forward Neural Network (FFNN)**: A position-wise MLP layer within a transformer block that processes individual token representations independently.
- **GQA (Grouped-Query Attention)**: An efficient attention mechanism where multiple Query heads share a single Key/Value head group.
- **KV Cache**: A reserved VRAM buffer storing Key and Value matrix states for historical context tokens to accelerate generation steps.
- **Language Modeling Head**: A linear classification projection layer converting transformer hidden vectors into vocabulary logit scores.
- **MHA (Multi-Head Attention)**: The original attention layer setup where Query, Key, and Value projections maintain equal head counts.
- **MoE (Mixture of Experts)**: A sparse neural network design replacing dense MLPs with dynamic router-selected expert sub-networks.
- **Nucleus (Top-$P$) Sampling**: A decoding strategy sampling tokens from the smallest dynamic set whose cumulative probability mass exceeds threshold $P$.
- **Pre-Layer Normalization (Pre-LN)**: An architectural design placing normalization layers before self-attention and FFNN blocks to stabilize gradient flow in deep networks.
- **RoPE (Rotary Position Embedding)**: A positional encoding method applying rotational transformations to $Q$ and $K$ vectors to encode relative position.
- **Softmax**: A mathematical function converting arbitrary vector logit scores into normalized probability distributions that sum to 1.
- **Temperature**: A hyperparameter scaling logit values before softmax, controlling randomness in text generation.

---

## 20. One-Page Cheat Sheet

### Core Mathematical Equations

$$\text{Scaled Dot-Product Attention: } \text{Attention}(Q, K, V) = \text{softmax}\left( \frac{QK^\top}{\sqrt{d_k}} + M \right) V$$

$$\text{Temperature Softmax Scaling: } P(w_i) = \frac{\exp(z_i / T)}{\sum_{j} \exp(z_j / T)}$$

$$\text{MoE Output Aggregation: } y = \sum_{i \in \text{TopK}} \text{Softmax}(\text{TopK}(\mathbf{x}W_g, k))_i \cdot E_i(\mathbf{x})$$

$$\text{KV Cache Memory (Bytes): } 2 \times b \times L \times h_{kv} \times d_h \times s \times p_{\text{bytes}}$$

---

### Architectural Differences Summary

| Component | Standard Encoder (BERT) | Standard Decoder (GPT / Llama) | Mixture of Experts Decoder (Mixtral) |
| :--- | :--- | :--- | :--- |
| **Primary Objective** | Contextual Token Representations | Autoregressive Text Generation | High-Throughput Autoregressive Generation |
| **Attention Masking** | Bidirectional (No Causal Mask) | Causal (Upper-Triangular Mask) | Causal (Upper-Triangular Mask) |
| **FFNN Architecture** | Single Dense MLP per layer | Single Dense MLP per layer | Multiple Sparse Expert MLPs per layer |
| **Positional Encoding** | Absolute Learned / Sinusoidal | Absolute or Rotary (RoPE) | Rotary Embeddings (RoPE) |
| **Active vs. Loaded Parameters** | Active == Loaded | Active == Loaded | Active $\ll$ Loaded (Sparse Execution) |

---

### Decoding Hyperparameter Quick Reference

```
Setting: Temperature = 0.0  -> Greedy Decoding  (Deterministic, reproducible, rigid)
Setting: Temperature = 0.7  -> Balanced Sampling (Natural flow, conversational, creative)
Setting: Top-P       = 0.9  -> Nucleus Filter    (Cuts off long-tail improbable tokens)
Setting: Top-K       = 40   -> Candidate Filter  (Restricts choices to top 40 tokens)
```

---

## 21. Flash Cards

- **Card 1 | Foundations**
  - **Q:** What are the three core functional components of a Transformer LLM architecture?
  - **A:** 1. The Subword Tokenizer, 2. The Stack of Transformer Blocks (Self-Attention + FFNN), 3. The Language Modeling Head.

- **Card 2 | Attention Mechanics**
  - **Q:** What roles do Queries ($Q$), Keys ($K$), and Values ($V$) play in self-attention?
  - **A:** Queries represent the current token seeking context; Keys represent available tokens to attend to; Values contain the actual content merged into the output representation based on $Q \cdot K$ relevance scores.

- **Card 3 | Efficiency Optimizations**
  - **Q:** How does KV Caching accelerate text generation speed?
  - **A:** It stores key and value projection vectors of historical tokens in GPU memory, avoiding redundant re-computations during autoregressive generation.

- **Card 4 | Attention Architecture**
  - **Q:** What distinguishes Grouped-Query Attention (GQA) from Multi-Head Attention (MHA)?
  - **A:** MHA assigns dedicated Key and Value heads for every Query head; GQA groups multiple Query heads to share single Key/Value pairs, reducing memory overhead.

- **Card 5 | Mixture of Experts**
  - **Q:** What is the difference between sparse parameters and active parameters in an MoE model?
  - **A:** Sparse parameters represent the total weights loaded into VRAM across all experts; active parameters are the subset executed during a single token forward pass.

- **Card 6 | Positional Encodings**
  - **Q:** How do Rotary Position Embeddings (RoPE) encode token order?
  - **A:** RoPE rotates Query and Key vectors in complex space by angles proportional to sequence position, encoding relative distance into self-attention.

---

## 22. Quiz

### Q1: What was the primary structural flaw of Bag-of-Words (BoW) text representations?
- A) Vectors were too small to store in memory.
- B) Vectors were order-agnostic and failed to capture semantic word meaning.
- C) Computations required quadratic self-attention layers.
- D) It generated dynamic representations that changed constantly.  
**Correct Answer:** B  
**Explanation:** Bag-of-Words counts isolated token frequencies without considering sentence structure or semantic context, making "dog bites man" and "man bites dog" identical.

---

### Q2: Word2Vec introduced dense embeddings, but suffered from which major limitation?
- A) It was limited to 10 vocabulary tokens.
- B) It could only execute on GPUs.
- C) It assigned static embeddings to words, failing to adapt to context (polysemy).
- D) It required causal attention masks during training.  
**Correct Answer:** C  
**Explanation:** Word2Vec produces a single static vector per word, meaning homonyms like "bank" receive the exact same vector regardless of context.

---

### Q3: Why did Recurrent Neural Networks (RNNs) struggle with machine translation of long sentences?
- A) They could not process text on CPUs.
- B) Sequential hidden states created an information bottleneck when compressing long sequences into a single context vector.
- C) They were blocked by rotary positional encodings.
- D) They executed self-attention across multiple heads simultaneously.  
**Correct Answer:** B  
**Explanation:** Standard RNN sequence-to-sequence models compressed the entire input sentence into a single fixed-size vector, losing critical details from early sequence positions.

---

### Q4: The original Vaswani et al. (2017) Transformer was designed for machine translation using which paradigm?
- A) Decoder-only
- B) Encoder-only
- C) Encoder-Decoder
- D) Mixture of Experts  
**Correct Answer:** C  
**Explanation:** The original 2017 Transformer used an Encoder block to extract context from input sentences (e.g., English) and a Decoder block to generate output translations (e.g., German).

---

### Q5: In modern Decoder-only LLMs, what is the specific role of the Causal Attention Mask?
- A) To prevent models from accessing the internet.
- B) To force tokens to attend only to current and preceding tokens, blocking future positions.
- C) To route tokens to specific expert networks.
- D) To compress key-value vector caches into 4-bit integers.  
**Correct Answer:** B  
**Explanation:** Causal masking sets upper-triangular attention values to $-\infty$, ensuring autoregressive models predict the next token relying exclusively on past context.

---

### Q6: What computational operation takes place inside the Language Modeling Head?
- A) Applying rotary matrices to input subword tokens.
- B) Multiplying the final hidden state vector by the vocabulary projection matrix to compute logit scores.
- C) Sorting token vectors into 8 distinct expert categories.
- D) Normalizing hidden representations using Pre-LN.  
**Correct Answer:** B  
**Explanation:** The LM Head projects the transformer block output vector across the vocabulary dimension $|V|$, deriving raw logit scores for every token in the vocabulary.

---

### Q7: If an LLM's decoding Temperature is set to $T = 0$, what decoding strategy is enforced?
- A) Top-$P$ Nucleus Sampling
- B) Random Walk Sampling
- C) Greedy Decoding
- D) Beam Search with penalty $1.5$  
**Correct Answer:** C  
**Explanation:** $T=0$ enforces Greedy Decoding, selecting the highest probability token at every step.

---

### Q8: What scaling factor is used in scaled dot-product attention $\text{softmax}\left(\frac{QK^\top}{\text{scale}}\right)$?
- A) $d_{\text{model}}$
- B) $\sqrt{d_k}$ (Square root of head dimension)
- C) $\log(|V|)$
- D) Total active parameter count  
**Correct Answer:** B  
**Explanation:** Scaling dot products by $\sqrt{d_k}$ prevents values from growing excessively large at high dimensions, avoiding vanishing gradients during softmax backpropagation.

---

### Q9: Which component of a Transformer block acts as a static memory store for world knowledge?
- A) The Masked Self-Attention layer
- B) The Rotary Embedding layer
- C) The Feed-Forward Neural Network (FFNN) layer
- D) The Tokenizer  
**Correct Answer:** C  
**Explanation:** Self-attention handles contextual sequence-mixing across tokens, while the FFNN acts as a static memory store containing learned facts and associations.

---

### Q10: How does Multi-Query Attention (MQA) achieve fast inference throughput?
- A) By eliminating all feed-forward layers.
- B) By forcing all Query heads to share a single Key/Value head.
- C) By dynamically dropping half of the input tokens.
- D) By replacing softmax with linear activations.  
**Correct Answer:** B  
**Explanation:** MQA compresses the KV cache footprint by sharing a single Key and Value head across all Query heads, minimizing memory transfers during generation.

---

### Q11: What primary benefit does Grouped-Query Attention (GQA) offer over MQA?
- A) It removes positional encodings completely.
- B) It balances output quality and memory efficiency by grouping query heads across a small set of Key/Value heads.
- C) It enables bi-directional attention in decoder models.
- D) It reduces the vocabulary size from 128,000 to 32,000.  
**Correct Answer:** B  
**Explanation:** GQA offers a middle ground between MHA (high quality, high VRAM use) and MQA (extreme memory savings, potential quality drop) by assigning query head groups to a reduced set of KV heads.

---

### Q12: Where are Rotary Position Embeddings (RoPE) applied inside modern Transformers?
- A) Directly to text prior to tokenization.
- B) To Query and Key vectors inside self-attention heads at every layer.
- C) Exclusively inside the final LM Head layer.
- D) To the output logits before softmax scaling.  
**Correct Answer:** B  
**Explanation:** RoPE applies rotational transformations directly to Query and Key vectors inside the self-attention mechanism at each layer, encoding relative token positioning.

---

### Q13: What is the main function of the Router network within a Mixture of Experts (MoE) layer?
- A) To tokenize incoming text prompts into subwords.
- B) To classify incoming token vectors and assign them to the top-$k$ most suitable expert networks.
- C) To compress model parameters into 4-bit representations.
- D) To apply causal masks to attention scores.  
**Correct Answer:** B  
**Explanation:** The MoE router evaluates token vectors and computes probability scores to direct execution to the top-$k$ expert sub-networks.

---

### Q14: In the Mixtral 8x7B MoE model, how many experts are actively selected per token?
- A) 1 Expert
- B) 2 Experts
- C) 4 Experts
- D) All 8 Experts  
**Correct Answer:** B  
**Explanation:** Mixtral 8x7B uses top-2 routing ($k=2$), directing each token through 2 of its 8 available expert networks per layer.

---

### Q15: Why does an MoE model require high VRAM to host, despite running inference with low compute resources?
- A) Because all sparse expert parameters must remain loaded in memory, even though only a small active subset runs per token pass.
- B) Because self-attention matrix sizes double in MoE architectures.
- C) Because routers recalculate embedding tables at every step.
- D) Because KV caching is disabled in sparse networks.  
**Correct Answer:** A  
**Explanation:** Full model weights (sparse parameters) must reside in GPU RAM to allow dynamic routing, but execution (active parameters) runs on only the selected expert subset per token.

---

## 23. Action Items

- [ ] **Step 1: Code Architecture Exploration**: Run the provided PyTorch/Transformers script to load an open-source model (e.g., `Phi-3-mini` or `Llama-3-8B`) and inspect layer attributes, parameters, and logit outputs.
- [ ] **Step 2: Attention Visualizations**: Implement a scaled dot-product self-attention module in PyTorch and plot attention score heatmap matrices across input sequence tokens.
- [ ] **Step 3: KV Cache Profiling**: Measure GPU VRAM usage during generation with and without KV caching enabled across sequence lengths ranging from 512 to 16,384 tokens.
- [ ] **Step 4: MoE Resource Calculation**: Benchmark inference speed and VRAM requirements for a dense model (e.g., `Llama-3-8B`) versus a sparse MoE model (e.g., `Mixtral-8x7B`).

---

## 24. Recommended Further Reading

- **Foundational Papers**:
  - Vaswani et al. (2017): [*Attention Is All You Need*](https://arxiv.org/abs/1706.03762) (Introduces original Transformer).
  - Devlin et al. (2018): [*BERT: Pre-training of Deep Bidirectional Transformers*](https://arxiv.org/abs/1810.04805) (Encoder-only architecture).
  - Su et al. (2021): [*RoFormer: Enhanced Transformer with Rotary Position Embedding*](https://arxiv.org/abs/2104.09864) (RoPE positional encodings).
  - Ainslie et al. (2023): [*GQA: Training Generalized Multi-Query Transformer Models*](https://arxiv.org/abs/2305.13245) (Grouped-Query Attention).
  - Jiang et al. (2024): [*Mixtral of Experts*](https://arxiv.org/abs/2401.04088) (Sparse MoE decoder implementation).
- **Books & Educational Resources**:
  - Alammar, J., & Grootendorst, M. (2024): *Hands-On Large Language Models* (O'Reilly Media).
  - DeepLearning.AI: *Short Course Series on Transformer Mechanics & Attention Implementation*.