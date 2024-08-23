### A Systematic Analysis of Subwords and Cross-Lingual Transfer in Multilingual Translation

By Francois Meyer and Jan Buys in NAACL 2024 Findings

Presented by Aarohi Srivastava on August 23, 2024

#### BPE
<BPE image>

#### Introduction

Premise: Tradeoff between *synergy* and *interference*.
* Synergy - shared parameter space and subword vocabulary can facilitate positive transfer from high to low resource languages.
* Interference - there can also be negative cross-lingual interactions which can result in sub-optimal performance for the high-resource languages.

Goal: To analyze the role of subword segmentation in MT with respect to:  
* type of subword tokenization used
* linguistic features of languages (morphology and orthography)
* scenario (multilingual MT vs. cross-lingual fine-tuning)
* synergy, interference, and knowledge transfer

#### Languages

| Language | Family | Morphology | Orthography |
| -------- | ------ | ---------- | ----------- |
| Siswati (ss) | Bantu/Nguni | agglutinative | conjunctive |
| isiXhosa (xh) | Bantu/Nguni | agglutinative | conjunctive |
| Setswana (ts) | Bantu/Sotho-Tswana | agglutinative | disjunctive |
| Afrikaans (af) | Germanic | analytic | disjunctive |

* Agglutinative (vs. analytic) morphology - morphemes serve clearer, smaller roles on their own and join together, leading to a higher number of morphemes per word (longer words).
* Conjunctive vs. disjunctive orthography - the various elements of a word (such as a verb and its person) are written as separate words in disjunctive orthography, but as together in one word in conjunctive orthography.
  *  In particular, even if languages are less closely related, if they are both disjunctive, it might make MT easier as there is stronger one-to-one alignment (see Setswana and Afrikaans).

#### Subword tokenization methods
* BPE - deterministic
* Unigram (ULM) w/ subword regularization - non-deterministic, model can see different tokenizations of the same sequence
* Subword segmental MT (SSMT) - segmentation learned jointly with MT training (optimized to MT performance)
* Overlap BPE (OBPE) - boost subword overlap across languages
* Extended BPE (XBPE) - new subwords (pertaining to the target language) are added to the vocabulary

#### Models trained
* Low-resource: English --> Siswati
* Higher resource: English --> isiXhosa / Setswana / Afrikaans
* Trilingual: English --> Siswati and isiXhosa/Setswana/Afrikaans
* Cross-lingual fine-tuning - fine-tune these on English to one of the unseen target languages (e.g., FT en-->xh on en-->ts)

If ChrF++ of the trilingual model is higher than that of the bilingual model, there is synergy. If it is lower, there is interference.

#### "Which subwords promote synergy and minimise interference?"
#### "Which subwords transfer cross-lingually?"
#### "What is the role of linguistic typology?"



