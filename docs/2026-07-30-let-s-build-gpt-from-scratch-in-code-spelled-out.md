# Master Knowledge Document: Let's Build GPT from Scratch

---

## 1. Executive Summary

In *Let's build GPT: from scratch, in code, spelled out*, Andrej Karpathy presents a comprehensive, code-first tutorial demonstrating how to build an autoregressive Generatively Pre-trained Transformer (GPT) using Python and PyTorch. Starting from a character-level Bigram baseline trained on the "Tiny Shakespeare" corpus, the guide progressively builds up to the full Transformer architecture introduced in *Attention Is All You Need*. 

Karpathy details the core mechanics of generative language modeling: predicting the next token in a sequence using historic context. The video explores data preparation, character vs. subword tokenization, scaled dot-product self-attention (Queries, Keys, and Values), causal masking, multi-head attention, feed-forward sub-networks, residual skip connections, and layer normalization. The resultant architecture, "nanoGPT," mirrors the fundamental design of state-of-the-art models like GPT-2, GPT-3, and ChatGPT, demonstrating that modern generative AI relies on structured matrix operations and optimization pathways.

---

## 2. Key Takeaways

1. **Autoregressive Language Modeling**: GPT models operate on next-token prediction, generating output token-by-token by conditioning on historical context ($T$ tokens).
2. **Self-Attention as Information Exchange**: Self-attention functions as a communication mechanism over a directed graph where tokens calculate affinity through dot products of Queries ($Q$) and Keys ($K$), aggregating contextual content from Values ($V$).
3. **Causal Masking**: Autoregressive decoding requires a lower-triangular mask matrix ($M$) set to $-\infty$ above the diagonal to prevent past tokens from attending to future tokens.
4. **Variance Scaling**: Dot-product attention scores must be scaled by $\frac{1}{\sqrt{d_k}}$ to prevent `softmax` saturation and vanishing gradients during initialization.
5. **Multi-Head Attention**: Multiple parallel attention heads allow tokens to simultaneously gather information across diverse representations and relational channels.
6. **Architectural Stabilization**: Deep Transformer stacks rely on **Residual Connections** to preserve gradient highways during backpropagation and **Pre-Layer Normalization** to keep activations normalized.
7. **Representation Granularity**: Character-level tokenization provides small vocabulary sizes ($S$) with longer context lengths, whereas subword tokenization (e.g., Byte-Pair Encoding via `tiktoken`) balances vocabulary size and context length.

---

## 3. Topics Covered

* **Dataset Ingestion & Exploration**: Reading text files and constructing minimal data processing pipelines.
* **Tokenization & Vocabulary Mapping**: Converting raw string characters to numerical indices via lookups (`stoi` and `itos`).
* **Data Ingestion & Context Windowing**: Structuring inputs ($X$) and targets ($Y$) across context windows (`block_size`) and training batches (`batch_size`).
* **Bigram Baseline Model**: Implementing simple context-free lookup tables using `nn.Embedding`.
* **Optimization & Training Loop**: Setting up loss calculation (`cross_entropy`), backpropagation (`loss.backward()`), and optimizer updates using `AdamW`.
* **Self-Attention Mechanics (v1 to v4)**: Iterating from manual nested `for`-loops to parallel matrix operations using queries, keys, and values.
* **Positional Encodings**: Adding spatial context to permutation-invariant self-attention operations.
* **Multi-Head Attention**: Running multiple scaled dot-product attention operations in parallel.
* **Feed-Forward Networks (FFN)**: Applying per-token non-linear computations to process aggregated attention state.
* **Residual Connections & Layer Normalization**: Scaling model depth through skip connections and pre-layer normalization (`LayerNorm`).
* **nanoGPT Assembly & Text Generation**: Combining components into an autoregressive Transformer decoder and sampling outputs.

---

## 4. Timeline with Timestamps

* **[00:00] Intro: ChatGPT, Transformers, nanoGPT, Shakespeare**: Overview of generative language modeling, the Transformer revolution, and project roadmap.
* **[00:07:52] Reading and Exploring the Data (Tiny Shakespeare)**: Ingesting the 1.1MB Tiny Shakespeare dataset for training.
* **[00:09:28] Tokenization and Train/Validation Split**: Character-level lookup tables, train/val split (90/10), and BPE alternatives (`tiktoken`).
* **[00:14:27] Data Loader: Batches and Block Size**: Batching input tensors across context windows (`block_size`) and target alignment.
* **[00:22:11] Simplest Baseline: Bigram Language Model, Loss, Generation**: Building a lookup-table model, computing cross-entropy, and basic decoding.
* **[00:34:53] Training the Bigram Model and Porting to a Script**: Setting up the `AdamW` training loop and refactoring from Jupyter to Python scripts.
* **[00:42:13] Self-Attention v1: Averaging Past Context (for-loop version)**: Naive implementation of context aggregation via explicit iterative loops.
* **[00:47:11] Self-Attention v2: Matrix Multiply as Weighted Aggregation**: Vectorizing context aggregation using lower-triangular matrix multiplication.
* **[00:51:54] Self-Attention v3: Softmax and Attention Weights**: Computing dynamic data-dependent attention weights using dynamic scaling and `softmax`.
* **[00:58:26] Code Cleanup and Setup**: Modular refactoring in preparation for the complete attention block.
* **[01:00:18] Positional Encoding and Why It Matters**: Injecting token order using spatial embeddings.
* **[01:02:00] Self-Attention v4 (Full Mechanism)**: Implementing formal Scaled Dot-Product Attention with Query, Key, and Value projections.
* **[01:11:38] Note 1: Attention as Communication in a Graph**: Conceptualizing attention as a message-passing graph architecture.
* **[01:12:46] Note 2: Attention over Sets**: Permutation invariance in self-attention across unstructured data sets.
* **[01:13:31] Multi-Head Attention**: Parallelizing self-attention heads and concatenating outputs.
* **[01:17:34] Feed-Forward Network (FFN)**: Integrating token-wise linear transformations and non-linearities (ReLU).
* **[01:18:49] Transformer Block**: Combining multi-head attention and feed-forward sub-layers into an extensible block module.
* **[01:21:40] Residual Connections (Skip Connections)**: Adding skip paths to support deep gradient propagation.
* **[01:22:20] Layer Normalization**: Implementing unit variance and mean-zero scaling before sub-layer operations.
* **[01:25:22] Scaling Up (hyperparameters, dropout, etc.)**: Adding dropout regularizers and enlarging layer depth, embedding size, and head count.
* **[01:31:00] nanoGPT Final Model & Training**: Training the full `nanoGPT` network and monitoring cross-entropy loss convergence.
* **[01:35:48] Generating Text with nanoGPT**: Sampling text iteratively from the trained decoder.
* **[01:39:19] Connections to GPT-2/GPT-3/ChatGPT & Conclusion**: Comparing nanoGPT with production LLMs, pre-training, instruction fine-tuning, and RLHF.

