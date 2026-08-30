# Table-of-Contents

<!-- toc -->

- [What happens when LLM receives your query](#what-happens-when-llm-receives-your-query)
        * [**Github Repo (for code):** [**https://github.com/vv176/Week1_AI_Engineering**](https://github.com/vv176/Week1_AI_Engineering)](#github-repo-for-code-httpsgithubcomvv176week1_ai_engineeringhttpsgithubcomvv176week1_ai_engineering)
  * [**1. Context Engineering, Prompt Engineering, vs. Fine-tuning**](#1-context-engineering-prompt-engineering-vs-fine-tuning)
  * [**2. How ChatGPT Works**](#2-how-chatgpt-works)
      - [A high-level flow of how text input is processed inside models like GPT-3/4:](#a-high-level-flow-of-how-text-input-is-processed-inside-models-like-gpt-34)
  * [**3. Tokenization and Byte Pair Encoding (BPE)**](#3-tokenization-and-byte-pair-encoding-bpe)
        * [Tokenization splits text into smaller units (tokens) for the model to process.](#tokenization-splits-text-into-smaller-units-tokens-for-the-model-to-process)
      - [**Token IDs**](#token-ids)
        * [Example:](#example)
      - [**Positional Embeddings**](#positional-embeddings)
        * [Transformers are order-agnostic. Positional embeddings inject sequence order.](#transformers-are-order-agnostic-positional-embeddings-inject-sequence-order)
      - [**Final embedding for each token:**](#final-embedding-for-each-token)
        * [Final Embedding = Token Embedding + Positional Embedding](#final-embedding--token-embedding--positional-embedding)
      - [**Mergeable Tokens**](#mergeable-tokens)

<!-- tocstop -->

---

## What happens when LLM receives your query

###### **Github Repo (for code):** [**https://github.com/vv176/Week1_AI_Engineering**](https://github.com/vv176/Week1_AI_Engineering)

  

### **1. Context Engineering, Prompt Engineering, vs. Fine-tuning** 

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756808815/Screenshot_2025-09-02_at_3.49.05_PM_k5moou.png)

  

### **2. How ChatGPT Works** 

##### A high-level flow of how text input is processed inside models like GPT-3/4: 

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756809781/Screenshot_2025-09-02_at_4.08.57_PM_zfhrwi.png)

  

### **3. Tokenization and Byte Pair Encoding (BPE)** 

###### Tokenization splits text into smaller units (tokens) for the model to process.

##### **Token IDs**

- Each token is assigned a unique integer ID.
- These IDs are used to look up embeddings in the model’s vocabulary matrix.

###### Example: 

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756810517/Screenshot_2025-09-02_at_4.24.56_PM_mpujlj.png)

  

##### **Positional Embeddings**

###### Transformers are order-agnostic. Positional embeddings inject sequence order. 

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756810832/Screenshot_2025-09-02_at_4.30.09_PM_mrgdli.png)

##### **Final embedding for each token:** 

###### Final Embedding = Token Embedding + Positional Embedding 

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756811253/Screenshot_2025-09-02_at_4.37.11_PM_ifreyb.png)

##### **Mergeable Tokens** 

- Tokenizers start from single characters or bytes.
- Frequently co-occurring pairs are merged to form larger, efficient tokens.

Example:    **p  +  izza   -->  pizza**