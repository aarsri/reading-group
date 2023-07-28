## How Can We Know What Language Models Know? 

Zhengbao Jiang, Frank F. Xu, Jun Araki, Graham Neubig - TACL 2020

Presented by Aarohi Srivastava on July 28, 2023

## Introduction
Main Idea: We know prompting can be a brittle method. But many people use prompting as a way of extracting knowledge from an LM and evaluating what the LM knows.  Arguably, this is unfair.

This paper: How can we discover/mine/generate prompts in the most appropriate form for the LM.

Achievement: Using their method, BERT's performance on the LAMA benchmark for knowledge extraction (with reformulated prompts) increases by 8 points, indicating:
1. Their method helps formulate a prompt in such a way that it will actually work on BERT.
2. BERT knows more than you may have thought.

You may be wondering how you can give a prompt to BERT (rather than GPT) -- they just mean test BERT with the fill-mask objective (so the generated text could probably not be more than a token).

