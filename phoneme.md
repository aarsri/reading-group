# On the Use/Misuse of the Term 'Phoneme'
_Roger K. Moore, Lucy Skidmore at Interspeech 2019_

_Presented by Aarohi Srivastava on 2/13/26_

## Background

NLP research is grounded in ideas from other disciplines, particularly linguistics. But not everyone working in NLP maintains a tight interface with linguistic theory, and as a result, technical linguistic terms sometimes get borrowed and used loosely. Today's paper looks at a specific instance of this, the use/misuse of the term "phoneme," particularly in speech conference papers.

Phoneme: "the smallest unit of speech that distinguishes one word from another _in a particular language_."

## Phonetic vs. Phonemic

The distinction between phonetic and phonemic levels of representation are probably the root of most misuse or misunderstanding. 
* Phonetic: language-independent fine-grained acoustic/articulatory realization, denoted between square brackets ([]).
* Phonemic: language-specific contrastive category, denoted between slashes (//).

### Developmental Intuition

Babies start out as universal listeners of language. They are sensitive to phonetic distinctions across languages and are primed to natively acquire any language. You could imagine that early on, they operate at something like a phonetic level, tracking fine-grained acoustic distinctions without yet grouping them into language-specific categories. As they acquire a particular language (or multiple languages), they reorganize perception into phonemic categories. **They stop distinguishing certain differences because those differences are not contrastive in their language.**

### Understanding the Distinction via English vs. Hindi

A while back, my mom had found a tea party invitation I wrote when I was 3. For context, I was bilingual in English and Hindi (but speaking and hearing 75% Hindi) and had learned the English alphabet but very little about spelling. I wrote:

This is a **invathashane** for the **thea** partea so plees kame.

What I see now (after Darcey pointed it out a while back) is that I maybe contrasted **aspirated** vs. unaspirated t in my writing. Though this is not a distinction we explicitly make in English, it is in Hindi (they are separate consonants). 

A few notes:
* Aspiration: burst of air that follows the release of a stop consonant.
* Voiced vs. voiceless consonant: Voiced consonants involve vibration of the vocal folds, while voiceless consonants do not. Consider [p] vs. [b] or [t] vs. [d].

Per English phonology, when a voiceless stop consonant is word-initial and/or at the onset of a stressed syllable (e.g., tea, pin, underpin), it is aspirated. Examples where it is not aspirated: stove, spy. (This is not a complete phonological analysis of aspiration in English.)
* Consider: tea → [tʰi]. If we replaced [tʰ] with [t] in tea, it might sound slightly accented, but it would still be perceived as *tea*.
* **[tʰ] and [t] are allophones of the same phoneme, /t/, in English.**
* This is not true in all languages.

An English phonemic transcription need not make this distinction (e.g., /ti/), while a phonetic transcription must. (It may still be noted in phonemic transcriptions because it is a more obvious feature, but it does not need to be there.)

This brings us to another definition of phoneme provided in the paper: "a family of uttered sounds (segmental elements of speech) in a particular language which count for practical purposes as if they were one and the same."
  
In Hindi, stop aspiration is a **contrastive feature** that can change the meaning of a word. So is the place of articulation for t!
* ताल - /t̪aːli/ - to clap
* थाल - /t̪ʰaːli/ - plate
* टोक - /ʈoːk/ - to pester/interrupt/object
* ठोक - /ʈʰoːk/ - to hammer (hit a nail)

Dental, alveolar and retroflex refer to the **place of articulation**, or where the tip of the tounge is when producing the sound. Alveolar is towards the front of the roof of the mouth while retroflex is further back. Dental would be directly behind/on the front teeth.
* alveolar: [t]
* dental: [t̪]
* retroflex: [ʈ]

In Hindi, a phonemic transcription must distinguish /t̪/, /t̪ʰ/, /ʈ/, and /ʈʰ/, because both place (dental vs retroflex) and aspiration are contrastive. In English, by contrast, these distinctions are not phonemic. **Even if a Hindi-accented speaker realized “tea” as [ti], [t̪i], or [ʈi], all of these would be mapped onto the same English phoneme /t/**, since none of these differences signal a change in lexical meaning in English.

Takeaway: I hope these examples clarify the distinction between phonetic and phonemic levels of representation, and when to use square brackets [] versus slashes // (which I have tried to do consistently). 
* Phonemic transcription (//) represents contrastive categories within a particular language. It abstracts away from predictable variation and encodes only those distinctions that differentiate words in that language. Because phonemes are language-specific, a phonemic transcription must always be interpreted relative to a particular language. There is no such thing as a language-independent phonemic representation.
* Phonetic transcription ([]) represents precise articulatory or acoustic realization, independent of whether they are contrastive in any given language. When discussing sounds across languages, square brackets are appropriate.
* Importantly, when using phonetic notation, we should be as precise as necessary. We cannot collapse distinctions simply because they are not contrastive in English. For example, [t], [t̪], and [ʈ] are articulatorily distinct sounds, even if English speakers map them all to /t/. Using [t] to represent all of them would impose an English phonological interpretation onto what is meant to be a language-neutral phonetic description. I suspect that is a key issue if non-linguists try to write a phonetic transcription.

*A phoneme is defined by the contrastive structure of a particular language, not by universal acoustic substance. The brackets we choose signal which level of representation we are committing to.*

## Implications

The paper points out three implications of why this phonetic–phonemic distinction matters, and in particular why a phonemic representation is often valuable.

1. Phoneme restoration effect: if a short section of speech was cut out and replaced by another sound, a native or proficient listener would not detect anything was missing. This is a strong demonstration that we are not simply decoding acoustic input; we are mapping it onto phonemic expectations shaped by the structure of our language. A phonetic transcription, being language-independent, is not supposed to fill in those blanks.
    *  I think it is important to note that the paper implicitly assumes native or highly proficient listeners in most descriptions. Such effects depend on having a stable internalized phonemic system for a language. If someone is not yet fully locked into the phonology of a language, like an early L2 learner, a lot of this would not apply.
2. When we expect to hear a particular sequence of sounds in a given linguistic context, we tend to perceive it as such, even if the acoustic signal is degraded. For example, if you yell across the house, “Johnny, did you feed your fish yet?”, you are realistically only expecting to hear something like “yes,” “no,” or “I don’t know.” What reaches your ears may just be muffled grunts, but you will often reconstruct the intended response anyway. (Though this is mentioned as a distinct implication in the paper, I think it conveys the same point as #1.)
3. Coarticulation: articulatory gestures overlap and unfold over time, so phonemic information is distributed across neighboring segments rather than packaged into clearly bounded acoustic units, challenging the idea that speech is composed of tidy, bead-like elements. Why they mention this in the paper is unclear to me, but I think the point is that due to this reality, phonemes are inherently not acoustic units and should not be treated as such (while phones are).

### Implications to Computational Research
I think the paper is missing this section.



## Results

of the
265 accepted papers in INTERSPEECH-2018 that contained
“phoneme”, only six had anything approaching a definition and,
of those, none were satisfactory:
“Speech signal consists of various ba-
sic speech sound units, which are called as phonemes.”
“. . . treat any sub-word acoustic unit as a phoneme.”
“In phonetics, it is be-
lieved that when one pronounces two neighbouring phonemes,
there often exists joint frames that can be a very short pause be-
longing to neither phoneme . . . ”
"sound symbols"
"sub words"

in the paper the authors say: "what is perhaps most concerning is that 98% of
the INTERSPEECH-2018 papers that used the term ‘phoneme’ did not provide any form of definition or explanation, presum-
ably because the authors assumed that everyone knows what it
means"
I personally don't find this concerning at all. If it is used correctly, it is an appropriate term to use in the context of speech so I don't think you need to define it (unless you are crucially distinguishing phonemic and phonetic to describe your method or something like that). And if it is used incorrectly you definitely shouldn't try to define it but that's just incorrect.

"of the 265 accepted papers in INTERSPEECH-
2018 that contained the term ‘phoneme’ showed that around
40% used the term in a way that could be construed as mis-
use"
examples: 
papers that clearly don't understand /.../ vs. [...] notation
"phonemic segmentation" - wrong because...
"phoneme durations" - wrong because...
"phoneme chunks" - wrong because...

They also present a long list of phrases in Interspeech 2018 papers misusing "phoneme." I don't doubt that for many of those the author writing the phrase did not really know the precise definition of phoneme or the difference between phonemic and phonetic. At the same time, I can't say all of these are really a misuse...
"If a phoneme lasts for more than 5ms" - I guess you could say "if the acoustic realization of a phoneme lasts more than 5 ms" but that seems very picky given that NLP writing may not always be known for its precision...
"treating filled pause as a special ‘phoneme’" - this is a way of concisely describing a modification they are making to their label space, even they seem to know that a filled pause is not actually a phoneme due to the use of quotes around phoneme

at the same time of course there are plenty that are misuse/don't make sense:
"universal phoneme mapping"
"We propose a language-independent phoneme segmentation
method"
"We have 252 phonemes, of which there are 213 Mandarin
and 39 English."

I will also say using phoneme to describe the label space is not totally accurate because per my discussion above it is not clear whether the model is learing phonetic or phonemic representations and it is probably a mix of the two. It begs the question of what is the appropriate term in such contexts (as you could not use phone or phoneme).

## Other linguistics terms that are misused in a lot of NLP writing
The only one that came to mind is writing system vs. alphabet vs. script (which are all different things). I wonder if anyone thinks of another collection of terms?

## A note about Speech LMs
Popular speech LMs:
The one that might provide the best coverage of foundational knowledge: 
