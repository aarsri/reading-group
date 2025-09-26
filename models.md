# Title

## Introduction

* One challenge in language modeling is how to **achieve robustness across dialects**. Pre-training on more linguistically diverse data helps, but models still fail to generalize to unseen language varieties.
* Because language is constantly evolving, increasing linguistic diversity in pretraining data is valuable but is not always enough.
* A unifying idea in exploring this is **invariance learning**. A model that succeeds in dialectal robustness learns to represent meaning in a way that is **stable under surface variation**.
* I examined six families of language models, thinking about:
  * Where are the vulnerabilities to dialectal variation?
  * What aspects of the model can we take advantage of to improve robustness?
  * Which aspects need improvement?

## Encoder-Only Text Models (e.g., BERT, RoBERTa)
Goal: Robustness in representations.

* Input: token sequence → Output: predict masked tokens from context.
* Encoder-only models are trained with the **masked language modeling** objective, encouraging the model to build context-sensitive embeddings so that different surface token sequences that point to the same semantics map to nearby representations.
* **Key vulnerability**: This is limited by the **tokenization bottleneck**. Dialectal text is often split into uncommon token sequences (particularly oversegmented ones), so the model may never see it enough to learn a stable representation.
  * Tokenization is rigid: small spelling variation → very different token sequences → embeddings don't align → semantic drift
  * Example:
    * tokenizer(student) → student
    * tokenizer(studebt) → stud, eb, t
* **Strengthening robustness**:
  * Intentionally manipulating token sequences during training, particularly via character-level corruption and subword regularization, encourages the model to build invariant representations across surface variation.
  * **Learning goal**: multiple tokenizations (even unusual ones) can yield the same contextual meaning → stronger invariance in the embedding space.

## Decoder-Only Text Models (e.g,. GPT, LLaMA)
Goal: Robustness in generative mappings.

* Input: left context tokens → Output: next token prediction
* Decoder-only models are trained with **causal language modeling**, predicting the next token given previous context.
* **Key vulnerability**: The autoregressive setup of decoder-only models can amplify the tokenization bottleneck in encoder-only models, because the error can propogate forward in generation. 
  * In addition to the tokenization bottleneck and rigidity issues found in encoder-only models, unusual token sequences can disrupt both understanding of the input *and* quality of generated output.
  * This is especially visible in interactive generative contexts when dialect variation is present.
* **Strengthening robustness**:
 * Manipulating token sequences during training can potentially help with two goals:
   * varied input forms can still lead to the same intent → input robustness
   * meanings map to multiple valid surface continuations → output diversity
 * However, success may require being more intentional with how this is carried out.

## Speech Encoders (e.g., wav2vec2, HuBERT)
Goal: Robustness in latent acoustic/phonetic space.

* Input: speech signal → Output: predict masked/clustered latent units from context
* Speech encoders are trained with self-supervised objectives such as masked prediction or clustering. Rather than mapping directly to text, they learn to form stable, context-aware embeddings of the speech signal itself.
* This means they capture latent acoustic-phonetic structure. In principle, this could support dialect robustness, since the model learns to represent speech frames independent of text.
* Despite the self-supervised setup, encoders often overfit to the distribution of speakers and dialects in the training data. If the representation space is too narrow, dialectal pronunciations may not be normalized properly, leading to embeddings that are inconsistent with those of the same words in the standard variety.
* **Key Vulnerability**: If the encoder overfits to narrow acoustic realizations, embeddings diverge across dialects → downstream models see them as different phones when they shouldn’t.
 * For example, a vowel shift common in one dialect might push embeddings into a region the model associates with entirely different phonetic content. Downstream models consuming these embeddings (e.g., ASR, SLU) would then inherit the fragility.
* Strengthening
 * Augmentation during pretraining (pitch shifting, time stretching, simulated accents) can encourage the model to form embeddings that are invariant to surface-level differences while preserving phonetic identity.
 * Ideas: Mask spans in audio signal, regularization/dropout in the latent space, channel noise, speed/pitch
 * Learning goal: diverse acoustic evidence can still yield the same latent category → stronger invariance in phonetic/phonological space

## Automatic Speech Recognition (ASR) Models (e.g., wav2vec2 + CTC, Whisper)
Goal: Robustness in acoustic-to-orthographic mapping.

* Input: speech signal → Output: token sequence (text transcription)
* ASR models are trained to map variable acoustic waveforms to standardized orthographic tokens. Architecturally, this is usually a speech encoder front-end (e.g., wav2vec2) paired with a sequence transduction method such as CTC.
* From a dialect robustness perspective, the key is that these models learn an acoustic-to-orthographic mapping. They must normalize across speakers, accents, and environments, but the orthographic target is rigid: if the dialectal realization diverges significantly from the canonical spelling, the model has no incentive to preserve it.
* When the orthography of the dialect differs from the canonical form, ASR models are doubly stressed: the acoustic signal is different and the target transcription is assumed to be the standard.
* Larger-scale models gain robustness from training diversity, but the vulnerability remains whenever dialects are underrepresented in the training mix.

