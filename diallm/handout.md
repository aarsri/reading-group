# DiaLLM: Bridging Dialect Robustness and Generation Through Post-Training
Presented by Aarohi Srivastava on July 31, 2026

## Introduction

* Most work on dialect adaptation asks whether the model can *understand* dialectal input. DiaLLM is more interested in a related but less-explored problem: whether the model can generate in a particular dialect well.
* LLMs may display strong understanding of a dialect, but the response is typically in generic standard English (in line with typical post-training).
* An interesting aspect of this paper to me is that they empirically separate the training stages that influence understanding/robustness versus generation.
* Throughout the paper, understanding benchmarks (classic NLP tasks) and generation (free-form response) evaluation often tell different stories.

## Method

The overall pipeline has three stages:
1. continual pretraining (CPT)
2. supervised fine-tuning (SFT)
3. post-training (DPO, GRPO, or GSPO)

These stages are useful for bringing about dialect robustness and generation capabilities, and the authors are able to measure performance after each stage to understand how robustness and generation ability change with each step.

### Models

The authors evaluate three models: Llama 3.2 3B Base, Qwen 2.5 3B Base, and Gemma 3 4B Instruct. 

### Continued Pre-training (CPT)

* All models first undergo CPT on naturally occurring text from 18 English varieties in the International Corpus of English (e.g., Australian, Indian, Northern British, Nigerian, Singaporean English).
* CPT is essentially continuing the model's original language-model training on new data. Rather than teaching the model a specific task (as in fine-tuning), the objective remains next-token prediction. CPT is commonly used to adapt models to a new domain or language.
* In this case, the goal is to expose the model to a broad range of naturally occurring dialectal English. Importantly, this stage does not teach the model how to follow instructions or how to generate a particular dialect on demand, it simply expands the language distribution the model has seen.
* One of the paper’s central questions is whether this broad dialect exposure is already sufficient, or whether later post-training is needed to produce dialectal output.

### Instruction Tuning via SFT

After CPT, the models are instruction-tuned on UltraFeedback, an existing dataset of prompts and preferred responses. The paper compares two training pipelines after CPT:
* Implicit Pipeline: The goal is simply to restore instruction-following capabilities after CPT; as a result, SFT uses original responses from UltraFeedback (assumed standard English, no dialects at this stage).
* Explicit Pipeline: Instead of using the standard responses from UltraFeedback, they first transform each example into a target variety (Australian, Indian, or Northern British English) using Multi-VALUE. The model is therefore instruction-tuned directly on dialectal responses, producing a separate SFT checkpoint for each target variety.
  * Unlike the implicit pipeline, there is no intermediate standard-English SFT stage, which the authors say helps preserve the dialectal signal introduced during CPT.

### Multi-VALUE and eWAVE
* Multi-VALUE performs rule-based transformations of English text into other varieties of English on the basis of linguistic features.
  * These linguistic features come from eWAVE (Electronic World Atlas of Varieties of English), a linguistic database describing morphosyntactic and lexical features associated with different English varieties.
  * Examples:
    * stative progressives: I'm liking this
    * possessive *me*: He's me brother
* The transformed responses used for SFT preserve the original semantic content and typically make relatively localized edits rather than rewriting the response in a substantially different voice.
* The authors also discuss several limitations of this approach.
  * Because the transformations are rule-based, they may introduce stereotypical or overgeneralized patterns, and individual eWAVE features are not always appropriate in every context or for every speaker.
  * More broadly, eWAVE describes population-level grammatical tendencies rather than complete dialects, so it cannot capture many aspects of natural language use such as register, discourse style, or broader cultural variation.
  * One consequence is that the explicit SFT stage teaches the model to generate synthetic realizations of morphosyntactic and lexical dialect features, while the earlier continual pre-training exposes it to naturally occurring dialect text. The paper does not investigate how these two sources of supervision interact, for example, whether synthetic SFT reinforces the richer dialect competence learned during CPT or instead narrows it toward the specific feature inventory represented by Multi-VALUE.

### Post-Training via DPO, GRPO, or GSPO
These methods are used to encourage dialectal generation, usually through preference optimization.
