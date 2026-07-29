## DiaLLM: Bridging Dialect Robustness and Generation Through Post-Training
Presented by Aarohi Srivastava on July 31, 2026

### Motivation
Most work on dialect adaptation asks whether the model can *understand* dialectal input. DiaLLM is more interested in a related but less-explored problem: whether the model can generate in a particular dialect well. They find that even when LLMs display understanding of a dialect (e.g., Indian English), the response is typically in generic standard English.

An interesting aspect of this paper to me is that they empirically separate the training stages that influence understanding/robustness versus generation. Throughout the paper, understanding benchmarks (classic NLP tasks) and generation (free-form response) evaluation often tell different stories.

### Background

The paper builds on two main resources.

ICE (International Corpus of English)

A collection of naturally occurring English from 18 regional varieties (~20M tokens total).

Examples include:

* Australian English
* Indian English
* Northern British English
* Nigerian English
* Singapore English
* etc.

This corpus is used for continual pretraining (CPT).

Importantly, this is real language rather than synthetic transformations.

UltraFeedback

UltraFeedback is an existing instruction-tuning dataset.

Each example contains:

* a prompt
* a preferred (“chosen”) response
* a rejected response

DiaLLM uses it in two different ways.

For ordinary SFT:

Prompt
↓
Chosen response

For dialect-specific training, they first transform the chosen response with Multi-VALUE, producing a dialectal version:

Prompt
Standard response
↓
Multi-VALUE
↓
Dialectal response

This becomes the supervision target.

Multi-VALUE vs. eWAVE

These were initially confusing because they are closely related but different.

eWAVE

The Electronic World Atlas of Varieties of English is a linguistic resource describing dialect features.

Examples:

Indian English

“I’m liking this.”

Northern British English

“He’s me brother.”

Australian English

“It was good, like.”

These are descriptions of dialectal features.

Multi-VALUE

Multi-VALUE is a rule-based transformation system built from these linguistic descriptions.

It automatically converts standard English into synthetic dialectal text by applying eWAVE-inspired grammatical transformations.

Thus:

eWAVE
    ↓
describes dialect features
Multi-VALUE
    ↓
applies those features

The synthetic outputs are then used for supervised fine-tuning.

⸻

Overall Training Pipeline

Every experiment begins the same way.

Base model
      ↓
Continual pretraining (ICE)

After that, the paper branches into two pipelines.

⸻

1. Broad (“Implicit”) Adaptation

The goal here is general dialect competence, not one particular variety.

ICE continual pretraining
        ↓
Standard UltraFeedback SFT
        ↓
DPO / GRPO / GSPO
(using pooled dialect information)

Important:

The SFT stage itself contains no explicit dialect transformations.

Its purpose is simply to restore instruction-following after continual pretraining.

Dialect information instead comes from:

* the ICE continual pretraining
* pooled dialect preference data
* rewards over all dialect features

⸻

2. Explicit (Targeted) Adaptation

Separate models are trained for Australian, Indian, and Northern British English.

ICE continual pretraining
        ↓
Dialect-specific SFT
(Multi-VALUE outputs)
        ↓
Targeted DPO / GRPO / GSPO

Now the SFT stage already teaches the model to answer prompts using the target variety.

Alignment then reinforces that same variety.

⸻

What is the SFT stage actually doing?

Initially I expected SFT to be “the dialect learning stage.”

It actually serves a broader purpose.

After continual pretraining, instruction-following degrades.

Standard SFT restores assistant behavior.

For explicit adaptation, they replace the ordinary responses with Multi-VALUE outputs so that instruction tuning simultaneously teaches the desired dialect.

One question I still have is whether standard SFT suppresses some of the richer dialect competence learned during continual pretraining. The paper evaluates downstream behavior rather than directly measuring retention of dialect representations.

⸻

Post-Training (the interesting part)

The post-training comparison is really the core contribution.

The paper compares three popular alignment methods while keeping the rest of the pipeline fixed.

⸻

DPO

DPO is an offline preference optimization method.

Each training example contains

Prompt
Chosen:
dialect-transformed response
Rejected:
original standard-English response

Unlike typical DPO, the difference is not answer quality.

The semantic content is intended to stay nearly identical.

Instead the preference simply says

prefer the dialectal realization over the standard-English realization.

Broad adaptation pools Australian, Indian, and Northern British examples together.

Explicit adaptation trains only on one variety.

I think this is an elegant use of DPO because it directly learns from paired examples rather than optimizing a hand-designed reward.

⸻

GRPO

GRPO is online reinforcement learning.

For each prompt the current model generates four candidate responses.

Each response receives a reward.

The rewards are normalized relative to the other responses from that prompt.

The model is updated to increase the probability of the higher-reward responses.

Unlike PPO, GRPO does not require a learned value model.

⸻

GSPO

GSPO uses essentially the same setup:

* same prompts
* same sampled responses
* same reward

The main difference is how credit is assigned.

Rather than computing policy updates token-by-token, GSPO performs sequence-level optimization.

The authors motivate this by arguing that dialectal style is naturally a sequence-level property rather than a collection of independent token decisions.

⸻

The Composite Reward

GRPO and GSPO optimize a weighted combination of three scores.

[
R

0.8\phi_{dialect}
+
0.1\phi_{COMET}
+
0.1\phi_{cosine}
]

where

Dialect score (80%)

Measures how strongly the generated response exhibits the desired dialectal features.

COMET (10%)

Measures semantic similarity to the original SFT response.

Embedding cosine similarity (10%)

