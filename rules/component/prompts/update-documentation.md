## Prompt Template: Update Prismatic Component Documentation

**Goal:** Update or generate documentation for a Prismatic component, including JSDoc comments in source files and updating the main `README.md`.

**User Provides:**

- **Target Component Path:** (e.g., `./src/actions` or a specific file like `./src/actions/getWebinar.ts`)
- **Scope (Optional):** Specify what to update:
  - `"all"` (Default): Review/update JSDoc across actions, connections, triggers, etc., and update `README.md` (if one exists at the root or component level).
  - `"readme"`: Focus only on `README.md`.
  - `"jsdoc"`: Focus only on JSDoc comments in `src/**/*.ts` within the target path.
  - Specific file path(s): (e.g., `./src/actions/getWebinar.ts`) to update JSDoc for that file only.
  - Specific section: (e.g., "README setup guide", "JSDoc for createRecord action")

**AI Actions:**

1.  **Confirm Understanding:** Restate the target path and scope.
2.  **(MANDATORY) Prerequisite Check & Reading:**
    - **Identify and READ all mandatory prerequisite documents** relevant to documentation. This **MUST** include:
      - `rules/component/00-core-principles.md` (Specifically Section 7: Documentation)
      - `rules/component/01-component-structure.md` (If updating `README.md`, for structure guidance)
    - **State which prerequisites were identified and read.** _Do not proceed to step 3 until this is complete._
3.  **Read Context & Files:**
    - Verify `Target Component Path` exists.
    - If scope includes JSDoc: Read relevant `.ts` files within the specified path/scope (actions, connections, triggers, client, index, types).
    - If scope includes README: Read the relevant `README.md` file.
4.  **Analyze & Identify JSDoc Updates (If in scope):**
    - Scan `.ts` files for missing or incomplete JSDoc blocks for exported functions, classes, types, actions, inputs, etc.
    - Check for presence and clarity of `@param`, `@returns`, `@throws`, `@description` tags, and inline `# Reason:` comments where applicable.
    - Prepare proposed additions/modifications for each relevant file.
5.  **Analyze & Identify README Updates (If in scope):**
    - Review `README.md` for completeness and accuracy based on component features (identified from reading source files like `index.ts`, `actions/index.ts`, etc.).
    - Check for:
      - Accurate component description.
      - Clear setup instructions (installation, connection configuration).
      - List of available actions/triggers/data sources with brief descriptions and key inputs.
      - Usage examples (simple integration flows).
      - Link to API documentation (if applicable).
    - Prepare proposed additions/modifications, focusing on clarity and accuracy.
6.  **Request Confirmation:** Propose the file edits for JSDoc and/or README using `edit_file` for each planned change and ask for user approval.
7.  **(Upon Approval):** Report completion, listing modified files (`README.md` and any `.ts` files with updated JSDoc).

**Constraints:**

- Strict adherence to prerequisites identified in Step 2 is mandatory.
- Follow documentation guidelines from `rules/component/00-core-principles.md` (Section 7).
- Ensure `README.md` structure aligns with `rules/component/01-component-structure.md` recommendations, if applicable.
- JSDoc should be clear, accurate, and cover all exported members.
- Inline comments (`// Reason: ...`) should explain non-obvious logic.
- `README.md` updates must accurately reflect the component's current functionality and inputs.
