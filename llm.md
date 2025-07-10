# Getting to Know Modern Language Models
Aarohi Srivastava | July 11, 2025

## From BERT to GPT: Encoder-Only vs. Decoder-Only Language Models

The rise of decoder-only models like GPT and LLAMA is tightly coupled with a paradigm shift in NLP tasks. Traditional NLP emphasized understanding (e.g., sentiment analysis, entity recognition, classification), while modern applications emphasize interaction and generation (e.g., chatbots, summarization, reasoning, coding, instruction-following). 

**Encoder-only** models (e.g., BERT) use bidirectional attention, where each token attends to all others in the sequence. They're trained with masked language modeling, where some tokens are hidden and the model predicts them. This allows the model to build contextualized representations of language, especially for classification or token-level tasks.

  Pretraining: "The [MASK] barked at the mailman."

**Decoder-only** models (e.g., GPT, LLaMA) use causal (unidirectional) attention, where each token only attends to previous tokens. They're trained with causal language modeling (next-token prediction). This setup is naturally suited to sequence generation and dialog.

  Pretraining: "The dog barked at the"

### Why are LLMs autoregressive and decoder-only?

* The shift toward decoder-only models reflects the dominance of generative, open-ended tasks in current NLP: chatbots, summarization, code generation, reasoning, etc.
* Masked language modeling disrupts the continuity of natural text during training, while causal language modeling keeps the flow intact, which may be important for learning discourse structure and pragmatic function.
* Decoder-only models support in-context learning (few-shot, zero-shot, CoT) without task-specific heads or retraining, which makes them ideal for large, general-purpose foundation models.
* Architecturally, they scale better with fewer complications (e.g., no need to coordinate an encoder-decoder interface or support bidirectional attention).

Encoder-only models remain strong in classification, retrieval, and analysis. They're typically more efficient when generation is not needed. For researchers in computational linguistics or language understanding, encoder models offer better layer-level interpretability and more modular control. At the same time, the rise of decoder-only LLMs does signal a shift in what tasks and capabilities are considered central to NLP.

### Do decoder-only models encode linguistic structure?

An influential finding from the "BERTology" era was that Transformer layers aligned with linguistic structure. Work like *BERT Rediscovers the Classical NLP Pipeline* showed that different layers specialize in different linguistic levels (POS tagging < syntax trees < dependency relations < coreference). These insights helped bridge deep learning with linguistic theory, and offered a compelling narrative for why these models “understand” language. But what about LLMs?

As we know, decoder-only models do capture linguistic structure, but the way they represent this information differs from encoder models:
* Causal attention means tokens only attend to previous tokens, so representations are inherently asymmetric. This limits certain types of structured representations, like complete parse trees, from forming as explicitly.
* Layer specialization is less clear. Instead of neat separation by linguistic task, capabilities seem to be more entangled and distributed.
* Probing tasks (e.g., syntactic agreement, morphological generalization) show high accuracy in LLaMA and GPT models, but these representations are harder to localize to specific layers.

**What does this mean?**
* Structural probes (e.g., linear classifiers predicting syntactic trees from hidden states) often work well, but not as cleanly or robustly as with BERT.
* Linguistic abilities may emerge more from general pretraining than from learning explicit structure.
* Encoder-based models remain more conducive to scientific study of language representations. Their clean separation of tasks and modular design makes them more accessible for analysis.

### Opportunities and Open Directions
* Can we reintroduce structural linguistic signals into LLMs?
* How can encoder-style probing be used to interpret or augment decoder-only LLMs?

---

## Model Landscape

| Model   | Year | Open Weights | Smallest Size  | Largest Size    | Tokenizer         | Vocab Size | Pretraining Data         | Objective             | Languages    |
| ------- | ---- | ------------ | -------------- | --------------- | ----------------- | ---------- | ------------------------ | --------------------- | ------------ |
| GPT-2   | 2019 | YES            | 117M           | 1.5B            | BPE     | \~50K      | WebText (8M docs)        | CLM                   | English      |
| GPT-3.5 | 2022 | NO            | 6.7B (Davinci) | 175B            | GPT BPE           | \~50K      | ?            | CLM                   | English      |
| GPT-4   | 2023 | NO            | -              | >500B           | Custom (Tiktoken) | \~100K     | ?            | Mixture of Objectives | Multilingual |
| LLaMA 2 | 2023 | YES            | 7B             | 65B             | SentencePiece BPE | 32K        | Common Crawl + books     | CLM                   | Multilingual |
| LLaMA 3 | 2024 | YES            | 8B             | 70B             | Custom BPE        | 128K       | Expanded corpus          | CLM                   | Multilingual |
| Mistral | 2023 | YES            | 7B             | 12.9B (Mixtral) | BPE               | 32K        | Web-scale corpus         | CLM                   | Multilingual |
| mT5     | 2020 | YES            | 60M            | 13B             | SentencePiece     | 250K       | C4 | MLM (T5-style)        | Multilingual         |

