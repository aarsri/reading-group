# On the Use/Misuse of the Term 'Phoneme'
_Roger K. Moore, Lucy Skidmore at Interspeech 2019_

_Presented by Aarohi Srivastava on 2/13/26_

## Background

NLP research is grounded in ideas from other disciplines, particularly linguistics. But not everyone working in NLP maintains a tight interface with linguistic theory, and as a result, technical linguistic terms sometimes get borrowed and used loosely. Today's paper looks at a specific instance of this, the use/misuse of the term "phoneme," particularly in speech conference papers.

Phoneme: "the smallest unit of speech that distinguishes one word from another _in a particular language_.

## Phonetic vs. Phonemic

The distinction between phonetic and phonemic levels of representation are probably the root of most misuse or misunderstanding. 
* Phonetic: language-independent fine-grained acoustic/articulatory realization, denoted between square brackets ([]).
* Phonemic: language-specific contrastive category, denoted between slashes (//).

### Developmental Intuition

Babies start out as universal listeners of language. They are sensitive to phonetic distinctions across languages and are primed to natively acquire any language. You could imagine that early on, they operate at something like a phonetic level, tracking fine-grained acoustic distinctions without yet grouping them into language-specific categories. As they acquire a particular language (or multiple languages), they reorganize perception into phonemic categories. **They stop distinguishing certain differences because those differences are not contrastive in their language.**

### Different versions of 't'

A while back, my mom had found a tea party invitation I wrote when I was 3. For context, I was bilingual in English and Hindi (but speaking and hearing 75% Hindi), and had formally learned the English alphabet but very little about spelling. I wrote:

This is a **invathashane** for the **thea** partea so plees kame.

What I see now (after Darcey pointed it out a while back) is that I maybe contrasted **aspirated** vs. unaspirated /t/. Though this is not a distinction we explicitly make in English, it is in Hindi (they are separate consonants). 

A few notes:
* Aspiration: burst of air that follows the release of a stop consonant.
* Voiced vs. voiceless consonant: minimal feature of articulation where the vocal chords would (voiced) or would not (voiceless) vibrate. Consider [p] vs. [b] or [t] vs. [d].

Per English phonology, when a voiceless stop consonant is word-initial and/or at the onset of a stressed syllable (e.g., tea, pin, underpin), it is aspirated. Examples where it is not aspirated: stove, spy. (This is not a complete phonological analysis of aspiration in English.)
* Consider: tea → [tʰi]. If we replaced [tʰ] with [t] in tea, it might sound slightly accented, but it would still be perceived as *tea*.
* **[tʰ] and [t] are allophones of the same phoneme, /t/, in English.**
* This is not true in all languages.

A phonemic transcription need not make this distinction (e.g., /ti/), while a phonetic transcription must. (It may still be noted in phonemic transcriptions because it is a more obvious feature, but it does not need to be there.)

This brings us to another definition of phoneme provided in the paper: "a family of uttered sounds (segmental elements of speech) in a particular language which count for practical purposes as if they were one and the same."
  
In Hindi, stop aspiration is a **contrastive feature** that can change the meaning of a word. So is the place of articulation for t!
* ताल - /t̪aːli/ - to clap
* थाल - /t̪ʰaːli/ - plate
* टोक - /ʈoːk/ - to pester/interrupt/object
* ठोक - /ʈʰoːk/ - to hammer (hit a nail)

Some notes:
* Dental, alveolar and retroflex refer to the place of articulation, or where the tip of the tounge is when producing the sound. Alveolar is towards the front of the roof of the mouth while retroflex is further back. Dental would be directly behind/on the front teeth.
* [t]: alveolar
* [t̪]: dental
* [ʈ]: retroflex

In Hindi, a phonemic transcription must distinguish /t̪/, /t̪ʰ/, /ʈ/, and /ʈʰ/, because both place (dental vs retroflex) and aspiration are contrastive. In English, by contrast, these distinctions are not phonemic. **Even if a Hindi-accented speaker realized “tea” as [ti], [t̪i], or [ʈi], all of these would be mapped onto the same English phoneme /t/**, since none of these differences signal a change in lexical meaning in English.

Takeaway: I hope these examples illustrate the difference between phonetic and phonemic levels of transcription, and when to use [] and // (which I tried to do correctly). When we are talking about a particular language, we should use // because we are on the level of the phonology of the language. When we are talking across languages we should use [] or if using // specify which language that phonemic transcription is for (it cannot be language-independent). When we are just talking about sounds in general we should use [], but that also means we should be as precise as possible with our IPA symbols and cannot collapse them to an English view of the sounds (e.g., using [t] instead of [t̪] because we thought they were the same)

A phoneme is defined by the contrastive structure of a particular language, not by universal acoustic substance. The brackets we choose signal which level of representation we are committing to.

## Implications

The paper points out three implications of why we care about this distinction, particularly, why a phonemic representation is often valuable. The first is the phoneme restoration effect, meaning if a short section of speech was cut out and replaced by another sound, a native or proficient listener would not detect anything was missing. (The reason i say native for all of these is also worth mentioning. Depending on someone's level of proficiency in a langauge these things may or may not apply, i.e., they may have too much trouble to restore what the word must have been if they are not locked into the language in their language faculty in their brain the way they are in their native language). The second is an extension, that when you're expecting to hear a certain sequence of sounds in a particular language, you will perceive it as such. Basically if you yell a question like johnny did you feed your fish yet across the house you will expect to hear just yes no or i don't know and so the way it reaches you might really just be muffled grunts but you will put it together. I honestly don't see how that one is relevant. 

in terms of implications to nlp, 
you need context no matter what, but you need context much more to do phonemic rather than phonetic transcription. it begs the question of which task is easier or what it takes to train a model to do either one. also if one model is trained to do one it can be difficult to adapt it to do the other. i think this is particularly true if a model was trained to do phonemic transcription (which i think it is bc you could think of orthography as a phonemic representation in languages with an alphabetic script) and then you try to have it do phonetic transcription or in some way differentiate between accent or dialect. same for us, we have an intuition of the phonemic transcription and we certainly know how to write in our alphabetic script, but if we are asked to do a phonetic transcription, aside from needing to know the IPA symbols, it would take us a lot of effort to gain that intuition of how to differentiate stuff like aspiration vs. unaspirated and other distinguishing features that we otherwise don't distinguish in our language.

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
