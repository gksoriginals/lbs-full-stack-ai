# Large Language Models (LLMs)

**Poster topic:** How LLMs work, capabilities, and limitations  
**Audience:** Engineering-college faculty  
**Suggested duration:** 55–65 minutes

## Narrative route

~~~text
Language models assign probabilities to sequences
        ↓
LLMs scale that idea with data, parameters, and compute
        ↓
A prompt is converted into tokens, embeddings, and positions
        ↓
The original Transformer encodes a source and decodes a target
        ↓
Attention explains how tokens exchange relevant information
        ↓
Encoder-only, decoder-only, and encoder–decoder models use this differently
        ↓
Training and decoding produce capabilities—and predictable limitations
~~~

---

## Slide 1 — What is a language model?

### On slide

> A **language model** estimates how likely a sequence is, or how likely the next language unit is given its context.

~~~text
P(next token | permitted context)
~~~

It can rank continuations, predict a missing unit, or generate a sequence one unit at a time.

### The term predates Generative AI

~~~text
Statistical language models
  n-grams: limited recent word history
        ↓
Neural language models
  learned representations replace fixed counts
        ↓
Modern LLMs
  scaled neural language models, usually Transformer-based
~~~

Language models are not defined by chat. They have supported speech recognition, spelling correction, autocomplete, translation, and text generation.

### Speaker notes

Statistical language modelling treated natural language as a probability distribution over sequences. Modern LLMs retain that basic prediction problem, but use learned representations and far broader context than classical n-gram systems.

Chat is only one interface over a language model. The next question is what makes some neural language models large and broadly adaptable rather than narrow models built for one task.

---

## Slide 2 — What makes a language model “large”?

### On slide

> **Large language model (LLM)** is an engineering term, not a standardised size category.

~~~mermaid
flowchart LR
  P["Parameters\nmodel capacity"] --> L["Large language model"]
  D["Data\nbreadth and volume"] --> L
  C["Compute\ntraining at scale"] --> L
  A["Architecture + training method"] --> L
~~~

Scale enables:

- broad reusable language representations;
- prompted task behaviour without changing parameters;
- adaptation to many downstream tasks.

### Speaker notes

There is no universal threshold at which a language model becomes an LLM. Parameter count is visible, but data, compute, architecture, and the self-supervised objective all shape what the model learns and how broadly it can be reused.

GPT-3 made this shift visible in 2020 by showing strong few-shot task performance from a 175-billion-parameter autoregressive model. A foundation model is broader than an LLM: it may operate across language, vision, audio, or several modalities.

---

## Bridge — Learn patterns once; generate step by step

### On slide

```text
1. Train      Repeated examples adjust parameters to reduce prediction error.
        ↓
2. Represent  Parameters capture patterns in syntax, style, code, and meaning.
        ↓
3. Infer      A new prompt enters the learned processing pipeline.
        ↓
4. Generate  Select a likely next token, append it, then repeat.
```

### Speaker notes

During pre-training, the model adjusts parameters over a very large corpus. At runtime, it does not update those parameters from the user’s prompt. It uses the learned parameters and the tokens currently in context to produce a next-token distribution, selects one token, appends it, and repeats.

---

## Slide 3 — Token and tokenizer: text becomes model input

### On slide

### Why does this exist?

> Neural networks operate on numbers, but human language is open-ended text. A tokenizer converts arbitrary text into a finite, reusable vocabulary of numeric units.

> A **token** is a unit from the model vocabulary. It may be a word, subword, punctuation mark, whitespace pattern, or byte sequence.

> A **tokenizer** maps text to token IDs, adds required special tokens, and converts generated IDs back to text.

~~~mermaid
flowchart LR
  TXT["Text\n'LLMs predict tokens'"] --> TOK["Tokenizer"]
  TOK --> IDs["Token IDs\ninteger sequence"]
  IDs --> OUT["Model input"]
  OUT --> DE["Generated IDs"]
  DE --> TXT2["Decoded text"]
~~~

~~~text
"unpredictable"  →  "un" | "predict" | "able"  →  [491] | [10321] | [987]
~~~

