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

## Methodology


