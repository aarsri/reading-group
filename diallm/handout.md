# DiaLLM: Bridging Dialect Robustness and Generation Through Post-Training
Presented by Aarohi Srivastava on July 31, 2026

## Motivation

Most work on dialect adaptation asks whether the model can *understand* dialectal input. DiaLLM is more interested in a related but less-explored problem: whether the model can generate in a particular dialect well. They find that LLMs often may display strong understanding of a dialect, but the response is typically in generic standard English (in line with typical post-training).

An interesting aspect of this paper to me is that they empirically separate the training stages that influence understanding/robustness versus generation. Throughout the paper, understanding benchmarks (classic NLP tasks) and generation (free-form response) evaluation often tell different stories.

## Method

The overall pipeline has three stages:
1. continual pretraining (CPT)
2. supervised fine-tuning (SFT)
3. post-training (DPO, GRPO, or GSPO)

These stages are useful for bringing about dialect robustness and generation capabilities, and the authors are able to measure performance after each stage to understand how robustness and generation ability change with each step.

### Models

The authors work with the following models: 
All of these are the base models except ? which is the instruct model.

### Continued Pre-training (CPT)

All models first undergo CPT on naturally occurring text from 18 English varieties in the International Corpus of English. CPT is like fine-tuning while retaining the pre-training objective and using a large amount of general rather than task data (usually used for domain or language adaptation). This step gives the model broad exposure to dialectal language, but it does not directly teach the model how to respond to instructions or generate in a particular target dialect.

### Instruction Tuning via SFT

The models are then instruction-tuned using UltraFeedback, an existing dataset of prompts and preferred responses. There are two versions of the training pipeline after CPT: implicit and explicit:
* Implicit Pipeline: SFT uses original responses from UltraFeedback (assumed standard English).
* Explicit Pipeline: SFT uses synthetically transformed responses from UltraFeedback into English varieties (e.g., Australian English, Indian English).

### Post-Training via DPO, GRPO, or GSPO
These methods are used to encourage dialectal generation, usually through preference optimization.
