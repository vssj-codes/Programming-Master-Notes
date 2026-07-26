# Table-of-Contents

<!-- toc -->

- [Prompt:](#prompt)
- [1. Intro](#1-intro)
- [2. **Claude Code Setup & Workflow**](#2-claude-code-setup--workflow)
    + [**Free Alternatives (Ollama)**](#free-alternatives-ollama)
- [3. Slash Commands in Claude Code](#3-slash-commands-in-claude-code)
    + [**Core Concepts**](#core-concepts)
    + [**Key Built-in Commands**](#key-built-in-commands)
    + [**Best Practices**](#best-practices)
- [4. **Claude Code Workflow: Landing Page Improvements**](#4-claude-code-workflow-landing-page-improvements)
- [5. **Context Window Management in Claude Code: Quick Reference**](#5-context-window-management-in-claude-code-quick-reference)
- [6.](#6)

<!-- tocstop -->

---
# Prompt:

 Give ultra concise bullet point notes on this video. i will use this note as my reference while practically implementing what is being taught. It should also serve as a revision sheet when i come back to it for revision

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
---
# 4. **Claude Code Workflow: Landing Page Improvements**

**Core Workflow (Session → Prompt → Commit → Push)**
*   **Start Session:** Type `claude` in the terminal to initialize.
*   **Naming:** Use `/rename [Session Name]` to track project goals.
*   **File Context:** Always use the `@` symbol before filenames (e.g., `@base.html`) to ensure the AI modifies the correct file.
*   **Iterative Prompting:** Provide specific instructions. If the initial output isn't perfect, follow up with additional prompts to fix styles or logic.
*   **Version Control:** Commit changes incrementally (`git add`, `git commit`) after each successful feature implementation.

**Key Development Tasks**
*   **Adding Footer Links:** Prompt Claude to add "Terms & Conditions" and "Privacy Policy" to `base.html` (03:29).
*   **Generating Pages:** Request route creation in `app.py` and template creation for new pages. Ensure CSS consistency with the existing theme (07:57, 11:54).
*   **Multi-modal UI Redesign:** Paste a design image into the chat and instruct Claude to update the hero section in `landing.html` and `landing.css` based on that visual context (13:37).
*   **Implementing Modals:** Use vanilla JavaScript for functional popups (e.g., YouTube video embeds) to avoid heavy dependencies (17:11).

**Best Practices**
*   **Curate Prompts:** Draft and polish your prompts in a separate file before pasting them into Claude Code to avoid mistakes.
*   **Review Diffs:** Always check the "red" (removed) and "green" (added) code snippets provided by Claude before confirming with `y`.
*   **Iterate:** If errors occur (e.g., wrong filename referenced), simply issue a follow-up command to merge or fix the mistake (20:10).
*   **Finalize:** Push changes to the repository (`git push origin main`) only after verifying functionality across all updated pages (18:39).
---
# 5. **Context Window Management in Claude Code: Quick Reference**

**Core Concept:**
* **Context Window:** The model's "working memory" (limited in tokens) containing codebases, chat history, tool schemas, and instructions.
* **Claude Code Limit:** ~200k tokens total, with ~150k usable (others reserved for system prompts, tools, and auto-compaction).

**Token Consumption Factors:**
* **Bi-directional:** Both your input prompts and Claude’s output consume tokens.
* **Growth:** Every turn re-sends the entire conversation history; long sessions scale linearly/rapidly.
* **Degradation:** Response quality drops as the window fills (approaching 150k limit).

**Proactive Management Commands:**
* `/context`: Check real-time token usage and distribution.
* `/compact`: Manually compress conversation history to free space (use before sessions get too large).

**Best Practices for Efficiency:**
* **Session Hygiene:** Limit each session to **one specific feature/task**. Start a new session for unrelated work.
* **Manual Control:** Use `/compact` proactively (when near 70-75% usage) rather than waiting for auto-compaction, which may trigger during critical tasks.
* **Sub-agents:** Use for isolated tasks; they have their own independent, fresh context windows.
* **File Filtering:** Use `.claudeskill` or `.clauderc` (and experiments like `.claudesignore`) to keep unnecessary files out of the context.
* **Terminal Advantage:** The CLI is preferred over GUI/VS Code extensions to access advanced features like memory management and hook configurations.

**Troubleshooting:**
* If context is exhausted or quality degrades: Clear current context (`/clear`) or start a completely new session.
---
# 6.