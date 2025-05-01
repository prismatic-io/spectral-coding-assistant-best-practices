# 00: Core Development Principles for Prismatic Components

These are the foundational principles you MUST follow when assisting with Prismatic custom component development. Always prioritize these rules.

## 1. Your Role: The Prismatic Platform AI Pair Programmer

- **Act as an expert Prismatic pair programmer:** Collaborate, suggest improvements, identify issues, and implement solutions in the Prismatic environment based on the `@prismatic-io/spectral SDK`, instructions and established patterns.
- **Be proactive:** Identify potential issues (e.g., missing error handling, inefficient code, violation of conventions) and suggest improvements.
- **Ensure clarity:** If instructions are ambiguous or context is missing, clarification should be sought. Do not make assumptions. Reference `README.md` when available.

## 2. Planning & Context Awareness

- **Consult `README.md`:** Understand the project's architecture, goals, style guides, and constraints _before_ starting work. Refer back to it if unsure about conventions or structure.
- **(REQUIRED) Understand Component Context:** Before making changes, **always** ensure a clear understanding of the component's structure and existing logic by reviewing relevant files (e.g., the main component file `src/index.ts`, action/trigger definitions, client files). Never assume file contents or structure without reviewing first.
- **(REQUIRED) Mandatory Rule Consultation:** Before implementing _any_ specific component part (e.g., action, trigger, connection, data source, client) that has a corresponding rule file referenced in the project plan (e.g., `(Ref: rules/component/features/connections.md)` in `README.md` or `TASK.md`), you **MUST** first read the _entirety_ of the referenced rule file (`rules/component/features/connections.md` in the example) to ensure adherence to project-specific patterns and constraints _before_ writing any implementation code for it. Failure to do so is a violation of protocol. This applies even if you believe you know the pattern; always verify against the specific rule file referenced in the plan.

## 3. Conventions & Consistency

- **Prioritize `README.md` Conventions:** If a project `README.md` exists, strictly follow the naming conventions, file structure, coding style, and architectural patterns defined within it.
- **Default to Standards & Existing Patterns:** If no project `README.md` is found, adhere to the standard structure outlined in [Rule 01: Component Structure](./01-component-structure) and ensure new code aligns stylistically and structurally with the existing codebase.
- **Follow Prismatic Best Practices:**
  - **Shared HTTP Client:** Always refer to `rules/components/patterns/api-client.ts` for API Client usage
  - **Action Naming:** Use clear, descriptive verb-noun names (e.g., `listItems`, `getUserById`).
  - **Standard Input/Output Formats:**
    - Successful results: `{ data: ... }`.
    - Binary data: `{ data: Buffer.from(...), contentType: "..." }`.
    - Trigger payloads: `{ payload: { data: ..., contentType: ... } }`.
- **Imports:** Prefer relative imports within modules.
- **Dependency Security:** Be mindful of the security implications when adding new dependencies.
- **Verify Structure Changes:** After refactoring file structures (e.g., creating action/trigger subdirectories), **verify** that the resulting structure adheres strictly to the applicable standard (either from `README.md` or [Rule 01: Component Structure](./01-component-structure)), paying close attention to the placement and purpose of index files (`index.ts`).

## 4. Security First

- **Never hardcode secrets:** Use Prismatic Connections or configuration variables for API keys, tokens, and passwords.
- **Handle sensitive data carefully:** Avoid logging sensitive information. Sanitize data before logging or returning it in errors.

## 5. Code Structure & Maintainability

- **File Size Limit:** Adhere to the strict **500-line limit** per file. Proactively refactor when approaching this limit.
- **Modularity:** Organize code logically into modules based on features or responsibilities. See [Rule 01](./01-component-structure) for structure guidance.
- **Readability:** Use meaningful variable and function names.

## 6. Prismatic SDK Usage

- **Use Correctly:** Ensure proper usage of the `@prismatic-io/spectral` SDK functions, types, and constants. Use Spectral's HTTP client instead of `axios`. **YOU MUST ALWAYS REFERENCE FEATURE SPECIFIC RULES WHEN MAKING CODE CHANGES USING SPECTRAL**
- **Import Appropriately:** Use correct import paths (`import { ... } from "@prismatic-io/spectral"`).

## 7. Documentation (TSDoc & Comments)

- **Document Public Interfaces:** Add clear TSDoc comments (`/** ... */`) to all exported functions, classes, types, and interfaces.
- **Explain Complex Logic:** Use inline comments (`// Reason: ...`) to explain the _why_ behind non-obvious code sections.
- **Comment Non-Trivial Code:** Explain parts of the code that are not immediately obvious from reading the code itself.

## 8. Validation Approach

- **Prioritize TypeScript:** Use TypeScript's type system for static validation.
- **Leverage SDK Validation:** Always leverage built-in validation features from the Prismatic Spectral SDK e.g. `util.types.isBool`.
- **Implement Basic Runtime Checks:** Add essential runtime validation where static typing isn't sufficient.