Another semantic-preservation signal.

Notice that 80% of the reward is devoted to dialect feature detection.

Naturalness, authenticity, pragmatic appropriateness, or cultural fit are not directly optimized.

⸻

How is the dialect reward computed?

This was the part I most wanted to understand.

The authors train a BERT-based multilabel classifier that predicts the probability of each eWAVE feature appearing in a sentence.

For example, if targeting Indian English, the active feature set includes things like

* stative progressive
* lack of subject–auxiliary inversion
* etc.

The reward is

[
\phi_{dialect}(y)

\log
\left(
1+
\sum_{i\in F}
p_i
\right)
]

where

* (p_i) is the classifier probability for feature (i)
* (F) is the active feature set

Broad adaptation uses all 135 detected features.

Explicit adaptation uses only the features associated with the target dialect.

One interesting consequence is that the reward measures feature density, not whether the overall response sounds authentic to a speaker of that variety.

⸻

How is robustness evaluated?

This paper is not using representation probes.

Instead, robustness is measured using downstream task performance.

Examples include

* GLUE
* VALUE (dialectal GLUE)
* BESSTIE sentiment
* BESSTIE sarcasm
* DialectBench
* BBH
* GPQA

So robustness here means

can the model still solve tasks when the inputs are dialectal?

⸻

How is generation evaluated?

Generation is almost the opposite.

The prompts are ordinary open-ended questions.

Examples from the paper include

What’s the best piece of advice someone’s ever given you?

What’s your take on people who are always late?

There is no objectively correct answer.

Instead they ask

Does the response convincingly resemble the target dialect?

Evaluation is done with

* human annotators
* automatic dialect classifiers
* Phi-4 as an LLM judge

⸻

Main Result: Robustness and generation are only loosely coupled

This is probably the result I found most interesting.

The paper evaluates checkpoints after

Base
↓
Continual pretraining
↓
SFT
↓
Alignment

Across these stages they observe

* continual pretraining changes robustness substantially
* SFT restores most benchmark performance
* alignment causes relatively modest benchmark changes

Yet generation changes much more dramatically after explicit dialect SFT and alignment.

The takeaway is not that robustness and generation are unrelated.

Rather,

robustness benchmarks tell us surprisingly little about what has changed in generation.

This seems particularly relevant for dialect robustness research.

⸻

Main Result: Explicit adaptation produces better dialect generation

When the goal is producing a specific dialect,

targeted adaptation consistently outperforms broad adaptation.

This makes intuitive sense.

Learning “dialect in general” is a different objective from producing a recognizable Australian or Indian English response.

I don’t think this should be interpreted as evidence that targeted adaptation is better for robustness.

Instead it highlights that

broad competence and controlled generation are different goals.

⸻

Main Result: DPO vs. GRPO vs. GSPO

This turned out to be much more nuanced than I expected.

GRPO

* achieves the highest training reward
* maximizes detected dialect features

Yet humans generally do not prefer its outputs.

Instead

* DPO is preferred for Indian and Northern British English
* GSPO performs best for Australian English

One possible interpretation is that optimizing the classifier-derived reward is not the same thing as producing authentic dialect.

DPO never directly optimizes the reward.

Instead it learns from paired examples, which may help preserve more natural distributions despite the synthetic data.

⸻

LLM-as-a-Judge

The paper also evaluates outputs with Phi-4.

Overall it broadly recovers the same trends as humans, especially for comparing broad versus targeted adaptation.

However, giving Phi-4 the eWAVE feature inventory changes its judgments noticeably.

This suggests it becomes more sensitive to explicit grammatical markers rather than necessarily judging dialect the same way humans do.

The paper therefore treats LLM evaluation as complementary rather than replacing human judgments.

⸻

Thoughts / Discussion

One thing I appreciated is that the paper doesn’t simply compare alignment algorithms.

Instead it asks what different stages of training contribute to dialect adaptation.

That framing feels much more useful for understanding where robustness and generation actually come from.

At the same time, I was left with several questions.

Natural pretraining vs. synthetic post-training

Continual pretraining uses naturally occurring dialectal text.

Post-training uses synthetic Multi-VALUE transformations.

Does post-training preserve the richer dialect competence learned during CPT, or does it collapse it into a relatively small inventory of recognizable features?

⸻

Is morphosyntax enough?

Multi-VALUE primarily modifies morphosyntax.

Real dialectal communication also involves

* lexical choice
* orthography
* discourse markers
* pragmatics
* culturally grounded language

These dimensions seem especially important for generation but are largely absent from the reward.

⸻

Reward design

The reward emphasizes

“Can I detect dialect features?”

rather than

“Does this sound authentic?”

Is GRPO exposing weaknesses in the reward, or does dialect generation simply require richer objectives than feature detection?

⸻

Robustness vs. generation

Improving robustness means users should not need to standardize their language to be understood.

Generation is a different design question.

Should models automatically respond in a user’s dialect?

When is accommodation helpful?

When might it feel stereotyped or inappropriate?

The paper motivates dialect generation as an important capability, but I think these interaction questions remain largely open.

⸻

Broad vs. targeted adaptation

For my own work, broad dialect robustness is the primary objective.

A model that understands many varieties well should not necessarily produce strongly marked output for any one variety.

This paper suggests that broad robustness and targeted generation may be optimized by different stages of training, which I think is an interesting direction for future work.

Based on the DiaLLM paper and our discussion. All methodological descriptions, equations, and reported findings are drawn from the paper unless explicitly framed as interpretation or discussion.
