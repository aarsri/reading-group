# High-Dimension Human Value Representation in Large Language Models

_Samuel Cahyawijaya, Delong Chen, Yejin Bang, and others at NAACL 2025_

_Presented by Aarohi Srivastava on 9/11/26_

## Motivation

-   LLM alignment work usually asks how to *put* values into a model.
    This paper asks a complementary question: **once a model has been
    trained, how can we characterize the values it actually expresses?**
-   A common approach is to administer an existing human-values survey
    to an LLM and report its position along the survey's predefined
    dimensions.
    -   Examples include Hofstede's cultural dimensions (6 dimensions),
        the World Values Survey (10 dimensions in the framing used
        here), and Schwartz-style value taxonomies.
    -   This is interpretable, but it also constrains the analysis to
        whatever axes the survey designers decided in advance.
-   The paper's central proposal is **UniVaR (Universal Value
    Representation)**: instead of representing an LLM with a small
    hand-designed vector of survey scores, learn a high-dimensional
    embedding from many value-relevant model responses.
-   The intended analogy is to ordinary representation learning:
    -   a sentence embedding tries to retain information about sentence
        meaning while ignoring irrelevant surface details;
    -   UniVaR tries to retain information about a model's **value
        tendencies** while ignoring things like wording, syntax,
        model-specific style, and the language in which the answer was
        originally produced.
-   Importantly, UniVaR is **not supposed to tell us whether a model's
    values are good or bad**. It is a measurement/representation tool:
    models (or model-language combinations) that express similar value
    tendencies should be close in the embedding space.

> **Figure to print: Figure 1.** This is probably the best result figure
> to have in color. It gives the intuitive payoff of the whole method
> before getting into how the representation is learned.

## What exactly is being represented?

The paper conceptually decomposes the factors governing an LLM's
behavior into

$$
\theta = \phi(\vartheta_{\text{value}}, \vartheta_{\text{other}})
$$

where:

-   $\vartheta_{\text{value}}$ = latent factors that affect
    value-relevant decisions;
-   $\vartheta_{\text{other}}$ = everything else that affects an answer,
    such as linguistic style, syntax, general knowledge, reasoning
    ability, etc.

We obviously cannot literally invert a neural network and isolate
$\vartheta_{\text{value}}$. This would also be impossible for closed
models whose parameters are unavailable.

Instead, UniVaR learns a representation $Z$ intended to contain
information about $\vartheta_{\text{value}}$ while discarding as much
irrelevant information as possible:

$$
\max_Z I(\vartheta_{\text{value}}; Z) - H(Z).
$$

The equation is mainly a statement of intent rather than something they
can optimize directly, because $\vartheta_{\text{value}}$ is unobserved.
The practical question becomes: **what observable data should contain
information about a model's values, and what learning signal can
separate that information from nuisance factors?**

## Step 1: Elicit values through questions

-   Not every model output tells us much about values.
    -   "What is $17 \times 24$?" mostly probes knowledge/reasoning.
    -   A question about whether individual freedom should outweigh a
        public-health obligation plausibly depends much more on value
        judgments.
-   The authors therefore construct a large collection of
    **value-eliciting questions**.
-   They begin with **87 reference values** collected from five
    established families of human-value research, including the World
    Values Survey, Hofstede, Schwartz's value theories, and the Rokeach
    Value Survey.
    -   The 87 values are used to obtain broad coverage; they are **not
        the dimensions of the final UniVaR embedding**.
    -   This distinction matters. UniVaR is advertised as escaping a
        small fixed taxonomy, but the training questions are still
        seeded from existing taxonomies. The method is therefore less
        constrained than reporting 6 or 10 survey scores, but it is not
        taxonomy-free.
-   An LLM generates 50 candidate questions for each reference value.
    After manual filtering, the authors retain **4,296 questions**.
-   Each is paraphrased four times, giving **21,480 English questions**.
-   These are translated into **25 languages**, asked to the LLMs, and
    the resulting question-answer pairs are translated **back into
    English** before representation learning.
-   Overall, the training pipeline produces roughly **1 million QA
    pairs**.

