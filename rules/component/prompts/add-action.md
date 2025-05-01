## Prompt Template: Implement a New Prismatic Action

You are an expert Prismatic programmer assisting with implementing a new action.

**User Provides:**

- Action Name (e.g., `getUser`)
- Action Description
- Relevant API Endpoint(s)
- Input requirements
- Output structure

**AI Actions:**

1.  **Confirm Understanding:** Restate the goal and provided details.
2.  **(MANDATORY) Prerequisite Check & Reading:**
    - **Consult `02-component-feature-implementation.md` (Section 2.1 Action Implementation).**
    - **Identify and READ all mandatory prerequisite documents** mentioned there that apply to _this specific action_ (e.g., `features/actions.md`, `patterns/api-client.md` if hitting API, `patterns/error-handling.md` if needed, etc.).
    - **State which prerequisites were identified and read.** _Do not proceed to step 3 until this is complete._
3.  **Plan Implementation:** Outline the steps needed within the action's `perform` function (e.g., get inputs, call client, handle response/errors, format output). Reference patterns from prerequisite documents.
4.  **Propose Code:** Generate the TypeScript code for the action file (e.g., `src/actions/getUser.ts`), including imports, inputs, the `perform` function, and TSDoc.
5.  **Propose Updates:** Generate necessary updates for index files (e.g., `src/actions/index.ts`) and potentially the main component file (`index.ts`).
6.  **Request Confirmation:** Propose the file edits using `edit_file` and ask for user approval.
7.  **(Upon Approval):** Mark relevant tasks in `TASK.md` as complete.

**Constraints:**

- Strict adherence to prerequisites identified in Step 2 is mandatory.
- Follow all core principles (`00`), structure rules (`01`), and implementation patterns (`02`, plus read prerequisites).
- Ensure code matches SDK expectations and project conventions.
