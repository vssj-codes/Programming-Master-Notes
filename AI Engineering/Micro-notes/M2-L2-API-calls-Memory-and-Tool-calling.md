# Table-of-Contents

<!-- toc -->

- [🧠 **MICRO-NOTES — API Calls, Memory & Tool Calling**](#%F0%9F%A7%A0-micro-notes--api-calls-memory--tool-calling)
  * [**1️⃣ Core Idea**](#1%EF%B8%8F%E2%83%A3-core-idea)
  * [**2️⃣ LLM API Call Lifecycle**](#2%EF%B8%8F%E2%83%A3-llm-api-call-lifecycle)
  * [**3️⃣ Memory Types**](#3%EF%B8%8F%E2%83%A3-memory-types)
    + [**Short-Term Memory**](#short-term-memory)
    + [**Long-Term Memory**](#long-term-memory)
    + [**System / Profile Memory**](#system--profile-memory)
  * [**4️⃣ Context Window & Trade-offs**](#4%EF%B8%8F%E2%83%A3-context-window--trade-offs)
  * [**5️⃣ What Tool Calling Really Is**](#5%EF%B8%8F%E2%83%A3-what-tool-calling-really-is)
  * [**6️⃣ Tool Selection & Orchestration**](#6%EF%B8%8F%E2%83%A3-tool-selection--orchestration)
  * [**7️⃣ Production Architecture Pattern**](#7%EF%B8%8F%E2%83%A3-production-architecture-pattern)
  * [**8️⃣ Failure Modes (Interview Gold)**](#8%EF%B8%8F%E2%83%A3-failure-modes-interview-gold)
  * [**9️⃣ One-Minute Interview Explanation**](#9%EF%B8%8F%E2%83%A3-one-minute-interview-explanation)

<!-- tocstop -->

---

# 🧠 **MICRO-NOTES — API Calls, Memory & Tool Calling**

---

## **1️⃣ Core Idea**

> **LLMs are stateless engines.**  
> Your backend provides **memory, tools, and control.**

The model only sees:

```
current prompt → response
```

Everything else is **your system’s job.**

---

## **2️⃣ LLM API Call Lifecycle**

**Mental Diagram**

```
User Input
 → Backend
   → Inject system prompt
   → Inject memory
   → Inject tool schemas
 → LLM API Call
 → LLM decides: text or tool
 → Backend executes tools
 → Results appended to context
 → LLM produces final answer
```

**Interview line:**

> “The LLM is just a reasoning engine; orchestration lives in the backend.”

---

## **3️⃣ Memory Types**

### **Short-Term Memory**

- Chat history in context window
    
- Limited by token budget
    

### **Long-Term Memory**

- Stored externally (DB / vector store)
    
- Retrieved and injected when relevant
    

### **System / Profile Memory**

- User preferences
    
- App configuration
    
- Long-term persona info
    

**Key Point:**

> “Memory is not inside the model — it’s reconstructed every request.”

---

## **4️⃣ Context Window & Trade-offs**

LLM only knows what’s inside the prompt.

Trade-offs:

- More context → better grounding
    
- More context → higher cost & slower
    
- Too much context → performance drops
    

**Production rule:**

> Always send **only the most relevant memory.**

---

## **5️⃣ What Tool Calling Really Is**

LLMs **cannot call APIs**.

Instead:

- You provide **tool schemas** (name, inputs, description)
    
- The LLM outputs:
    
    ```
    { tool_name, arguments }
    ```
    
- Backend executes the tool
    
- Tool result is fed back into the LLM
    

**Interview line:**

> “LLMs suggest actions; the backend executes them.”

---

## **6️⃣ Tool Selection & Orchestration**

Model chooses tools using:

- Tool descriptions
    
- Current prompt
    
- Conversation context
    

Flow:

```
LLM → Tool decision → Backend runs tool → Result → LLM → Final answer
```

Multiple tools can be chained.

---

## **7️⃣ Production Architecture Pattern**

```
User
 → Backend
   → Memory retrieval
   → Prompt construction
   → Tool schemas
 → LLM
   → Tool request
 → Backend executes tools
 → LLM final response
 → User
```

---

## **8️⃣ Failure Modes (Interview Gold)**

- Hallucinated tool calls
    
- Wrong tool selection
    
- Context window overflow
    
- Memory leakage between users
    
- Cost explosion from oversized prompts
    

**Engineering fix:**  
Strict schemas, validation, pruning memory, evals.

---

## **9️⃣ One-Minute Interview Explanation**

> “LLMs are stateless. Each API call is a fresh request where we inject memory, instructions, and tool definitions.  
> The model reasons over this context and may request a tool.  
> The backend executes it, returns results, and the LLM produces the final answer.  
> Real AI systems are mostly orchestration, not modeling.”

---