### Why translate everything back to English?

Suppose a Chinese answer and an English answer end up far apart in the
embedding. Without normalization, we would not know whether the distance
reflects:

1.  different values, or
2.  the trivial fact that one response is Chinese and the other is
    English.

Back-translation is an attempt to remove (2), so that the representation
cannot simply use language identity as a shortcut.

This creates a methodological tradeoff: translation removes an obvious
linguistic confound, but it can also introduce **translationese** or
erase culturally meaningful distinctions that are expressed through
language. The authors explicitly test the first problem later.

> **Figure to print: Figure 3.** Useful if the group will want the full
> data-generation pipeline. Otherwise Figure 2 is the more important
> methods figure.

## A "value identity" is a model-language pair

One of the most important choices in the paper is that the authors **do
not assume a model has one fixed value system independent of language**.

For example:

-   ChatGPT answering in English = one value identity;
-   ChatGPT answering in Chinese = another value identity.

Across the models/languages that are supported, this produces **127
distinct model-language pairs**.

The motivation is empirical work showing that multilingual LLM behavior
can change with prompting language. Conceptually, this means UniVaR
represents **values as expressed under a particular linguistic
context**, rather than claiming to recover one immutable set of values
stored inside the model.

This distinction is useful when interpreting the paper's later claim
that language clusters correspond to cultures: the evidence is about
**behavior elicited through languages**, not direct access to an
internal cultural identity.

## Step 2: Multi-view learning

A single value question is noisy. An answer might reveal the model's
stance on one issue, but it cannot characterize its broader value
distribution.

So the input to UniVaR is a **view**: a randomly sampled set of
$\lambda$ value-eliciting QA pairs from the same model-language
identity.

During training:

1.  Sample two different sets of QA pairs, $X_1$ and $X_2$, from the
    **same model-language pair**.
2.  Encode both using the same encoder $g$: $$
    Z_{X_1}=g(X_1), \qquad Z_{X_2}=g(X_2).
    $$
3.  Train the representations of these two views to be similar.
4.  Representations from different model-language identities serve as
    negatives.

The encoder is initialized from **Nomic Embed v1 (137M parameters)**,
and the paper uses an **InfoNCE contrastive loss**.

### Why should this isolate values?

The key assumption is that two randomly sampled sets of value questions
from the same model-language pair share the underlying value tendencies,
but do **not** necessarily share the same topic, exact wording, or
individual answers.

Therefore, the easiest information that is consistently useful for
matching arbitrary views should be information stable across many
value-eliciting responses from that model-language pair.

This is the same general logic behind contrastive/multi-view
representation learning: construct two observations that share the
factor you care about while varying nuisance factors, then train the
model to recover what is common.

There is an important caveat: **anything else that is stable within a
model-language pair can also become useful for the contrastive task**.
Model-specific prose style, refusal behavior, translation artifacts, or
recurring lexical choices could all be shortcuts. This is why the
paper's confounder tests are not optional side analyses; they are
necessary evidence for the central interpretation of the embedding.

> **Figure to print: Figure 2.** I would definitely include this one.
> The right side makes the multi-view idea much easier to explain than
> the equations alone.

## Training vs. evaluation

-   The authors study **15 LLMs** total.
-   QA outputs from **8 LLMs** are used to train UniVaR.
-   The remaining **7 LLMs are unseen during UniVaR training**, which is
    important: the representation is intended to generalize beyond the
    particular generators it was trained on.
-   Evaluation questions also come from sources not directly used to
    create the training questions:
    -   PVQ-RR
    -   World Values Survey
    -   GLOBE
    -   ValuePrism
-   These sources are converted into natural questions, translated into
    the target languages, answered by the LLMs, and translated back to
    English.

So the strongest evaluation is not "can the embedding remember the
training questions?" It asks whether a learned notion of model-language
value identity transfers to **new value questions and unseen models**.

## Does UniVaR actually contain value-related information?

The main quantitative evaluation is **value identification**.

Given an embedding of value-eliciting QA(s), can a simple classifier
identify which model-language value identity generated them?

The reasoning is:

-   if different model-language pairs systematically express different
    values;
-   and UniVaR captures those differences;
-   then their representations should be distinguishable.

This is tested with both k-nearest neighbors and a linear probe on
frozen embeddings.

### Main result

  Representation               k-NN accuracy   Linear Acc@10
  -------------------------- --------------- ---------------
  Random                               0.78%            7.8%
  GloVe                                2.27%          27.72%
  BERT                                 1.78%          42.20%
  RoBERTa                              1.88%          41.17%
  LaBSE                                4.03%          47.48%
  **UniVaR ($\lambda=1$)**        **18.68%**      **57.98%**
  **UniVaR ($\lambda=5$)**        **20.37%**      **61.70%**

The absolute classification accuracy is not enormous, but the comparison
to ordinary semantic embeddings is the more informative result. UniVaR
is learning something substantially more diagnostic of the
model-language source on value questions than generic sentence
similarity alone.

### An interesting result: more context is not always better

They train UniVaR with view sizes $\lambda\in\{1,5,20,80\}$.

Performance peaks at **$\lambda=5$**, then declines:

-   $\lambda=5$: 20.37% k-NN accuracy
-   $\lambda=20$: 19.99%
-   $\lambda=80$: 18.01%

The authors suggest the very wide dynamic range of view sizes may make
the model underfit the single-QA case. I would not interpret this as
evidence that five questions are intrinsically sufficient to identify a
model's values. It is better viewed as an optimization/generalization
property of this particular training setup.

## But is the classifier just recognizing the model's style?

This is probably the most important methodological sanity check in the
paper.

If UniVaR can identify "ChatGPT-English" because ChatGPT has a
recognizable writing style, then high value-identification accuracy
would not demonstrate that it learned values.

The authors therefore feed UniVaR **non-value-eliciting questions from
LIMA**, such as programming questions. If the embedding is genuinely
specialized for values, model identification should become much harder
when the answer does not require a value judgment.

That is what they observe: source-identification performance drops
substantially for non-value questions. They also separately test
translationese and report that UniVaR retains less translation-origin
information than the baseline representations.

This does not prove the representation contains *only* values, but it
rules out the simplest alternative explanation that UniVaR is merely a
model/style fingerprint.

> **Figure to print: Figure 4.** I would include this. It supports the
> interpretation of the entire method, not just an auxiliary result.

## What does the learned value space look like?

The authors project UniVaR embeddings into two dimensions with UMAP.

The striking pattern is that **responses in the same language tend to
cluster together even when they come from different LLMs**.

Examples the paper highlights:

-   Chinese / Japanese / Korean are relatively close;
-   German / French / Spanish are relatively close;
-   Indonesian / Malaysian / Arabic are relatively close;
-   English is relatively separated from several continental European
    languages.

The authors compare this organization with the **Inglehart-Welzel World
Cultural Map** and argue that the geometry resembles known cultural
groupings.

### Why this result is interesting

Remember that all QAs have already been translated back to English
before UniVaR sees them. Therefore, a simple explanation like "Japanese
strings cluster because they contain Japanese tokens" is unavailable.

If the pipeline is working as intended, the remaining clustering must
come from systematic differences in the **content of the answers**
produced when the models were originally prompted in different
languages.

This is one of the paper's strongest high-level findings: **prompting
language appears to be associated with a reproducible shift in the
value-relevant behavior of LLMs, and that shift can dominate model
identity.**

### But "language = culture" is too strong

The authors often use language as a proxy for culture. This is
convenient experimentally, but the mapping is obviously imperfect:

-   English is used across many culturally different societies.
-   Arabic spans many countries and communities.
-   Multilingual speakers do not acquire a new culture simply by
    switching languages.
-   Training-data composition and model alignment can create
    language-conditioned behavior that does not faithfully represent
    human speakers of that language.

So I would phrase the result as:

> **Model responses elicited in the same language show similar
> value-related patterns, and the geometry of several language groups
> resembles patterns in human cultural surveys.**

That is well supported. "The model has learned the culture associated
with each language" is a stronger causal/representational claim than the
experiments establish.

