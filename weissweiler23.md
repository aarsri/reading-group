# Counting the Bugs in ChatGPT’s Wugs: A Multilingual Investigation into the Morphological Capabilities of a Large Language Model
_Leonie Weissweiler et al. at EMNLP 2023_

_Presented by Aarohi Srivastava on 5/6/26_

## Wug Test (Jean Berko Gleason, 1958)

> This is a wug. Now there's another one. These are two ___.

> This is a man who zibs. He is a ___.

Child language acquisition experiment that demonstrated that young children do not just memorize words but internalize abstract, systematic grammatical rules.

This paper conducts the Wug Test using ChatGPT (gpt-3.5-turbo-0613) with nonce words from typologically diverse 4 languages: English, German, Tamil,
and Turkish.

## Background

The Wug Test focuses on inflectional morphology (changes to form) rather than derivational morphology (changes role/meaning). 
* Inflection: I listen, she listens. In the past we listened.
* Derivational: I listen, so I am a listener, and the audio is listenable.

Types of morphology:
* Isolating: morphemes are separated
* Fusional: morphemes are fused in a word and may or may not be distinguishable
* Agglutinative: words contain several morphemes which are all distinguishable
* Polysyntetic: word vs. sentence blurred, morphemes are not necessarily distinguishable

Languages used:
* English: almost fully isolating
* German: between isolating and fusional
* Tamil: between fusional and agglutinative
* Turkish: fully agglutinative
In addition to different morphological systems, these languages come from different language families and vary in resourcedness.

## Data Creation

A big part of this work is to develop sets of nonce words in the four languages to be used in the Wug Tests. Data is available here: github.com/dmort27/chatgpts-wugs

### English
* Focus: past tense formation of verbs.
* Starting point: 50 short irregular English verbs from UniMorph, then altered one or two letters to create nonce verbs not found in the dataset.
* Annotation: 
  * Sentence frame: They {nonce} all the time. In fact, they ___ just yesterday.
  * 28 volunteer annotators produced possible past-tense forms.
  * Most common human responses were regular past-tense forms with -ed.

### German
* Focus: plural noun formation, which is complex because German has several competing plural strategies.
* Starting point: Generated 200 nonce nouns from Unipseudo (4-7 characters), then filtered down to 174.
* Annotation: 
  * A native German speaker first assigned each nonce noun an article: der, die, or das, then generated a plural.
  * Sentence frame: Hier ist ein/e {nonce}. Jetzt sind es zwei ___.
  * 21 volunteer annotators provided plural forms.
  * The result was a ranked list of plausible plural forms for each nonce noun.

### Tamil
* Focus: past tense verb inflection
  * Tamil verbs are morphologically rich: they can encode tense, transitivity, person, number, and sometimes gender.
  * The authors simplified the task by focusing on past tense, intransitive verbs, third-person singular masculine agreement.
* Starting point: Sampled 86 common Tamil verbs, generated conjugations, and had two native speakers validate them. Nonce verbs were built by combining syllables from real Tamil verb roots, then checked against a Tamil dictionary to ensure they were not real words.
* Annotation:
  * Five native Tamil speakers supplied past-tense forms; the most common response was treated as the gold answer.
  * Inter-annotator agreement was relatively low, partly because Tamil verb-class assignment can depend on historical/linguistic context.
  * I don't think this one has a sentence frame; the format seems to be {nonce} --> ___ [must be past tense]

### Turkish
* Focus: inflection and reinflection
  * first-person singular agreement + past tense [reinflection]
  * second-person plural agreement + reported/inferential past + negation [reinflection]
  * dative case + first-person possessive [reinflection]
  * accusative singular [simpler inflection]
* Annotation
  * Example sentence: I [verb]-ed, (I heard that) you have not [verb]-ed, to my [noun].
  * Each of the four had up to five real-word examples and 10 nonce-root test examples.
  * Stimuli and gold annotations came from one Turkish annotator.

## Experiments

### Setup

* Compare `gpt-3.5-turbo-0613` against human annotations and supervised morphology baselines on the above languages/tasks.
  * Training data for baselines come from a few sources including SIGMORPHON 2023 dataset.
* Evaluation metric: accuracy at *k* (`acc@k`).
  * A prediction is correct if it matches one of the top *k* human responses.
  * Main evaluation uses `acc@5` (Turkish uses `acc@1`).
* Only the first word of ChatGPT’s response is scored; non-word characters are removed.