The pieces and ID values are illustrative; each model has its own tokenizer and vocabulary. Subword and byte-level units avoid requiring one vocabulary entry for every possible word, identifier, spelling variant, or code fragment.

### Speaker notes

Tokenisation is model-specific. The visible word count is only an approximation of token count: code, URLs, punctuation-heavy material, and some languages can consume tokens differently from ordinary English prose.

This affects context-window usage, API cost, truncation, and output limits. It also explains why a model may appear to stop at an unexpected boundary: generation happens in vocabulary units, not necessarily in whole words or sentences.

---

## Slide 4 — Embeddings: token IDs become vectors

### On slide

### Why does this exist?

> A token ID is only an arbitrary index. The model needs a continuous numeric representation in which learning can express useful similarities and differences.

> An **embedding** is a learned vector representation. The token ID is an index; the embedding is the numeric representation used by the neural network.

~~~mermaid
flowchart LR
  ID["Token ID\n[10321]"] --> LOOKUP["Embedding matrix lookup\nread row 10321"]
  LOOKUP --> V["Initial embedding vector\n[0.12, −0.84, 0.37, …]"]
  V --> TR["Transformer layers"]
~~~

The vector values are illustrative. Real embeddings have hundreds or thousands of dimensions.

Two distinctions:

| Term | Meaning |
| --- | --- |
| **Input embedding** | the initial learned vector looked up for a token |
| **Contextual representation** | the vector after Transformer layers mix in context |

~~~text
"bank" + river context  →  contextual representation A
"bank" + loan context   →  contextual representation B
~~~

### Speaker notes

An embedding matrix is a learned table: each vocabulary ID retrieves one vector. That vector initially knows only the token identity. After attention and feed-forward layers process it, the same token receives a context-dependent representation.

Embedding vectors let gradient-based learning alter relationships between token representations; an integer ID by itself has no useful geometry. This distinction also matters for retrieval systems: an embedding model creates vectors intended for similarity comparison, while a generative decoder produces next-token probabilities.

---

## Slide 5 — Context window: working memory for this generation

### On slide

### Why does this exist?

> The model needs a bounded set of tokens to attend over for each prediction. The context window is that bounded working-memory region.

> The **context window** is the maximum token budget the model can attend to for a current request and response.

~~~text
instructions + prompt + history + retrieved passages + tool results + generated output
──────────────────────────────────────────────────────────────────────────
                         total context window
~~~

Consequences:

- input tokens and output tokens share the available budget;
- material outside the window is unavailable unless reintroduced;
- irrelevant context can dilute relevant context;
- discarded context is not permanent memory.

### Speaker notes

A larger context window increases capacity, not factual reliability. A model can receive a long document and still miss a detail, conflate sections, or follow an untrusted instruction embedded in the document.

The context window answers **what is available** to the model. Attention answers the next question: **which available tokens should influence the current token or prediction?**

Attention has memory and compute costs that grow with sequence length, so a model cannot treat an unbounded history as input. Context should therefore be selected, structured, and evaluated; durable storage and retrieval remain responsibilities of the surrounding application.

---

## Attention-first teaching sequence

Present the next material in this order: **attention problem** → **Slide 8 — QKV inside one head** → **Slide 9 — multi-head attention** → **Architecture follow-up** → **Transformer components**. The audience first frames the relevance question, then follows the QKV computation that produces one contextual update, and only then asks why several learned attention patterns must run in parallel.

## Slide 6 — Architecture follow-up: the original Transformer

### On slide

> The 2017 Transformer was introduced for sequence-to-sequence tasks such as machine translation.