> **Figure to print: Figure 5.** Definitely worth printing in color. The
> side-by-side comparison with the World Values Survey map is one of the
> most discussion-worthy figures.

## A particularly useful exception: Aya and JAIS

The general trend is that **language matters strongly**: different
languages within a model often occupy different regions of UniVaR space.

However, the paper notes that **Aya and JAIS show unusually similar
values across their languages**. Both were trained with substantial
amounts of translated/multilingual data.

This is interesting because it suggests that multilingual training
strategy can change the relationship between language and expressed
values. One plausible interpretation is that heavy use of
parallel/translated data encourages a more language-invariant response
distribution, including on value questions.

The paper does not establish this causally, so I would treat it as a
hypothesis rather than a conclusion. But it points to a useful future
experiment: hold architecture and alignment constant while varying the
amount/type of translated training data, then measure whether
cross-language value distances shrink.

## Can distance in UniVaR space be interpreted semantically?

The authors inspect pairs of nearby and distant embeddings.

One example compares vaccination responses:

-   **ChatGPT-English** emphasizes individual liberty/choice;
-   **ChatGPT-Chinese** emphasizes social responsibility.

These embeddings are relatively far apart.

Conversely, nearby **ChatGPT-French** and **Mixtral-German** responses
to a question about tracking a criminal's IP address both emphasize
rule-of-law considerations.

This is helpful because it gives a concrete meaning to "distance" in the
learned space: at least in selected examples, proximity corresponds to
similar normative reasoning and distance corresponds to contrasting
priorities.

The important limitation is that UniVaR dimensions themselves are **not
directly labeled or interpretable**. We can compare positions and
distances, but unlike a 10-dimensional survey vector, we cannot say
"dimension 37 = collectivism." High-dimensional learned representations
trade some direct interpretability for expressive capacity.

> **Figure to print: Figure 6** if you want one qualitative example to
> make distances concrete. If space is limited, I would prioritize
> Figures 2, 4, and 5 over it.

## What I think the paper establishes

1.  **Generic semantic embeddings are not enough for this task.** A
    representation trained specifically across many value-eliciting
    outputs carries much more information about model-language value
    identity.
2.  **Value-relevant behavior changes with prompting language.** Across
    many models, language is a surprisingly strong organizer of the
    learned value space.
3.  **The effect is not trivially reducible to input/output language**,
    because the representation is trained and evaluated on QAs
    translated back to English.
4.  **The representation is not trivially just a model-style
    classifier**, because its source-identification ability drops
    strongly on non-value questions.
5.  **Cross-language geometry resembles some known human cultural
    groupings**, although this should be interpreted as correspondence
    rather than proof that LLMs faithfully contain those cultures.
6.  **Training choices may affect cross-language value consistency.**
    Aya and JAIS provide suggestive evidence that translation-heavy
    multilingual training can make expressed values more similar across
    languages.

## Methodological considerations / things I would be careful about

### 1. UniVaR does not discover values from nothing

The paper motivates UniVaR as an alternative to low-dimensional
predefined taxonomies, but the questions used to train it are seeded
from **87 values taken from existing taxonomies**.

The distinction is:

-   traditional survey approach: the taxonomy defines the *output
    coordinates*;
-   UniVaR: the taxonomy helps define the *elicitation distribution*,
    while the learned embedding is free to organize responses in a much
    higher-dimensional way.

So UniVaR is more flexible, but its notion of "value-relevant behavior"
is still shaped by the questions researchers chose to ask.

### 2. Value identity is operationalized as model × language

The contrastive objective is trained to make views from the same
model-language pair similar and different pairs dissimilar. This is a
clever source of self-supervision, but it means that "value" is not
independently labeled during representation learning.

The authors' confounder experiments provide evidence that the learned
signal is value-related. Still, the representation should be understood
as **a learned model-language fingerprint specialized to value-eliciting
behavior**, rather than a ground-truth measurement of an inaccessible
latent variable.

### 3. Translation is both a control and an intervention

Back-translating everything to English is a strong control against
superficial language clustering. At the same time, translation may:

-   normalize distinctions that matter culturally;
-   introduce systematic artifacts;
-   perform differently across languages.

The paper checks translationese, which is reassuring, but no translation
pipeline can guarantee perfect preservation of pragmatic or culturally
specific meaning.

### 4. UMAP is evidence for structure, not the structure itself

The colorful 2D maps are compelling, but UMAP is a nonlinear
dimensionality-reduction method. Local neighborhoods are generally more
trustworthy than exact global distances or axis positions.

The stronger evidence is therefore the combination of:

-   quantitative identification results;
-   confounder tests;
-   robustness across four evaluation corpora;
-   qualitative inspection of nearby/distant examples;

rather than the 2D visualization alone.

### 5. Similarity to human cultural maps needs careful interpretation

A language cluster resembling a WVS cultural cluster is interesting
validation, but there are multiple possible causal routes:

-   pretraining data written by speakers of those languages;
-   multilingual alignment/RLHF data;
-   translation data;
-   language-conditioned prompting effects;
-   model safety policies;
-   artifacts of the value questions or translation pipeline.

UniVaR measures the resulting behavior; it does not identify which
training stage caused it.

## Questions for discussion

1.  Is **model × language** the right unit for a "value identity," or
    should we instead condition on country, dialect, persona, or
    explicit cultural context?
2.  How much of a model's value behavior comes from pretraining versus
    instruction tuning / preference optimization?
3.  Could we train UniVaR on the *same model before and after RLHF/DPO*
    and use movement in embedding space to measure exactly what
    alignment changed?
4.  If a model gives culturally different answers in different
    languages, is that desirable pluralism, undesirable inconsistency,
    or context-sensitive behavior we actually want?
5.  The paper removes language information through back-translation.
    Could a multilingual representation model remove surface language
    while preserving culturally meaningful pragmatics **without
    translating**?
6.  What should count as evidence that a learned representation is
    genuinely a "value representation" rather than a specialized
    behavioral fingerprint? What additional negative controls would we
    want?
7.  Can UniVaR eventually be made interpretable enough to say *which*
    values changed, rather than only that two distributions are far
    apart?

## Figures I would print

If printing only **three** figures:

1.  **Figure 2 --- UniVaR overview:** best explanation of the method.
2.  **Figure 4 --- value vs. non-value identification:** crucial sanity
    check against model/style confounding.
3.  **Figure 5 --- UniVaR map vs. human cultural map:** clearest
    substantive result and likely the best discussion figure.

If there is room for more:

-   **Figure 1** --- visually striking overview of the full learned map.
-   **Figure 3** --- useful for explaining exactly where the \~1M QA
    pairs come from.
-   **Figure 6** --- concrete qualitative interpretation of near/far
    embeddings.
-   **Figure 7** --- useful evidence that the broad clustering pattern
    is robust across the four different evaluation corpora.

## Takeaway

The paper's most useful idea is not simply that LLMs have different
"values." It is that **value-related behavior can be treated as a
representation-learning problem**.

By repeatedly eliciting normative judgments, constructing multiple views
of the same model-language behavior, and contrastively learning what
remains stable across those views, UniVaR produces a representation that
is much more sensitive to value-relevant differences than ordinary
semantic embeddings.

The resulting space suggests a striking empirical pattern: **for many
multilingual LLMs, the language used to interact with the model is
strongly associated with the values it expresses, often strongly enough
that responses group by language across different model families.** At
the same time, models trained heavily on translated/multilingual data
provide an interesting exception, suggesting that this relationship is
partly a property of training rather than an unavoidable property of
language itself.

The main conceptual caution is that UniVaR does not directly recover a
model's latent "true values." It learns a compact representation of
**observable, value-elicited behavior** under a particular experimental
pipeline. That is still useful: it gives us a scalable way to compare
models, languages, and potentially alignment interventions without
forcing every comparison into a small predefined set of survey
dimensions.

------------------------------------------------------------------------

Paper: Cahyawijaya et al., *High-Dimension Human Value Representation in
Large Language Models*, NAACL 2025.\
https://aclanthology.org/2025.naacl-long.274/
