# Morphological Capabilities of LLMs

_Presented by Aarohi Srivastava on 5/8/26_

## Counting the Bugs in ChatGPT’s Wugs: A Multilingual Investigation into the Morphological Capabilities of a Large Language Model
_Leonie Weissweiler et al. at EMNLP 2023_

### Wug Test (Jean Berko Gleason, 1958)

> This is a wug. Now there's another one. These are two ___.

> This is a man who zibs. He is a ___.

Child language acquisition experiment that demonstrated that young children do not just memorize words but internalize abstract, systematic grammatical rules.

This paper conducts the Wug Test using ChatGPT (gpt-3.5-turbo-0613) with nonce words from typologically diverse 4 languages: English, German, Tamil,
and Turkish.

### Background

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

### Data Creation

A big part of this work is to develop sets of nonce words in the four languages to be used in the Wug Tests. Data is available here: github.com/dmort27/chatgpts-wugs

#### English
* Focus: past tense formation of verbs.
* Starting point: 50 short irregular English verbs from UniMorph, then altered one or two letters to create nonce verbs not found in the dataset.
* Annotation: 
  * Sentence frame: They {nonce} all the time. In fact, they ___ just yesterday.
  * 28 volunteer annotators produced possible past-tense forms.
  * Most common human responses were regular past-tense forms with -ed.

#### German
* Focus: plural noun formation, which is complex because German has several competing plural strategies.
* Starting point: Generated 200 nonce nouns from Unipseudo (4-7 characters), then filtered down to 174.
* Annotation: 
  * A native German speaker first assigned each nonce noun an article: der, die, or das, then generated a plural.
  * Sentence frame: Hier ist ein/e {nonce}. Jetzt sind es zwei ___.
  * 21 volunteer annotators provided plural forms.
  * The result was a ranked list of plausible plural forms for each nonce noun.

#### Tamil
* Focus: past tense verb inflection
  * Tamil verbs are morphologically rich: they can encode tense, transitivity, person, number, and sometimes gender.
  * The authors simplified the task by focusing on past tense, intransitive verbs, third-person singular masculine agreement.
* Starting point: Sampled 86 common Tamil verbs, generated conjugations, and had two native speakers validate them. Nonce verbs were built by combining syllables from real Tamil verb roots, then checked against a Tamil dictionary to ensure they were not real words.
* Annotation:
  * Five native Tamil speakers supplied past-tense forms; the most common response was treated as the gold answer.
  * Inter-annotator agreement was relatively low, partly because Tamil verb-class assignment can depend on historical/linguistic context.
  * I don't think this one has a sentence frame; the format seems to be {nonce} --> ___ [must be past tense]

#### Turkish
* Focus: inflection and reinflection
  * first-person singular agreement + past tense [reinflection]
  * second-person plural agreement + reported/inferential past + negation [reinflection]
  * dative case + first-person possessive [reinflection]
  * accusative singular [simpler inflection]
* Annotation
  * Example sentence: I [verb]-ed, (I heard that) you have not [verb]-ed, to my [noun].
  * Each of the four had up to five real-word examples and 10 nonce-root test examples.
  * Stimuli and gold annotations came from one Turkish annotator.

### Experiments

#### Setup

* Compare `gpt-3.5-turbo-0613` against human annotations and supervised morphology baselines on the above languages/tasks.
  * Training data for baselines come from a few sources including SIGMORPHON 2023 dataset.
* Evaluation metric: accuracy at *k* (`acc@k`).
  * A prediction is correct if it matches one of the top *k* human responses.
  * Main evaluation uses `acc@5` (Turkish uses `acc@1`).
* Only the first word of ChatGPT’s response is scored; non-word characters are removed.

#### Baselines
* Affix Rule Learner (ARL): Non-neural system that learns prefix/suffix edit rules from training data.
* Minimal Generalization Learner (MinGen): Rule-based model that learns transformations from lemma → inflected form.
* Feature Invariant Transformer (FIT): Character-level transformer for generating inflected forms from features.
* Principal Parts for Inflection (PPI): Uses key paradigm slots to infer other inflected forms.
* Analogical Encoder-Decoder (AED): Neural encoder-decoder model using analogical morphological patterns.

