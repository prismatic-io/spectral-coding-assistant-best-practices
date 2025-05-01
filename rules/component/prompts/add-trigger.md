## Prompt Template: Implement a New Prismatic Trigger

You are an expert Prismatic programmer assisting with implementing a new trigger.

**User Provides:**

- Trigger Name (e.g., `newRecordWebhook`, `pollUpdatedContacts`)
- Trigger Type (e.g., `webhook`, `polling`, `app-event-webhook`)
- Trigger Description
- Relevant External Details (e.g., API endpoint for polling, expected webhook source/events, details for `onInstanceDeploy`/`onInstanceDelete` if applicable)
- Input requirements (e.g., connection, configuration options like event types, polling interval)
- Expected Output/Payload Structure (What the trigger should pass to the flow)
- Specific Response Requirements (For webhook triggers, e.g., custom status code, headers, body needed for the caller)
- Synchronous/Schedule Support (`synchronousResponseSupport`, `scheduleSupport` flags)

**AI Actions:**

1.  **Confirm Understanding:** Restate the goal, trigger type, and provided details.
2.  **(MANDATORY) Prerequisite Check & Reading:**
    - **Consult `02-component-feature-implementation.md` (Section 2.2 Trigger Implementation)** (_Assuming this section exists or will be created_).
    - **Identify and READ all mandatory prerequisite documents** mentioned there that apply to _this specific trigger_. This **MUST** include:
      - `00-core-principles.md`
      - `rules/component/examples/polling-triggers.md` (if type is `polling`)
      - `rules/component/examples/webhook-patterns.md` (if type is `webhook` or `app-event-webhook`)
      - Other relevant patterns (e.g., `patterns/api-client.md`, `patterns/error-handling.md`).
    - **Reference External Docs:** Review the [Prismatic Trigger Documentation](https://prismatic.io/docs/custom-connectors/triggers/).
    - **State which prerequisites and external docs were identified and read.** _Do not proceed to step 3 until this is complete._
3.  **Plan Implementation:**
    - Outline the steps needed within the trigger's `perform` function (e.g., handle payload, validate signature, call client, process results, handle errors, format output payload).
    - If `polling`, detail the state management (`getState`/`setState`) and data fetching logic (potentially using `invokeAction`).
    - If `app-event-webhook`, detail the `onInstanceDeploy` and `onInstanceDelete` logic (e.g., registering/deregistering webhooks).
    - Reference specific patterns from the prerequisite documents read in Step 2.
4.  **Propose Code:** Generate the TypeScript code for the trigger file (e.g., `src/triggers/newRecordWebhook.ts`), including imports, inputs, the `perform` function (and `onInstanceDeploy`/`onInstanceDelete` if applicable), TSDoc, and appropriate flags (`synchronousResponseSupport`, `scheduleSupport`).
5.  **Propose Updates:** Generate necessary updates for index files (`src/triggers/index.ts`) and the main component file (`src/index.ts`).
6.  **Request Confirmation:** Propose the file edits using `edit_file` and ask for user approval.
7.  **(Upon Approval):** Mark relevant tasks in `TASK.md` as complete.

**Constraints:**

- Strict adherence to prerequisites identified in Step 2 is mandatory.
- Follow all core principles (`00`), structure rules (`01`), and implementation patterns (`02`, plus read prerequisites like `polling-triggers.md` or `webhook-patterns.md`).
- Ensure code matches SDK expectations ([Prismatic Trigger Docs](https://prismatic.io/docs/custom-connectors/triggers/)) and project conventions.
- Correctly implement polling logic (`getState`, `setState`, `invokeAction`, `polledNoChanges`) as shown in `polling-triggers.md` if applicable.
- Correctly implement webhook/event logic (`onInstanceDeploy`, `onInstanceDelete`, custom `response`) as shown in `webhook-patterns.md` if applicable.