~~~mermaid
flowchart LR
  subgraph EN["Encoder stack × N"]
    EI["Source token embeddings\n+ positional encoding"] --> ESA["Multi-head self-attention"]
    ESA --> EAN1["Add residual + LayerNorm"]
    EAN1 --> EFFN["Position-wise FFN / MLP"]
    EFFN --> EAN2["Add residual + LayerNorm"]
  end

  subgraph DE["Decoder stack × N"]
    DI["Target tokens shifted right\n+ positional encoding"] --> DMSA["Masked multi-head self-attention"]
    DMSA --> DAN1["Add residual + LayerNorm"]
    DAN1 --> DCA["Encoder-decoder cross-attention\nQ from decoder; K,V from encoder"]
    DCA --> DAN2["Add residual + LayerNorm"]
    DAN2 --> DFFN["Position-wise FFN / MLP"]
    DFFN --> DAN3["Add residual + LayerNorm"]
    DAN3 --> OUT["Linear projection + Softmax\nnext-token distribution"]
  end

  EAN2 --> DCA
~~~

### Speaker notes

The encoder reads the full source sequence and produces contextual representations. The decoder generates the target sequence one token at a time: it uses masked self-attention over earlier target tokens and cross-attention over the encoder output. Its final decoder vector is passed through a **vocabulary projection**, producing one raw score (**logit**) for every token in the vocabulary; softmax converts those logits into candidate probabilities for the next token.

After the attention, QKV, and multi-head slides, return here to locate the same mechanism in the full model: encoder self-attention, decoder masked self-attention, and encoder–decoder cross-attention.

The architecture therefore answers a different question: **where is attention used in the Transformer, and what information can each instance see?**

---

## Slide 7 — What each Transformer component does

### On slide

| Component | Role in the architecture |
| --- | --- |
| Token embeddings + positional encoding | represent identity and order |
| Multi-head self-attention | mix relevant information within one sequence |
| Masked self-attention | hides future target tokens from a decoder |
| Cross-attention | lets decoder tokens consult encoder outputs |
| Residual / skip connection | preserves earlier signal and supports deep optimisation |
| Layer normalisation | stabilises activations across features |
| Feed-forward network (FFN / MLP) | applies non-linear transformation at each position |
| Dropout in the original design | regularises during training |
| Linear projection + softmax | converts final hidden state to vocabulary probabilities |

~~~text
one Transformer block:
attend to context  →  add + normalise  →  transform with MLP  →  add + normalise
~~~

### Speaker notes

“Add & Norm” in the original paper means residual addition followed by layer normalisation. The attention sub-layer exchanges information across positions. The FFN then applies the same non-linear transformation independently to every position.

Modern LLMs use variants—pre-normalisation, RMSNorm, rotary position methods, grouped-query attention, and alternate MLP activations—but these are evolutions of the same functional roles.

---

## Bridge — Why Transformers scaled beyond LSTMs

### On slide

- **LSTMs could generate text:** they also predict the next token, but process positions serially through one evolving hidden state.
- **Transformers retain token-level representations:** attention lets the current token relate directly to relevant earlier tokens instead of forcing all information through one compressed state.
- **This made broad pre-training practical at scale:** training can be parallelised efficiently, and long prompts can contain instructions and examples that influence the current response.
- **In-context learning is not weight update:** the model temporarily follows the pattern in prompt examples; its learned parameters remain unchanged during that request.

### Speaker notes

LSTMs are not incapable of generation or using context. An autoregressive LSTM can generate text and sometimes infer a pattern from examples in its prefix. The limitation is scale and reliability over long, complex prompts. An LSTM processes tokens serially and repeatedly compresses the past into one hidden state. A Transformer keeps a representation for every token and attention can directly select relevant earlier positions. This made long-context pattern use and large-scale GPU training much more effective. GPT-3 demonstrated the result: prompts with instructions and example input-output pairs can be continued as a temporary task, without parameter updates.

---

> Attention asks: **among the available tokens, what context should influence this token now?** The QKV computation below produces that decision.

## Slide 8 — Inside one attention head: Query, Key, Value

### On slide

> **Objective:** explain how the same Head 1 that highlights **it ← animal** produces that relationship.

Q, K, and V are three learned views of the same token representations. They separate three jobs: **what the focus token needs** → **how candidates are matched** → **what information selected candidates contribute**.

~~~text
Input representations H:   The | animal | was | tired | so | it | slept
                              ├── H × WQ → Q(it), the focus Query
                              ├── H × WK → one Key per token
                              └── H × WV → one Value per token

