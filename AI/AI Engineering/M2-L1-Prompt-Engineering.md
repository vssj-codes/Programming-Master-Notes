# Table-of-Contents

<!-- toc -->

- [Best practices](#best-practices)
        * [**Github repo (for code):** [**https://github.com/vv176/SimpleExamples**](https://github.com/vv176/SimpleExamples)](#github-repo-for-code-httpsgithubcomvv176simpleexampleshttpsgithubcomvv176simpleexamples)
- [**Topic 1: Prompt Engineering — What, Why & How**](#topic-1-prompt-engineering--what-why--how)
      - [**🔍 What is Prompt Engineering?**](#%F0%9F%94%8D-what-is-prompt-engineering)
        * [Prompt Engineering is the art and science of **adapting the behavior of a large language model (LLM)**](#prompt-engineering-is-the-art-and-science-of-adapting-the-behavior-of-a-large-language-model-llm)
        * [to perform a desired task — without changing its internal weights, architecture, or doing any fine-tuning.](#to-perform-a-desired-task--without-changing-its-internal-weights-architecture-or-doing-any-fine-tuning)
        * [It’s the process of shaping **what the model does**, just by tweaking what you say to it (i.e., the prompt).](#its-the-process-of-shaping-what-the-model-does-just-by-tweaking-what-you-say-to-it-ie-the-prompt)
      - [**💡 Why Prompt Engineering?**](#%F0%9F%92%A1-why-prompt-engineering)
        * [Vivek called it the **"most underrated yet crucial part"** of AI engineering, and made one thing very clear:](#vivek-called-it-the-most-underrated-yet-crucial-part-of-ai-engineering-and-made-one-thing-very-clear)
        * [❝ In production-grade AI applications, **90% of the grunt work** is prompt engineering. ❞](#%E2%9D%9D-in-production-grade-ai-applications-90%25-of-the-grunt-work-is-prompt-engineering-%E2%9D%9E)
        * [It's not just about writing clever instructions — it's about **making LLMs behave the way your product needs**, under unpredictable, noisy, and diverse conditions.](#its-not-just-about-writing-clever-instructions--its-about-making-llms-behave-the-way-your-product-needs-under-unpredictable-noisy-and-diverse-conditions)
      - [**🛠 Analogy: ML Models Before Prompting Era**](#%F0%9F%9B%A0-analogy-ml-models-before-prompting-era)
        * [In the traditional ML world:](#in-the-traditional-ml-world)
        * [This meant _huge development overhead_ for every new task.](#this-meant-_huge-development-overhead_-for-every-new-task)
      - [**⚙ What Prompt Engineering Unlocks**](#%E2%9A%99-what-prompt-engineering-unlocks)
        * [❝ Just supply the right prompt, and the behavior adapts. That’s your superpower. ❞](#%E2%9D%9D-just-supply-the-right-prompt-and-the-behavior-adapts-thats-your-superpower-%E2%9D%9E)
      - [**🧬 Key Insight: Prompt Engineering = Behavior Hacking Without Code**](#%F0%9F%A7%AC-key-insight-prompt-engineering--behavior-hacking-without-code)
        * [Prompt engineering is about:](#prompt-engineering-is-about)
        * [You’re not editing the model’s neurons — you're manipulating its perception.](#youre-not-editing-the-models-neurons--youre-manipulating-its-perception)
        * [It’s like giving the same person different instructions and getting them to write poems, solve math, or](#its-like-giving-the-same-person-different-instructions-and-getting-them-to-write-poems-solve-math-or)
        * [summarize policy — **without ever changing who they are.**](#summarize-policy--without-ever-changing-who-they-are)
      - [**TL; DR;**](#tl-dr)
- [**Topic 2: The Myth of Prompt Engineering**](#topic-2-the-myth-of-prompt-engineering)
    + [**❌ Myth #1: “Prompting is Easy”**](#%E2%9D%8C-myth-%231-prompting-is-easy)
      - [Many believe:](#many-believe)
      - [✅ But in reality:](#%E2%9C%85-but-in-reality)
        * [**Prompting is hard-core engineering** — often messier, more unpredictable, and more trial-and-error than writing traditional software.](#prompting-is-hard-core-engineering--often-messier-more-unpredictable-and-more-trial-and-error-than-writing-traditional-software)
    + [**❌ Myth #2: “It’s Not Engineering at All”**](#%E2%9D%8C-myth-%232-its-not-engineering-at-all)
        * [People question the term “engineering” in prompt engineering.](#people-question-the-term-engineering-in-prompt-engineering)
        * [Vivek addressed this directly:](#vivek-addressed-this-directly)
        * [❝ Prompt engineering requires more engineering than deterministic code. You’ll understand why when you try to test and debug LLM behavior. ❞](#%E2%9D%9D-prompt-engineering-requires-more-engineering-than-deterministic-code-youll-understand-why-when-you-try-to-test-and-debug-llm-behavior-%E2%9D%9E)
    + [**⚙ So, Why Is It Actually Engineering?**](#%E2%9A%99-so-why-is-it-actually-engineering)
        * [Because it shares the same **rigor, iteration, and debugging** mindset as software engineering — but applied to a **non-deterministic black box.**](#because-it-shares-the-same-rigor-iteration-and-debugging-mindset-as-software-engineering--but-applied-to-a-non-deterministic-black-box)
        * [You still need to:](#you-still-need-to)
        * [The only difference? Your code is not **executable logic**, it's **natural language** with embedded logic.](#the-only-difference-your-code-is-not-executable-logic-its-natural-language-with-embedded-logic)
    + [**🧪 Software Engineering vs. Prompt Engineering: A Testing Analogy**](#%F0%9F%A7%AA-software-engineering-vs-prompt-engineering-a-testing-analogy)
      - [**✅ In Traditional Software (Deterministic)**](#%E2%9C%85-in-traditional-software-deterministic)
        * [if user_id is None:](#if-user_id-is-none)
        * [raise NullPointerException](#raise-nullpointerexception)
        * [→ Always fails for None. Always fixable with one if-check.](#%E2%86%92-always-fails-for-none-always-fixable-with-one-if-check)
      - [**❌ In Prompt Engineering (Non-Deterministic)**](#%E2%9D%8C-in-prompt-engineering-non-deterministic)
        * [❝ Same prompt, same input → 3 different behaviors. ❞](#%E2%9D%9D-same-prompt-same-input-%E2%86%92-3-different-behaviors-%E2%9D%9E)
        * [This means:](#this-means)
        * [🔁 And when you fix one issue, others might get worse. That’s the ripple effect (a.k.a. side effects).](#%F0%9F%94%81-and-when-you-fix-one-issue-others-might-get-worse-thats-the-ripple-effect-aka-side-effects)
      - [**🧠 Real-Life Analogy: Doctor Diagnosing Side Effects**](#%F0%9F%A7%A0-real-life-analogy-doctor-diagnosing-side-effects)
        * [Prompt engineering is like a doctor prescribing medicine:](#prompt-engineering-is-like-a-doctor-prescribing-medicine)
        * [You need to:](#you-need-to)
      - [**🧠 Black Box & Non-Determinism: Root Cause**](#%F0%9F%A7%A0-black-box--non-determinism-root-cause)
        * [Two properties make LLM prompting inherently hard:](#two-properties-make-llm-prompting-inherently-hard)
        * [So you can’t treat LLMs like APIs with clear contracts.](#so-you-cant-treat-llms-like-apis-with-clear-contracts)
      - [**⚡ Key Takeaway**](#%E2%9A%A1-key-takeaway)
        * [❝ If software engineering is deterministic Lego-building, prompt engineering is **behavioral psychology** with an unpredictable alien. ❞](#%E2%9D%9D-if-software-engineering-is-deterministic-lego-building-prompt-engineering-is-behavioral-psychology-with-an-unpredictable-alien-%E2%9D%9E)
        * [It’s not about language. It’s about **behavior design** under uncertainty.](#its-not-about-language-its-about-behavior-design-under-uncertainty)
      - [**TL; DR;**](#tl-dr-1)
- [**Topic 3: Prompt Template — How to Write a V0 Prompt**](#topic-3-prompt-template--how-to-write-a-v0-prompt)
        * [**This is the foundational structure you can use as your first commit — your Version 0 — when building LLM-based apps.**](#this-is-the-foundational-structure-you-can-use-as-your-first-commit--your-version-0--when-building-llm-based-apps)
    + [**🧱 V0 Prompt Structure (3-Part Template)**](#%F0%9F%A7%B1-v0-prompt-structure-3-part-template)
      - [**1. 🧠 Task Description (System Prompt Style)**](#1-%F0%9F%A7%A0-task-description-system-prompt-style)
        * [Tell the model **what it needs to do, who it should be,** and **how to respond.**](#tell-the-model-what-it-needs-to-do-who-it-should-be-and-how-to-respond)
        * [This part includes:](#this-part-includes)
        * [🧠 **Think of this like a system prompt** — you’re configuring the model's behavioral blueprint here.](#%F0%9F%A7%A0-think-of-this-like-a-system-prompt--youre-configuring-the-models-behavioral-blueprint-here)
    + [**2. 🔁 Examples (Few-shot / Explanation-based)**](#2-%F0%9F%94%81-examples-few-shot--explanation-based)
        * [Provide examples of how to do the task.](#provide-examples-of-how-to-do-the-task)
        * [These can include:](#these-can-include)
      - [**🧠 Vivek's insight:**](#%F0%9F%A7%A0-viveks-insight)
        * [❝ Examples are not just I/O — they’re a way to shape the model’s thinking process. ❞](#%E2%9D%9D-examples-are-not-just-io--theyre-a-way-to-shape-the-models-thinking-process-%E2%9D%9E)
      - [**💬 Example Case (Movie Recommender Prompt from class Q&A):**](#%F0%9F%92%AC-example-case-movie-recommender-prompt-from-class-qa)
        * [If user input is:](#if-user-input-is)
        * [“Movies about prison life”](#movies-about-prison-life)
        * [You can provide example output:](#you-can-provide-example-output)
        * [Shawshank Redemption, because it’s widely recognized, fits the theme, and has a great IMDb score.](#shawshank-redemption-because-its-widely-recognized-fits-the-theme-and-has-a-great-imdb-score)
      - [**🧠 Why it matters:**](#%F0%9F%A7%A0-why-it-matters)
        * [Without this, LLM might return Arabic or Korean prison movies — **not aligned with your user’s context.**](#without-this-llm-might-return-arabic-or-korean-prison-movies--not-aligned-with-your-users-context)
    + [**3. 🧾 The Task (Actual User Input)**](#3-%F0%9F%A7%BE-the-task-actual-user-input)
        * [This is **the real user query** or input data that needs to be processed now.](#this-is-the-real-user-query-or-input-data-that-needs-to-be-processed-now)
      - [**🧪 Class Example (Stand-Up Summary Task):**](#%F0%9F%A7%AA-class-example-stand-up-summary-task)
        * [**Practice Problem:**](#practice-problem)
        * [=============](#)
        * [You have transcript of morning standup that happened in team.](#you-have-transcript-of-morning-standup-that-happened-in-team)
        * [You want to take that transcript and convert it into a short summary also action items(“task”, “assignee”, “due date”)](#you-want-to-take-that-transcript-and-convert-it-into-a-short-summary-also-action-itemstask-assignee-due-date)
        * [V0 Prompt](#v0-prompt)
        * [========](#)
        * [**Task description:**](#task-description)
      - [**Rules:**](#rules)
      - [**Example(s) of how to do this task**](#examples-of-how-to-do-this-task)
    + [**The task:**](#the-task)
    + [**TL;DR:** 3-Part V0 Prompt Template](#tldr-3-part-v0-prompt-template)
      - [**🔧 Pro Tip:**](#%F0%9F%94%A7-pro-tip)
        * [Start with this format. Once you hit edge cases or regressions (e.g. LLM skips steps, mislabels fields), go back and:](#start-with-this-format-once-you-hit-edge-cases-or-regressions-eg-llm-skips-steps-mislabels-fields-go-back-and)
        * [This is how prompt engineering **iterates like product engineering.**](#this-is-how-prompt-engineering-iterates-like-product-engineering)
- [**Topic 4: Rules of Prompting + Role of Evals**](#topic-4-rules-of-prompting--role-of-evals)
      - [**🎯 1. Be Explicit**](#%F0%9F%8E%AF-1-be-explicit)
        * [Vivek repeatedly emphasized:](#vivek-repeatedly-emphasized)
        * [❝ If your prompt is vague, the LLM will hallucinate. Be absurdly specific. ❞](#%E2%9D%9D-if-your-prompt-is-vague-the-llm-will-hallucinate-be-absurdly-specific-%E2%9D%9E)
      - [**✅ Real Example from Class:**](#%E2%9C%85-real-example-from-class)
        * [In his homework extraction use-case (images from NCERT textbook):](#in-his-homework-extraction-use-case-images-from-ncert-textbook)
        * [→ “Extract all problems from these pages.”](#%E2%86%92-extract-all-problems-from-these-pages)
        * [⚠ Result: The LLM hallucinated questions from random paragraphs.](#%E2%9A%A0-result-the-llm-hallucinated-questions-from-random-paragraphs)
        * [✅ Fix: He rewrote the prompt to say:](#%E2%9C%85-fix-he-rewrote-the-prompt-to-say)
        * [“Only extract solved examples and exercise questions. Solved examples have a labeled solution. Skip all concept-explaining paragraphs, even if they contain a question mark.”](#only-extract-solved-examples-and-exercise-questions-solved-examples-have-a-labeled-solution-skip-all-concept-explaining-paragraphs-even-if-they-contain-a-question-mark)
        * [📌 **Lesson: Explicit rules reduce hallucination**, especially in noisy, unstructured inputs.](#%F0%9F%93%8C-lesson-explicit-rules-reduce-hallucination-especially-in-noisy-unstructured-inputs)
    + [**👤 2. Use a Persona / Role**](#%F0%9F%91%A4-2-use-a-persona--role)
        * [Tell the model who it's supposed to be.](#tell-the-model-who-its-supposed-to-be)
        * [Common pattern:](#common-pattern)
        * [“You are a domain expert / experienced engineer / school teacher / recruiter…”](#you-are-a-domain-expert--experienced-engineer--school-teacher--recruiter)
        * [🧠 Why? It primes the model’s tone, domain knowledge, and response boundaries.](#%F0%9F%A7%A0-why-it-primes-the-models-tone-domain-knowledge-and-response-boundaries)
        * [Even though Vivek mentioned that engineers at Anthropic don’t always favor this, he acknowledged:](#even-though-vivek-mentioned-that-engineers-at-anthropic-dont-always-favor-this-he-acknowledged)
        * [❝ It works in most real-world prompts. That’s why it’s used — not because it’s a guarantee, but because it helps.❞](#%E2%9D%9D-it-works-in-most-real-world-prompts-thats-why-its-used--not-because-its-a-guarantee-but-because-it-helps%E2%9D%9E)
    + [**🧾 3. Specify Output Format**](#%F0%9F%A7%BE-3-specify-output-format)
        * [Never let LLMs respond freely when structure matters.](#never-let-llms-respond-freely-when-structure-matters)
        * [**Example from Class (Standup Transcript Task):**](#example-from-class-standup-transcript-task)
        * [Output must include:](#output-must-include)
        * [{ "task": "Add load test to CI", "assignee": "Carol", "due_date": "Wednesday" }](#-task-add-load-test-to-ci-assignee-carol-due_date-wednesday-)
    + [**🧪 4. Add & Analyze Edge Cases**](#%F0%9F%A7%AA-4-add--analyze-edge-cases)
        * [Vivek shared multiple examples where **new errors** forced him to **add edge-case rules** in the prompt:](#vivek-shared-multiple-examples-where-new-errors-forced-him-to-add-edge-case-rules-in-the-prompt)
- [**📚 5. In-Context Learning (ICL)**](#%F0%9F%93%9A-5-in-context-learning-icl)
      - [**📖 Chapter 10 Example (Homework Extraction):**](#%F0%9F%93%96-chapter-10-example-homework-extraction)
      - [**✅ Rule in Prompt:**](#%E2%9C%85-rule-in-prompt)
      - [**📊 Role of Evals in Prompt Engineering**](#%F0%9F%93%8A-role-of-evals-in-prompt-engineering)
      - [**🧠 Analogy: Medical Testing**](#%F0%9F%A7%A0-analogy-medical-testing)
      - [**How evals work:**](#how-evals-work)
      - [**✅ TL;DR Summary**](#%E2%9C%85-tldr-summary)

<!-- tocstop -->

---

## Best practices

###### **Github repo (for code):** [**https://github.com/vv176/SimpleExamples**](https://github.com/vv176/SimpleExamples) 


## **Topic 1: Prompt Engineering — What, Why & How**


##### **🔍 What is Prompt Engineering?**

###### Prompt Engineering is the art and science of **adapting the behavior of a large language model (LLM)**

###### to perform a desired task — without changing its internal weights, architecture, or doing any fine-tuning.

###### It’s the process of shaping **what the model does**, just by tweaking what you say to it (i.e., the prompt).

  

##### **💡 Why Prompt Engineering?**

###### Vivek called it the **"most underrated yet crucial part"** of AI engineering, and made one thing very clear:

###### ❝ In production-grade AI applications, **90% of the grunt work** is prompt engineering. ❞

###### It's not just about writing clever instructions — it's about **making LLMs behave the way your product needs**, under unpredictable, noisy, and diverse conditions.

  

##### **🛠 Analogy: ML Models Before Prompting Era**

###### In the traditional ML world:

- You had a trained model that could do **Task A** — say, text summarization.
- Now you want to do **Task B** — spam classification.
- You had no choice but to:
    - **Retrain the model**
    - **Rewire the architecture**
    - **Adjust the weights**

###### This meant _huge development overhead_ for every new task. 

  

##### **⚙ What Prompt Engineering Unlocks**

- With LLMs + prompts, you can:
- Use the **same base model**
- Without retraining or modifying its internals
    - And get it to do:
    - Summarization ✅
    - Classification ✅
    - Code generation ✅
    - Translation ✅
    - Anything else ✅

###### ❝ Just supply the right prompt, and the behavior adapts. That’s your superpower. ❞ 

  

##### **🧬 Key Insight: Prompt Engineering = Behavior Hacking Without Code** 

###### Prompt engineering is about: 

“Adapting behavior of a system you don’t control, by designing the right input signal.”

###### You’re not editing the model’s neurons — you're manipulating its perception.

###### It’s like giving the same person different instructions and getting them to write poems, solve math, or

###### summarize policy — **without ever changing who they are.** 

  

##### **TL; DR;**

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756889583/Screenshot_2025-09-03_at_2.05.22_PM_oruzvf.png)

  

## **Topic 2: The Myth of Prompt Engineering** 

######   

#### **❌ Myth #1: “Prompting is Easy”**

##### Many believe:

- "Prompt engineering is just good English.”
- “Anyone can do it with basic language skills.”
- “Why even call it engineering?”

##### ✅ But in reality:

###### **Prompting is hard-core engineering** — often messier, more unpredictable, and more trial-and-error than writing traditional software.

  

#### **❌ Myth #2: “It’s Not Engineering at All”**

###### People question the term “engineering” in prompt engineering.

###### Vivek addressed this directly:

###### ❝ Prompt engineering requires more engineering than deterministic code. You’ll understand why when you try to test and debug LLM behavior. ❞

######   

#### **⚙ So, Why Is It Actually Engineering?**

###### Because it shares the same **rigor, iteration, and debugging** mindset as software engineering — but applied to a **non-deterministic black box.**

###### You still need to:

- Analyze failures
- Refactor inputs 
- Design for edge cases
- Monitor performance
- Build guardrails

###### The only difference? Your code is not **executable logic**, it's **natural language** with embedded logic.

######   

#### **🧪 Software Engineering vs. Prompt Engineering: A Testing Analogy**

#####   

##### **✅ In Traditional Software (Deterministic)**

- Code behaves the same way every time for a given input
- If it fails once, it’ll fail every time for the same condition
- Example: 

######                      if user_id is None:

######                                 raise NullPointerException

######                                        → Always fails for None. Always fixable with one if-check.

- Testing = **unit tests, integration tests, regression tests**
- Easy to isolate root causes and patch them

######   

##### **❌ In Prompt Engineering (Non-Deterministic)**

###### ❝ Same prompt, same input → 3 different behaviors. ❞

- First run: excellent response
- Second run: slightly worse
- Third run: totally off
- Even **zero-shot prompts** can perform inconsistently

###### This means:

- You **can’t write 10 unit tests** and call it done
- You must **analyze 1000+ prompt invocations**
- Then **categorize failures** (e.g. 40% = hallucination, 30% = format errors, etc.)

###### 🔁 And when you fix one issue, others might get worse. That’s the ripple effect (a.k.a. side effects).

  

##### **🧠 Real-Life Analogy: Doctor Diagnosing Side Effects**

###### Prompt engineering is like a doctor prescribing medicine:

- You fix one symptom (e.g. hallucination)
- But your fix creates new issues (e.g. skips some valid answers)

###### You need to:

- Run evals = diagnostic tests
- Track what gets better vs what regresses
- Iterate again 

######   

##### **🧠 Black Box & Non-Determinism: Root Cause**

###### Two properties make LLM prompting inherently hard: 

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756890472/Screenshot_2025-09-03_at_2.37.03_PM_ryt8fu.png)

###### So you can’t treat LLMs like APIs with clear contracts. 

  

##### **⚡ Key Takeaway**

###### ❝ If software engineering is deterministic Lego-building, prompt engineering is **behavioral psychology** with an unpredictable alien. ❞

###### It’s not about language. It’s about **behavior design** under uncertainty. 

  

##### **TL; DR;**

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756890614/Screenshot_2025-09-03_at_2.39.56_PM_tvcrqx.png)

  

## **Topic 3: Prompt Template — How to Write a V0 Prompt**

###### **This is the foundational structure you can use as your first commit — your Version 0 — when building LLM-based apps.**

######   

#### **🧱 V0 Prompt Structure (3-Part Template)**

######   

##### **1. 🧠 Task Description (System Prompt Style)**

###### Tell the model **what it needs to do, who it should be,** and **how to respond.**

###### This part includes: 

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756890811/Screenshot_2025-09-03_at_2.43.18_PM_lwmy1y.png)

  

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756890849/Screenshot_2025-09-03_at_2.43.53_PM_jpehcf.png)

###### 🧠 **Think of this like a system prompt** — you’re configuring the model's behavioral blueprint here.

######   

#### **2. 🔁 Examples (Few-shot / Explanation-based)**

###### Provide examples of how to do the task.

###### These can include:

- Input-output pairs
- Annotated examples (explain why this output is correct)
- Format styles
- Do’s and Don'ts

##### **🧠 Vivek's insight:**

###### ❝ Examples are not just I/O — they’re a way to shape the model’s thinking process. ❞

##### **💬 Example Case (Movie Recommender Prompt from class Q&A):**

###### If user input is:

###### “Movies about prison life”

###### You can provide example output:

###### Shawshank Redemption, because it’s widely recognized, fits the theme, and has a great IMDb score.

  

##### **🧠 Why it matters:**

###### Without this, LLM might return Arabic or Korean prison movies — **not aligned with your user’s context.**

######   

#### **3. 🧾 The Task (Actual User Input)**

###### This is **the real user query** or input data that needs to be processed now.

- After setting up the behavior (task + examples), this is where the actual question or data is injected.
- The better your first two sections, the more accurate and aligned the LLM’s response will be here.

##### **🧪 Class Example (Stand-Up Summary Task):**

###### **Practice Problem:**

###### =============

###### You have transcript of morning standup that happened in team.

###### You want to take that transcript and convert it into a short summary also action items(“task”, “assignee”, “due date”)

###### V0 Prompt

###### ========

###### **Task description:**

You are a note-taker for engineering standups.

Summarize the meeting for a PM audience in ≤120 words and extract action items (task, assignee, due_date if mentioned).

Output only valid JSON: 

{

      "summary": "≤120 words, PM-friendly",

       "actions": [ {

                "task": "string",

                "assignee": "string|null",

                "due_date": "YYYY-MM-DD|null"

        }]

} 

######   

##### **Rules:**

If no assignee/due date is stated, set field to null

  

##### **Example(s) of how to do this task**

Sample Input :

Alice: Search latency spiked to 420ms after Friday's deployment.

Bob: Root cause is the new logging middleware; reverting cuts it to ~180ms.

Carol: I'll add a load test to CI by Wed.

Bob: Cool, I will go through the code of middleware.

  

  

Sample Output :

{

         "summary": "Latency rose to 420ms post-deploy; reverting logging middleware drops it to ~180ms. Team will add a CI load test and read middleware’s code.",

          "actions": [

                    {

                           "task": "Add load test to CI",

                            "assignee": "Carol",

                             "due_date": "null"

                    },

                    { 

                              "task": "Code study of middleware",

                               "assignee": "Bob",

                                "due_date": "null"

                    }

          ]

}

  

#### **The task:**

Summarize and extract actions from the transcript below. Respond with JSON only.

Priya: We’ve encountered a NullPointerException in the order-service module after the latest deploy.

Arjun: That’s likely on me—my recent code shipped and I suspect I missed a null check in the

OrderValidator. I’ll dig into the stack trace now.

Riya: I’m on-call; I’ll create a hotfix and ship it behind a flag.

Karan: Great—Riya, ping me once your code is ready. I’ll review the PR immediately. 

  

#### **TL;DR:** 3-Part V0 Prompt Template 

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756891720/Screenshot_2025-09-03_at_2.55.54_PM_flyuou.png)

  

##### **🔧 Pro Tip:**

###### Start with this format. Once you hit edge cases or regressions (e.g. LLM skips steps, mislabels fields), go back and: 

- Add rules
- Refactor examples
- Split prompt into smaller parts if needed

###### This is how prompt engineering **iterates like product engineering.** 

######   

## **Topic 4: Rules of Prompting + Role of Evals**

##### **🎯 1. Be Explicit**

###### Vivek repeatedly emphasized:

###### ❝ If your prompt is vague, the LLM will hallucinate. Be absurdly specific. ❞

######   

##### **✅ Real Example from Class:**

###### In his homework extraction use-case (images from NCERT textbook):

- Initially, the prompt said:

######                → “Extract all problems from these pages.”

  

###### ⚠ Result: The LLM hallucinated questions from random paragraphs.

###### ✅ Fix: He rewrote the prompt to say:

###### “Only extract solved examples and exercise questions. Solved examples have a labeled solution. Skip all concept-explaining paragraphs, even if they contain a question mark.”

###### 📌 **Lesson: Explicit rules reduce hallucination**, especially in noisy, unstructured inputs. 

######   

#### **👤 2. Use a Persona / Role**

###### Tell the model who it's supposed to be.

###### Common pattern:

###### “You are a domain expert / experienced engineer / school teacher / recruiter…”

###### 🧠 Why? It primes the model’s tone, domain knowledge, and response boundaries.

###### Even though Vivek mentioned that engineers at Anthropic don’t always favor this, he acknowledged:

###### ❝ It works in most real-world prompts. That’s why it’s used — not because it’s a guarantee, but because it helps.❞

######   

#### **🧾 3. Specify Output Format**

###### Never let LLMs respond freely when structure matters.

- In free-text mode: model can be verbose, inconsistent, or unordered
- With a defined output schema (JSON, Markdown, numbered list), you can:
    - Parse the output
    - Compare against expectations
    - Integrate into systems

  

###### **Example from Class (Standup Transcript Task):**

###### Output must include: 

- A summary
- A structured list of action items:

######           { "task": "Add load test to CI", "assignee": "Carol", "due_date": "Wednesday" }

  

#### **🧪 4. Add & Analyze Edge Cases**

###### Vivek shared multiple examples where **new errors** forced him to **add edge-case rules** in the prompt: 

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756892228/Screenshot_2025-09-03_at_3.06.53_PM_hojgkt.png)

  

## **📚 5. In-Context Learning (ICL)**

Teach the model how to solve a task by showing examples within the prompt.

##### **📖 Chapter 10 Example (Homework Extraction):**

LLM had to extract questions from textbook pages — but not all were straightforward.

Problem:

- Sometimes the LLM found questions with no solution
- Other times, the solution was split across pages
- Sometimes, it hallucinated from examples

Fix:

- Vivek explicitly said:

           “A solved example is only valid if it contains both the problem and the solution. If either is missing, **say so.** ”

  

##### **✅ Rule in Prompt:**

- If there is:
    - ✅ Evidence → extract problem
    - ❌ Missing info → report it
    - 🕵 No evidence → skip the image

This is _in-context teaching_ the LLM what grounding means. 

  

##### **📊 Role of Evals in Prompt Engineering** 

**Evals = Unit Tests for Prompts**

Used to:

- Catch regressions
- Measure performance across inputs
- Quantify side effects

##### **🧠 Analogy: Medical Testing**

Vivek’s metaphor:

“Imagine a doctor giving you medicine for leg pain. After 7 days, the leg is fine, but your liver enzymes are off. That’s what evals show — the unexpected side effects of your prompt change.”

  

##### **How evals work:**

- You define:
    - ✅ Inputs (e.g. folder of 16 image sets)
    - ✅ Expected outputs
    - ✅ Actual outputs from the LLM
- Run an eval script to:
    - Compare actual vs expected
    - Categorize failure cases (e.g. 0.5% hallucination, 10% missing info)

🧠 You can even use LLM as **a judge** in the eval pipeline (to compare semantic similarity).

  

##### **✅ TL;DR Summary** 

![](https://res.cloudinary.com/dfgtstjy6/image/upload/v1756892912/Screenshot_2025-09-03_at_3.16.59_PM_rpb5vg.png)