Contrasts
* Because encoders stop short of producing orthography, they are in some ways better positioned than ASR models to support dialect robustness: they build flexible representations that can later be aligned with dialect-aware text or semantic supervision.
* Both start from acoustics, but ASR must commit to text early, which makes it more brittle. Encoders remain in the latent space, which gives more room for invariance learning before downstream supervision.
* This contrast highlights a structural advantage: encoders can be repurposed or fine-tuned with dialect-aware signals, whereas ASR systems are “locked in” to orthography unless retrained end-to-end.

Strengthening
* Learning goal: many sounds correspond to the same transcription token → robustness to speech variation
* Alternate learning goal: same utterance can be composed of different sounds and can be expressed differently orthographically (hard)

## Speech Language Models (e.g., AudioLM, Vall-E)
Goal: Robustness in generative unit distributions.

* Input: sequence of discrete audio units (from quantizer) → next unit prediction
* Speech language models extend the decoder-only LM paradigm to audio. Instead of predicting text tokens, they predict discrete acoustic units (from quantizers like SoundStream or EnCodec).
* This means they learn the distribution of speech units in context — modeling fluency in speech directly, including prosody, rhythm, and phonotactics.
* In principle, this allows them to capture dialect-specific patterns of pronunciation and prosody. But in practice, the quantizer is usually trained on standardized input, which bakes in a bias toward canonical realizations.
* **Key Vulnerability**: Dialectal pronunciations may be quantized into units that are rare or out-of-distribution relative to the training corpus. Once unusual units are introduced, the model can struggle to continue fluently.

Strengthening
* Injecting controlled noise into the unit sequence (masking, dropping, or permuting units) could help the model recover from irregularities. Perhaps a key ingredient is to be able to update the quanitzed units we have to work with.
* Learning goal: semantic intent is compatible with many surface-level acoustic/prosodic realizations → strengthens generalization across speaking styles

Contrast
* Compared to ASR, speech LMs bypass orthography entirely, which is an opportunity: they could in principle represent dialectal variants as equally valid continuations, rather than forcing them into standard spelling.
* Whether this works depends on the design of the unit space — a structural difference that makes quantization a crucial site of dialect robustness or fragility.

## Speech Language Understanding (SLU) Models
Goal: Robustness in acoustic-to-semantic mapping.

* SLU systems aim to map speech directly to semantic labels (e.g., intent classification, slot filling), sometimes bypassing text altogether. Architecturally, this often means a speech encoder feeding into a task-specific head.
* For dialect robustness, the key is that SLU models must learn invariances that align acoustic variation with stable semantic categories. This is a different demand than orthographic transcription: semantic consistency matters more than surface form fidelity.
* **Key Vulnerability**: variation in pronunciation or word choice → embeddings shift → model misclassifies meaning

Strengthening
* Learning goal: acoustically and lexically diverse utterances can point to the same semantic intent → robust semantic invariance

## Insights

### Structural bottlenecks differ across model type.
* In encoder-only text models, the key vulnerability is the tokenization bottleneck. Dialectal spellings are fragmented or rare, destabilizing embeddings.
* In decoder-only text models, the vulnerability compounds: odd spellings disrupt not just representations but the generative trajectory, amplifying mistakes forward.
* In speech encoders, the bottleneck is representation space bias. If the latent space reflects only a narrow set of acoustic realizations, dialect pronunciations fall into unstable or mismatched regions.
* In ASR models, the bottleneck is acoustic coverage. Dialectal phonetic shifts map poorly to standardized orthographic targets, forcing the model into hallucinations or collapses.
* In speech LMs, the weak link is quantization. Dialectal variants often land in rare or untrained unit codes, derailing autoregressive generation.
* In SLU models, the challenge is the semantic collapse. Dialectal pronunciation and lexical differences can both misalign embeddings with meaning categories.

### The role of pretraining objectives in invariance learning.
* Masked prediction (encoder-only text, speech encoders) encourages representation invariance: mapping diverse surface forms into stable embeddings.
* Autoregressive prediction (decoder-only text, speech LMs) encourages continuation fluency, but also makes models more fragile because they are end-to-end.
* ASR enforces canonicalization, which increases brittleness: dialectal input is forced into standardized orthography.
* SLU enforces meaning invariance, but risks overlooking systematic dialectal richness if only exposed to standard forms.

### Noise as a unifying tool, but with different effects.
* For encoders (text or speech), noise enforces embedding invariance, teaching the model that multiple realizations should collapse into the same representation.
* For autoregressive models, noise encourages trajectory recovery, training the system to continue fluently despite irregular or unexpected tokens or units.
* For ASR, noise can broaden speaker and accent tolerance, but without dialect-aware supervision, it risks blurring meaningful variation into generic robustness.
* For SLU, noise aligned with semantic labels is most directly effective, since the goal is to preserve meaning across variation.
* Noise is versatile but contextual. The same principle (perturb input to encourage invariance) plays out differently depending on whether the model builds embeddings, generates outputs, or maps directly to semantics.

### The deeper insight: why dialects remain challenging.
* All six families struggle not because dialects are inherently “hard,” but because their architectural commitments and pretraining setups bias them toward invariances that do not align perfectly with dialectal variation.
* Text models assume orthographic regularity; ASR assumes standard mappings; encoders assume training coverage; LMs assume unit distributions.
* Dialects systematically push against these assumptions, exposing what each family takes as “invariant” and what it discards.