Q(it) compares with every K(token)
        ↓
scaled dot-product scores → softmax weights
        ↓
0.03V(The) + 0.70V(animal) + ... → attention output for it
        ↓
a(it) = 0.03V(The) + 0.70V(animal) + ...
original H(it) + a(it) → contextual H′(it)
~~~

For every token representation, the model learns three projection matrices:

| Component | Role |
| --- | --- |
| **WQ, WK, WV** | learned matrices that project the same input representation into three different spaces |
| **Query (Q)** | learned request made by the focus token **it** |
| **Key (K)** | learned match tag for every permitted candidate token |
| **Value (V)** | learned information mixed into the focus representation after weighting |

~~~mermaid
flowchart LR
  I["Focus token: it"] --> Q["Query: what should update me?"]
  C["Permitted candidates: The | animal | …"] --> K["Keys: learned match tags"]
  C --> V["Values: learned information to contribute"]
  Q --> MATCH["Scaled dot-product score: QKᵀ / √dₖ"]
  K --> MATCH
  MATCH --> W["Softmax → relevance weights"]
  W --> MIX["Weighted sum of Values"]
  V --> MIX
  MIX --> O["Updated representation of it\nmostly informed by animal"]
~~~

### Speaker notes

The preceding multi-head visual establishes the reason for several heads. Now zoom into Head 1. The point is not that Q, K, and V are separate objects in language; they are three learned projections of the same incoming token representations. “It” supplies a Query: a learned request for useful context. Every permitted token—including “animal” and “street”—supplies a Key for matching and a Value for information transfer. In the original Transformer, Query–Key comparisons use a **scaled dot product**, not cosine similarity: `softmax(QKᵀ / √dₖ)` creates the attention weights. The model then combines Values using those weights, giving more weight to “animal” in this illustrative case. Actual token boundaries depend on the model’s tokenizer.

For the animated visual: (1) start from one input row, (2) reveal `H × WQ`, `H × WK`, and `H × WV` to form Q, K, and V, (3) show `Q(it)` matching against every Key, (4) reveal illustrative softmax weights with `animal` high, (5) show each weight multiplying its aligned Value, and (6) reveal `H′(it)`, the contextual vector for `it`. This output is not the word “animal”; it is a new vector for `it` that is strongly informed by the information carried by `animal`.

The final residual line is deliberately simplified to one head: a full Transformer also applies an output projection and LayerNorm around this sublayer.

**Why use three projections rather than one?** Matching and information transfer are different learned jobs. A key can represent how a token should be matched by other tokens, while a value can represent the information it contributes after a match; the query represents what the focus token needs. Q, K, and V are not hand-written semantic fields—they are learned projections that give attention separate spaces for relevance and content.

The database analogy is useful but incomplete. Queries, keys, and values are learned vector projections. Attention is a differentiable weighted computation, not an exact symbolic lookup or a retrieved fact.

---

## Slide 9 — Multi-head attention and masks

### On slide

> **Multi-head attention** performs several learned attention operations in parallel, then concatenates and projects their outputs.

For the sentence **“The animal was tired, so it slept”**, one head may give high weight from **it** to **animal**, while another may capture a different pattern such as the relation between **slept** and **tired**. These are illustrative patterns, not manually assigned questions or guaranteed one-to-one meanings for heads.

### Masks define what is permitted

| Mask | Permitted attention | Typical purpose |
| --- | --- | --- |
| **Padding mask** | real tokens, not artificial padding | batch unequal-length sequences |
| **Causal mask** | current and earlier tokens only | next-token generation |
| **Cross-attention mask** | decoder tokens to source/encoder positions | source-to-target generation |

~~~text
Target tokens:   [The] [model] [will] [predict]
At "will":       ✓ The   ✓ model   ✗ predict
~~~

### Speaker notes

One head is not “the antecedent head” by definition. Every head has separate learned Q, K, and V projections, so it can score and mix context in a different representation subspace. The model learns which patterns are useful from training data; the examples on the slide make the reason for parallel heads intuitive, rather than claiming heads have fixed human labels.