---

## 5. Detailed Explanation

### Dataset Exploration, Tokenization, and Context Creation
Language modeling converts unstructured text into numerical tokens. A character-level tokenizer extracts every unique character in the corpus (65 characters in Tiny Shakespeare) and maps each to an integer using lookup tables (`stoi` and `itos`).

```
Text: "hello"  ---> Encoding ---> [8, 5, 12, 12, 15]
```

To enable autoregressive learning, the dataset is split into training sequences of fixed context length `block_size` ($T$). For an input array $X = [t_1, t_2, \dots, t_T]$, the model is tasked with concurrently predicting targets $Y = [t_2, t_3, \dots, t_{T+1}]$. This structure trains the network across $T$ different contextual lengths in parallel.

```
Context & Target breakdown for block_size=4:
When context is [t1], target is t2
When context is [t1, t2], target is t3
When context is [t1, t2, t3], target is t4
When context is [t1, t2, t3, t4], target is t5
```

---

### The Bigram Baseline Model
The initial baseline relies on a standard Bigram model implemented using `nn.Embedding(vocab_size, vocab_size)`. This model evaluates context using only the current token index $t_i$, projecting it directly to logits representing probabilities for the next token $t_{i+1}$.

$$\text{Logits} = \text{Embedding}(t_i)$$

Because it lacks broader context, the Bigram model produces low cross-entropy optimization performance, generating incoherent text streams.

---

### Iterative Development of Self-Attention

Self-attention allows tokens to look back across historic tokens and dynamically gather context.

#### Self-Attention v1: Spatial Averages via Loops
A basic context implementation averages preceding token vectors. Using nested `for`-loops, each token index $t$ averages all vectors from index $0$ to $t$:

$$x_{avg}[b, t] = \frac{1}{t+1} \sum_{i=0}^{t} x[b, i]$$

While conceptually straightforward, explicit Python loops are computationally inefficient for GPUs.

---

#### Self-Attention v2: Lower-Triangular Matrix Multiplication
Loop-based averaging can be vectorized using matrix multiplication. Constructing a lower-triangular matrix $W$ of size $(T, T)$ populated with ones, normalizing across rows yields row sums equal to $1.0$:

$$W = \begin{bmatrix} 1 & 0 & 0 \\ 0.5 & 0.5 & 0 \\ 0.33 & 0.33 & 0.33 \end{bmatrix}$$

Multiplying $W \cdot X$ yields spatial sequence averages across all time steps in a single vectorized matrix operation.

---

#### Self-Attention v3: Data-Dependent Weights via Softmax
Uniform spatial weights limit sequence modeling because token interactions vary by context. Self-attention solves this by making weights dynamic and data-dependent:

1. **Queries ($Q$)**: What the current token is looking for.
2. **Keys ($K$)**: What information the historical token contains.

Taking the inner product $Q \cdot K^T$ generates raw interaction scores. To prevent context leaks from future tokens, values above the diagonal are set to $-\infty$. Applying `softmax` normalizes the interactions:

$$W = \text{softmax}\left(\text{Tril}(Q \cdot K^T)\right)$$

---

#### Self-Attention v4: The Complete Scaled Dot-Product Mechanism
The complete self-attention block incorporates Values ($V$). Tokens project their state into three vectors using linear weights ($W_Q, W_K, W_V$):

$$Q = X W_Q, \quad K = X W_K, \quad V = X W_V$$

Attention weights are computed by scaling dot products by $\frac{1}{\sqrt{d_k}}$ to maintain unit variance across initial inputs, preserving gradient stability:

$$\text{Attention}(Q, K, V) = \text{softmax}\left( \frac{Q K^T}{\sqrt{d_k}} + M \right) V$$

Where $M$ represents the causal mask:

$$M_{ij} = \begin{cases} 0 & i \ge j \\ -\infty & i < j \end{cases}$$

```
                +-------------------+
                | Input Embeddings  |
                +---------+---------+
                          |
             +------------+------------+
             |            |            |
             v            v            v
          +----+       +----+       +----+
          | Q  |       | K  |       | V  |
          +-+--+       +-+--+       +-+--+
            |            |            |
            +-----+------+            |
                  |                   |
                  v                   |
             ( Q x K^T )              |
                  |                   |
                  v                   |
             Scale ( / sqrt(d_k) )     |
                  |                   |
                  v                   |
             Apply Mask (Tril)        |
                  |                   |
                  v                   |
               Softmax                |
                  |                   |
                  +---------+---------+
                            |
                            v
                   ( Attention x V )
                            |
                            v
                         Output
```

