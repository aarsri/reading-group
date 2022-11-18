# Multi-view Subword Regularization

Xinyi Wang, Sebastian Ruder, Graham Neubig @ NAACL 2021

Presented by Aarohi Srivastava on November 18, 2022

### Let's set the scene.
* Context: Pre-trained multilingual language models (e.g., mBERT, XLM-Roberta).
  * Pre-trained LMs operate by first arriving at a desirable vocabulary of tokens by applying a subword tokenization algorithm to the training corpus, and then learning appropriate embeddings for said tokens.
  * Popular multilingual LMs operate under the same paradigm.  The goal is to first create a shared multilingual vocabulary by applying some subword tokenization algorithm to data comprised of several languages.
* The problem: 
  * When segmenting text from multiple languages, for a word with the same meaning and usage, we may arrive at very different tokenizations.  Each of these tokens will eventually live somewhere in a shared embedding space, but depending on the severity of the discrepancy, they may live very far apart.  
    * EN | excitement
    * FR | excita tion
    * PT | excita ção
    * DE | Auf re gung
    * RU | волн ение
  * **This does not bode well for cross-lingual transfer, especially for low-resource and unseen languages.** 
* Their contributions: 
  * Examining how existing subword regularization (previously used in one- (LMs) or two-language (MT) settings) impacts cross-lingual transfer --> it helps a bit.
  * Developing multi-view subword regularization *for use in fine-tuning* to promote consistency between original tokenizations and regularized/probabilistic tokenizations --> this helps even more.

### Wait...isn't this already a thing?

#### Deterministic Segmentation
* Byte-Pair Encoding 
  * BPE is one of the most commonly used subword tokenization algorithms.  It is a *deterministic* segmentation algorithm.
  * Given a training corpus, the goal of the BPE algorithm is to create a vocabulary of tokens.  The vocabulary is initialized with all possible characters/symbols.  Based on *frequency*, new vocabulary items are created by progressively merging the most frequent pairs of tokens appearing in the training data, for a set number of merges.
  * In practice, BPE tokenization is applied to new data by looking at the characters in a word and applying merges greedily.  This means the most frequent words will be left whole, while rare or unseen words will be segmented into smaller pieces.  These pieces are not necessarily morphologically-sound.
* Unigram Language Model Method
  * The ULM method is another deterministic segmentation algorithm.  
  * You start with a large vocabulary, and keep pruning it for a set number of iterations with the goal of maximizing the training data likelihood under a ULM model of the subwords.  
  * For new data, this method decodes the segmentation using the Viterbi algorithm.

#### Probabilistic Segmentation and Subword Regularization
* Probabilistic segmentation methods are used to fight the *rigidity* of deterministic segmentation algorithms and support the multiple possible tokenizations given the same vocabulary/data.  
* This has been shown in past work to be especially helpful in contexts where the data may have typos, come from a closely related but unseen language, come from another domain, etc.  (i.e., the "real world"... :))
* BPE-dropout: When applying BPE to tokenize some text, randomly drop certain merge operations (given some probability).  This way, if we tokenize the same word multiple times (i.e., it appears multiple times in the data), we may end up with different tokenizations each time:
  * excitement
  * ex cite ment
  * e x c i t e m e n t
* ULM-sample: Sample the possible segmentations based on their language modeling scores instead of always picking the most likely one.

### But for Cross-lingual Transfer?
* The above methods have been successful to improve transfer in monolingual settings as well as in machine translation (two languages).  However, it was not explored as much for multilingual settings.
* Even though a multilingual LM has a shared multilingual embedding space, the way segmentation occurs can be very different when an algorithm like BPE is applied to text of different languages.
  * Low-resource languages will be *over-segmented* because even the most common words in these languages will not come close in frequency to words of high-resource languages.  So a word in a high-resource language may be one token, while a near-cognate in a closely-related langauge may be several tokens, baring no resemblance to its close relative.
  * This is also related to inconsistency in tokenization of related words in different langauges.  Taking the below examples, we might imagine that cross-lingual transfer between FR and PT is better supported than with EN.   
    * EN | excitement
    * FR | excita tion
    * PT | excita ção
  * If BPE-dropout had been applied, sometimes excitement may appear as excite ment, for example, motivating why that can help with cross-lingual transfer.
* Pre-training from scratch is (obviously) challenging for many reasons.  However, when we want to fine-tune a model on something other than the canonical downstream tasks like GLUE tasks, we may encounter a few issues:
  * The fine-tuning task requires so much disruption to the model that it is not able to perform well.
  * The nature of the fine-tuning input (particularly the text itself) is so different from the pre-training data that the model is not able to effectively use or transfer existing knowledge.
    * For instance, if we only do BPE-dropout in fine-tuning, there will be a big discrepancy between the pre-training and fine-tuning data.  This is a driving force for the authors' solution. 

### Multi-View Subword Regularization
* Multi-View Subword Regularization is "a method for learning from multiple segmented versions of the same data and enforcing the consistency of predictions over different segmentations.
* Given the BPE tokenization of a sequence $\hat{x}_i$, and the BPE-dropout tokenization of the same sequence $x'_i$, the objective for multi-view subword regularization is as follows:
* There are three parts:
  1. Det.  Seg CrossEnt: cross-entropy loss using the standard deterministic segmentation (maximizing the benefit of the pre-trained representations).
  2. Prob. Seg CrossEnt: cross-entropy loss using the probabilistic segmentation (allowing the model to learn from different segmentations of the same input).
  3. Consistency loss: calculates the distance between the model prediction distributions for the two input versions $\hat{x}_i$ and $x'_i$ (enforcing consistent predictions under different segmentations).
* The consistency loss portion has a big role to play, but there is a risk that it may not work that well if the model predicion becomes overly confident on certain classes.  This is addressed by modifying the consistency loss calculation by using softmax temperature $\tao$ to flatten the prediction distribution.  Higher temperatures make the probabilities more evenly distributed across the classes.
* Note that this technique is only used during fine-tuning training, not testing/inference.

### Let's see the receipts...
* Evaluation is done on the XTREME benchmark for zero-shot cross-lingual transfer from English to other languages.  
  * XTREME has a lot of tasks including text classification, structured prediction, sentence retrieval, and cross-lingual question answering.
* mBERT (BPE) and XLM-Roberta (ULM) are both tested.  The subword regularization (SR) method used depends on the model (BPE-dropout for mBERT, ULM-sample for XLM-Roberta).
* Hyperparameters (including BPE-dropout probability, ULM-sampling temperature) are mostly set based on performance on English NER. An exception is changing softmax temperature for QA tasks.



### Conclusion 