#### Prompting 

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

### Results

| Method | English | German | Tamil | Turkish |
|---|---:|---:|---:|---:|
| Human annotator | 87.64 ± 12.13 | 87.88 ± 10.34 | 43.85 ± 26.95 | — |
| ARL | **100.00** | **94.25** | 61.48 | 60.00 |
| MinGen | 62.00 | 64.37 | 49.18 | 40.00 |
| FIT | 98.00 ± 1.26 | 92.87 ± 0.74 | **63.28 ± 3.36** | 67.00 ± 4.58 |
| PPI | 94.60 ± 2.54 | 85.98 ± 5.91 | 55.33 ± 1.84 | **68.00 ± 4.00** |
| AED | 57.60 ± 6.62 | 48.51 ± 5.45 | 58.69 ± 5.46 | 56.00 ± 4.90 |
| ChatGPT: long 0-shot | 58.40 ± 5.28 | 86.49 ± 1.07 | 0.00 | 28.00 ± 14.00 |
| ChatGPT: long 1-shot | 73.60 ± 6.97 | 85.42 ± 2.52 | 14.52 ± 7.48 | 20.00 ± 14.14 |
| ChatGPT: long few-shot | 76.40 ± 4.45 | 87.36 ± 2.37 | 42.70 ± 3.96 | 54.00 ± 10.20 |
| ChatGPT: short 0-shot | 75.40 ± 5.87 | 88.62 ± 1.64 | 0.00 | 3.00 ± 4.58 |
| ChatGPT: short 1-shot | _**82.80**_ ± 5.60 | _**88.94**_ ± 2.35 | 3.28 ± 3.99 | 58.00 ± 7.48 |
| ChatGPT: short few-shot | 78.60 ± 2.84 | 88.33 ± 1.15 | _**43.36**_ ± 3.12 | _**59.00**_ ± 9.43 |

#### Takeaways

* For each language, best ChatGPT config never outperforms the best baseline.
* For English and kind of for Turkish, short prompts work better than long. For German and Tamil, no major difference.
* Substantial increase in performance going from zero-shot to one- or few-shot for Tamil and Turkish, but not major for English and German.

| Language | Best baseline(s)          | Neural/non-neural                                           |
| -------- | ------------------------- | ----------------------------------------------------------- |
| English  | **ARL 100.00**, FIT 98.00 | Best is non-neural; neural FIT is close.                    |
| German   | **ARL 94.25**, FIT 92.87  | Best is non-neural; neural FIT is close.                    |
| Tamil    | **FIT 63.28**, ARL 61.48  | Best is neural, but non-neural ARL is close.                |
| Turkish  | **PPI 68.00**, FIT 67.00  | Best is non-neural; neural FIT is close. |

### Analysis
* Task: The German version of the task was more complicated because rules are not clear; despite this, ChatGPT's performance was the highest for German, suggesting morphological complexity alone does not explain performance differences. At the same time, ChatGPT overgeneralizes in picking _-en_ and _-s_ German plural markers, and so frequency could play a role too.
* Tokenization: The number of tokens a nonce word was split into did not significantly affect ChatGPT’s performance, suggesting tokenization was not a major factor in these experiments.
* Impact of *k* in `acc@k`: The gap between ChatGPT and the baselines increases as k increases. Baselines generate a wider range of plausible human-like inflections, while ChatGPT often produces either the top response or an implausible one.
* Real world bias: ChatGPT sometimes outputs an inflected form of a real word instead of properly inflecting the nonce word. This bias is strongest in English and German.
  * English:
    * dedo → did
    * blus → blushed
    * fride → fried
  * German:
    * Ozeak → Ozeane
    * Instite → Institute
    * Schlave → Sklaven

### Conclusion

