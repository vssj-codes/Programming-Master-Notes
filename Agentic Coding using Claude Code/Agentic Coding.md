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
- [6. **Claude.md: Project Context & Memory**](#6-claudemd-project-context--memory)
- [7. Spec-Driven Development vs. Vibe Coding](#7--spec-driven-development-vs-vibe-coding)
    + [The Spec-Driven Workflow](#the-spec-driven-workflow)
    + [Implementation Strategy (Claude Code)](#implementation-strategy-claude-code)
- [8. **Project Workflow: Spec-Driven Development**](#8-project-workflow-spec-driven-development)
    + [**Plan Mode Optimizations**](#plan-mode-optimizations)
    + [**Key Commands Summary**](#key-commands-summary)
- [9. This video covers automating development workflows in *Claude Code* using custom slash commands and implementing authentication features.](#9-this-video-covers-automating-development-workflows-in-claude-code-using-custom-slash-commands-and-implementing-authentication-features)
    + [**Custom Slash Commands**](#custom-slash-commands)
    + [**Development Workflow Automations**](#development-workflow-automations)
    + [**Feature Implementation Steps**](#feature-implementation-steps)
- [10. **Claude Code Skills: Essential Revision Sheet**](#10-claude-code-skills-essential-revision-sheet)
- [11. **Subagents in Claude Code: Summary Notes**](#11-subagents-in-claude-code-summary-notes)
- [12. **Claude Custom Subagents: Quick Reference**](#12-claude-custom-subagents-quick-reference)
- [13. ### **What is MCP?**](#13-%23%23%23-what-is-mcp)
    + [**Practical Implementations**](#practical-implementations)
    + [**Useful MCP Servers (Summary)**](#useful-mcp-servers-summary)
    + [**Pro-Tips & Management**](#pro-tips--management)
- [14. ### **Understanding Claude Code Hooks**](#14-%23%23%23-understanding-claude-code-hooks)
- [15. ### **Claude Code Plugins: Revision Sheet**](#15-%23%23%23-claude-code-plugins-revision-sheet)

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
# 6. **Claude.md: Project Context & Memory**

*   **Purpose:** LLMs lack session memory. `claude.md` acts as a persistent system prompt to maintain project context, coding conventions, and instructions across sessions.
*   **Setup:**
    *   **Manual:** Create a file named `claude.md` in the project root.
    *   **Automated:** Run `/init` to let *Claude Code* scan and generate an initial file (30% complete, requires your manual updates for the remaining 70%).
*   **Ideal Content:**
    *   Project Overview: 1-line description.
    *   Architecture & Folder Structure.
    *   Coding Style & Conventions (e.g., Type hints).
    *   Preferred Tools & Libraries.
    *   Commands (Installation, Dev, Test).
    *   Critical Constraints: What to avoid (e.g., "don't edit X file").
*   **Best Practices:**
    *   **Keep it under 200 lines:** Use split files if it grows too large.
    *   **Living Document:** Update periodically as the project evolves.
    *   **Scoping:** Use `Important` labels sparingly.

**Advanced Configuration (.claude folder)**

*   **Project-Level:** Located in root; committed to repo; shared with team.
*   **Global/User-Level:** Located in `~/ .claude`; personal settings, not shared.
*   **Rules Folder:** Store modular rules (e.g., `security.md`, `style.md`) here to be loaded only when relevant.
*   **Claude.local.md:** Use for personal, local-only overrides that should never be pushed to git.

**Auto-Memory**

*   **Mechanism:** *Claude Code* automatically observes patterns and stores insights in `memory.md`.
*   **Access:** Use `/memory` command to manage or view Project, User, or Auto-memory files.
*   **Manual Update:** Use the prompt "Update your memory files" after a successful session to explicitly save learnings.
---
# 7.  Spec-Driven Development vs. Vibe Coding

*   **Vibe Coding (0:00 - 4:55):** Fast, conversational, experimental. **Problem:** Loss of control, decision-making delegated to AI, prone to repetitive patching loops, unsuitable for production systems.
*   **Spec-Driven Development (4:55 - 6:55):** Structured, proactive planning. **Goal:** Maintain total control by defining requirements *before* coding; AI acts as an executor, not a decision-maker.

### The Spec-Driven Workflow

1.  **Spec Document (6:00 - 12:40):** The "Single Source of Truth."
    *   **Components:** Problem Statement, Functional Requirements, Input/Output flow, System Constraints, Edge Cases, Acceptance Criteria.
2.  **Technical Design Plan (13:20 - 16:50):** The "How" document.
    *   **Components:** Tech stack, architecture, data models, API endpoints, development roadmap.
    *   *Why separate?* Allows changing tech stacks without rewriting product specs.
3.  **Task Extraction (17:00 - 18:00):** Break design into atomic tasks (e.g., DB models -> Backend -> Frontend -> Integration).
4.  **Coding & Validation (18:00 - 19:00):** Execute tasks, then validate against the original Acceptance Criteria.

### Implementation Strategy (Claude Code)

*   **Workflow Integration (23:06 - 27:14):**
    *   **Pull/Branch:** Always start on a new Git feature branch.
    *   **Generation:** Use AI (Claude) to generate the Spec Doc, Tech Design, and Task list using custom commands/Plan mode.
    *   **Review:** Human review of all AI-generated plans is **mandatory**.
    *   **Execution:** Use single or parallel sub-agents for coding.
    *   **Completion:** Validate -> Git Commit -> Pull Request -> Merge -> Cleanup.
---
# 8. **Project Workflow: Spec-Driven Development**

1.  **Branching Strategy:** Use `git checkout -b feature/feature-name` to isolate development.
2.  **Spec Documentation:** Define project goals, database schema, and acceptance criteria in `.clotted/specs/`.
3.  **Plan Mode:** Trigger with `Shift + Tab` (2x) or `/plan`. 
    *   **Purpose:** Allows AI agents to read files and build a technical implementation roadmap without writing code.
    *   **Process:** Generate plan -> Review logic -> Execute via `Enter` to approve edits.
4.  **Validation:** Run the app and verify database/schema against the original spec acceptance criteria.
5.  **Git Cleanup:** After verification: `git add .` -> `git commit` -> `git push` -> Create/Merge PR -> Delete feature branch.

### **Plan Mode Optimizations**

*   **Model Selection:** Use `/model` to switch to *Opus* for complex architectural tasks or large-scale refactoring.
*   **Extended Thinking:** Toggle via `/config` (Thinking Mode: True). AI reasons in a "scratchpad" before responding, improving plan quality.
*   **Effort Level:** Use `/effort` (Low/Medium/High/Max) to control tokens spent on reasoning.
*   **UltraPlan:** Use `/ultraplan` for superior, cloud-based planning via remote containers when local plans are insufficient. Features a rich web-based editing interface.

### **Key Commands Summary**
*   `/plan`: Activate local planning mode.
*   `/model`: Select AI model (e.g., Opus for complexity).
*   `/config`: Toggle thinking/settings.
*   `/effort`: Manage AI reasoning depth.
*   `/ultraplan`: Trigger remote cloud-based planning.
---
# 9. This video covers automating development workflows in *Claude Code* using custom slash commands and implementing authentication features.

### **Custom Slash Commands**
*   **Concept:** Saved prompts that execute repeatable workflows via `/command_name`.
*   **Setup:** Create a markdown file (`.md`) inside the `.claude/commands/` directory.
*   **Scope:** 
    *   **Project:** Saved in `project_dir/.claude/commands/` (local to project).
    *   **User:** Saved in `home_dir/.claude/commands/` (global access).
*   **Implementation:** Commands must contain a detailed description, allowed tool access (e.g., *bash*, *read*), and clear step-by-step instructions.
*   **Workflow Integration:** 
    *   After adding/modifying a command file, **restart** *Claude Code* for it to be recognized.
    *   Commands can accept arguments (e.g., `{{arguments}}`) for dynamic inputs (0:12:15).

### **Development Workflow Automations**
*   **Seeding Data:** Create commands (e.g., `seed-user`, `seed-expense`) to populate local databases with dummy data for UI testing (0:04:20).
*   **Spec Generation (`create-spec`):** Automates documentation by parsing requirements, checking existing codebases, and outputting structured specs (0:16:00).
*   **Branching Automation:** Enhanced `create-spec` to check for clean working directories, auto-switch to `main`, pull updates, and create/checkout new feature branches (0:32:00).

### **Feature Implementation Steps**
1.  **Plan:** Use `create-spec` command to define the feature spec (0:21:00).
2.  **Verify:** Review generated spec for accuracy against acceptance criteria (0:22:00).
3.  **Technical Plan:** Run `plan mode` and provide the path to the spec file to generate an implementation plan (0:23:40).
4.  **Execute:** Approve edits/code changes generated by *Claude* (0:25:00).
5.  **Test:** Validate against defined acceptance criteria (0:27:00).
6.  **Commit/Push:** Standardize git flow (add, commit, push, PR/merge, delete branch) (0:29:00, 0:43:00).
---
# 10. **Claude Code Skills: Essential Revision Sheet**

**1. The Problem: General AI vs. Specialized Tasks**
*   LLMs are excellent at general reasoning but struggle with repetitive, domain-specific tasks (e.g., custom PPT layouts, specific coding styles, or company guidelines).
*   System prompts are inefficient: they clutter the context window and are difficult to manage, version, or share (0:01:05 - 0:08:43).

**2. What are Skills?**
*   **Definition:** Reusable, folder-based resources that provide task-specific expertise, workflows, and best practices to *Claude Code* (0:08:43).
*   **Progressive Disclosure:** Skills are not pre-loaded; they load "just-in-time" only when the model identifies the relevant trigger, preserving context window space (0:14:50 - 0:17:30).

**3. Structure & Organization**
*   **Hierarchy:** `project_folder` -> `.claude` -> `skills` -> `skill_name_folder` (0:10:00).
*   **Required Files:**
    *   `skill.md`: Contains **YAML front matter** (Name, Description, Triggers) and the **Markdown body** (detailed instructions/workflows).
    *   **Resources:** Additional folders (e.g., `/scripts`, `/templates`) for supporting files used by the skill (0:10:00 - 0:14:24).

**4. Skill Lifecycle & Creation**
*   **Scope:** 
    *   *Personal Skills:* Stored in home `~/.claude` directory (available globally).
    *   *Project Skills:* Stored in local `.claude` directory (specific to project) (0:17:30).
*   **Creation Methods:**
    *   *Claude AI:* Use the built-in "Skill Creator" tool (recommended for beginners) (0:18:58).
    *   *Manual:* Create folder/file structure yourself.
*   **Workflow:** Identify need → Create → Test → Iterate (0:22:00).

**5. Commands vs. Skills (Key Update)**
*   Anthropic has merged commands and skills. Both are now structured as skills.
*   To keep an old-style "command" behavior, add `disable_model_invocation: true` to the YAML front matter to prevent auto-loading (0:46:06 - 0:49:17).
---
# 11. **Subagents in Claude Code: Summary Notes**

**Core Problem: Stateless LLMs**
* **Statelessness:** LLMs have no memory; every request requires sending the entire conversation history (context).
* **Context Window Overload:** Sending large codebases repeatedly causes token explosion, high costs, and the "Lost in the Middle" effect (reduced accuracy).

**What are Subagents?**
* **Definition:** Specialized AI assistants created by the "main agent" to run tasks in **isolated context windows** (10:00-10:52).
* **Workflow:** Main agent delegates a specific task $\rightarrow$ Subagent spawns with a fresh context $\rightarrow$ Performs heavy lifting $\rightarrow$ Returns only the essential output $\rightarrow$ Subagent context is destroyed.

**Key Advantages**
* **Context Isolation:** Keeps the main conversation clean and reduces token waste (15:08-15:27).
* **Specialization:** Each agent can have custom system prompts, tools, and permissions (e.g., security auditor, researcher) (15:28-16:04).
* **Modularity & Parallelism:** Divide complex pipelines into smaller tasks; run independent tasks (like data analysis) in parallel (16:16-18:28).

**Primary Use Cases**
* **Codebase Exploration:** Automatically used by Claude to scan projects (18:46-19:32).
* **Code Review & Testing:** Using a separate agent prevents inherent bias from the original code-writer (19:33-20:47).
* **Multi-stage Pipelines:** Hand-offs between specific tasks (e.g., API design $\rightarrow$ Code $\rightarrow$ Test) (21:09-21:50).
* **Security Audits:** Isolate security scanning to dedicated, high-capability models (22:25-22:53).

**Types of Subagents**
1. **Built-in:**
    * *Explore:* Scans codebase/summarizes findings (23:50-24:25).
    * *Plan:* Generates implementation steps for a task (24:28-24:55).
    * *General Purpose:* Handles miscellaneous read/write tasks (24:56-25:13).
2. **Custom:**
    * Configurable via system prompts, specific models (e.g., Opus vs. Sonnet), and tool access (26:35-28:18).
    * Can be *User Level* (available for all projects) or *Project Level* (local to specific repo) (26:44-27:08).

**Implementation Tip:** Use tools like `agents-observe` to monitor real-time subagent triggering and verify your workflows during development (31:09-32:45).

---
# 12. **Claude Custom Subagents: Quick Reference**

**Core Concept**
* **Subagents:** Independent AI workers with specific context, tools, and system prompts to handle specialized tasks (e.g., security audits, testing pipelines) that generic agents cannot perform optimally (0:01:14-0:05:47).

**Creation Process**
1. **Structure:** Create a `.md` (Markdown) file containing **YAML front matter** (name, description, tools, model, color) and a **system prompt/instruction body** (0:06:30-0:07:44).
2. **Storage:** Save files in `.claude/agents/` (Project-level) or your home directory (Personal-level) (0:07:44-0:08:10).
3. **Helper:** Use the `slash agents` command in Claude Code to initialize and configure new agents (0:19:00-0:22:12).

**Triggering Workflows**
* **Manual:** Invoked via custom slash commands (e.g., `/test-feature`) defined in `.claude/commands/` (0:09:00-0:10:00).
* **Automatic:** Triggered by Claude based on the agent's description during development (0:09:00-0:09:35).

**Recommended Practical Pipeline (Example Workflow)**
1. **Testing Stage (`/test-feature`):**
    * **Test Writer:** Generates tests based on feature specs (0:15:30).
    * **Test Runner:** Executes tests and provides a summary (0:15:30).
2. **Code Review Stage (`/code-review`):**
    * **Security Agent:** Performs parallel vulnerability checks (0:16:30).
    * **Quality Agent:** Checks for best practices and code standards (0:16:30).

**Workflow Best Practices**
* **Spec-Driven:** Always develop using a specs/plan-first approach before implementing (0:13:24-0:14:00).
* **Modularization:** Build separate agents for distinct, independent tasks to ensure scalability (0:15:47).
* **Validation:** Treat testing and code review as mandatory pre-commit stages before pushing to Git (0:14:00-0:15:00).
---
# 13. ### **What is MCP?**
*   **Model Context Protocol (MCP):** An open standard by *Anthropic* acting as a **universal connector** between LLMs and external tools/data (0:56).
*   **Benefits:** Eliminates custom/brittle integration code; standardizes how Claude interacts with local/external services (1:25).
*   **Capability Shift:** Extends *Claude Code* beyond basic read/write/bash tools to access real-time databases, design files, and Git repositories (3:52).

### **Practical Implementations**
*   **SQLite Database Integration (7:41):**
    *   Use the `mcp-server-sqlite` for local database management.
    *   Command: Add your database file path to the MCP setup config.
    *   Usage: Query via natural language (e.g., "List all tables," "Show schema of expenses," "Total spending by category") (10:05).
*   **Figma Integration (15:30):**
    *   Install via: `claude plugin install figma` (16:03).
    *   Workflow: Use Figma as your UI design tool; Claude reads the design URL to generate production-ready code (HTML/CSS/Templates) (18:00).
*   **GitHub Integration (25:01):**
    *   Requires: **Personal Access Token (PAT)** from GitHub (23:21).
    *   Capabilities: Query repositories, issues, and pull requests; automate commits and PR management (26:53).

### **Useful MCP Servers (Summary)**
*   **Context7:** Live, up-to-date documentation for libraries/frameworks (41:41).
*   **Jira:** Syncs project management tasks and bug tracking directly into your IDE (43:15).
*   **Notion:** Connects knowledge bases and product requirement docs to your coding environment (44:49).
*   **Slack:** Automates notifications, incident reporting, and team communication (46:12).
*   **AWS:** Manages deployments, logs, and cloud services (47:58).
*   **Docker:** Optimizes images and automates containerization workflows (49:26).

### **Pro-Tips & Management**
*   **List/Verify:** Use `/mcp` command to see status and active servers (7:05).
*   **Remove:** Use `claude mcp remove <server_name>` to clean up unused connections (51:18).
*   **Performance:** Keep active MCP servers **minimal**. Loading too many unnecessary tool descriptions into the context window can degrade LLM performance (53:02).
---
# 14. ### **Understanding Claude Code Hooks**

**Core Concept** (30:05):
* **What are Hooks?** Custom scripts triggered automatically by the *coding harness* at specific events within the *session lifecycle*.
* **Why use them?** To inject **determinism** into a probabilistic LLM-based system, enforcing safety and consistency (33:35).

**Architecture Definitions** (4:22 - 27:27):
* **Coding Harness:** The system interface (harness) that controls the LLM (the "engine") to ensure safe, stable execution.
* **Agent Loop:** A multi-step process where the LLM plans, uses tools (read/write/bash), and executes tasks iteratively.
* **Session Lifecycle:** The span from `session start` to `exit`, containing various events (e.g., `pre-tool-use`, `post-tool-use`, `stop`).

**Practical Implementation** (43:11 - 54:29):
* **Configuration:** Define hooks in `.claudecode/settings.json`.
* **Anatomy of a Hook:**
    * **Event:** When to trigger (e.g., `pre-tool-use`).
    * **Matcher:** Filter for specific tools or actions (e.g., `bash`).
    * **Action:** The script/command executed (exits with `0` for success or `2` to abort/stop).

**Common Use Cases** (34:37 - 42:13):
* **Safety:** Prevent dangerous file operations (e.g., protecting `.env` or database files) by aborting the command (exit code 2).
* **Formatting:** Automatically run tools like *Black* after edits to ensure code consistency (36:01).
* **Linting:** Catch bugs/bad patterns post-execution (37:53).
* **Automation:** Custom notifications or telemetry for sub-agent monitoring (40:13).

**Developer Workflow Integration** (58:27 - 1:04:07):
* **GitHub Integration:** Use *MCP* to handle PRs, merging, and branch deletion directly from the terminal.
* **Custom Commands:** Bundle repetitive actions into shell commands (e.g., `ship-feature`) to automate the end-to-end deployment/cleanup flow.
---
# 15. ### **Claude Code Plugins: Revision Sheet**

**Core Concept**
* **Plugins:** Packaged entities (folders) containing *skills*, *hooks*, *custom slash commands*, *subagents*, and *MCP tools* to standardize workflows across teams.
* **Benefit:** Replaces manual setup/configuration with a single, distributable package.

**Plugin Structure**
* Must include a `.claude/plugins/` directory and a mandatory `plugin.json` manifest file (contains metadata: name, version, description, etc.).
* **Folder Organization:** Use sub-folders like `skills/`, `hooks/`, `agents/`, and include `mcp.json` for tool configurations.

**Marketplaces & Distribution**
* **Marketplace:** A *GitHub repository* containing a `marketplace.json` file that hosts/lists multiple plugins.
* **Workflow:** 
    1. Install the marketplace (via `slash-plugin` -> `Add Marketplaces`).
    2. Browse/Discover plugins.
    3. Install the specific plugin to your machine or project.

**Practical Implementation: Spendly Project Deployment**
* **Step 1: Feature Completion:** Built the "Delete Expense" functionality using standard *Claude Code* agentic flow (spec -> plan -> execute -> ship).
* **Step 2: Deployment via Plugin:** 
    * Opted for *Railway* (better suited for Flask) over *Vercel*.
    * Used `railway login` and the official Railway plugin for seamless infrastructure automation.
    * **Deployment command:** `Deploy this Flask application to Railway and give me a public URL`.

**Key Commands & Navigation**
* `/plugin`: Primary entry point for discovery, installation, and managing marketplaces.

**Recommended Plugins for Exploration**
* *Super Powers* (Workflow enhancement), *Front-end Design* (UI generation), *Context Seven* (Library documentation), *Code Simplifier*, *Playwright* (Browser automation).
---