Do not conflate **heads** with **layers**. Heads operate in parallel inside one attention layer; stacked Transformer layers then repeat attention and MLP processing sequentially, refining every token representation further.

The causal mask is essential for decoder generation. Training can process target positions efficiently in parallel, but the mask prevents each position from seeing the future token it is supposed to predict.

---

## Slide 11 — Encoder models, decoder models, and encoder–decoder models

### On slide

| Family | Visibility pattern | Training objective | Representative models | Used for |
| --- | --- | --- | --- | --- |
| **Encoder-only** | each token sees left and right context | masked-token reconstruction | BERT, RoBERTa | classification, token classification, semantic embeddings, extractive QA |
| **Decoder-only** | each token sees earlier tokens only | next-token prediction | GPT, Llama, Claude-style LLMs | chat, text generation, code completion, open-ended continuation |
| **Encoder–decoder** | encoder sees source; decoder sees past target + source | source-to-target reconstruction | T5, BART | translation, summarisation, controlled text transformation |

~~~mermaid
flowchart LR
  E["BERT-style encoder\ninput → contextual vectors\n→ classifier / retriever"]
  D["GPT-style decoder\nprompt → next token\n→ next token → ..."]
  ED["T5/BART encoder-decoder\nsource → encoder\n→ decoder → target"]
~~~

### Speaker notes

BERT is an encoder-only Transformer. Its bidirectional representations are useful when the output is a label, a token annotation, an embedding, or an answer span rather than an open-ended generated paragraph.

GPT-style models are decoder-only. The causal restriction is what makes autoregressive generation possible, so this family dominates current conversational and code-generation LLMs. T5 and BART retain the original encoder–decoder pattern, which is particularly natural for source-to-target tasks such as translation and summarisation.

---

## Slide 12 — Pre-training learns patterns; prompting supplies this task

### On slide

~~~mermaid
flowchart LR
  D["Broad corpus\ntext, code, documents"] --> TOK["Tokenise training sequences"]
  TOK --> OBJ["Self-supervised objective"]
  OBJ --> ERR["Compare prediction with known token"]
  ERR --> UPD["Update parameters"]
  UPD --> REP["Repeat at scale"]
~~~

| Architecture | Pre-training objective | Learned behaviour |
| --- | --- | --- |
| Decoder-only | causal language modelling: predict next token | continuation and generation |
| Encoder-only | masked language modelling: reconstruct hidden token | bidirectional representations |
| Encoder–decoder | corrupt input then reconstruct, or map source to target | sequence transformation |

> Pre-training learns broad statistical regularities. A prompt supplies the task, constraints, examples, and available evidence for one request.

### Speaker notes

During pre-training, the target token already exists in the corpus. The model predicts it, receives an error signal, and updates parameters by gradient-based optimisation. Repeating this over large corpora creates broad regularities in language and code without a human label for every sentence.

Instruction tuning and preference optimisation can make a model more useful in conversation, but they do not turn it into a live database or a verified source of truth.

---

## Slide 13 — Inference: from prompt to response

### On slide

~~~mermaid
flowchart LR
  P["Prompt + conversation\n+ retrieved context"] --> T["Tokenise"]
  T --> M["Decoder-style LLM"]
  M --> L["Logits:\nscore for each next token"]
  L --> D["Decoding strategy"]
  D --> N["Selected next token"]
  N --> M
  N --> O["Stop condition → response"]
~~~

| Term | Meaning | Not a guarantee of |
| --- | --- | --- |
| **Logits** | raw model scores over the vocabulary | probability until softmax |
| **Temperature** | controls how concentrated the sampling distribution is | factuality or intelligence |
| **Top-k** | retains the k highest-scoring candidates | quality |
| **Top-p** | retains the smallest set reaching probability mass p | validation |
| **Max output / stop sequence** | limits when generation ends | completeness |

### Speaker notes

A decoder-style LLM generates autoregressively: select one token, append it to context, and repeat. Greedy decoding selects the highest-probability token; sampling chooses among candidates. Temperature adjusts how concentrated or diverse the candidate distribution becomes.

