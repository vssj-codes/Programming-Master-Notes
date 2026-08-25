# Uber Project Discussion — Round 3 Prep

---

## How to Structure Your Project Introduction

Follow this pattern — concise, confident, technical:

```
"I worked on [project name] — a [what it does] used by [who].
 My role was [role]. I was responsible for [key areas].
 The stack was [tech]. One major feature I built was [feature]
 which involved [technical challenge and how you solved it]."
```

---

## Questions to Prepare Answers For

---

### 1. Explain Your Project, Role, and Major Features

**Template:**
- What does the product do?
- Who uses it? (internal tool, end users, clients)
- What was your specific role? (solo, team, lead)
- What were the 2-3 major features you built?
- What tech stack did you use?

**Prepare for follow-ups:**
- "Walk me through the most complex feature you built"
- "How many users does the product serve?"
- "What was your team size?"

---

### 2. Project Architecture & Technical Decisions

Be ready to explain:
- Why React? (vs Vue, Angular)
- Why Redux? (vs Context API, Zustand)
- Why this folder structure?
- How is the app deployed? (Vercel, AWS, Docker)
- How do you handle authentication?
- How do you handle API errors globally?

**Template for any architecture decision:**
```
"We chose [X] because [reason].
 We considered [alternative] but [why we didn't choose it].
 The trade-off was [downside] which we handled by [mitigation]."
```

---

### 3. How You Started/Built a Project from Scratch

Be ready to describe:
1. Gathered requirements from stakeholders
2. Chose tech stack based on project needs
3. Set up project structure (feature-based)
4. Configured build tools (Vite/CRA), linting, formatting
5. Set up CI/CD pipeline
6. Defined reusable component library / design system
7. Established coding standards and PR review process

---

### 4. Service Configuration & API Organization

```
src/
  api/
    client.js         ← Axios instance with base URL, interceptors
    userApi.js        ← all user-related endpoints
    orderApi.js       ← all order-related endpoints
    index.js          ← export all APIs
```

```javascript
// client.js
const api = axios.create({
    baseURL: import.meta.env.VITE_API_URL,
    timeout: 10000,
});

api.interceptors.request.use(config => {
    config.headers.Authorization = `Bearer ${getToken()}`;
    return config;
});

api.interceptors.response.use(
    res => res.data,
    err => {
        if (err.response?.status === 401) handleUnauthorized();
        return Promise.reject(err);
    }
);
```

---

### 5. Authorization Approach

Common approach to describe:
1. User logs in → backend returns JWT token
2. Store token in httpOnly cookie (secure) or localStorage
3. Attach token to all API requests via Axios interceptor
4. Backend validates token on every request
5. Frontend: protected routes check auth state
6. On 401 → clear token → redirect to login
7. Refresh token flow for long sessions

---

### 6. Challenges Faced and How You Solved Them

Think of 2-3 real challenges. Structure:
```
Situation → Problem → What you tried → What worked → Result
```

**Common good examples to adapt:**
- Performance issue with large data sets → virtualization / pagination
- Inconsistent API responses → data normalization layer
- State sync issues across multiple components → moved to Redux
- Slow initial load → code splitting + lazy loading
- Race conditions in search → debounce + request cancellation

---

### 7. How You Ensure Application Quality Before Release

Good answer covers:
- **Unit tests** — component-level testing (React Testing Library)
- **Integration tests** — feature-level testing
- **E2E tests** — critical flows (Playwright/Cypress)
- **Code review** — PR review checklist
- **Staging environment** — test against real API before production
- **Linting and type checking** — ESLint, TypeScript
- **Performance profiling** — React DevTools, Lighthouse
- **Accessibility checks** — axe, screen reader testing

---

### 8. Learning/Adapting to New Technologies

Structure:
```
"When we needed to [solve problem], I researched [technology X].
 I started by [official docs / small PoC].
 Within [timeframe] I was able to [what you built].
 Key things I learned: [2-3 insights]."
```

Shows: proactiveness, learning ability, practical application.

---

### 9. Collaboration with Limited Team Support

Good answer:
- Used documentation and official sources aggressively
- Built small proof of concepts before committing to a solution
- Asked focused, specific questions rather than broad ones
- Shared learnings with team through internal docs/demos
- Pair programming for knowledge transfer

---

### 10. Familiarity with Cursor

Cursor is an AI-powered code editor (based on VS Code). Key points:

- AI autocomplete (like GitHub Copilot but more integrated)
- Chat with your codebase — "explain this file", "refactor this function"
- Cmd+K — inline AI edits
- Cmd+L — chat sidebar
- Knows your codebase context automatically

If you've used it:
```
"I use Cursor for [specific use case — debugging, refactoring].
 It speeds up [task] significantly.
 I use it for [boilerplate generation / understanding unfamiliar code / writing tests]."
```

If you haven't:
```
"I haven't used Cursor specifically, but I'm familiar with AI-assisted
 development through [GitHub Copilot / Claude Code].
 I'm comfortable picking up new tools quickly."
```

---

## Project Discussion — Golden Rules

1. Lead with impact — "This reduced load time by 40%" beats "I implemented lazy loading"
2. Always mention the why, not just the what
3. Be specific — vague answers lose credibility
4. Know your numbers — users, data size, team size, performance metrics
5. Prepare for "tell me more" on everything you say
6. It's okay to say "I don't know the exact number but approximately..."
