# Bottlenecks to Robustness in Text and Speech Models: A Dialectal Perspective

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

### Overview
* Input: token sequence → predict masked tokens from context → Output: continuous embeddings
* Encoder-only models are trained with the masked language modeling objective, encouraging the model to build context-sensitive embeddings so that different surface token sequences that point to the same semantics map to nearby representations.

### Underlying Vulnerabilities wrt Dialects
* For encoder-only models, I view robustness as *being able to build meaningful representations for linguistically diverse text*.
* **Key vulnerability**: This is limited by the **tokenization bottleneck**. Dialectal text is often split into uncommon token sequences (particularly oversegmented ones), so the model may never see it enough to learn a stable representation.
  * Tokenization is rigid: small spelling variation → very different token sequences → embeddings don't align → semantic drift
  * Example:
    * tokenizer(student) → student
    * tokenizer(studebt) → stud, eb, t

### Strengthening the Model
* Intentionally manipulating token sequences during training, particularly via character-level corruption and subword regularization, encourages the model to build invariant representations across surface variation.
* **Learning goal**: multiple tokenizations (even unusual ones) can yield the same contextual meaning → stronger invariance in the embedding space.

### Takeaway
Dialect robustness here is mostly about escaping the tokenization bottleneck and learning useful embeddings. The challenge is that tokenization schemes are usually baked in, making this a structural limitation.

## Decoder-Only Text Models (e.g,. GPT, LLaMA)
Goal: Robustness in generative mappings.

### Overview
* Input: left context tokens → Output: next token prediction
* Decoder-only models are trained with causal language modeling, predicting the next token given previous context.

### Underlying Vulnerabilities wrt Dialects
* For decoder-only models, I view robustness as (1) *being able to understand prompts in different varieties*, and (2) *being able to generate appropriate responses*. An additional concern is whether the model should generate in the standard or in the dialect.
* **Key vulnerability**: The autoregressive setup of decoder-only models can amplify the tokenization bottleneck in encoder-only models, because the error can propogate forward in generation. 
  * In addition to the tokenization bottleneck and rigidity issues found in encoder-only models, unusual token sequences can disrupt both understanding of the input *and* quality of generated output.
  * This is especially visible in interactive generative contexts when dialect variation is present.

### Strengthening robustness
* Manipulating token sequences during training can potentially help with two goals:
  * varied input forms can still lead to the same intent → input robustness
  * meanings map to multiple valid surface continuations → output diversity
* However, success may require being more intentional with how this is carried out:
  * *Speculation*: Because a decoder-only model does not have an encoder, it does not necessarily have as much incentive to build robust representations. We need to find a way to leverage the autoregressive paradigm and the next-word prediction objective.

### Takeaway
Manipulating token sequences can teach robustness both at input (dialect forms → same intent) and output (many continuations → same meaning). The difficulty is that the autoregressive setup doesn’t naturally reward invariant embeddings, so robustness has to emerge indirectly, making it harder to control.

## Speech Encoders (e.g., wav2vec2, HuBERT)
Goal: Robustness in latent acoustic/phonetic space.

### Overview
* Input: speech signal → predict masked/clustered latent units from context → Output: continuous embeddings
* Speech encoders are trained with self-supervised and unsupervised objectives like masked prediction and clustering. Rather than mapping directly to text, they learn to form stable, context-aware embeddings of the speech signal itself.
* This means they capture latent acoustic-phonetic structure. In principle, this could support dialect robustness, since the model learns to represent speech frames independent of text.

### Underlying Vulnerabilities wrt Dialects
* Encoders often overfit to the distribution of speakers and dialects in the training data. If the representation space is too narrow, dialectal pronunciations may not be realized properly, leading to embeddings that are inconsistent with those of the same words in the standard variety.
* For speech encoders, I view robustness as *being able to represent acoustically diverse inputs* (e.g., accent, phonological variation) well and closely in the representation space. Because downstream tasks are usually built on top of speech encoders, other aspects of variation that are important to be robust to can come at those stages instead. Here, it's about the representations being built so they can be used later.
* **Key Vulnerability**: If the encoder overfits to narrow acoustic realizations, embeddings diverge across dialects → downstream models see them as different when they shouldn’t.
 * For example, a vowel shift common in one dialect might push embeddings into a region the model associates with entirely different phonetic content. Downstream models consuming these embeddings (e.g., ASR, SLU) would then inherit the fragility.

