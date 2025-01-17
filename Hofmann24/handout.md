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

**Cognitive Models**: To probe the underlying generalization mechanisms in GPT-J, the authors employ two cognitive models, one rule-based and one analogical, and see which one best aligns with GPT-J's outputs. 
* The rule-based model is fit by identifying consistent patterns in the data (e.g., which suffix is most often used with specific adjective classes).
* The analogical model is fit by analyzing how similar examples in the training data are distributed.
* They compare GPT-J's predictions with the rule-based and analogical cognitive models to determine which model better explains the LLM’s behavior for different linguistic phenomena (regular vs. irregular patterns, word frequencies, etc.).

