## Prompt Template: Generate Prismatic Component Unit Tests

**Goal:** Generate Jest unit tests for a specific component file (e.g., an action, connection, client function, utility), ensuring adequate coverage and adherence to testing rules.

**User Provides:**

- **Target Source File Path:** (e.g., `./src/actions/getWebinar.ts`)
- **Related Files (Optional):** (e.g., `client.ts` path if testing an action that uses it). Provide if not standard location.

**AI Actions:**

1.  **Confirm Understanding:** Restate the target file path.
2.  **(MANDATORY) Prerequisite Check & Reading:**
    - **Identify and READ all mandatory prerequisite documents** that apply to generating unit tests for _this specific file type_. This **MUST** include:
      - `rules/component/00-core-principles.md`
      - `rules/component/patterns/testing.md`
      - The rule relevant to the _type_ of file being tested (e.g., `rules/component/features/actions.md` if testing an action, `rules/component/patterns/api-client.md` if testing the client).
      - `rules/component/patterns/error-handling.md` if applicable based on source code.
    - **State which prerequisites were identified and read.** _Do not proceed to step 3 until this is complete._
3.  **Read Context & Target File:**
    - Verify the `Target Source File Path` exists.
    - Read the `Target Source File Path`.
    - Read related files needed for context (e.g., `client.ts`, `connections/index.ts`, shared types) identified from imports or user input.
4.  **Determine Test File Path:** Generate the corresponding test file path (e.g., `src/__tests__/actions/getWebinar.test.ts`) following conventions in the testing rules read in Step 2.
5.  **Identify Test Cases:** Analyze the target source file and relevant rules (e.g., `testing.md`) to identify necessary test cases:
    - **Happy Path:** Test successful execution with valid inputs.
    - **Error Handling:** Test expected error scenarios (e.g., API errors, invalid inputs, connection failures). Refer to error handling patterns if used.
    - **Edge Cases:** Test boundary conditions, optional parameters, different input combinations.
6.  **Propose Test File Code:**
    - Determine the target test file path based on conventions.
    - Generate the TypeScript code for the test file, including:
      - Imports for Jest (`describe`, `it`, `expect`, `jest`), the target function/component, and Prismatic testing utilities (`invoke`, `createConnection`, `logger` if needed).
      - Necessary mocks using `jest.mock()` or `jest.spyOn()` for dependencies (API client, connections, etc.), following strategies from the testing rules.
      - `describe` blocks for organization.
      - `it` or `test` blocks for each identified test case.
      - Appropriate testing utilities (e.g., `invokeAction`, `invokeTrigger`).
      - `expect` assertions to verify outcomes.
7.  **Request Confirmation:** Propose creating/updating the test file using `edit_file` with the generated code and ask for user approval.
8.  **(Upon Approval):** Report completion, listing the created/modified test file path. Suggest running tests (`npm test` or `yarn test`).

**Constraints:**

- Strict adherence to prerequisites identified in Step 2 is mandatory.
- Follow all core principles (`rules/component/00-core-principles.md`), structure rules (`rules/component/01-component-structure.md`), and testing patterns (`rules/component/patterns/testing.md`).
- Ensure generated test code correctly mocks dependencies and uses Prismatic testing utilities (`invoke`, `createConnection`).
- Test file location and naming must follow conventions defined in the relevant rules.
- Ensure sufficient test coverage for happy paths, error handling, and edge cases identified in Step 5.
