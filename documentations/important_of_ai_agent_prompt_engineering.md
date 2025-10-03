Great question. For a fully automated software engineering (SWE) AI agent, prompt engineering is critical—but it’s one piece of a larger system (tools, memory, planning, verification). Still, good prompts often make the difference between a flaky agent and a reliable one.

Below are the most important prompt-engineering principles and patterns specifically for SWE agents, plus concrete examples you can adapt.

Key principles for SWE agent prompt engineering
- Define the agent’s contract clearly
  - Role: What the agent is (e.g., “You are a senior backend engineer and test-first advocate”).
  - Objectives: What success looks like (e.g., “Produce a working solution with passing tests and a minimal diff”).
  - Scope/constraints: Languages, frameworks, coding standards, security/privacy rules.
  - Non-goals: What not to do (e.g., “Do not change unrelated files”).
- Align with the development workflow
  - Mirror real SWE steps: requirement parsing, design, implementation, tests, run/debug, refactor, document, PR.
  - Make each step explicit so the model knows when to stop and what artifact to output.
- Use tool-aware prompting
  - Describe available tools (filesystem, git, test runner, linter, container, browser) and how/when to use them.
  - Provide I/O formats for tools and require the agent to call them instead of hallucinating outputs.
- Make state and memory explicit
  - Summarize project context, architecture, and decisions in the prompt or accessible memory.
  - Provide a scratchpad for partial plans and running notes; instruct the model to update it.
- Constrain output formats
  - Enforce schemas for code changes, logs, decisions, and commit messages. This reduces ambiguity and makes automation robust.
  - Prefer diffs/patches or file blocks over freeform code dumps.
- Force verification and self-checks
  - Require the model to list test coverage, run tests, explain failures, and fix iteratively.
  - Add checklists: correctness, performance, security, style, dependency safety, backward compatibility.
- Plan before coding
  - Ask for a brief plan (design, interfaces, edge cases) before writing code. This reduces rework and helps debugging.
- Data grounding over guessing
  - Paste relevant files, configs, logs, stack traces. Instruct the agent to only reason from provided context or tool outputs.
- Encourage minimality
  - “Smallest viable change” prompts reduce blast radius and increase maintainability.
- Provide exemplars
  - Few-shot examples of good patches, tests, and commit messages improve consistency.
- Deterministic, step-bounded loops
  - Define iteration limits and exit criteria. Provide explicit “stop” signals once success criteria are met.
- Post-change explainability
  - Require a short rationale and risk notes. Useful for code reviews and future agents.

Prompt patterns and templates
1) System prompt skeleton (for a repo agent)
- Role and goals
  - You are a senior software engineer agent. Your goal is to implement tasks with minimal diffs, strong tests, and passing CI.
- Tools
  - You can: read/write files, run tests, run lints/formatters, execute code, and run shell commands. Use tools rather than guessing outputs.
- Workflow
  - Steps:
    1. Understand task and constraints.
    2. Inspect relevant files and dependencies.
    3. Draft a plan and test strategy.
    4. Make minimal code changes.
    5. Add/adjust tests.
    6. Run tools; fix failing tests/lints.
    7. Prepare a commit with a clear message.
    8. Summarize risks and follow-ups.
- Rules
  - Do not modify unrelated files.
  - Keep changes small and reversible.
  - Prefer pure functions and clear interfaces.
  - Ensure idempotent scripts.
  - Never fabricate tool outputs.

2) Task prompt template
- Task: Short description of the feature/bug.
- Constraints: Language, style, performance/security limits, compatibility.
- Acceptance criteria: Specific behaviors, test expectations, performance budgets, error handling.
- Context bundle: Key files, configs, logs, stack traces, API specs.
- Output format:
  - Proposed plan (bulleted).
  - Patch (unified diff).
  - New/updated tests.
  - Commands to run.
  - Commit message (conventional commits).
  - Brief verification report.

3) Planning + self-check prompts
- Before coding
  - “List the minimal files to touch and why.”
  - “Enumerate edge cases and error conditions.”
  - “Draft function signatures and data models.”
