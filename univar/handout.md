# High-Dimension Human Value Representation in Large Language Models

_Samuel Cahyawijaya, Delong Chen, Yejin Bang, and others at NAACL 2025_

_Presented by Aarohi Srivastava on 9/11/26_

## Motivation

-   LLM alignment methods such as RLHF, DPO, safety tuning, etc. are
    intended to shape model behavior according to human preferences and
    values. But after training, **how do we characterize what values a
    model actually expresses?**
-   One option is to give the model an existing human-values survey and
    score it on the survey's predefined axes.
    -   Hofstede, for example, represents cultures with 6 dimensions;
        Schwartz's theory has 19; the World Values Survey is often
        summarized using a relatively small set of dimensions.
    -   This is interpretable, but we only learn about distinctions that
        the survey designers chose in advance.
-   This paper proposes **UniVaR (Universal Value Representation)** to
    learn a high-dimensional embedding based on LLM answers to
    various value-related questions.
-   The basic idea is similar to other embeddings:
    -   GloVe maps words to vectors so words used similarly end up
        nearby.
    -   A sentence embedding model maps sentences to vectors so
        semantically similar sentences end up nearby.
    -   **UniVaR is an embedding model trained so that sets of answers
        expressing similar LLM value behavior end up nearby.**
-   UniVaR is therefore **not a new LLM and does not modify the LLM
    being studied**. It is a separate, fixed embedding model that is trained
    once and can then be used to obtain embeddings for value-related responses from LLMs, including models it never saw during training.

### What does a UniVaR representation actually look like?

Suppose we want to compare Model A and Model B. We ask both models
questions such as:

-   Is individual success more important than the welfare of the
    community?
-   Should someone prioritize personal well-being over societal
    expectations?
-   How important is it to preserve cultural traditions?

Model A's answers might consistently emphasize personal choice and
autonomy, while Model B's emphasize social obligation and harmony.

We then give each model's question-answer pairs to **the same UniVaR
encoder**. UniVaR converts them into vectors. If the learned
representation is meaningful, Model A's value-related answers should
occupy one region of the space and Model B's another.

This is much closer to using a pretrained sentence embedding model than
to fine-tuning a separate UniVaR for every LLM: **after UniVaR has been
trained, the encoder is static and can be applied to outputs from a new
model.**

The harder question is what the distances in this space *mean*. The
dimensions are not labeled "individualism," "tradition," etc.; the paper
has to provide empirical evidence that proximity is actually capturing
value-related behavior rather than model style, translation artifacts,
or something else.

> **Figure to print: Figure 1** is useful here as a preview of the
> learned space, although I would explain how it is constructed before
> interpreting the clusters.

## How should we think about "extracting values" from an LLM?

The paper begins with a **conceptual model, not something they directly
implement**.

Imagine that an LLM's response behavior comes from many different
factors. Some affect value judgments --- e.g., whether the model tends
to favor individual freedom versus collective responsibility --- while
others affect things like wording, syntax, knowledge, or writing style.

Ideally, we would like a representation that captures the first category
while ignoring the second.

The authors formalize these as latent **value-related factors** and
**other factors** inside an LLM. This is mainly a way of stating the
goal: these factors are not known variables or identifiable groups of
parameters that the authors locate inside the network. They explicitly
point out that we do not know how value-related behavior is encoded
among billions of parameters, and for closed models we do not have the
parameters anyway.

**What they actually do instead:** observe the model's behavior. They
ask many questions designed to make value preferences relevant to the
answer, then train a separate embedding model to capture the patterns
that are consistent across those answers.

So the logic is:

**latent values are inaccessible → value-sensitive behavior is
observable → learn a representation from that behavior.**

## Where do the value questions come from?

The quality of the representation depends heavily on what questions we
decide are "value-eliciting."

The authors start with **87 reference values drawn from several
established human-value frameworks**, including the World Values Survey,
Hofstede/cultural-dimensions work, Schwartz's value theories, and the
Rokeach Value Survey.

These are fairly broad concepts rather than 87 highly specific moral
rules. Examples given in the paper include:

  -----------------------------------------------------------------------
  Reference value                     Roughly what it probes
  ----------------------------------- -----------------------------------
  Individualism vs. Collectivism      independence/personal goals
                                      vs. interdependence/group goals

  Harmony vs. Mastery                 fitting into/harmonizing with the
                                      world vs. actively
                                      changing/mastering it

  Performance vs. Humane Orientation  achievement/performance
                                      vs. compassion and concern for
                                      others

  Affective Autonomy                  freedom to pursue personally
                                      rewarding experiences and desires
  -----------------------------------------------------------------------