---

### Modern Transformer Extensions

#### Positional Encodings
Self-attention computes operations over token sets without built-in spatial sequencing awareness. To encode order, a spatial embedding table `nn.Embedding(block_size, n_embd)` is added directly to token representations before processing:

$$X_{input} = X_{token\_embeddings} + X_{positional\_embeddings}$$

---

#### Multi-Head Attention
Multi-head attention runs multiple attention projections in parallel, allowing the network to capture different types of contextual relationships simultaneously:

$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \dots, \text{head}_h)W^O$$

Where each head operates over sub-dimensional spaces $d_{head} = \frac{d_{model}}{h}$.

---

#### Feed-Forward Networks (FFN)
Following context aggregation via multi-head attention, each token passes through a per-token Feed-Forward Network. This applies non-linear transformations independently across time steps:

$$\text{FFN}(x) = \text{ReLU}(x W_1 + b_1) W_2 + b_2$$

---

#### Residual Connections and Layer Normalization
To prevent vanishing gradients across deep layer stacks, residual skip connections pass inputs directly around sub-blocks:

$$x_{l+1} = x_l + \text{SubLayer}(\text{LayerNorm}(x_l))$$

Pre-Layer Normalization stabilizes optimization by normalizing feature distributions across channels prior to attention and feed-forward operations.

---

## 6. Beginner Explanation (ELI5)

Imagine writing a story one word at a time:

1. **Bigram Model (The Forgotten Memory)**: Imagine predicting the next word by looking *only* at the word you just wrote. If the current word is "The", you guess "cat". If it's "cat", you guess "sat". You ignore everything written earlier. The story quickly becomes meaningless.
2. **Self-Attention (The Classroom Chat)**: Imagine a classroom of students representing words in a sentence. Each student holds a notebook:
   * **Query**: A student asks a question ("I am a verb, where is my subject?").
   * **Key**: Other students hold up tags matching their roles ("I am a noun at position 2!").
   * **Value**: Students pass relevant notes to the questioner based on how well their tags match.
3. **Causal Masking (No Peeking Ahead)**: When writing line 3 of a story, you can read lines 1 and 2, but you cannot look at line 4 because it hasn't been written yet. Causal masking blocks future words from leaking into current predictions.
4. **Multi-Head Attention (Multiple Perspectives)**: One student checks grammar, another tracks story characters, and a third watches rhyming structures. Combining these perspectives helps the model capture complex context.
5. **Skip Connections (Elevators for Information)**: In tall models, passing information through every floor can distort the original signal. Skip connections act like elevators, carrying the core message directly to higher floors while adding small updates along the way.

---

## 7. Technical Deep Dive

### Mathematical Formulation

#### Scaled Dot-Product Attention
Given input representations $X \in \mathbb{R}^{B \times T \times C}$:

Linear projections compute Query, Key, and Value representations:

$$Q = X W_Q \in \mathbb{R}^{B \times T \times d_k}$$

$$K = X W_K \in \mathbb{R}^{B \times T \times d_k}$$

$$V = X W_V \in \mathbb{R}^{B \times T \times d_v}$$

Raw inner products evaluate sequence interactions:

$$A = Q K^T \in \mathbb{R}^{B \times T \times T}$$

Scaling by $\frac{1}{\sqrt{d_k}}$ preserves unit variance for normal distributions, preventing numerical saturation in the `softmax` function:

$$A_{scaled} = \frac{A}{\sqrt{d_k}}$$

Causal masking enforces autoregressive temporal structure:

$$M_{i,j} = \begin{cases} 0 & i \ge j \\ -\infty & i < j \end{cases}$$

$$S = \text{softmax}(A_{scaled} + M) \in \mathbb{R}^{B \times T \times T}$$

$$\text{Output} = S V \in \mathbb{R}^{B \times T \times d_v}$$

---

### Layer Normalization Mechanics
Unlike Batch Normalization, which computes stats across batch instances, Layer Normalization computes stats across feature dimensions for each token independently:

$$\mu = \frac{1}{C} \sum_{i=1}^{C} x_i, \quad \sigma^2 = \frac{1}{C} \sum_{i=1}^{C} (x_i - \mu)^2$$

$$\hat{x}_i = \frac{x_i - \mu}{\sqrt{\sigma^2 + \epsilon}}$$

$$y_i = \gamma_i \hat{x}_i + \beta_i$$

Where $\gamma$ and $\beta$ are learnable affine parameters.

---

### Full Transformer Block (Pre-LN Variant)

```
       x
       |-------------------+
       v                   |
  LayerNorm                |
       |                   |
 Multi-Head Attention      |
       |                   |
       v                   |
     ( + ) <---------------+
       |
       |-------------------+
       v                   |
  LayerNorm                |
       |                   |
  Feed-Forward Net         |
       |                   |
       v                   |
     ( + ) <---------------+
       |
       v
    Output
```

---

## 8. Important Definitions

