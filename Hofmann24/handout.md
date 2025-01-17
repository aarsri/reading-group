# Derivational Morphology Reveals Analogical Generalization in Large Language Models

What mechanisms underlie linguistic generalization in large language models (LLMs)? Are these mechanisms rule-based or analogical?

## Background

**Rule-Based Generalization**: Past work has approached this question from a rule-based perspective, meaning for some linguistic concept (e.g., subject-verb agreement), they believe the LLM has inferred a set of symbolic rules from the training data.
* Example: If a word ends in -ish, add -ness to nominalize (selfish --> selfishness) 

**Analogical Generalization**: The other (understudied) option is that the LLM generalizes by comparing new inputs to previously-encountered examples ("analogically processes operating on stored exemplars").
* Example: Model has seen selfish → selfishness, so when it encounters a new word "friquish," it might predict “friquishness” by analogy to the known “selfish → selfishness” pair.
* Analogical processes can allow the model to handle variability and irregular patterns better than strict rule-based approaches.
* Analogical generalization is believed to be a central learning mechanism for humans that allows us to form abstract conjectures.

**Adjective Nominalization**: To distinguish between rule-based and analogical generalization in LLMs, the authors analyze how GPT-J learns English adjective nominalization wiith -ity and -ness, focusing on adjectives that already contain a derivational suffix (available --> availability, selfish --> selfishness). This is a narrow class that still exhibits plenty of variability to be able to observe analogical learning.

## Cognitive Models

To probe the underlying generalization mechanisms in GPT-J, the authors employ two cognitive models, one rule-based and one analogical, and see which one best aligns with GPT-J's outputs. 
* The rule-based model (Minimal Generalization Learner, **MGL**) is fit by identifying consistent patterns in the data (e.g., which suffix is most often used with specific adjective classes).
* The analogical model (Generalized Context Model, **GCM**) is fit by analyzing how similar examples in the training data are distributed.
* The goal is to compare GPT-J's predictions with the rule-based and analogical cognitive models to determine which model better explains the LLM’s behavior for different linguistic phenomena (regular vs. irregular patterns, word frequencies, etc.).

## Generalization to Nonce Words

**Cognitive Models**
Train cognitive models on adjective-derivative pairs found in Pile (GPT-J's training corpus), and then see what nominalization they produce for unseen *nonce* (made up) adjectives.
* Example: train on pairs like available --> availability, selfish --> selfishness. At test time, the model would be asked to predict the corresponding noun for nonce adjectives with the same suffixes, like tegornable and friquish.
* The adjectives used include four possible suffixes: -able and -ish have *high* regularity for nominalization, and -ive and -ous have *low* regularity.

**LLM Predictions**
To obtain LLM predictions, GPT-J was prompted with a text snippet containing the nonce adjective and asked to complete the nominalization.
* They use 12 different prompts (e.g., "Noun: ", "Turn the given adjective into a noun. ") to elicit the nominalized form and report results based on the average across the 12 prompt results.

**Regular Adjectives (-able and -ish)**
* Both cognitive models, MGL and GCM, always predict -ity for -able and -ness for -ish.
* GPT-J predicts -ity for -able, and it predicts -ness for -ish in all but two cases for one prompt
(turgeishity and prienishity).

**Low Regularity Adjectives (-ive and -ous)**
* MGL (rule-based model) and GCM (analogy-based model) give variable predictions for these classes. They agree on predictions for only 54% of the adjective types.  
* *Token-based GCM* (which considers word frequency) matches GPT-J’s predictions better than the rule-based MGL or type-based GCM.  
* Example: Nonce Word "pepulative":
  * MGL (rule-based) predicts "-ity" (pepulativity) because it applies a general rule for adjectives ending in -ive.
  - GCM (analogy-based) is influenced by seen examples. There are more "-ity" derivatives overall for similar adjectives (e.g., “-lative”), but high-frequency neighbors like manipulativeness (1,544 occurrences) bias GCM toward "-ness."
  - **GPT-J** predicts "-ness" (e.g., *pepulativeness*), aligning with the **token-based GCM**.  
* Conclusion: GPT-J’s behavior supports analogy-based reasoning, particularly influenced by token frequency, over strict rule application.

## Predictions for Seen Words

## Frequency Effects and Analogical Pressure

## Human Use of Word Types Versus Tokens