For each reference value, the authors first use GPT-4 to generate
situations and then Mixtral to turn them into many concrete questions.
For **Individualism vs. Collectivism**, examples include whether one
prioritizes independence or interdependent relationships and whether
credit for a successful outcome should be shared or taken individually.
For **Affective Autonomy**, one example asks whether protecting one's
mental well-being should take precedence over meeting societal
expectations.

This distinction is important:

-   the **87 values are seeds used to generate a broad set of
    situations/questions**;
-   UniVaR does **not** ultimately output an 87-dimensional vector with
    one score for each value.

After filtering, they retain **4,296 distinct English questions**. Each
is paraphrased four times, giving **21,480 formulations**. They
translate the questions into the languages supported by each LLM,
collect answers, and translate the QA pairs back into English.

The final training set contains roughly **1 million QA pairs**.

### Why paraphrase and translate back to English?

Paraphrasing makes it harder for UniVaR to associate a value with one
particular wording.

Back-translation has a different purpose. Suppose responses originally
produced in Japanese cluster together. If UniVaR actually receives
Japanese text, that result is uninteresting: it could simply recognize
the language. By translating all QA pairs back to English, the authors
try to force the representation to rely on **differences in what the
models say**, rather than the language's vocabulary or script.

This introduces another possible confound --- translated English may
retain clues about its source language ("translationese") --- which the
authors test later.

> **Figure 3** is useful if we want the data-generation pipeline visible
> during discussion.

## Models: what variation are they trying to capture?

The study covers **15 chat/instruction-following LLMs and 25
languages**, producing 127 supported model-language combinations.

The models are deliberately heterogeneous. They include:

-   general multilingual models such as **Aya 101**;
-   region/language-focused models such as **SeaLLM**, **ChatGLM-3**,
    and **JAIS**;
-   widely used general-purpose families such as **Mistral/Mixtral,
    Llama, Yi, and ChatGPT**;
-   models with different post-training histories, including
    preference-tuned models and models the paper does not mark as
    preference-tuned.

This variation is useful because we have plausible reasons for some
models to differ: their training data, intended language coverage,
developers, and post-training procedures are not identical.

### The 8 training models vs. 7 unseen models

**Used to train UniVaR:**\
Mixtral Instruct, Aya 101, SeaLLM, BLOOMZ-RLHF, ChatGLM-3, Nous Hermes
Mixtral, SOLAR Instruct, and Mistral Instruct.

**Not used to train UniVaR:**\
JAIS Chat, Yi Chat, Llama 2 Chat, Maral, Command-R, Llama 3, and
ChatGPT.

This is meaningful because UniVaR never sees outputs from those seven
models while learning its embedding space. It can later encode their
answers because it consumes **textual QA pairs, not model parameters**.

The two sides are not completely unrelated populations: they contain
broadly similar modern instruction/chat LLMs, and some
architectures/families in the overall set are related. The training side
itself also contains especially close relatives --- Mixtral Instruct and
Nous Hermes Mixtral, for example. However, the held-out set includes
genuinely different model families and organizations.

A useful interpretation is: **does a value-sensitive text representation
learned from one collection of LLM outputs remain useful when we feed it
outputs from new LLMs?**

## A model in different languages counts as different "value identities"

The paper does not assume that ChatGPT has one value representation
regardless of language.

Instead:

-   ChatGPT answering in English = one value identity
-   ChatGPT answering in Chinese = another
-   Aya answering in English = another
-   Aya answering in Chinese = another

This choice is based on previous evidence that LLM behavior changes with
prompting language.

It also means that one of the paper's central questions is built into
the setup from the beginning: **how much does the expressed value
behavior of the same model change when we interact with it in another
language?**

## How UniVaR is trained: multi-view learning

The high-level problem is that **one answer tells us very little about a
model's overall values**.

Imagine randomly taking several value-related answers from
ChatGPT-in-English. Call that one sample. Then independently take
several *different* value-related answers from ChatGPT-in-English. Call
that another sample.

Although the questions differ, both samples come from the same model in
the same language. The authors train UniVaR to place these two samples
near each other.

At the same time, a sample from a different model-language combination
--- for example ChatGPT-in-Chinese or Aya-in-English --- is encouraged
to be farther away.