### Strengthening the Model
* Augmentation during pretraining (pitch shifting, time stretching, simulated accents) can encourage the model to form embeddings that are invariant to surface-level differences while preserving phonetic identity.
* Ideas: Mask spans in audio signal, regularization/dropout in the latent space, channel noise, speed/pitch.
* **Learning goal**: diverse acoustic input can still yield the same latent category → stronger invariance in phonetic/phonological space.

### Takeaway
Speech Encoders are well-positioned for dialect robustness. The challenge is that overfitting to narrow acoustic distributions is subtle. Encoders may look generalized but still miss dialect variation.

## Automatic Speech Recognition (ASR) Models (e.g., wav2vec2 + CTC, Whisper)
Goal: Robustness in acoustic-to-orthographic mapping.

### Overview
* Input: speech signal → Output: token sequence (text transcription)
* ASR models are trained to map variable acoustic waveforms to standardized orthographic tokens. Architecturally, this is usually a speech encoder front-end (e.g., wav2vec2) paired with a sequence transduction method such as CTC.
 * Connectionist temporal classification (CTC) adds a linear layer + softmax over characters to the wav2vec2 encoder.

### Underlying Vulnerabilities wrt Dialects
* For ASR models, I view robustness as *being able to provide valid, readable transcriptions for acoustically diverse inputs*. An additional concern is whether the output transcription should reflect standard or nonstandard orthography.
* From a dialect robustness perspective, the key is that these models learn an acoustic-to-orthographic mapping. They must normalize across speakers, accents, and environments, but the orthographic target is usually rigid.
* **Key Vulnerability**: If the dialectal realization diverges significantly from the canonical spelling, the model may not have incentive to preserve it.
* When the orthography of the dialect differs from the canonical form, ASR models are doubly stressed: the acoustic signal is different and the target transcription is assumed to be the standard.
* Larger-scale models gain robustness from greater linguistic diversity in the training data, but the vulnerability remains.

### Speech Encoders vs. ASR
* Because encoders do not produce orthography, they are in some ways better positioned than ASR models to support dialect robustness. They can build flexible representations that can later be aligned with dialect-aware text or semantic supervision.
* ASR models must commit to text, which makes them more brittle. Encoders remain in the latent space, which gives more room for invariance learning before downstream supervision.
* This contrast highlights a structural advantage: encoders can be repurposed or fine-tuned with dialect-aware signals, whereas ASR systems are “locked in” to orthography unless retrained end-to-end.

### Strengthening the Model
* Ideas: audio-native manipulation (e.g., speech/pitch perturbation, additive noise), phoneme substitutions using TTS model.
* **Learning goal**: many sounds correspond to the same transcription token → robustness to speech variation.
* Alternate learning goal: same utterance can be composed of different sounds and can be expressed differently orthographically (hard).

### Takeaway
ASR brittleness comes from committing to orthography. Augmentation can broaden tolerance, but without dialect-aware labels, robustness risks looking like noisier transcriptions rather than genuinely dialect-sensitive.

## Speech Language Models (e.g., AudioLM, Vall-E)
Goal: Robustness in generative unit distributions.

### Overview
* Input: sequence of discrete audio units (from quantizer) → next unit prediction (generative)
* Speech language models extend the decoder-only LM paradigm to audio. Instead of predicting text tokens, they predict discrete acoustic units.
* In principle, this allows them to capture dialect-specific patterns of pronunciation and prosody.

### Underlying Vulnerabilities wrt Dialects
* In practice, the quantizer is usually trained on standardized input, which bakes in a bias toward canonical realizations. This is similar to the tokenization bottleneck.
* **Key Vulnerability**: Dialectal pronunciations may be quantized into units that are rare or out-of-distribution relative to the training corpus. Once unusual units are introduced, the model can struggle to continue fluently.
* For Speech LMs, I view robustness as *being able to generate next units fluently despite acoustically diverse input*.

