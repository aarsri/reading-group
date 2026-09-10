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

### What do UniVaR representations look like?

Suppose we want to compare Model A and Model B. We ask both models
questions such as:

-   Is individual success more important than the welfare of the
    community?
-   Should someone prioritize personal well-being over societal
    expectations?
-   How important is it to preserve cultural traditions?

Model A's answers might consistently emphasize personal choice and
autonomy, while Model B may emphasize social obligation and harmony.

Each model's question-answer pairs can be transformed into a value representation by UniVaR. LLM response behavior comes from many different
factors; some affect value judgments, while
others affect things like wording, syntax, knowledge, or writing style. UniVaR aims to capture the first category
while ignoring the second. The authors formalize these as latent **value-related factors** inside an LLM. 

After UniVaR has been trained, the encoder is static and can be applied to outputs from a new
model. If the learned
representations are meaningful, Model A's value-related answers should
occupy one region of the embedding space and Model B's should occupy another.

An interesting question is what distance/spatial relationships in the embedding space *mean*. The
dimensions are not labeled "individualism," "tradition," etc. as they are learned; the paper provides some empirical evidence that proximity is actually capturing
value-related behavior rather than model style, translation artifacts, or something else.

## Where do the value questions come from?

The quality of the representation depends heavily on what questions we
decide are value-eliciting.

The authors start with 87 reference values drawn from several
established human-value frameworks, including the World Values Survey,
Hofstede cultural-dimensions, Schwartz's value theories, and the
Rokeach Value Survey.

These are fairly broad concepts rather than 87 highly specific moral
rules. Examples given in the paper include:

| Reference value | Meaning |
| :---     | :---     |
| Individualism vs. Collectivism | independence/personal goals vs. interdependence/group goals |
| Harmony vs. Mastery | fitting into/harmonizing with the world vs. actively changing/mastering it |
| Performance vs. Humane Orientation | achievement/performance vs. compassion and concern for others |
| Affective Autonomy | freedom to pursue personally rewarding experiences and desires |

For each reference value, the authors first use GPT-4 to generate
situations and then Mixtral to turn them into many concrete questions.
For *Individualism vs. Collectivism*, examples include whether one
prioritizes independence or interdependent relationships and whether
credit for a successful outcome should be shared or taken individually.
For *Affective Autonomy*, one example asks whether protecting one's
mental well-being should take precedence over meeting societal
expectations.

This distinction is important:

-   the 87 values are seeds used to generate a broad set of
    situations/questions;
-   UniVaR does **not** ultimately output an 87-dimensional vector with
    one score for each value.

After filtering, they retain **4,296 distinct English questions**. Each
is paraphrased four times, giving **21,480 formulations**. The purpose of paraphrasing is to make it harder for UniVaR to associate a value with one
particular wording. 

They translate the questions into the languages supported by each LLM,
collect answers, and translate the QA pairs back into English. By translating all QA pairs back to English, the authors
try to force the representation to rely on differences in what the
models say, rather than the language's vocabulary or script. The final training set contains about **1 million QA pairs**.

Note that this unit of this study is **model-language pairs**. This choice is based on previous evidence that LLM behavior changes with
prompting language. It also means that one of the paper's central questions is built into
the setup from the beginning: **how much does the expressed value
behavior of the same model change when we interact with it in another
language?** Of course, the representations operate on the back-translated English versions, but the LLM QA interaction was in a specific non-English language.

## Models Used

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

### 8 training models vs. 7 unseen models

**Used to train UniVaR:**\
Mixtral Instruct, Aya 101, SeaLLM, BLOOMZ-RLHF, ChatGLM-3, Nous Hermes
Mixtral, SOLAR Instruct, and Mistral Instruct.

**Not used to train UniVaR:**\
JAIS Chat, Yi Chat, Llama 2 Chat, Maral, Command-R, Llama 3, and
ChatGPT.

This is meaningful because UniVaR never sees outputs from those seven
models while learning its embedding space. It can later encode their
answers because it consumes **textual QA pairs, not anything model-specific**. This allows them to answer: **does a value-sensitive text representation
learned from one collection of LLM outputs remain useful when we feed it
outputs from new LLMs?** Of course, the types of models represented are similar in the seen vs. unseen groups.

