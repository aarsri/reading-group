## All About ChatGPT

Presented by Aarohi Srivastava on May 24, 2024

See electronic version here: [https://github.com/aarsri/reading-group/blob/main/chatgpt/chatgpt.md](https://github.com/aarsri/reading-group/blob/main/chatgpt/chatgpt.md)

### Overview

OpenAI has three flagship text generation models, all of which are multilingual.
| | GPT-4o | GPT-4 Turbo | GPT-3.5 Turbo |
| - | ------ | ----------- | ------------- |
| inputs | text, image | text, image | text |
| context length | 128k | 128k | 16k |
| cost (per 1M tokens) for input | $5 | $10 | $0.50 |
| cost (per 1M tokens) for output | $15 | $30 | $1.50 |

The blog post [here](https://dev.to/maximsaplin/gpt-4-128k-context-it-is-not-big-enough-1h02) (Saplin, 2023) provides insight into what this context length means (for English).

Below are examples of short English texts and how many fit inside 128k tokens (Saplin, 2023):

<img src="smaller.png" width="600">

Below are examples of long English texts and how many fit inside 128k tokens (Saplin, 2023):

<img src="larger.png" width="600">

Note that context window is different from maximum sequence length, which is 4096 tokens for all the models. It is a different question of how easy it is for the model to digest or make use of so much information (see below).

Related to many of the issues regarding tokenization of nonstandard text or text from different scripts and low-resource languages, the following paper by Ahia et al. (2024) provides insight into the inequity in cost when using ChatGPT for different languages. 

[Do All Languages Cost the Same? Tokenization in the Era of Commercial Language Models](https://arxiv.org/pdf/2305.13707) (Ahia et al., 2024)

<img src="numberoftokens.png" width="400">

<img src="costratio.png" width="400">

### OpenAI API
Information about the OpenAI API can be found at: [https://platform.openai.com/docs/overview](https://platform.openai.com/docs/overview)

API calls can be done using curl, Python, and Node.js. In this handout, we focus on Python.

In addition to the GPT-based text generation models, OpenAI API can also be used for several other models, including DALL-E (text2image) and Whisper (speech2text).

Here are the basics to get started:
```
from openai import OpenAI
client = OpenAI()

completion = client.chat.completions.create(
  model="gpt-3.5-turbo",
  messages=[
    {"role": "system", "content": "You are a poetic assistant..."},
    {"role": "user", "content": "Compose a poem..."}
  ]
)

print(completion.choices[0].message)
```
The possible roles are system, user, and assistant. The `messages` list consists of turns of the conversation. The first element is typically to specify the behavior of the system, though this is not required. The generic message if omitted would be something like "You are a helpful assistant."


and then the roles "user" and "assistant" would alternate.

There is also a [fine-tuning UI](https://platform.openai.com/docs/guides/fine-tuning) that simply involves uploading the training and testing data and specifying hyperparameters (or using defaults). They suggest that the training size should be 50-100 well-crafted examples. The estimated cost to train on 100,000 tokens of text for 3 epochs is $2.40.

The OpenAI [cookbook](https://cookbook.openai.com) has lots of examples of accomplishing specific tasks.

### Directions of NLP Research


### Directions of Non-NLP Research


### Opinions from AI Researchers


### Usage and Thoughts from General Audience





AI is always framed as an assistant, and when it has a voice or name it is always a woman.