ChatGPT (gpt-3.5-turbo-0613) seems imitate morphology quite well in certain settings, but it does not exhibit consistent morphological abstraction or generalization capability, especially when it comes to typologically diverse languages and unseen nonce words.

## Evaluating Morphological Compositional Generalization in Large Language Models
_Ismayilzada et al._ at NAACL 2025

### Setup

- The paper asks whether LLMs can do morphological compositional generalization. In simpler terms: can models combine roots and morphemes in systematic, human-like ways, especially for novel words?
- They use 5-shot prompting and focus on two highly agglutinative languages, Turkish and Finnish.

### How it relates to Weissweiler et al. (2023)

- Like the earlier paper, it uses Wug tests to avoid simply testing memorization.
- Like the earlier paper, it compares LLMs against human performance.
- Like the earlier paper, it finds that LLMs struggle with novel morphological forms (nonce words).
- The main difference is that this paper focuses more explicitly on compositionality, so not just whether the model can produce the correct inflection, but also 
  - not just “can the model produce the right inflection?”
  - but “can the model combine morphemes productively and systematically?”

### What Ismayilzada et al. (2025) does differently

- It tests two capabilities:
  - Productivity: generate a valid word from a root plus morphemes.
    - Example: Nonce root: nisi, Morphemes: -ler, -lik, -in. Generate the correct composed form. Answer: nisiliklerin
  - Systematicity: judge whether a morpheme combination is valid.
    - Example: choose which is correct: nisiliklerin or nisiliklerin
- It evaluates newer multilingual LLMs, including GPT-4, Gemini-1.5, Aya-23, and Qwen-2.5.
- It compares in-distribution real roots with out-of-distribution nonce roots.

### Main findings

- **Performance**
  - LLMs still fall far below humans on morphological generalization. Human performance is also much more stable across real and nonce roots.
  - Models struggle especially with nonce roots, supporting Weissweiler et al.’s finding that LLMs do not robustly generalize to novel word forms.
  - Models do better on systematicity than productivity, but even with systematicity the models are not consistently applying a particular rule.
- **Tokenization** does not seem to be a driving factor. They compare morphologically aligned units (tokens are real morpheme pieces) vs. original tokenizer and do not see a difference in performance. At the same time, they do not do anything to tune the model to the new token sequences; the model is suddenly being asked to process inputs in a format that may not match its training distribution.
- **Real world bias**: The paper also finds evidence of real-word bias, similar to Weissweiler et al., where models often drift toward real/frequent words rather than following the requested morphological composition.
- **Order of morphemes**: Presenting morphemes in the correct order rather than shuffled order increases productivity task performance greatly.
- **Number of morphemes**: Performance declines sharply as morphological complexity increases.
- **Model choice**: GPT-4 does the best among the models tested. Otherwise, when comparing within the same model family, the larger size does a bit better, but scale does not seem to solve the problem.

### Effect of morphological complexity

- The authors ask whether models get worse as words become morphologically longer/more complex. They measure complexity by the number of bound morphemes (1-7 morphemes for Turkish, 1-6 for Finnish).
- In the productivity task, GPT-4’s performance drops sharply as the number of morphemes increases. For longer Turkish forms, performance falls close to zero. Humans do not show the same sharp drop.
- In the systematicity task, Macro-F1 stays more stable, but consistency declines with complexity. This means models may judge some individual forms correctly, but become less consistent as morpheme chains get longer.

[include Figure 6]

### Effect of context

- The authors test whether adding sentence context helps. Instead of only giving root + morphemes, they give a sentence with a blank.
- For productivity, context helps somewhat, since the model can use the sentence to guide the generated word.
- For systematicity, context often hurts performance, especially for smaller models and OOD nonce roots.
- Interpretation: Context does not solve the morphology problem. In some cases, it adds extra processing burden. This is important because the contextual version is closer to ordinary language modeling, so failure there suggests the problem is not just an artifact of an artificial task.

## Overall Takeaways

### Discussion