* **Autoregressive Model**: A model that predicts future sequence elements using past predictions as context.
* **Block Size ($T$)**: The maximum context window size (sequence length) the model can process.
* **Byte-Pair Encoding (BPE)**: A subword tokenization algorithm that iteratively merges frequent character pairs into subword tokens.
* **Causal Masking**: A masking technique that sets future sequence scores to $-\infty$ so models cannot attend to subsequent tokens.
* **Embedding**: A mapping of categorical token IDs into continuous, dense vector spaces.
* **Key ($K$)**: A projected representation encoding the attributes a token offers to other tokens.
* **Layer Normalization**: A normalization technique applied across feature channels per token instance to stabilize network activations.
* **Logits**: Unnormalized raw output scores produced by the final linear projection layer before applying `softmax`.
* **Multi-Head Attention**: An attention design that splits feature dimensions across multiple heads operating in parallel.
* **Positional Encoding**: Spatial vectors added to token embeddings to supply sequence order information.
* **Query ($Q$)**: A projected representation encoding what information a token is looking for.
* **Residual Connection**: A structural skip path that adds a sub-layer's input directly to its output ($x + f(x)$).
* **Self-Attention**: An attention mechanism where token interaction scores are calculated using tokens from the same sequence.
* **Softmax**: A function that converts raw scalar values into a normalized probability distribution summing to $1.0$.
* **Value ($V$)**: A projected representation encoding the contextual content passed along during attention aggregation.

---

## 9. Code Snippets & Configuration Examples

### Single-Head Scaled Dot-Product Causal Self-Attention

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class Head(nn.Module):
    """ One head of causal self-attention """
    def __init__(self, n_embd, head_size, block_size, dropout=0.2):
        super().__init__()
        self.key = nn.Linear(n_embd, head_size, bias=False)
        self.query = nn.Linear(n_embd, head_size, bias=False)
        self.value = nn.Linear(n_embd, head_size, bias=False)
        self.register_buffer('tril', torch.tril(torch.ones(block_size, block_size)))
        self.dropout = nn.Dropout(dropout)

    def forward(self, x):
        B, T, C = x.shape
        k = self.key(x)   # (B, T, head_size)
        q = self.query(x) # (B, T, head_size)
        
        # Compute attention scores ("affinities"), scale by sqrt(d_k)
        wei = q @ k.transpose(-2, -1) * (C ** -0.5) # (B, T, head_size) @ (B, head_size, T) -> (B, T, T)
        wei = wei.masked_fill(self.tril[:T, :T] == 0, float('-inf')) # (B, T, T)
        wei = F.softmax(wei, dim=-1) # (B, T, T)
        wei = self.dropout(wei)
        
        # Perform weighted aggregation of values
        v = self.value(x) # (B, T, head_size)
        out = wei @ v # (B, T, T) @ (B, T, head_size) -> (B, T, head_size)
        return out
```

---

### Multi-Head Attention & Feed-Forward Network

```python
class MultiHeadAttention(nn.Module):
    """ Multiple heads of self-attention running in parallel """
    def __init__(self, num_heads, head_size, n_embd, block_size, dropout=0.2):
        super().__init__()
        self.heads = nn.ModuleList([Head(n_embd, head_size, block_size, dropout) for _ in range(num_heads)])
        self.proj = nn.Linear(num_heads * head_size, n_embd)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x):
        out = torch.cat([h(x) for h in self.heads], dim=-1)
        out = self.dropout(self.proj(out))
        return out

class FeedForward(nn.Module):
    """ A simple linear layer followed by a non-linearity """
    def __init__(self, n_embd, dropout=0.2):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(n_embd, 4 * n_embd),
            nn.ReLU(),
            nn.Linear(4 * n_embd, n_embd),
            nn.Dropout(dropout),
        )

    def forward(self, x):
        return self.net(x)
```

---

### Transformer Block with Residual Skip Connections & Pre-LN

```python
class Block(nn.Module):
    """ Transformer block: communication followed by computation """
    def __init__(self, n_embd, n_head, block_size):
        super().__init__()
        head_size = n_embd // n_head
        self.sa = MultiHeadAttention(n_head, head_size, n_embd, block_size)
        self.ffwd = FeedForward(n_embd)
        self.ln1 = nn.LayerNorm(n_embd)
        self.ln2 = nn.LayerNorm(n_embd)

    def forward(self, x):
        # Pre-Layer Normalization architecture with residual connections
        x = x + self.sa(self.ln1(x))
        x = x + self.ffwd(self.ln2(x))
        return x
```

---

### Complete nanoGPT Language Model Blueprint

```python
class NanoGPT(nn.Module):
    def __init__(self, vocab_size, n_embd, block_size, n_layer, n_head):
        super().__init__()
        self.block_size = block_size
        self.token_embedding_table = nn.Embedding(vocab_size, n_embd)
        self.position_embedding_table = nn.Embedding(block_size, n_embd)
        self.blocks = nn.Sequential(*[Block(n_embd, n_head, block_size) for _ in range(n_layer)])
        self.ln_f = nn.LayerNorm(n_embd) # Final layer norm
        self.lm_head = nn.Linear(n_embd, vocab_size)

    def forward(self, idx, targets=None):
        B, T = idx.shape
        tok_emb = self.token_embedding_table(idx) # (B, T, n_embd)
        pos_emb = self.position_embedding_table(torch.arange(T, device=idx.device)) # (T, n_embd)
        x = tok_emb + pos_emb # (B, T, n_embd)
        x = self.blocks(x)   # (B, T, n_embd)
        x = self.ln_f(x)     # (B, T, n_embd)
        logits = self.lm_head(x) # (B, T, vocab_size)

        if targets is None:
            loss = None
        else:
            B, T, C = logits.shape
            logits = logits.view(B * T, C)
            targets = targets.view(B * T)
            loss = F.cross_entropy(logits, targets)

        return logits, loss

    def generate(self, idx, max_new_tokens):
        for _ in range(max_new_tokens):
            # Crop current context window to block_size
            idx_cond = idx[:, -self.block_size:]
            logits, _ = self(idx_cond)
            # Focus only on the last time step prediction
            logits = logits[:, -1, :] # (B, C)
            probs = F.softmax(logits, dim=-1) # (B, C)
            idx_next = torch.multinomial(probs, num_samples=1) # (B, 1)
            idx = torch.cat((idx, idx_next), dim=1) # (B, T+1)
        return idx