Repeated over many questions and model-language combinations, the hope
is that the encoder learns the patterns that are **consistent across
many different value judgments from the same source**, rather than
memorizing the content of one question.

This is **multi-view learning**: show the model different "views" of the
same underlying thing and train it to recognize what they have in
common.

### Two implementation terms in the paper

-   **Nomic Embed v1** is the pretrained text-embedding model they start
    from. Rather than training an encoder from scratch, they take an
    existing model that already turns text into useful vectors and
    fine-tune it for this specialized task.
-   **InfoNCE** is the contrastive training objective. Intuitively:
    *pull matching views together in embedding space and push
    non-matching views apart.*

Neither is the conceptual contribution; they are standard tools used to
implement the value-representation idea.

> **Figure 2** is probably the single best methods figure to print.

## Why might this capture values?

Because the matched samples contain **different questions and answers**,
exact topic and wording are unreliable ways to recognize the pair. What
should be more consistent is the model-language combination's broader
pattern of responses across value questions.

But model style, refusal tendencies, or translation artifacts could also
be consistent. The paper therefore needs controls showing that UniVaR is
especially sensitive to **value-related** differences.

## Does UniVaR actually contain value-related information?

The main quantitative task is **value identification**: given
value-related QA text, can its UniVaR embedding identify which
model-language combination produced it?

UniVaR substantially outperforms generic word/sentence embeddings:

  Representation                k-NN accuracy   Linear Acc@10
  --------------------------- --------------- ---------------
  GloVe                                 2.27%          27.72%
  BERT                                  1.78%          42.20%
  LaBSE                                 4.03%          47.48%
  **UniVaR (best setting)**        **20.37%**      **61.70%**

The useful comparison is not that 20% is intrinsically high. It is that
**a representation specifically trained on value-eliciting behavior
separates model-language sources far better than generic semantic
embeddings do**.

They also evaluate on value questions from four external sources ---
PVQ-RR, WVS, GLOBE, and ValuePrism --- rather than simply reusing the
generated training questions. The advantage persists even for GLOBE and
ValuePrism, whose values overlap less with the sources used to generate
the training data.

## Could UniVaR just be recognizing the model?

This is the most important sanity check.

ChatGPT, Llama, etc. have recognizable response styles. If UniVaR
identifies "ChatGPT-English" because of phrases, formatting, refusals,
or other stylistic habits, then calling it a **value representation**
would be misleading.

The authors therefore repeat source identification using **non-value
questions from LIMA**, such as programming/informational questions.
UniVaR's ability to distinguish the sources drops substantially.

They separately test whether UniVaR can identify which language an
English sentence was translated from. It is worse at this than the
generic embedding baselines, suggesting that source-language translation
artifacts are not the main thing it has learned.

Neither test proves that the remaining signal is "pure values." But they
make two simple alternative explanations --- generic model
fingerprinting and translationese --- considerably less plausible.

> **Figure 4** is worth printing because the interpretation of almost
> every later result depends on this sanity check.

## The most interesting result: models cluster strongly by language

When the authors project UniVaR embeddings into two dimensions,
responses elicited in the **same language often cluster together even
when they come from different LLMs**.

They highlight patterns such as:

-   Chinese / Japanese / Korean being relatively close;
-   German / French / Spanish being relatively close;
-   Indonesian / Malay / Arabic being relatively close.

The authors compare this with the **Inglehart-Welzel World Cultural
Map**, derived from human World Values Survey responses, and find
qualitatively similar groupings.

This is particularly interesting because UniVaR receives the QA pairs
**after they have been translated back into English**. Therefore, the
clusters cannot simply be "these strings are all written in Chinese."
Something about the answers elicited through Chinese survives
translation and is shared across models.

> **Figure 5** is probably the best results/discussion figure to print.

### Aya and JAIS are interesting exceptions

For many models, switching languages moves their responses substantially
in UniVaR space. Aya and JAIS show more cross-language similarity.

Both have multilingual/translated-data-oriented training histories,
which raises an interesting hypothesis: **training heavily across
languages or on parallel/translated data may make a model's
value-related behavior more language-invariant.**

The paper does not experimentally isolate training data as the cause, so
this should be treated as a hypothesis rather than a demonstrated
mechanism.

## Do these mappings actually mean what the authors say they mean?

This is where I think the paper is most interesting to discuss.

### What seems reasonably well supported

