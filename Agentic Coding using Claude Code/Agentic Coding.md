# Table-of-Contents

<!-- toc -->

- [1. Intro](#1-intro)
- [2. **Claude Code Setup & Workflow**](#2-claude-code-setup--workflow)
    + [**Free Alternatives (Ollama)**](#free-alternatives-ollama)
- [3. Slash Commands in Claude Code](#3-slash-commands-in-claude-code)
    + [**Core Concepts**](#core-concepts)
    + [**Key Built-in Commands**](#key-built-in-commands)
    + [**Best Practices**](#best-practices)

<!-- tocstop -->

---
# 1. Intro

---
# 2. **Claude Code Setup & Workflow**

*   **Installation:** Search for *Claude Code*, upgrade to a paid account ($1/day estimate), and run the provided command in your terminal (0:56 - 2:47).
*   **Project Initialization:** Download the project folder, open in *VS Code*, and run `claude` in the terminal to authorize (3:37 - 5:30).
*   **Environment Setup:** 
    *   Create a virtual environment (`python3 -m venv venv`) (5:50 - 6:53).
    *   Activate and install dependencies from `requirements.txt` (6:56 - 7:42).
    *   Run the application (7:49 - 8:32).
*   **Bash Mode:** Use `Shift+1` within *Claude Code* to execute commands directly; this allows the model to retain context of terminal operations for better assistance (9:50 - 11:29).
*   **Version Control:** Initialize *Git* and push your project to *GitHub* as a standard professional practice (11:29 - 14:22).
*   **Using Claude as an Agent:** Ask analytical questions like "What does this project do?" or "Explain the project structure" to help the model build context (14:22 - 17:05).

### **Free Alternatives (Ollama)**

*   **Ollama Setup:** Use *Ollama* to run local or cloud-based open-source models (17:05 - 18:59).
*   **Cloud Models (Limited):** Run `ollama launch claude` to access cloud models via Ollama's platform (18:59 - 22:41).
*   **Local Models (Fully Free):** Download models (e.g., *Qwen 2.5*) to your machine via `ollama pull` for unlimited, offline usage (22:41 - 25:14).

---
# 3. Slash Commands in Claude Code
This video provides a comprehensive guide to **Slash Commands** in *Claude Code*, which serve as shortcuts to trigger predefined workflows and improve development efficiency.

### **Core Concepts**
* **Slash Commands:** Shortcuts starting with `/` to execute automated tasks without writing full prompts (0:36).
* **Sessions:** Individual, persistent conversation histories with *Claude Code* (4:01). Use `claude -r` to resume past sessions (5:20).

### **Key Built-in Commands**
* **Session Management:** 
    * `/exit`: Close the current session (3:27).
    * `/resume`: Switch between active sessions (6:17).
    * `/rename [name]`: Label sessions for better organization (8:07).
    * `/export [file]`: Save conversation history to a markdown file (12:06).
* **Context & Utility:**
    * `/btw`: Ask side questions without polluting the main conversation history (9:23).
    * `/usage`: Track token usage and weekly limits (17:49).
    * `/models`: Switch between *Opus*, *Sonnet*, and *Haiku* models (17:20).
    * `/permissions`: Allow or deny specific tool access (e.g., web search or bash commands) (23:24).
    * `/theme`: Change the terminal UI appearance (28:55).
    * `/voice`: Enable hands-free prompting (29:31).

### **Best Practices**
* **One Task per Session:** Keeps context clean and manageable (6:56).
* **Rename Sessions:** Label early to avoid generic AI-assigned names (7:30).
* **Use `/btw`:** Utilize for quick lookups to keep the main context focused (10:30).
* **Model Strategy:** Use *Opus* for planning/architecture and *Sonnet* for implementation (16:27).