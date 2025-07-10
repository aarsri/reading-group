# Title

## 1  From BERT to GPT: Encoder-only vs. Decoder-only Language Models

The rise of decoder-only models like GPT and LLAMA is tightly coupled with a paradigm shift in NLP tasks. Traditional NLP emphasized understanding (e.g., sentiment analysis, entity recognition, classification), while modern applications emphasize interaction and generation (e.g., chatbots, summarization, reasoning, coding, instruction-following). 

**Encoder-only** models (e.g., BERT) use bidirectional attention, where each token attends to all others in the sequence. They're trained with masked language modeling, where some tokens are hidden and the model predicts them. This allows the model to build contextualized representations of language, especially for classification or token-level tasks.

  Pretraining: "The [MASK] barked at the mailman."

**Decoder-only** models (e.g., GPT, LLaMA) use causal (unidirectional) attention, where each token only attends to previous tokens. They're trained with causal language modeling (next-token prediction). This setup is naturally suited to sequence generation and dialog.

  Pretraining: "The dog barked at the"

### 1.1  Why are LLMs autoregressive and decoder-only?

* The shift toward decoder-only models reflects the dominance of generative, open-ended tasks in current NLP: chatbots, summarization, code generation, reasoning, etc.
* Masked language modeling disrupts the continuity of natural text during training, while causal language modeling keeps the flow intact, which may be important for learning discourse structure and pragmatic function.
* Decoder-only models support in-context learning (few-shot, zero-shot, CoT) without task-specific heads or retraining, which makes them ideal for large, general-purpose foundation models.
* Architecturally, they scale better with fewer complications (e.g., no need to coordinate an encoder-decoder interface or support bidirectional attention).

Encoder-only models remain strong in classification, retrieval, and analysis. They're typically more efficient when generation is not needed. For researchers in computational linguistics or language understanding, encoder models offer better layer-level interpretability and more modular control. At the same time, the rise of decoder-only LLMs does signal a shift in what tasks and capabilities are considered central to NLP.

### 1.2  Do decoder-only models encode linguistic structure?

An influential finding from the "BERTology" era was that Transformer layers aligned with linguistic structure. Work like *BERT Rediscovers the Classical NLP Pipeline* showed that different layers specialize in different linguistic levels (POS tagging < syntax trees < dependency relations < coreference). These insights helped bridge deep learning with linguistic theory, and offered a compelling narrative for why these models “understand” language. But what about LLMs?

As we know, decoder-only models do capture linguistic structure, but the way they represent this information differs from encoder models:
* Causal attention means tokens only attend to previous tokens, so representations are inherently asymmetric. This limits certain types of structured representations, like complete parse trees, from forming as explicitly.
* Layer specialization is less clear. Instead of neat separation by linguistic task, capabilities seem to be more entangled and distributed.
* Probing tasks (e.g., syntactic agreement, morphological generalization) show high accuracy in LLaMA and GPT models, but these representations are harder to localize to specific layers.

**What does this mean?**
* Structural probes (e.g., linear classifiers predicting syntactic trees from hidden states) often work well, but not as cleanly or robustly as with BERT.
* Linguistic abilities may emerge more from general pretraining than from learning explicit structure.
* Encoder-based models remain more conducive to scientific study of language representations. Their clean separation of tasks and modular design makes them more accessible for analysis.

### 1.3  Opportunities and Open Directions
* Can we reintroduce structural linguistic signals into LLMs?
* How can encoder-style probing be used to interpret or augment decoder-only LLMs?

---

## 2. Model Landscape

| Model   | Year | Open Weights | Smallest Size  | Largest Size    | Tokenizer         | Vocab Size | Pretraining Data         | Objective             | Languages    |
| ------- | ---- | ------------ | -------------- | --------------- | ----------------- | ---------- | ------------------------ | --------------------- | ------------ |
| GPT-2   | 2019 | YES            | 117M           | 1.5B            | BPE (English)     | \~50K      | WebText (8M docs)        | CLM                   | English      |
| GPT-3.5 | 2022 | NO            | 6.7B (Davinci) | 175B            | GPT BPE           | \~50K      | Not disclosed            | CLM                   | English      |
| GPT-4   | 2023 | NO            | ?              | >500B           | Custom (Tiktoken) | \~100K     | Not disclosed            | Mixture of Objectives | Multilingual |
| LLaMA 2 | 2023 | YES            | 7B             | 65B             | SentencePiece BPE | 32K        | Common Crawl + books     | CLM                   | Multilingual |
| LLaMA 3 | 2024 | YES            | 8B             | 70B             | Custom BPE        | 128K       | Expanded corpus          | CLM                   | Multilingual |
| Mistral | 2023 | YES            | 7B             | 12.9B (Mixtral) | BPE               | 32K        | Web-scale corpus         | CLM                   | Multilingual |
| mT5     | 2020 | YES            | 60M            | 13B             | SentencePiece     | 250K       | C4, multilingual corpora | MLM (T5-style)        | 100+         |

### Tokenizer Comparison

- **GPT**: \~50K BPE, optimized for English and code.
- **LLaMA 2**: 32K vocabulary with SentencePiece BPE.
- **LLaMA 3**: 128K vocabulary, optimized for compression of high-frequency phrases. Reduces token count for faster inference.
- **mT5**: 250K SentencePiece for multilingual coverage. Less efficient for English-only.

Tokenizer vocabulary and segmentation behavior can affect downstream performance, especially in low-resource or instruction-heavy setups.