- After coding
  - “Run tests. For each failure, explain root cause and fix with smallest diff.”
  - “Check for security issues: input validation, injection, deserialization, secrets, SSRF, path traversal.”
  - “Assess performance impact and big-O changes.”

4) Code-diff prompting
- Force patch format
  - “Output changes as a single unified diff with correct paths and line numbers. Do not include files with no changes.”
- Guard rails
  - “If you are unsure about a file’s content, read it first using the file tool.”

5) Test-first prompting
- “Write failing tests that capture the bug/feature. Confirm they fail. Then implement code to pass them. Keep tests deterministic and fast.”

6) Error-handling and ambiguity prompts
- “If requirements are ambiguous, list clarifying questions. If you cannot proceed, stop and ask.”
- “When a tool or command fails, capture stdout/stderr verbatim and propose next steps.”

7) Multi-agent or multi-phase prompts
- Designer agent: produce design doc and API.
- Implementer agent: produce diff and tests.
- Reviewer agent: run static checks, look for regressions, ensure style and security, suggest improvements.
- Merger agent: finalize commit message and PR description.

Concrete example (adaptable)
System prompt
- You are an autonomous SWE agent. Optimize for correctness, safety, and minimal diffs. Use tools to read/modify files, run tests, and execute code. Never invent tool outputs.

Developer/task message
- Task: Add pagination to GET /users.
- Constraints: Node.js/Express, do not change DB schema, default pageSize=25, max=100, return totalCount.
- Acceptance criteria:
  - Query params: page (>=1), pageSize (1..100).
  - Response shape: { data: User[], page, pageSize, totalCount }
  - Tests: integration tests covering bounds and invalid inputs (400).
- Context: package.json, app.js, routes/users.js, db/usersRepo.js, tests/users.test.js, sample logs.
- Output format:
  - Plan, diff, new tests, commands, commit message, verification notes.

Verification checklist prompt
- Before finalizing, answer yes/no and provide evidence:
  - All tests pass locally?
  - Lint/format pass?
  - Backward compatibility preserved?
  - Security: no unvalidated input, no leaked secrets, safe queries?
  - Performance within budget?
  - Minimal diff?
  - Documented behavior and edge cases?

Common pitfalls and how prompts prevent them
- Hallucinated files or commands
  - Fix: Tool-only outputs, “read before write,” schema-constrained patches.
- Over-broad edits
  - Fix: “Touch minimal files,” diff-only output, small PR size constraint.
- Fragile tests
  - Fix: Deterministic tests guidance, stable seeds, no live network.
- Ignored acceptance criteria
  - Fix: Explicit acceptance list the agent must restate and map to changes/tests.
- Getting stuck in loops
  - Fix: Step limits, explicit exit criteria, “ask for help if blocked.”
- Missing security considerations
  - Fix: Security checklist baked into the prompt; require explicit confirmation.

Quick starter: copy-paste templates
- Role preamble
  - “You are a senior SWE agent working in a monorepo. Aim for minimal, test-verified changes. Use available tools; never assume outputs.”
- Output schema
  - “Respond in JSON:
    {
      "plan": [...],
      "diff": "unified diff",
      "tests": "...",
      "commands": ["..."],
      "commit_message": "...",
      "verification": {
        "tests_pass": "yes/no/unknown",
        "lint_pass": "yes/no/unknown",
        "notes": "..."
      }
    }”
- Self-check addendum
  - “Before finalizing, list three plausible failure modes of your change and how they’re mitigated.”

Beyond prompts: what else matters
- Good tool design: deterministic, fast, clear error messages, idempotent actions.
- Context management: smart retrieval of only relevant files/logs to avoid context bloat.
- Guarded execution: sandboxing, resource/time limits, rollback on failure.
- Observability: logs of actions, diffs, test results, decision traces.
- Human-in-the-loop options: approval gates and overrides.
- Evaluation suite: realistic tasks, regression tests, and metrics (pass rate, time to fix, patch size, review quality).

If you want, tell me your stack and agent runtime (e.g., CLI tools available, repo size, languages), and I’ll tailor concrete prompts and output schemas for your setup.