# 🧠 From BERT to GPT: A Deep Dive into Modern Large Language Models (LLMs)

*A technical guide and discussion handout for NLP researchers familiar with encoder-only models, exploring the structure, usage, fine-tuning, and evaluation of large-scale decoder models.*

---

## 1. Introduction: Why LLMs, and Why Now?

In the shift from encoder-based models (e.g., BERT, RoBERTa, mT5) to large-scale decoder-only architectures like LLaMA, the field of NLP has embraced models capable of not only understanding language but **generating and reasoning with it**. These models underpin applications such as chatbots, code assistants, agents, and instruction-following systems.

**Key architectural difference**:

- **Encoder models**: Bidirectional, optimized for tasks requiring context from both directions (e.g., classification, span extraction).
- **Decoder models**: Causal (unidirectional), optimized for sequence generation, auto-completion, and free-form outputs.

**Discussion prompts:**

- Are encoder-only models obsolete in current NLP workflows?
- What applications benefit most from decoder-only models?

---

## 2. Decoder-Only vs. Encoder-Only Architectures

### 2.1 Input Processing

- **Encoder-only (e.g., BERT)**: Ingests a full sentence with attention across all tokens. Often requires input segmentation ([CLS], [SEP]) and special treatment for pairs.
- **Decoder-only (e.g., LLaMA)**: Tokens are processed autoregressively. The model can only attend to previous tokens in the sequence.

### 2.2 Pre-training Objective

- **Encoders**: Masked Language Modeling (MLM).
  - Example: "The [MASK] barked at the mailman."
- **Decoders**: Causal Language Modeling (CLM).
  - Example: "The dog barked at the"

### 2.3 Output Behavior

- **Encoders**: Output a representation per token (good for classification).
- **Decoders**: Generate token by token, producing a sequence (good for open-ended generation).

### 2.4 Inference Use Cases

- **Encoders**: Sentence classification, NER, QA (span extraction).
- **Decoders**: Chatbots, summarization, translation, instruction following.

---

## 3. Model Landscape Overview

| Model   | Year | Open Weights | Smallest Size  | Largest Size    | Tokenizer         | Vocab Size | Pretraining Data         | Objective             | Languages    |
| ------- | ---- | ------------ | -------------- | --------------- | ----------------- | ---------- | ------------------------ | --------------------- | ------------ |
| GPT-2   | 2019 | ✅            | 117M           | 1.5B            | BPE (English)     | \~50K      | WebText (8M docs)        | CLM                   | English      |
| GPT-3.5 | 2022 | ❌            | 6.7B (Davinci) | 175B            | GPT BPE           | \~50K      | Not disclosed            | CLM                   | English      |
| GPT-4   | 2023 | ❌            | ?              | >500B           | Custom (Tiktoken) | \~100K     | Not disclosed            | Mixture of Objectives | Multilingual |
| LLaMA 2 | 2023 | ✅            | 7B             | 65B             | SentencePiece BPE | 32K        | Common Crawl + books     | CLM                   | Multilingual |
| LLaMA 3 | 2024 | ✅            | 8B             | 70B             | Custom BPE        | 128K       | Expanded corpus          | CLM                   | Multilingual |
| Mistral | 2023 | ✅            | 7B             | 12.9B (Mixtral) | BPE               | 32K        | Web-scale corpus         | CLM                   | Multilingual |
| mT5     | 2020 | ✅            | 60M            | 13B             | SentencePiece     | 250K       | C4, multilingual corpora | MLM (T5-style)        | 100+         |

### Tokenizer Comparison

- **LLaMA 3**: 128K vocabulary, optimized for compression of high-frequency phrases. Reduces token count = faster inference.
- **LLaMA 2**: 32K SentencePiece. Well-rounded but less efficient than LLaMA 3 on instruction data.
- **mT5**: 250K SentencePiece for multilingual coverage. Less efficient for English-only.
- **GPT**: \~50K BPE, optimized for English and code.

**Takeaway**: Tokenizer vocabulary and segmentation behavior affect downstream performance, especially in low-resource or instruction-heavy setups.

---

## 4. What Does "Bigger Model" Mean?

Model capacity is typically described in terms of **width** (hidden size, intermediate size) and **depth** (number of layers).

### 4.1 Width (Hidden Dimension)

- Defines size of internal vector representations.
- LLaMA 2 7B: hidden dim = 4096
- LLaMA 2 13B: hidden dim = 5120
- LLaMA 2 65B: hidden dim = 8192

### 4.2 Depth (Number of Layers)

- LLaMA 2 7B: 32 layers
- LLaMA 2 13B: 40 layers
- LLaMA 2 65B: 80 layers

### 4.3 Total Parameter Count

A function of:

- `#layers × hidden_dim² × 12` (approx for transformer-based models)

**Discussion prompts:**

- Are deeper models always better than wider ones?
- How do scaling laws guide architecture design?

---

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

