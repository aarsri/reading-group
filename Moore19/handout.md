# On the Use/Misuse of the Term 'Phoneme'
_Roger K. Moore, Lucy Skidmore at Interspeech 2019_

_Presented by Aarohi Srivastava on 2/13/26_

## Background

NLP work is grounded in ideas from other disciplines, including linguistics. However, not all researchers maintain a strong NLP-linguistics interface (we all do!), meaning that when it comes time to describe specific situations in NLP with a pretty technical linguistic term, it can instead get oversimplified and overused to mean an instead imprecise meaning. This paper is focused on this situation when it comes to the term phoneme, and specifically calculates statistics of use/misuse in Interspeech papers (a speech conference).

Phoneme: "the smallest unit of speech that distinguishes one word from another _in a particular language_.

## Phonetic vs. Phonemic

This is probably where the confusion starts. These are two different things. Define and disentangle.
Babies start out as universal listeners of language. You could imagine that they start out with phonetic representations in their brain because they do not have the notion of what a particular language is, and they are primed to natively acquire any language. At some point once they actually start speaking a particular language (or set of languages if bi/trilingual), this distinction will come into play. This is probably why I was looking back at a tea party invitation I made when I was 3 and I wrote "invithashun" and some other words [need to remember] all with an h for the aspirated consonants because I distinguished between aspirated and unaspirated. In Hindi unaspirated t is more common while in English both are used in different settings (per the surrounding sounds). what that means is that since phonemic representation/transcription is _within a language_ you would just use a _t_ in English words like tea and invitation, but phonetic transcription is across languages meaning you would have to use th for aspirated t regardless of the language.
To sum that up another way phoneme is defined in the paper is "a
family of uttered sounds2 (segmental elements of speech) in a
particular language3 which count for practical purposes as if
they were one and the same" i.e., each phonemic symbol would be a collection of close but different sounds. Another example could be with accent or dialect. In English we hear so many different accents (e.g., my mom doesn't really say thank you with an american th it's different) but we squash them into the same representation in our head like you will still hear her thank you as thank you. So at this point in our head with the frame of reference of a particular language we have phonemic rather than phonetic representations. 
If it's confusing to think about the fact that a single phoneme is a cluster of sounds and thinking about where to draw that line, one way to think about it is if two phones are swapped out as minimal pairs in the word, does it make the word different to a native listener, or would they perceive it as the same. So if you think about it, whether we said "tea" with an aspirated t or with an unaspirated t, we will take it as the same word (maybe you will perceive an accent but nothing more). In contrast if we said "dea" party with a d there it would no longer sound like the same word. This brings about another pertinent concept of allophone. In English, aspirated and unaspirated t are _allophones of the same phoneme_, while t and d are distinct phonemes. This phrase "allophones of the same phoneme" truly sticks in my head bc I think it was drilled into our head in intro linguistics. In another language perhaps unaspirated and aspirated t would actually make the word different and would thus be considered distinct phonemes for that language. And to be clear the phonetic transcription would mean to use [] and you would always have to do t^h vs t while in a phonemic transcription you use // and you may not have to differentiate t^h and t you might just write t.

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