```

---

## 10. Best Practices

1. **Pre-Layer Normalization Layout**: Modern GPT architectures apply Layer Normalization *before* multi-head attention and feed-forward operations (`Pre-LN`), rather than after (`Post-LN`), which improves training stability in deep networks.
2. **Dimension Scaling for Feed-Forward Networks**: Expand internal feed-forward hidden dimensions by a factor of 4 ($4 \times d_{model}$) to increase capacity for non-linear feature processing.
3. **Softmax Variance Scaling**: Always scale inner products by $\frac{1}{\sqrt{d_k}}$ prior to `softmax` evaluation to keep initial variance near $1.0$ and prevent vanishing gradients.
4. **Registering Causal Masks as Buffers**: Store static causal triangular masks using `register_buffer` in PyTorch module definitions so they are properly assigned to hardware devices without being tracked as learnable parameters.
5. **Context Bounds Checks During Inference**: Always truncate input sequence tensors to `block_size` (`idx[:, -block_size:]`) during iterative autoregressive generation to prevent position index out-of-bounds errors.

---

## 11. Common Mistakes

* **Causal Mask Leakage**: Omitting the lower-triangular causal mask during self-attention allows past tokens to attend to future tokens, corrupting training for autoregressive generation tasks.
* **Unscaled Dot Products**: Omitting the $\frac{1}{\sqrt{d_k}}$ scaling factor can push large dot products into saturated regions of the `softmax` function, causing zero-valued gradients during backpropagation.
* **Applying Softmax Across Incorrect Dimensions**: Applying `softmax` along sequence or batch dimensions rather than feature channels (`dim=-1`) destroys probability distribution calculations.
* **Mixing Up Pre-LN and Post-LN Layers**: Applying `Post-LN` transformations in deep networks without learning rate warmups often leads to unstable training or divergence.
* **Reshaping Tensors Incorrectly Before Loss Calculations**: PyTorch's `cross_entropy` function expects input logits of shape `(Batch * Time, Vocab_Size)` and target vectors of shape `(Batch * Time)`. Passing un-flattened 3D tensors directly causes execution errors.

---

## 12. Interview Questions

### Q1: Why is scaled dot-product attention scaled by $\frac{1}{\sqrt{d_k}}$?
**Answer:** Assuming vector components of $Q$ and $K$ are independent random variables with zero mean and unit variance ($1.0$), their dot product has a mean of $0$ and a variance equal to $d_k$ (the projection dimension size). For large $d_k$ values, unscaled dot products grow significantly in magnitude, pushing inputs to the `softmax` function into extreme saturated regions where output gradients approach zero. Dividing dot products by $\frac{1}{\sqrt{d_k}}$ rescales the distribution variance back to $1.0$, preserving healthy gradient flow during backpropagation.

---

### Q2: What is the difference between Self-Attention and Cross-Attention?
**Answer:** In **Self-Attention**, the Query ($Q$), Key ($K$), and Value ($V$) projections are all derived from the same input sequence, allowing tokens within that sequence to interact with one another. In **Cross-Attention** (commonly used in encoder-decoder models like T5 or translation models), Queries ($Q$) are projected from the decoder sequence, while Keys ($K$) and Values ($V$) are generated from the encoder output. This enables output tokens to attend to information from an external source context.

---

### Q3: Why are positional encodings required in Transformer models?
**Answer:** The core operations of self-attention—matrix multiplications and weighted sums—are permutation-invariant. If you shuffle the input tokens in a sequence, the calculated attention weights and output values will remain identical (just permuted accordingly). Because language order is critical to meaning, explicit spatial information must be added to token representations using positional encodings (either static sinusoidal functions or learned position lookup tables).

---

### Q4: Why do Transformers place a Feed-Forward Network (FFN) after the Self-Attention block?
**Answer:** Self-attention is primarily a contextual communication mechanism that aggregates information across tokens. However, it relies entirely on linear projections and weighted averages. The Feed-Forward Network (FFN) applies per-token non-linear transformations (such as `ReLU` or `GELU` activations across expanded hidden dimensions), allowing the network to process and refine the aggregated representations independently.

---

## 13. Certification Questions

### Question 1
In an autoregressive decoder model using a context context window of size $T=8$, what values must be placed in the upper-triangular region (above the main diagonal) of the attention affinity matrix prior to passing it through the `softmax` activation function?
* A) $0.0$
* B) $1.0$
* C) $-\infty$
* D) $-1.0$

**Correct Answer:** **C**  
**Explanation:** Setting future affinity values to $-\infty$ ensures that applying `e^x` in the `softmax` calculation produces $e^{-\infty} = 0$, completely blocking future token information from leaking into past context predictions.

---

### Question 2
Given a Transformer hidden dimension size $n\_embd = 384$ and $n\_head = 6$ attention heads, what is the feature dimension size $head\_size$ ($d_k$) for each individual attention head?
* A) 384
* B) 64
* C) 128
* D) 2304

**Correct Answer:** **B**  
**Explanation:** Multi-head attention splits the global embedding dimension $n\_embd$ equally across the parallel attention heads:

$$\text{head\_size} = \frac{n\_embd}{n\_head} = \frac{384}{6} = 64$$

---

### Question 3
How do Pre-Layer Normalization (`Pre-LN`) Transformer blocks differ from Post-Layer Normalization (`Post-LN`) architectures?
* A) `Pre-LN` applies normalization after feed-forward calculations, whereas `Post-LN` applies it before attention layers.
* B) `Pre-LN` applies normalizations directly to the inputs of attention and feed-forward sub-layers within residual paths, improving gradient flow during deep network training.
* C) `Pre-LN` eliminates the need for residual skip connections.
* D) `Pre-LN` normalizes features across the batch dimension instead of feature channel dimensions.

**Correct Answer:** **B**  
**Explanation:** The `Pre-LN` layout normalizes activations before passing them into multi-head attention and feed-forward sub-layers ($x + f(\text{LN}(x))$). This keeps gradient pathways clearer along the main residual stream, enabling stable training of deeper Transformer models without complex learning rate warmups.

---

## 14. Real-World Examples

1. **GitHub Copilot**: Uses decoder-only Transformer architectures trained on code repositories to generate context-aware code completions autoregressively.
2. **ChatGPT (OpenAI)**: Extends pre-trained GPT architectures by applying Supervised Fine-Tuning (SFT) and Reinforcement Learning from Human Feedback (RLHF) to align raw text generation with conversational instructions.
3. **Enterprise Document Summarization**: Uses custom subword-tokenized Transformer models to read long context blocks and generate structured executive summaries.

---

## 15. Analogies

### 1. The Database Search (Queries, Keys, Values)
Think of self-attention as a search query over a database:
* **Query ($Q$)**: The search bar string you type in (e.g., "recipe for chocolate cake").
* **Key ($K$)**: The titles and tags assigned to each database record (e.g., "vanilla frosting", "chocolate dessert", "baked goods").
* **Value ($V$)**: The actual text content stored inside each record.

The system calculates relevance scores by comparing your Query with every Key, turns those scores into normalized percentages, and uses them to retrieve a weighted combination of the database Values.

```
       Query ("chocolate cake recipe")
                     |
                     v
   +------------------------------------+
   | Compare Query vs. Keys (Dot Product)|
   +-----------------+------------------+
                     |
       +-------------+-------------+
       |                           |
