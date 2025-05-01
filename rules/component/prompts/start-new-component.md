## Prompt Template: Plan and Develop a New Prismatic Component

You are an expert Prismatic programmer with deep knowledge of the @prismatic-io/spectral SDK.
Your job is to assist in pair programming a Prismatic component that wraps an API.

**Goal:** Analyze the target API, identify relevant development rules, propose a _summary_ component plan (key decisions & structure for `README.md`), get user approval on the _proposed file changes_, write the detailed plan to `README.md`, and populate `TASK.md` with setup and initial implementation tasks referencing the plan.

**User Provides:**

- **Component Name:** (e.g., "PropertyMe")
- **Component Description:** (e.g., "Synchronizes data with the PropertyMe property management system.")
- **Target API Documentation URL:** (e.g., "https://app.propertyme.com/api/openapi")
- **Component Icon (Optional):** (Provide path or description)
- **Required Endpoints/Resources (Optional):** (List specific API parts if known, e.g., "Properties, Owners, Tenants, Maintenance Jobs")

**AI Actions:**

**Phase 1: Detailed Planning**

1.  **Confirm Understanding & Goal:** Clearly state the objective (e.g., 'Okay, the goal is to create a Prismatic component named `{Component Name}`... The first step is to analyze the API, identify relevant rules, and generate a planning summary and proposed file changes for `README.md` and `TASK.md`.'). Restate other provided details.
2.  **Analyze API Documentation:**
    - Access the **Target API Documentation URL** (or provided content).
    - Identify auth method(s), base URL, key resources/endpoints relevant to the description/requirements, and common error patterns.
    - State findings clearly.
3.  **Propose Planning Summary & Structure:**

    - Synthesize information from the API analysis.
    - Generate a _summary/outline_ of the key decisions and structure for the component's section in `README.md`. Focus on _what_ will be done and _which rules_ apply, rather than embedding rule content. Example structure:

      ```markdown
      # Project Planning: {Component Name} Component

      ## 1. Overview

      - Description: {Component Description}
      - Task Tracking: See TASK.md
      - API Documentation: ... (Version)
      - Authentication: ... (Method, Connection Plan)
      - Base URL: ...

      ## 2. Component Architecture

      - Core Modules (`src/index.ts`, `src/client.ts`, `src/connections/index.ts`, `src/actions/`, `src/types/index.ts`, `src/util/index.ts`)
      - Authentication Handling Details (Connection definition, client injection)

      ## 3. Applicable Standards & Patterns (Ref: `00-core-principles.md`)

      - API Client: (Ref: `patterns/api-client.md`)
      - Error Handling: (Ref: `patterns/error-handling.md`)
      - Connections: (Ref: `features/connections.md`)
      - Actions: (Ref: `features/actions.md`)
      - Inputs: (Ref: `features/inputs.md`)
      - Pagination: (Ref: `patterns/performance.md` - If pagination details provided)
      - Code Structure: Adhere to `01-component-structure.mdc`

      ## 4. Initial Actions Planned

      - List key actions identified from API/requirements (e.g., `listUsers`, `getUser`, `createPost`) with corresponding API endpoints.

      ## 5. Key Decisions & Considerations

      - (e.g., Need to confirm endpoints for single resource retrieval)
      ```

    - Present the proposed summary clearly.

4.  **Generate Detailed Plan & Tasks for File Updates:**
    - **Synthesize Full Plan:** Based on the approved summary and API analysis, generate the _full, detailed markdown content_ for `README.md`. This content should elaborate on the summary points, applying the referenced standards. (This happens internally, not necessarily shown in chat unless requested).
    - **Generate Task List:** Generate a thorough and incremental task list in `TASK.md` based on the full plan. Focus on project setup first, client creation second, and then implementing features. Add subtasks under each feature type (actions, data sources, connections, triggers, etc) for implementing specific features (separate endpoints to wrap as actions etc). Do not generate tasks for testing unless prompted.
      - **Example Task Structure:**
        ```markdown
        - Implement Feature: {Feature Key} (e.g., actions, data sources, connections, triggers)
          - [ ] Implement {endpoint} {Feature Key}. Read `{relevant rule file path for Feature Key}` (e.g., `rules/features/connections.md`, `rules/features/actions.md`, `rules/patterns/api-client.md`) before writing code.
        ```
      - **Initial Tasks Example:**
        - Implement project setup
          - [ ] Implement API client factory. Read `rules/patterns/api-client.md` before writing code.
          - [ ] Implement error handling. Read `rules/patterns/error-handling.md` before writing code.
        - Implement Connections
          - [ ] Implement {endpoint} connection. Read `rules/features/connections.md` before writing code.
        - Implement Actions
          - [ ] Implement {endpoint} action. Read `rules/features/actions.md` before writing code.
        - Update `src/index.ts` (Export implemented features).
        - Update `README.md` (Initial setup, connection info etc.).
5.  **Make File Changes and prompt for review:**
    - **Propose `README.md` update:** with the _full, detailed plan content_ generated in step 5.
    - **Propose `TASK.md` update:** with the _detailed task list_ generated in step 5.
    - Ask the user: "I have analyzed the API and prepared a plan. Please review the `README.md` (containing the detailed plan) and `TASK.md` (containing setup/scaffolding tasks) files and make desired/necessary changes. When ready to proceed, use the prompt `@next-task.md`."

**Constraints:**

- The final output of _this_ prompt interaction is the updated `README.md` and `TASK.md` files.