There is a **stable behavioral signal in answers to value-related
questions** that UniVaR can learn and that generalizes beyond the eight
LLMs used to train the encoder.

That signal is not easily explained by ordinary semantic similarity,
generic model style, or obvious source-language artifacts.

There is also convincing evidence that **prompting language
systematically changes value-related LLM responses**. The fact that
different models prompted in the same language often become closer is a
real and interesting behavioral result.

### What is not established

**1. UniVaR does not show that it has recovered a model's internal "true
values."**

The authors never observe the latent value factors from their motivating
formulation. UniVaR learns from behavior on questions researchers have
designated as value-related. I think the safest description is therefore
**a representation of value-elicited behavior**, rather than a direct
measurement of values stored inside the network.

**2. A language cluster is not automatically a cultural cluster.**

Language is being used as a proxy for culture, but English, Arabic,
Spanish, etc. each span many societies. A model may also behave
differently across languages because of differences in pretraining data,
instruction tuning, safety data, translation quality, or model
competence.

Therefore, "responses elicited in the same language show similar
value-related patterns, and some patterns resemble human cultural survey
groupings" is supported much better than "LLMs learn the culture
associated with each language."

**3. The geometry is learned from a particular definition of what counts
as a value question.**

The authors avoid forcing the final representation into a small
predefined taxonomy. But the training questions are still seeded from 87
values collected from existing human-value frameworks.

So UniVaR is **less constrained by a taxonomy, not independent of one**.

**4. The training objective itself does not know what a value is.**

It knows that two sets of answers came from the same model-language
source and should be close. Calling the learned common signal "values"
depends on the design of the questions and on the controls showing that
obvious non-value signals have been reduced.

This is why I find the non-value and translationese experiments more
important than the raw classification accuracy.

**5. The 2D map is suggestive, not conclusive evidence.**

UMAP is a visualization of a much higher-dimensional space. Exact
distances and global geometry in the 2D figure should not be
overinterpreted. The resemblance to the World Cultural Map is compelling
visually, but it is not itself a statistical demonstration that UniVaR
has rediscovered human cultural structure.

## My takeaways / questions

-   **The measurement idea is useful even if "value representation" is
    too strong a name.** A reusable encoder for comparing value-related
    behavior across models, languages, and checkpoints could be valuable
    without claiming that the vector corresponds to a model's internal
    value system.
-   I find the **cross-language result more convincing than the culture
    interpretation**. The experiments give good evidence that prompting
    language changes normative behavior; they give weaker evidence about
    *why*.
-   The most interesting next experiment would be controlled training:
    take the same base model and vary only multilingual data,
    instruction tuning, or preference tuning. Then UniVaR could help ask
    **which training stage causes the language-conditioned shifts**.
-   Another strong test would be intervention: deliberately change a
    known value preference during fine-tuning and ask whether UniVaR
    moves in the expected direction while unrelated behaviors stay
    fixed. The appendix contains an initial DPO value-transfer
    experiment along these lines, but this could be made much more
    controlled.
-   If two models are nearby in UniVaR space, **what predictions should
    that let us make about their behavior on unseen dilemmas?**
    Predictive validity would make the geometry much more convincing
    than visual similarity alone.
-   Finally, is language-conditioned value variation desirable? A model
    giving culturally/contextually appropriate answers in different
    languages may be useful. But if the variation comes from uneven
    alignment quality or stereotypes in training data, the same
    phenomenon could be undesirable.

## Figures I would print

If printing **three**:

1.  **Figure 2 --- UniVaR overview:** method.
2.  **Figure 4 --- value vs. non-value identification:** most important
    sanity check.
3.  **Figure 5 --- UniVaR vs. World Cultural Map:** main result and best
    discussion figure.

If there is room for a fourth, add **Figure 3** for the QA-generation
pipeline.

## Takeaway

UniVaR is a **separate embedding model** trained on many value-eliciting
responses from LLMs. Once trained, it can take value-related QA text
from a new LLM and place it in the same learned space, allowing
comparisons across models and languages.

The strongest finding is that **the language used to prompt an LLM is
systematically associated with the value-related behavior it
expresses**, often strongly enough that different models prompted in the
same language cluster together.

The paper makes a reasonable case that this is not simply language
recognition, translation artifacts, or generic model style. What remains
less certain is whether the resulting geometry should literally be
interpreted as a map of "human values" or "cultures." I would view
UniVaR first as a promising behavioral measurement tool; establishing
exactly what its distances mean is the more difficult open problem.