Key: "vanilla icing"        Key: "chocolate dessert"
Score: 0.1                  Score: 0.9
       |                           |
       v                           v
Value: [Icing recipe]       Value: [Cake recipe]
       |                           |
       +-------------+-------------+
                     |
                     v
  Weighted Result: (0.1 x Icing) + (0.9 x Cake)
```

---

### 2. The Highway System (Residual Connections)
Deep neural networks act like a city with many sequential street intersections. Without highway bypasses, traffic (gradients) slows down and gets blocked at every local traffic light. **Residual skip connections** act like elevated express highways running parallel to local streets. They allow the main traffic signal to flow through unhindered, while local exits process specific details before merging back onto the main highway.

---

## 16. Frequently Asked Questions

### What is the maximum context length for nanoGPT, and how is it adjusted?
The maximum context length is defined by the hyperparameter `block_size` (set to 8 or 64 in the video tutorial). To increase context length in production models (e.g., to 2048 or 4096 tokens), you increase `block_size` and scale up positional embeddings accordingly. Note that computational and memory costs scale quadratically ($O(T^2)$) with context length.

---

### What is the difference between character-level and subword tokenizers?
Character-level tokenizers map individual text characters directly to integers. This keeps the vocabulary size small (e.g., 65 characters) but results in long sequence lengths for a given text. Subword tokenizers (like Byte-Pair Encoding in `tiktoken`) use larger vocabularies (e.g., 50,000 tokens) to group common letter clusters and words into single tokens. This reduces overall sequence lengths by roughly $3\times$ to $4\times$, though it requires larger embedding lookup matrices.

---

### Why does training loss decrease while validation loss eventually plateaus or increases?
This behavior indicates overfitting. The network begins memorizing specific patterns in the training data rather than learning generalizable language structure. Overfitting can be mitigated by adding regularization (such as `Dropout`), increasing dataset size, or reducing model capacity.

---

### Why use PyTorch's `tril` lower-triangular operator in self-attention?
The `tril` operator constructs a matrix with ones on and below the main diagonal and zeros above it. When masking, zeros are replaced with $-\infty$, ensuring that subsequent `softmax` operations assign zero probability weight to future positions. This enforces causal masking during autoregressive training.

---

## 17. Related Technologies

* **PyTorch**: The open-source deep learning library used to construct and train nanoGPT models.
* **tiktoken**: OpenAI's fast Byte-Pair Encoding (BPE) subword tokenizer library used in GPT-2, GPT-3, and GPT-4 pipelines.
* **HuggingFace Transformers**: An open-source model repository and execution framework providing pre-trained Transformer architectures.
* **FlashAttention**: An optimized, GPU-aware implementation of scaled dot-product attention that reduces memory read/write overhead from $O(T^2)$ to linear memory complexity.
* **WandB (Weights & Biases)**: A metric tracking and visualization toolkit commonly used to monitor loss curves and hyperparameters during model training.

---

## 18. Important Quotes

> *"GPTs are autoregressive language models: they are trained to predict the next token in a sequence given all preceding context tokens."*

> *"Self-attention is a mechanism for performing massive weighted averages across continuous sequence vectors through dynamic matrix multiplication."*

> *"The Transformer architecture treats nodes in a directed graph as communication channels that exchange message representations via Query, Key, and Value projections."*

> *"Scaling dot products by $\frac{1}{\sqrt{d_k}}$ preserves activation variance near $1.0$, preventing saturation in the softmax activation function during training initialization."*

---

## 19. Glossary

| Term | Definition |
| :--- | :--- |
| **Autoregressive** | Generative process where predictions are produced sequentially, using previous outputs as context for future steps. |
| **BPE** | Byte-Pair Encoding; a subword tokenization strategy that merges frequent byte or character sequences into unified tokens. |
| **Causal Mask** | A matrix operation that sets future context locations to $-\infty$ to enforce past-only sequence visibility. |
| **Cross-Entropy** | A loss function measuring differences between predicted token probability distributions and target token representations. |
| **Dropout** | A regularization technique that randomly sets a fraction of input activations to zero during training steps to reduce overfitting. |
| **Feed-Forward Net** | A per-token neural network layer applying non-linear transformations across feature channels. |
| **Key ($K$)** | A linear projection representing the features a token exposes to incoming queries during self-attention. |
| **Layer Normalization** | A technique that normalizes activation distributions across channel features independently for each token. |
| **Logits** | The unnormalized linear outputs generated by a neural network before conversion to probabilities. |
| **Multi-Head Attention** | Running multiple parallel self-attention mechanisms over partitioned feature dimensions. |
| **Positional Encoding** | Spatial vectors added to input token embeddings to provide explicit token order information. |
| **Query ($Q$)** | A linear projection representing the context features a token is searching for during self-attention. |
| **Residual Connection** | Skip pathways that add sub-layer inputs directly to their outputs ($x + f(x)$) to maintain clear gradient highways. |
| **Self-Attention** | An attention mechanism that computes contextual dependencies across tokens within the same sequence. |
| **Softmax** | An activation function that converts raw scores into normalized probability distributions summing to $1.0$. |
| **Value ($V$)** | A linear projection representing the contextual content aggregated during self-attention. |

---

## 20. One-Page Cheat Sheet

### Key Architectural Equations

$$\text{Input Representation: } X_{in} = \text{Embedding}_{tok}(X) + \text{Embedding}_{pos}(\text{Positions})$$

$$\text{Linear Projections: } Q = X W_Q, \quad K = X W_K, \quad V = X W_V$$

$$\text{Scaled Attention Weight: } S = \text{softmax}\left( \frac{Q K^T}{\sqrt{d_k}} + M \right)$$

$$\text{Attention Output: } \text{Head}(X) = S \cdot V$$

$$\text{Residual Sub-Layer Output: } Y = X + \text{MultiHead}(\text{LayerNorm}(X))$$

$$\text{Feed-Forward Network: } \text{FFN}(Y) = \text{ReLU}(Y W_1 + b_1) W_2 + b_2$$

---

### Core Tensor Shapes Matrix

| Step / Matrix | Tensor Shape notation | Description |
| :--- | :--- | :--- |
| **Input Batch ($X$)** | `(B, T)` | Integer sequence tensor indices. |
| **Token Embeddings** | `(B, T, C)` | Continuous representations across embedding dimension $C$. |
| **Queries / Keys / Values** | `(B, T, head_size)` | Projection matrices per individual head ($d_k = \frac{C}{n\_head}$). |
| **Attention Scores ($QK^T$)** | `(B, T, T)` | Pairwise token affinity matrix across sequence time steps. |
| **Multi-Head Output** | `(B, T, C)` | Concatenated projections across all parallel attention heads. |
| **Final Logits** | `(B, T, vocab_size)` | Unnormalized next-token score distributions. |

---

## 21. Flash Cards

- **Card 1 | Architecture**
  - **Q:** What are the three projection matrices required to compute scaled dot-product self-attention?
  - **A:** Query ($Q$), Key ($K$), and Value ($V$) matrices.

- **Card 2 | Mathematical Stabilization**
  - **Q:** Why do we scale dot products by $\frac{1}{\sqrt{d_k}}$ prior to calculating `softmax`?
  - **A:** To maintain unit variance across initial scores, preventing `softmax` saturation and vanishing gradients during training setup.

- **Card 3 | Sequence Control**
  - **Q:** How does causal masking prevent future token information leakage?
  - **A:** By populating upper-triangular affinity matrix values with $-\infty$, which evaluate to zero probability weight after `softmax`.

- **Card 4 | Network Optimization**
  - **Q:** How do residual skip connections help train deep Transformer architectures?
  - **A:** They provide direct gradient pathways during backpropagation, bypassing complex non-linear sub-layers.

- **Card 5 | Positional Information**
  - **Q:** Why is positional encoding necessary in self-attention models?
  - **A:** Self-attention operations are permutation-invariant, so positional encodings must be added to provide token order information.

---

## 22. Quiz

### Q1: What task is an autoregressive generative language model optimized to perform?
- A) Classify text sequences into categories
- B) Predict the next token given preceding context tokens
- C) Reconstruct masked internal tokens in a bidirectional sequence
- D) Translate entire text inputs simultaneously without iterative steps
**Correct Answer:** **B**  
**Explanation:** Autoregressive language models learn to generate text by continuously predicting the next token in a sequence based on preceding historical context.

---

### Q2: Which matrix operation provides context-aware aggregation across past sequence representations?
- A) Multiplying Queries directly with Values
- B) Multiplying normalized `softmax` attention scores by Values ($S \cdot V$)
- C) Multiplying token embeddings by positional embeddings
- D) Passing token embeddings through non-linear activation layers
**Correct Answer:** **B**  
**Explanation:** Multiplying normalized `softmax` attention weight distributions ($S$) by Value vectors ($V$) yields a contextual sum across sequence positions.

---

### Q3: How do character-level tokenizers compare to subword Byte-Pair Encodings (BPE)?
- A) Character-level tokenizers require larger vocabulary mapping tables
- B) Character-level tokenizers shorten overall context sequence lengths
- C) Character-level tokenizers feature smaller vocabulary sizes but require longer context sequence steps for equivalent text
- D) Subword tokenizers eliminate the need for positional encodings
**Correct Answer:** **C**  
**Explanation:** Character-level tokenizers restrict vocabulary maps to unique single characters, resulting in longer token sequences to represent the same text compared to subword BPE tokenizers.

---

### Q4: What is the primary purpose of applying Layer Normalization before sub-layer operations in Pre-LN Transformer blocks?
- A) To convert outputs into discrete integer indices
- B) To stabilize activation feature distributions before processing, enabling smoother gradient propagation in deep models
- C) To mask future sequence positions
- D) To reduce model parameter counts
**Correct Answer:** **B**  
**Explanation:** Pre-Layer Normalization stabilizes internal activation distributions across feature channels prior to attention and feed-forward sub-layers, making deep Transformer networks easier to optimize.

---

### Q5: What is the computational complexity of standard self-attention relative to sequence length ($T$)?
- A) $O(T)$
- B) $O(T \log T)$
- C) $O(T^2)$
- D) $O(T^3)$
**Correct Answer:** **C**  
**Explanation:** Self-attention calculates pairwise interaction dot products across all time steps in a sequence, resulting in $O(T^2)$ computational and memory complexity relative to sequence length $T$.

---

### Q6: In the nanoGPT implementation, what does the hyperparameter `block_size` represent?
- A) The total number of layers in the network
- B) The maximum context window size (sequence length) the model can process
- C) The batch size used during parameter updates
- D) The feature embedding dimension size
**Correct Answer:** **B**  
**Explanation:** `block_size` sets the maximum context window size (number of time steps) the model can use for autoregressive training and prediction.

---

### Q7: What function is used to convert raw output logits into normalized probability distributions during sampling?
- A) `ReLU`
- B) `Sigmoid`
- C) `Softmax`
- D) `LayerNorm`
**Correct Answer:** **C**  
**Explanation:** Applying `softmax` across final linear logits converts raw output scores into non-negative values that sum to $1.0$, producing a valid probability distribution for sampling.

---

### Q8: What components form a standard Transformer Feed-Forward Network (FFN)?
- A) A single linear projection layer followed by a causal mask
- B) Two linear projection layers separated by a non-linear activation function (e.g., `ReLU` or `GELU`)
- C) Parallel multi-head attention blocks
- D) An embedding lookup table combined with Layer Normalization
**Correct Answer:** **B**  
**Explanation:** Transformer Feed-Forward Networks consist of two linear projection layers with an intermediate non-linear activation function, expanding and compressing feature dimensions to process context representations per token.

---

### Q9: What problem can occur if dot-product values in self-attention are left unscaled prior to `softmax` evaluation?
- A) The network becomes non-deterministic during inference
- B) Large score magnitudes can saturate `softmax` outputs, causing near-zero gradients during backpropagation
- C) Sequence token positions become permanently inverted
- D) Causal masking operations fail to enforce upper-triangular masking
**Correct Answer:** **B**  
**Explanation:** Large score magnitudes push `softmax` outputs into saturated regions (approaching $0$ or $1$), producing extremely small gradients that hinder learning during backpropagation.

---

### Q10: How are positional embeddings integrated into input representations in nanoGPT?
- A) They are concatenated to token embedding vectors, doubling feature length
- B) They are element-wise added directly to token embedding vectors
- C) They are multiplied by Query matrices during attention calculations
- D) They are passed through a separate Feed-Forward layer before processing
**Correct Answer:** **B**  
**Explanation:** In standard GPT architectures, positional embeddings are added element-wise directly to token embedding vectors ($X_{tok} + X_{pos}$) before entering the Transformer blocks.

---

## 23. Action Items

- [ ] **Set Up Environment**: Create a Python environment containing `torch`, `jupyter`, `numpy`, and `tiktoken`.
- [ ] **Download Corpus**: Fetch the 1.1MB Tiny Shakespeare dataset (`input.txt`).
- [ ] **Build Data Ingestion**: Write script functions to process character mappings (`stoi`, `itos`) and structure `train`/`val` dataloaders.
- [ ] **Implement Bigram Baseline**: Construct a basic PyTorch model using `nn.Embedding` to verify cross-entropy loss tracking.
- [ ] **Implement Causal Attention**: Construct single-head and multi-head scaled dot-product attention modules featuring lower-triangular causal masking.
- [ ] **Assemble nanoGPT**: Combine positional encodings, multi-head attention, feed-forward sub-layers, pre-layer normalization, and residual skip connections into a unified network architecture.
- [ ] **Train Model**: Run an optimization loop using `AdamW`, track loss convergence across training iterations, and sample generated text output autoregressively.

---

## 24. Recommended Further Reading

1. **Attention Is All You Need** (Vaswani et al., 2017): The foundational paper introducing the Transformer architecture.
2. **Language Models are Unsupervised Multitask Learners** (Radford et al., 2019): OpenAI's GPT-2 research paper detailing scaled decoder-only Transformer designs.
3. **Language Models are Few-Shot Learners** (Brown et al., 2020): The GPT-3 paper demonstrating strong few-shot performance from scaled autoregressive models.
4. **karpathy/nanoGPT Repository**: Andrej Karpathy's minimal, readable repository for training GPT models in PyTorch.
5. **Neural Networks: Zero to Hero**: Andrej Karpathy's video lecture series covering neural networks from fundamental backpropagation to complete GPT implementations.