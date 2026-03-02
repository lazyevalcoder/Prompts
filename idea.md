## One line summary:
chatgpt code interpreter /advanced data analysis like tool (but minimal)v

## Features: (think like a technical product manager)
- LLM backend is llama.cpp running locally. use OpenAI sdk and llama.cpp endpoint.
- sandbox code execution like repl (no containers/vm)
- when llm generates wrong code, code sent back to llm with error to get corrected.
- initially need a single working code file without front end
- need solid test cases with sample csv created and testing response from llm

## ux/ui: (think like a senior designer)
- for now, no front end.
- interactive command line mode

## control flow: (think like a senior design engineer)
- user runs the main.py
- prompted for csv file and task/question
- user provides file and the question
- user gets the result in natural English (no code)

## testing: (think like a senior qa engineer)
- validate critical functions and failure paths independently
- ensure components interact correctly across full execution path
- test user flows, invalid inputs, and clear feedback messaging

## Checklist: (think like an architect and product manager)
backend
- assumptions
- unknowns
- what must work
- what must not happen
- success criteria
- failure criteria
- failure modes
- hardest technical component
- why it may fail

frontend
- What must work perfectly?
- Who is this really for?
- What problem are we actually solving?
- What would confuse a first-time user?
- Where could users get stuck or drop off?
- What assumptions are we making?
- What happens when something fails?
- What are we overcomplicating?
- What would make this feel slow or heavy?
- What would make users lose trust?

## Implementation protocol:
1. Each implementation phase must include:
- Unit validation
- Integration validation (if applicable)
- Negative case validation
2. No feature may proceed to next phase without:
- Verified behavior
- Defined failure modes
- Explicit pass criteria
3. A phase is considered complete only if:
- All tests defined in the phase pass
- No known failure states remain unresolved
- Output matches defined success criteria
4. Refactoring of prior phases is allowed only if:
- It fixes a failing test
- It does not expand scope
5. A phase must not assume functionality from future phases. Each phase must be independently verifiable.