Temperature is not a factuality control. For structured extraction, lower randomness plus schema validation is usually more appropriate than relying on a high-temperature conversational response.

---

## Slide 14 — Capabilities and limitations follow from the design

### On slide

| Capability | Why it is possible | Quality check |
| --- | --- | --- |
| Drafting and continuation | next-token generation over learned patterns | rubric and human review |
| Summarisation and transformation | conditioning on supplied source text | fidelity to source |
| Translation | source-to-target sequence mapping | adequacy and fluency |
| Code assistance | learned regularities in language and code | tests, static analysis, review |
| Structured extraction | generate a constrained representation from text | schema + source evidence |
| In-context adaptation | examples and constraints alter current context | held-out task cases |

| Limitation | Why it occurs | Engineering response |
| --- | --- | ---|
| Hallucination | token likelihood is not external verification | retrieval, citations, verification |
| Knowledge freshness | parameters are not a live information source | current approved sources at runtime |
| Context limits | attention covers the supplied window only | select, chunk, and retrieve context |
| Non-determinism | sampling can choose different candidates | version prompts and configuration |
| Prompt injection | untrusted text can compete with instructions | separate data from instructions; enforce authority in code |
| No accountability | model predicts text; it does not own a decision | human or policy authority at consequence boundaries |

### Speaker notes

Capabilities are conditional, not blanket guarantees. An LLM can produce a plausible technical explanation, transform a supplied document, or synthesise code, but fluent output can still be wrong.

The model is neither a source of truth, a policy engine, nor an accountable actor. It is a probabilistic model inside a larger system, and the cost of error should determine retrieval, validation, and human review.

---

## Slide 15 — The faculty-level mental model

### On slide

~~~text
Language model: predicts likely language units from context
        ↓
Large language model: scaled in data, parameters, and compute
        ↓
Prompt → tokens → embeddings + position
        ↓
Transformer layers: attention mixes context; MLPs transform representations
        ↓
Architecture family determines permitted context:
encoder-only / decoder-only / encoder–decoder
        ↓
Decoding turns token scores into an output
        ↓
Evaluation and control determine whether that output is usable
~~~

> **An LLM is a probabilistic sequence model with learned representations—not a reasoning oracle, database, or autonomous authority.**

### Speaker notes

This model explains both the power and the limits. Attention makes token meaning context-sensitive; depth and scale build rich learned representations; autoregressive decoding makes open-ended generation possible.

The next chapter is Prompt Engineering. It will focus on how to structure model context, specify constraints, and evaluate the output a model is likely to generate.

---

## Research sources

1. Jurafsky and Martin, [N-gram Language Models](https://www.web.stanford.edu/~jurafsky/slp3/3.pdf).
2. [Carnegie Mellon: Language Modeling and Information Retrieval](https://www.cs.cmu.edu/~lemur/background.html).
3. Vaswani et al., [Attention Is All You Need](https://arxiv.org/abs/1706.03762), 2017.
4. [Hugging Face Course: How do Transformers work?](https://huggingface.co/docs/course/main/en/chapter1/4).
5. [Hugging Face: How Transformers solve tasks](https://huggingface.co/docs/course/en/chapter1/5).
6. Devlin et al., [BERT: Pre-training of Deep Bidirectional Transformers](https://arxiv.org/abs/1810.04805), 2018.
7. Raffel et al., [Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer](https://arxiv.org/abs/1910.10683), 2019.
8. Brown et al., [Language Models are Few-Shot Learners](https://arxiv.org/abs/2005.14165), 2020.
9. [Hugging Face Tokenizers documentation](https://huggingface.co/docs/transformers/fast_tokenizers).
10. [Hugging Face Generation and decoding parameters](https://huggingface.co/docs/transformers/main_classes/text_generation).
11. [Hugging Face Generation strategies](https://huggingface.co/docs/transformers/en/generation_strategies).
12. [NIST: Foundation model definition](https://csrc.nist.gov/glossary/term/foundation_model).