Tokenizer vocabulary and segmentation behavior can affect downstream performance, especially in low-resource or instruction-heavy setups. Some models (e.g., Llama 3) have customized tokenizers that make processing more efficient for common phrases and instruction setups, but could pose issues in nonstandard settings.

### What does "large" mean?

**Width** defines size of internal vector representations:
- LLaMA 2 7B: hidden dim = 4096
- LLaMA 2 13B: hidden dim = 5120
- LLaMA 2 65B: hidden dim = 8192

**Depth** is the number of layers:
- LLaMA 2 7B: 32 layers
- LLaMA 2 13B: 40 layers
- LLaMA 2 65B: 80 layers

**Total Parameter Count** is a function of width and depth, and also involves calculations for multi-head self-attention modules and the feedforward network parameters.

## Modern Evaluation Strategies

- Instruction Following
  - e.g., "Explain why the sky is blue."
  - Benchmarks: AlpacaEval, VicunaEval
- Reasoning & Math
  - e.g., "If Alice has 3 apples and gives 1 to Bob, how many does she have left?"
  - Benchmarks: GSM8K, MATH
- Multilingual Understanding
  - e.g., Translate "Good morning" to Swahili
  - Benchmarks: XTREME, FLORES, GLUE-X
- General Knowledge
  - e.g., "Who was the first president of the United States?"
  - Benchmarks: MMLU

### Prompting

- **Zero-shot**: "Translate: Bonjour"
- **Few-shot**:
  ```
  Translate the following:
  English: Hello → French: Bonjour
  English: Goodbye → French: Au revoir
  English: Thanks → French:
  ```
- **Chain-of-Thought**:
  ```
  Q: Mary had 5 oranges. She gave 2 to John and bought 3 more. How many does she have now?
  A: Mary starts with 5. She gives away 2, so she has 3. Then she buys 3 more, making it 6. Final answer: 6.
  ```
  
BERT-based models are evaluated with metrics like F1, accuracy on classification tasks, or EM/F1 on span-based QA, while LLMs are evaluated based on generation quality, reasoning consistency, and task transferability.

## Parameter-Efficient Fine-Tuning

Because LLMs are so large, fully fine-tuning all their parameters is often impractical due to memory, compute, and time constraints. Instead, we use parameter-efficient fine-tuning (PEFT) methods, which selectively update only a small subset of the model’s parameters. The key challenge then becomes strategically choosing which components to adapt, and how, in order to retain performance while keeping resource usage low.

### Prompt Tuning

- Learnable embeddings (`T × d_model`) prepended to input tokens.
- Only modifies input embedding table.
- Key advantage: Extremely lightweight.
- Best for tasks for which the model already shows zero/few-shot competence.

### Prefix Tuning

- Injects key/value vectors into each attention layer.
- Trainable parameters: `2 × T × d_model × num_layers`
- Key advantage: Layer-level signal injection.
- Best for mid-level tasks with strong contextual dependencies.

### Low-Rank Adaptation (LoRA)

- Only updates low-rank adapters for linear weights.
- Trainable parameters per layer per module: `2 × r × d`
- Hyperparameters:
  - **Rank** controls the size of the low-rank adaptation matrices (typically 4-16). A higher rank increases learning capacity but also introduces a greater risk of overfitting.
  - **Alpha** acts like a learning rate multiplier for the adapter, where higher values give stronger influence.
- Best for tasks needing deeper adaptation (e.g., domain shift, new outputs).
- Offers customization of target modules.

### Quantization

HuggingFace `transformers` supports native quantized loading and PEFT parameter injection. Use `bitsandbytes` for 4-bit and 8-bit loading. For example, adapter layers can be used in 16-bit while loading LLaMA in 4-bit.

### Code Example

```python
from peft import LoraConfig, get_peft_model
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig

bnb_config = BitsAndBytesConfig(load_in_4bit=True)
model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-2-7b-hf", quantization_config=bnb_config, device_map="auto")

tokenizer = AutoTokenizer.from_pretrained("meta-llama/Llama-2-7b-hf")

peft_config = LoraConfig(
    r=4,
    lora_alpha=16,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.1,
    bias="none",
    task_type="CAUSAL_LM"
)

model = get_peft_model(model, peft_config)
```

## Discussion Questions
* If you had the infrastructure to use a huge LLM for your research, what would want to do with it?
* If you could build a huge LLM, what would you want it to do?
* Are there tasks you work on that you think a decoder-only model shouldn't be able to solve (but might)?
* What do you think it means to “understand” language? Do you think Llama or GPT get close?
* What would a “linguistically informed” LLM look like to you?