### Baselines
* Affix Rule Learner (ARL): Non-neural system that learns prefix/suffix edit rules from training data.
* Minimal Generalization Learner (MinGen): Rule-based model that learns transformations from lemma → inflected form.
* Feature Invariant Transformer (FIT): Character-level transformer for generating inflected forms from features.
* Principal Parts for Inflection (PPI): Uses key paradigm slots to infer other inflected forms.
* Analogical Encoder-Decoder (AED): Neural encoder-decoder model using analogical morphological patterns.

### Prompting 

> Fill in the blank with the correct past tense of the word ‘wug’. Give your response in one word.

> They wug all the time. In fact, they ___ just yesterday.

Three prompting regimes:
* Zero-shot: no examples
* One-shot: one real-word example
* Few-shot: several real-word examples, usually one from each major inflection class

Two prompt formats:
* Long: Fill in the blank with the correct past tense of the verb X. Answer with one word. They X all the time. In fact, they ___ just yesterday! ___ : 
* Short: Form the correct past tense of the verb X. Answer with one word. X :

For Tamil, the instruction part of the prompt is omitted because ChatGPT was unreliable when given Tamil instructions.

## Results

| Method | English | German | Tamil | Turkish |
|---|---:|---:|---:|---:|
| Human annotator performance | 87.64 ± 12.13 | 87.88 ± 10.34 | 43.85 ± 26.95 | — |
| ARL | **100.00** | **94.25** | 61.48 | 60.00 |
| MinGen | 62.00 | 64.37 | 49.18 | 40.00 |
| FIT | 98.00 ± 1.26 | 92.87 ± 0.74 | **63.28 ± 3.36** | 67.00 ± 4.58 |
| PPI | 94.60 ± 2.54 | 85.98 ± 5.91 | 55.33 ± 1.84 | **68.00 ± 4.00** |
| AED | 57.60 ± 6.62 | 48.51 ± 5.45 | 58.69 ± 5.46 | 56.00 ± 4.90 |
| ChatGPT: long 0-shot | 58.40 ± 5.28 | 86.49 ± 1.07 | 0.00 | 28.00 ± 14.00 |
| ChatGPT: long 1-shot | 73.60 ± 6.97 | 85.42 ± 2.52 | 14.52 ± 7.48 | 20.00 ± 14.14 |
| ChatGPT: long few-shot | 76.40 ± 4.45 | 87.36 ± 2.37 | 42.70 ± 3.96 | 54.00 ± 10.20 |
| ChatGPT: short 0-shot | 75.40 ± 5.87 | 88.62 ± 1.64 | 0.00 | 3.00 ± 4.58 |
| ChatGPT: short 1-shot | 82.80 ± 5.60 | 88.94 ± 2.35 | 3.28 ± 3.99 | 58.00 ± 7.48 |
| ChatGPT: short few-shot | 78.60 ± 2.84 | 88.33 ± 1.15 | 43.36 ± 3.12 | 59.00 ± 9.43 |


#### English

- ChatGPT performs worse than the strongest baselines and below average human performance.
- Short prompts work better than long prompts.
- Best ChatGPT score: **82.80** with short 1-shot.
- The model sometimes outputs real English words instead of properly inflecting the nonce word.

#### German

- ChatGPT performs strongly.
- Best ChatGPT score: **88.94** with short 1-shot.
- This is close to human-level performance, given that human acc@5 is about 88%.
- Long vs. short prompt differences are small.

#### Tamil

- ChatGPT performs much worse than the supervised baselines.
- Zero-shot produces no correct outputs.
- Few-shot improves performance substantially.
- Best ChatGPT score: **43.36** with short few-shot.
- The authors note that this is still somewhat reasonable given low human agreement on Tamil nonce forms.

#### Turkish

- ChatGPT performs worse than English and German.
- Short prompts help more than long prompts.
- Best ChatGPT score on the main Turkish inflection task: **59.00** with short few-shot.
- For the three harder Turkish reinflection tasks, scores are lower overall:

| Prompt type | Turkish reinflection average |
|---|---:|
| Long 0-shot | 3.00 ± 1.80 |
| Long 1-shot | 20.67 ± 5.73 |
| Long few-shot | 33.33 ± 4.94 |
| Short 0-shot | 7.00 ± 4.33 |
| Short 1-shot | 18.67 ± 6.18 |
| Short few-shot | 31.00 ± 4.23 |

### 8. Overall takeaway

ChatGPT shows some ability to generalize morphological patterns, especially with short analogy-style prompts, but it does **not** outperform strong morphology-specific baselines. Performance is strongest for German, moderate for English, weaker for Turkish, and weakest for Tamil. Few-shot prompting helps in several cases, but the improvement is inconsistent across languages.


