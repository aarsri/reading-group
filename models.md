# Title

## Introduction

* One challenge in language modeling is how to **achieve robustness across dialects**. Pre-training on more linguistically diverse data helps, but models still fail to generalize to unseen language varieties. 
* A unifying idea in exploring this is **invariance learning**. A model that succeeds in dialectal robustness learns to represent meaning in a way that is **stable under surface variation**.
* I examined six families of language models, thinking about:
  * Where are the vulnerabilities to dialectal variation?
  * What aspects of the model can we take advantage of to improve robustness?
  * Which aspects need improvement?

## Encoder-Only Text Models (e.g., BERT, RoBERTa)
Goal: Robustness in representations

* Input: token sequence --> Output: predict masked tokens from context.
* Encoder-only models are trained with the **masked language modeling** objective, encouraging the model to build context-sensitive embeddings so that different surface token sequences that point to the same semantics map to nearby representations.
* **Key vulnerability**: This is limited by the **tokenization bottleneck**. Dialectal text is often split into uncommon token sequences (particularly oversegmented ones), so the model may never see it enough to learn a stable representation.
  * Tokenization is rigid: small spelling variation --> very different token sequences --> embeddings don't align --> semantic drift
  * Example:
    * tokenizer(student) --> student
    * tokenizer(studebt) --> stud, eb, t
* Strengthening robustness:
  * Intentionally manipulating token sequences during training, particularly via character-level corruption and subword regularization, encourages the model to build invariant representations across surface variation.
  * Learning goal: multiple tokenizations (even unusual ones) can yield the same contextual meaning → stronger invariance in the embedding space.

## Decoder-Only Text Models (e.g,. GPT, LLaMA)
Goal: Robustness in generative mappings

* Input: left context tokens --> Output: next token prediction
* Decoder-only models are trained with **causal language modeling**, predicting the next token given previous context.
* They need to learn:
  * Multiple prompts may share underlying intent (like encoders).
  * One meaning (prompt) may map to multiple surface token sequences (generations).
* **Key vulnerability**: The autoregressive setup of decoder-only models can amplify the tokenization bottleneck in encoder-only models, because the error can propogate forward in the generation. 
  * If the model encounters nonstandard spellings or word usage, its predictive distribution may shift dramatically, since the unusual tokens will disrupt the flow of expectations.
  * This is visible moreso in interactive generative contexts when dialect variation is present.


