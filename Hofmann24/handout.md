## Derivational Morphology Reveals Analogical Generalization in Large Language Models

Valentin Hofmann, Leonie Weissweiler, David Mortensen, Hinrich Schütze, Janet Pierrehumbert

Presented by Aarohi Srivastava on January 17, 2025

Big Idea: What mechanisms underlie linguistic generalization in large language models (LLMs)? Are these mechanisms rule-based or analogical?

### Background

**Rule-Based Generalization**: Past work has approached this question from a rule-based perspective, meaning for some linguistic concept (e.g., subject-verb agreement), they believe the LLM has inferred a set of symbolic rules from the training data.
* Example: If a word ends in -ish, add -ness to nominalize (selfish --> selfishness) 

**Analogical Generalization**: The other (understudied) option is that the LLM generalizes by comparing new inputs to previously-encountered examples ("analogically processes operating on stored exemplars").
* Example: Model has seen selfish → selfishness, so when it encounters a new word "friquish," it might predict “friquishness” by analogy to the known “selfish → selfishness” pair.
* Analogical processes can allow the model to handle variability and irregular patterns better than strict rule-based approaches.
* Analogical generalization is believed to be a central learning mechanism for humans that allows us to form abstract conjectures.

**Adjective Nominalization**: To distinguish between rule-based and analogical generalization in LLMs, the authors analyze how GPT-J learns English adjective nominalization wiith -ity and -ness, focusing on adjectives that already contain a derivational suffix (available --> availability, selfish --> selfishness). This is a narrow class that still exhibits plenty of variability to be able to observe analogical learning.

### Cognitive Models

To probe the underlying generalization mechanisms in GPT-J, the authors employ two cognitive models, one rule-based and one analogical, and see which one best aligns with GPT-J's outputs. 
* The rule-based model (Minimal Generalization Learner, **MGL**) is fit by identifying consistent patterns in the data (e.g., which suffix is most often used with specific adjective classes).
* The analogical model (Generalized Context Model, **GCM**) is fit by analyzing how similar examples in the training data are distributed.
* The goal is to compare GPT-J's predictions with the rule-based and analogical cognitive models to determine which model better explains the LLM’s behavior for different linguistic phenomena (regular vs. irregular patterns, word frequencies, etc.).

### Generalization to Nonce Words

**Cognitive Models**
Train cognitive models on adjective-derivative pairs found in Pile (GPT-J's training corpus), and then see what nominalization they produce for unseen *nonce* (made up) adjectives.
* Example: train on pairs like available --> availability, selfish --> selfishness. At test time, the model would be asked to predict the corresponding noun for nonce adjectives with the same suffixes, like tegornable and friquish.
* The adjectives used include four possible suffixes: -able and -ish have *high* regularity for nominalization, and -ive and -ous have *low* regularity.
* The cognitive models are trained in two ways: word *type* perspective and word *token* perspective. The token version will be much more sensitive to word frequency in the training data.

**LLM Predictions**
To obtain LLM predictions, GPT-J was prompted with a text snippet containing the nonce adjective and asked to complete the nominalization.
* They use 12 different prompts (e.g., "Noun: ", "Turn the given adjective into a noun. ") to elicit the nominalized form and report results based on the average across the 12 prompt results.

**Regular Adjectives (-able and -ish)**
* Both cognitive models, MGL and GCM, always predict -ity for -able and -ness for -ish.
* GPT-J predicts -ity for -able, and it predicts -ness for -ish in all but two cases for one prompt
(turgeishity and prienishity).

**Low Regularity Adjectives (-ive and -ous)**
* MGL (rule-based) and GCM (analogy) give variable predictions for these classes. They agree on predictions for only 54% of the adjective types.  
* *Token-based GCM* (which considers word frequency) matches GPT-J’s predictions better than the rule-based MGL or type-based GCM.
* Example: Nonce Word "pepulative":
  * MGL (rule-based) predicts "-ity" (pepulativity) because it applies a general rule for adjectives ending in -ive.
  - GCM (analogy) is influenced by seen examples. There are more "-ity" derivatives overall for similar adjectives (e.g., “-lative”), but high-frequency neighbors like manipulativeness (1,544 occurrences) bias GCM toward "-ness."
  - GPT-J predicts "-ness" (e.g., *pepulativeness*), aligning with the token-based GCM.  
* Conclusion: GPT-J’s behavior supports analogy-based reasoning, particularly influenced by token frequency, over strict rule application.
 * While analogical generalization seems to be used for cases with high variability, LLMs may also use rules for highly regular patterns, aligning with dual-mechanism theories in morphology.

[Include Figure 1]

### Predictions for Seen Words
* Four groups of adjectives (see Table 3). R- denotes high regularity, while V- denotes high variability.
* GPT-J’s nominalization predictions for -ity vs. -ness were tested using 48,995 adjectives seen by the model (in Pile).
* Probability assignments for each suffix were compared against training data statistics.
Results were obtained averaging over the same 12 prompts.
* GPT-J’s predictions closely matched the suffix distribution in its training data. It consistently preferred the suffix with higher frequency in the training data, even for variable cases.
* This suggests reliance on analogically reasoning rather than strict rules, though highly regular patterns may still involve some rule-based reasoning.

[Include Table 3]

### Frequency Effects and Analogical Pressure

**Goal** 
* Analyze the model's preference for attested (seen in training data) vs. unattested (not seen) nominalized forms using log probabilities as a measure of confidence.
* Larger differences indicate higher confidence in the chosen form.
* Adjectives were grouped into low frequency (up to 10) and high frequency (100 and up) as measured in Pile. These groups allowed testing the impact of word frequency on GPT-J’s confidence.
* Question: "We have already seen that the most regular outcomes can be generated by analogy, but
could they instead be generated by rule?"

**Results**
* Frequency Effects: GPT-J consistently showed higher confidence for high-frequency derivatives across all adjective classes. This finding contradicts rule-based explanations.
* Analogical: Frequently encountered derivatives exert stronger influence on predictions, aligning with analogical generalization.
 * Example: In the R-NESS class, GPT-J confidently predicts -ness for the high frequency group, but reflects *analogical pressure* for the low frequency group. (If it was rule-based, it would always predict -ness regardless of frequency.)
 * Variability in confidence for the low-frequency words is linked to the similarity and frequency of the most related words (*heterogeneous neighborhood*), which is the underlying operation in analogical reasoning.
* Conclusion: "GPT-J learns adjective nominalization by implicitly storing derivatives in its model weights."
* I think there could be another handout all about the statistical methods used in this paper :)

## Human Use of Word Types Versus Tokens
Context: Humans generalize linguistic patterns based on word types (distinct categories), while GPT-J generalizes based on tokens (specific word occurrences and their frequencies in training data).

**Judgments of Nonce Words**
* 22 native English speakers evaluated nonce adjectives for their preference between -ity and -ness. GPT-J, GPT-4, and cognitive models (MGL and GCM) were compared to the human responses.
* Type-based GCM best matched human preferences, especially for the variable adjective classes.
* Token-based variants of GCM and MGL aligned more with LLMs but were less human-like.
* GPT-J matched human responses the least across the board.
* GPT-4 matched humans even less than GPT-J for variable classes, showing an over-reliance on token frequencies.
* Key Finding: Both GPT-J and GPT-4 heavily rely on token frequencies, limiting their ability to generalize like humans. Larger models like GPT-4 may exacerbate this issue (*inverse scaling effects*).

**Familiarity of Complex Words**

