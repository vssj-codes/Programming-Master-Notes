# Table-of-Contents

<!-- toc -->

- [Sampling algorithms, SFT, RLHF](#sampling-algorithms-sft-rlhf)
  * [**Github Repo (for code):** [**https://github.com/vv176/Week1_AI_Engineering**](https://github.com/vv176/Week1_AI_Engineering "https://github.com/vv176/Week1_AI_Engineering")](#github-repo-for-code-httpsgithubcomvv176week1_ai_engineeringhttpsgithubcomvv176week1_ai_engineering-httpsgithubcomvv176week1_ai_engineering)
  * [**1. GPT-2 and Autoregressive Language Modeling**](#1-gpt-2-and-autoregressive-language-modeling)
        * [GPT-2 is a decoder-only Transformer trained to predict the next token in a sequence.](#gpt-2-is-a-decoder-only-transformer-trained-to-predict-the-next-token-in-a-sequence)
      - [**Training Objective**](#training-objective)
        * [For a sequence x1,x2,...,xTx_1, x_2, ..., x_T:](#for-a-sequence-x1x2xtx_1-x_2--x_t)
      - [**Inference Process**](#inference-process)
        * [1. Tokenize input prompt.](#1-tokenize-input-prompt)
        * [2. Forward pass → compute logits for next token.](#2-forward-pass-%E2%86%92-compute-logits-for-next-token)
        * [3. Convert logits to probabilities (softmax).](#3-convert-logits-to-probabilities-softmax)
        * [4. Sample or select next token.](#4-sample-or-select-next-token)
        * [5. Append and repeat until stop condition (e.g., EOS token).](#5-append-and-repeat-until-stop-condition-eg-eos-token)
      - [**Logits**](#logits)
        * [**Definition**: Raw, unnormalized scores from the model.](#definition-raw-unnormalized-scores-from-the-model)
        * [**Usage**: Transformed to probabilities with softmax.](#usage-transformed-to-probabilities-with-softmax)
        * [**Example:**](#example)
      - [**Sampling Strategies**](#sampling-strategies)
  * [**2. Attention Mechanism**](#2-attention-mechanism)
      - [**Key Concepts**](#key-concepts)
        * [For each token:](#for-each-token)
        * [**Query (Q):** What the token is looking for.](#query-q-what-the-token-is-looking-for)
        * [**Key (K):** What the token represents.](#key-k-what-the-token-represents)
        * [**Value (V):** The information carried by the token.](#value-v-the-information-carried-by-the-token)
      - [**Scaled Dot-Product Attention**](#scaled-dot-product-attention)
      - [**Multi-Head Attention**](#multi-head-attention)
        * [Multiple attention heads look at different relationships simultaneously.](#multiple-attention-heads-look-at-different-relationships-simultaneously)
        * [Outputs are concatenated and projected back to hidden size.](#outputs-are-concatenated-and-projected-back-to-hidden-size)
      - [**Causal Masking in GPT**](#causal-masking-in-gpt)
        * [Ensures tokens only attend to past and current tokens.](#ensures-tokens-only-attend-to-past-and-current-tokens)
        * [Prevents "cheating" by looking at future tokens.](#prevents-cheating-by-looking-at-future-tokens)
      - [**Example**](#example)
        * [Sentence: **"The cat sat"**](#sentence-the-cat-sat)
        * [When predicting **"sat"** , attention might assign:](#when-predicting-sat--attention-might-assign)
        * [**"cat"** → 0.7](#cat-%E2%86%92-07)
        * [**"The"** → 0.3](#the-%E2%86%92-03)
  * [**3. Extra Tools**](#3-extra-tools)
      - [**tiktoken**](#tiktoken)
      - [Token vocabularies may **change with each model** for efficiency and compression.](#token-vocabularies-may-change-with-each-model-for-efficiency-and-compression)

<!-- tocstop -->

---

## Sampling algorithms, SFT, RLHF

### **Github Repo (for code):** [**https://github.com/vv176/Week1_AI_Engineering**](https://github.com/vv176/Week1_AI_Engineering "<span data-offset-key=\"bqmkk-1-0\" style=\"font-family: Lato-Regular; box-sizing: inherit; scrollbar-width: none; font-weight: bold; text-decoration: underline;\"><span data-text=\"true\" style=\"font-family: Lato-Regular; box-sizing: inherit; scrollbar-width: none;\">https://github.com/vv176/Week1_AI_Engineering</span></span>")

  

### **1. GPT-2 and Autoregressive Language Modeling**

###### GPT-2 is a decoder-only Transformer trained to predict the next token in a sequence.  

##### **Training Objective**

###### For a sequence x1,x2,...,xTx_1, x_2, ..., x_T:

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756811993/Screenshot_2025-09-02_at_4.49.42_PM_zl0thj.png)

##### **Inference Process** 

###### 1. Tokenize input prompt.

###### 2. Forward pass → compute logits for next token.

###### 3. Convert logits to probabilities (softmax).

###### 4. Sample or select next token.

###### 5. Append and repeat until stop condition (e.g., EOS token). 

  

##### **Logits**

###### **Definition**: Raw, unnormalized scores from the model.

###### **Usage**: Transformed to probabilities with softmax. 

###### **Example:**

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756811504/Screenshot_2025-09-02_at_4.41.25_PM_rwwvea.png)

  

##### **Sampling Strategies** 

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756811586/Screenshot_2025-09-02_at_4.42.47_PM_eqapi6.png)

  

### **2. Attention Mechanism**

The attention mechanism enables models to focus on relevant parts of the input when generating or encoding tokens. 

  

##### **Key Concepts**

###### For each token:

###### **Query (Q):** What the token is looking for.

###### **Key (K):** What the token represents.

###### **Value (V):** The information carried by the token. 

  

##### **Scaled Dot-Product Attention** 

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756811892/Screenshot_2025-09-02_at_4.47.56_PM_ng2ivs.png)

  

##### **Multi-Head Attention**

###### Multiple attention heads look at different relationships simultaneously.

###### Outputs are concatenated and projected back to hidden size. 

  

##### **Causal Masking in GPT** 

###### Ensures tokens only attend to past and current tokens.

###### Prevents "cheating" by looking at future tokens.

  

##### **Example**

###### Sentence: **"The cat sat"**

###### When predicting **"sat"** , attention might assign:

###### **"cat"** → 0.7

###### **"The"** → 0.3 

  

### **3. Extra Tools**

##### [**tiktoken**](https://github.com/openai/tiktoken "<u style=\"font-family: Lato-Regular; box-sizing: inherit; scrollbar-width: none;\"><strong style=\"font-family: Lato-Regular; box-sizing: inherit; scrollbar-width: none; font-weight: 700;\">tiktoken</strong></u>")  — OpenAI’s library for fast tokenization.

##### Token vocabularies may **change with each model** for efficiency and compression.