## 4. What Does "Large" Mean?

**Width** defines size of internal vector representations:
- LLaMA 2 7B: hidden dim = 4096
- LLaMA 2 13B: hidden dim = 5120
- LLaMA 2 65B: hidden dim = 8192

**Depth** is the number of layers:
- LLaMA 2 7B: 32 layers
- LLaMA 2 13B: 40 layers
- LLaMA 2 65B: 80 layers

**Total Parameter Count** is a function of width and depth, and also involves calculations for multi-head self-attention modules and the feedforward network parameters.

## 5. Modern Evaluation Strategies

### 5.1 Task Categories with Examples

- **Instruction Following**
  - e.g., "Explain why the sky is blue."
  - Benchmarks: AlpacaEval, VicunaEval
- **Reasoning & Math**
  - e.g., "If Alice has 3 apples and gives 1 to Bob, how many does she have left?"
  - Benchmarks: GSM8K, MATH
- **Multilingual Understanding**
  - e.g., Translate "Good morning" to Swahili
  - Benchmarks: XTREME, FLORES, GLUE-X
- **General Knowledge**
  - e.g., "Who was the first president of the United States?"
  - Benchmarks: MMLU

### 5.2 Prompting Paradigms

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

### 5.3 How Evaluation Differs from BERT-Style Models

- **BERT-based** models are evaluated with metrics like F1, accuracy on classification tasks, or EM/F1 on span-based QA.
- **LLMs like LLaMA** are evaluated based on generation quality, reasoning consistency, and task transferability—typically with GPT-style prompting and multi-turn conversations.

**Discussion prompts:**

- Can we trust benchmark scores for multi-turn LLMs?
- How should LLaMA-based models be evaluated differently from encoder-only models?

---

## 6. Prompting vs. Fine-Tuning: Task Formatting Examples

### Example Task: Sentiment Classification

**Prompting (few-shot)**:

```
Review: "The movie was boring and predictable."
Sentiment: Negative
--
Review: "I loved the cinematography and the music."
Sentiment: Positive
--
Review: "The plot lacked depth and characters were flat."
Sentiment:
```

**Fine-Tuning Format (Supervised)**:

```json
{"input": "The movie was boring and predictable.", "label": "Negative"}
{"input": "I loved the cinematography and the music.", "label": "Positive"}
```

### Example Task: Text Generation

**Prompting**:

```
Write a short story about a robot learning to paint.
```

**Fine-Tuning Format**:

```json
{"input": "Write a short story about a robot learning to paint.", "output": "Once upon a time..."}
```

---

## 7. PEFT Techniques in Depth

### 7.1 Prompt Tuning

- Learnable embeddings (`T × d_model`) prepended to input tokens.
- Only modifies input embedding table.
- ✅ Extremely lightweight
- ❌ Context length is limited

### 7.2 Prefix Tuning

- Injects key/value vectors into each attention layer.
- Parameters: `2 × T × d_model × num_layers`
- ✅ Layer-level signal injection
- ❌ More memory than prompt tuning

### 7.3 LoRA (Low-Rank Adaptation)

- Only updates low-rank adapters for linear weights (e.g., q\_proj, v\_proj).
- Formula: `W_new = W + A @ B`, with A = `[d × r]`, B = `[r × d]`
- Parameters per layer per module: `2 × r × d`
- ✅ Efficient with memory, works well with quantization
- ❌ May need tuning for rank, alpha, target modules

**Design Considerations with LLaMA**:

- For LLaMA 7B, common config: `r=4`, `alpha=16`, `target_modules=['q_proj', 'v_proj']`
- Adapter layers can be used in 16-bit while loading LLaMA in 4-bit with `bitsandbytes`

---

## 8. Code Examples

### 8.1 LoRA with Transformers

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

### 8.2 Preparing Prompt Data for Fine-Tuning

```python
from datasets import load_dataset
from transformers import DataCollatorForLanguageModeling

data = load_dataset("json", data_files="ft_data.jsonl")

def format_prompt(example):
    return {"text": f"### Instruction:\n{example['input']}\n\n### Response:\n{example['output']}"}

data = data.map(format_prompt)
collator = DataCollatorForLanguageModeling(tokenizer, mlm=False)
```

---

## 9. Memory & Training Efficiency

### 9.1 Quantization

- Use `bitsandbytes` for 4-bit and 8-bit loading.
- HuggingFace `transformers` supports native quant loading + PEFT injection.

### 9.2 Memory Saving Tips

- Use `gradient_checkpointing=True`
- Set `gradient_accumulation_steps > 1`
- Enable `flash_attention` if supported

---

## 10. Resources for Experimentation

- 🤖 [LLaMA 3 Weights & Tokenizer](https://github.com/facebookresearch/llama)
- 🧹 [HuggingFace PEFT](https://github.com/huggingface/peft)
- 🔬 [Evaluation Benchmarks](https://paperswithcode.com/benchmark/language-models)
- 💠 [AI-Commandos Language-Specific LLaMA2](https://github.com/AI-Commandos/LLaMa2lang)

---

## 11. Closing Discussion Questions

- What does it mean for a model to "understand" a task in the PEFT era?
- Will PEFT methods scale with even larger models (e.g., 1T+)?
- How do we ensure evaluation matches the real-world use cases?
- Can language-specific models outperform multilingual ones with small-scale fine-tuning?
- Can LoRA and prefix tuning be combined meaningfully?

---