## How UniVaR is trained: multi-view learning

The high-level problem is that *one QA pair tells us very little about a
model's overall values*. A "view" is a single QA pair, but the UniVaR representation is meant to capture a model-language pair (aggregating patterns from several QA pairs eliciting that value). This is **multi-view learning**: show the model different "views" and train it to recognize what they have in common.

During training:

1.  Sample two different sets of QA pairs, $X_1$ and $X_2$, from the
    **same model-language pair**.
2.  Encode both using the same encoder.
3.  Train the representations of these two views to be similar.
4.  Representations from different model-language identities serve as
    negatives.

**Nomic Embed v1** is the pre-trained text-embedding model they start
from. Rather than training an encoder from scratch, they take an
existing model that already turns text into useful vectors and
fine-tune it for this specialized task.

**InfoNCE** is the contrastive training objective. Intuitively:
*pull matching views together in embedding space and push
non-matching views apart.*

> Include Figure 2

## Does UniVaR actually contain value-related information?

The main quantitative task is **value identification**: Can a classifier correctly assign values on the basis of UniVaR embeddings?

UniVaR embeddings yield substantially higher value classification scores than generic word/sentence embeddings, indicating that UniVaR embeddings do represent values.

| Representation | kNN accuracy | Linear Acc@10 |
| :---     | :---     | :---     |
| GloVe    |   2.27%   |   27.72% |
| BERT     |     1.78%    | 42.20% |
| LaBSE   |     4.03%   |  47.48% |
| **UniVaR**  |  **20.37%** | **61.70%** |

### Confounds
The authors consider various confounds including models having distinct styles, translationese, etc. and provide checks to show why their representations have not fallen prey to these confounds.

## Finding: Models cluster strongly by language.

When the authors project UniVaR embeddings into two dimensions,
**responses elicited in the same language often cluster together even
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

> Figure 5

## Finding: Aya and JAIS are interesting exceptions.

For many models, switching languages moves their responses substantially
in UniVaR space. Aya and JAIS show more cross-language similarity.

Both have multilingual/translated-data-oriented training histories,
which raises an interesting hypothesis: **training heavily across
languages or on parallel/translated data may make a model's
value-related behavior more language-invariant.**

The paper does not experimentally isolate training data as the cause, so this is still just a hypothesis.

## What is not established by the results?

**1. UniVaR does not show that it has recovered a model's internal "true
values."**

The authors never observe the latent value factors from their motivating
formulation. UniVaR learns from behavior on questions researchers have
designated as value-related. I think the safest description is therefore
**a representation of elicited behavior**, rather than a direct
measurement of values stored inside the model.

**2. A language cluster is not automatically a cultural cluster.**

In some ways, language is kind of a proxy for culture, but English, Arabic,
Spanish, etc. span many societies. A model may also behave
differently across languages because of differences in pre-training data,
instruction tuning, safety data, translation quality, or model
competence. 

Therefore, responses elicited in the same language show similar
value-related patterns, and some patterns resemble human cultural survey
groupings. However, we can't quite say LLMs learn the culture
associated with each language.

**3. Spatial relationships in the embedding space are determined by a particular definition of what counts
as a value question.**

The authors avoid forcing the final representation into a small
predefined taxonomy, but the training questions are still seeded from 87
values collected from existing human-value frameworks. So UniVaR is less constrained by a taxonomy, but *not independent of one*.

## Discussion

1.  Is **model × language** the right unit for a "value identity," or
    should we instead condition on country, dialect, persona, or
    explicit cultural context?
2.  How much of a model's value behavior comes from pre-training versus
    instruction tuning / preference optimization?
3.  Could we train UniVaR on the *same model before and after RLHF/DPO*
    and use movement in embedding space to measure exactly what
    the post-training changed?
4.  If a model gives culturally different answers in different
    languages, is that desirable pluralism, undesirable inconsistency,
    or context-sensitive behavior we actually want?