### Strengthening the Model
* Injecting controlled noise into the unit sequence (masking, dropping, or permuting units) could help the model recover from irregularities. Perhaps a key ingredient is to be able to update the quanitzed units we have to work with.
* **Learning goal**: semantic intent is compatible with many surface-level acoustic/prosodic realizations → strengthens generalization across speaking styles

### Speech LMs vs. ASR
* Compared to ASR, speech LMs bypass orthography entirely, which is an opportunity: they could in principle represent dialectal variants as equally valid continuations, rather than forcing them into standard spelling.
* Whether this works depends on the design of the unit space. This is a structural difference that makes quantization a crucial site of dialect robustness or fragility.

### Takeaways
* Quantization is a lot like the tokenization bottleneck. If dialectal realizations get poor unit codes, robustness collapses no matter how good the LM is. 
* Note that quanitzation is used in other speech models beyond Speech LMs, but the distinction is that quantization is an in-built part of typical Speech LMs, while it is just an auxiliary prediction target in speech encoders.
 * Specifically, HuBERT and wav2vec2 discretize intermediate representations, but the discrete units are not the output.
 * At inference for speech encoders, the encoder typically produces continuous embeddings (like BERT) rather than discrete codes.

## Speech Language Understanding (SLU) Models
Goal: Robustness in acoustic-to-semantic mapping.

### Overview
* Input: speech signal → Output: label
* SLU systems aim to map speech directly to semantic labels (e.g., intent classification, slot filling), sometimes bypassing text altogether. Architecturally, this often means a speech encoder feeding into a task-specific head.

### Underlying Vulnerabilities wrt Dialects 
* For SLU, I view robustness as *being able to predict the answer just as well whether the input is standard or nonstandard*.
* For dialect robustness, the key is that SLU models must learn invariances that align acoustic variation with stable semantic categories.
* **Key Vulnerability**: variation in pronunciation or word choice → embeddings shift → model misclassifies meaning

### Strengthening the Model
* **Learning goal**: acoustically and lexically diverse utterances can point to the same semantic intent → robust semantic invariance

## Insights and Takeaways

### Structural Bttlenecks Differ Across Model Types
* In encoder-only text models, the key vulnerability is the tokenization bottleneck. Dialectal spellings are fragmented or rare, destabilizing embeddings.
* In decoder-only text models, the vulnerability compounds: different tokens disrupt not just representations but the generative trajectory, amplifying mistakes forward.
* In speech encoders, the bottleneck is representation space bias. If the latent space reflects only a narrow set of acoustic realizations, dialect pronunciations fall into unstable or mismatched regions.
* In ASR models, the bottleneck is acoustic coverage. Dialectal phonetic shifts map poorly to standardized orthographic targets.
* In speech LMs, the weak link is quantization. Dialectal variants often land in rare or untrained unit codes, derailing autoregressive generation.
* In SLU models, the challenge is the semantic collapse. Dialectal pronunciation and lexical differences can both misalign embeddings with meaning categories.

### Role of Pretraining Objectives
* Assumptions that dialects break: ASR assumes standard mappings; encoders assume training coverage; LMs assume unit distributions.
* Masked prediction (encoder-only text, speech encoders) encourages representation invariance: mapping diverse surface forms into stable embeddings.
* Autoregressive prediction (decoder-only text, speech LMs) encourages continuation fluency, but also makes models more fragile because they are end-to-end.
* ASR enforces canonicalization, which increases brittleness: dialectal input is forced into standardized orthography.
* SLU enforces meaning invariance, but risks overlooking systematic dialectal richness if only exposed to standard forms.

### Noise as a Unifying Tool but with Different Effects
* For encoders (text or speech), noise enforces embedding invariance, teaching the model that multiple realizations should collapse into the same representation.
* For autoregressive models, noise encourages trajectory recovery, training the system to continue fluently despite irregular or unexpected tokens or units.
* For ASR, noise can broaden speaker and accent tolerance, but without dialect-aware supervision, it risks blurring meaningful variation into generic robustness.
* For SLU, noise aligned with semantic labels is most directly effective, since the goal is to preserve meaning across variation.
* Noise is versatile but contextual. The same principle (perturb input to encourage invariance) plays out differently depending on whether the model builds embeddings, generates outputs, or maps directly to semantics.
