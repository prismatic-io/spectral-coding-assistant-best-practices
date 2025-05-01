# 02: Component Feature Implementation

This rule guides the implementation of specific component features (Actions, Triggers, Data Sources) using the Prismatic Spectral SDK and established patterns. It builds upon the structure defined in [Rule 01: Component Structure](./01-component-structure) and follows the principles in [Rule 00: Core Principles](./00-core-principles).

## 1. General Implementation Guidelines

- **Use SDK Functions:** Utilize the appropriate functions from `@prismatic-io/spectral` (e.g., `action`, `trigger`, `dataSource`).
- **TSDoc:** Add TSDoc comments to explain purpose, inputs, and outputs for all exported features.
- **Reference Existing Code:** You may reference existing implementations within the project for similar features.

## 2. Feature-Specific Implementation Process

### 2.1 Action Implementation

- **Prerequisite 1 (Core Action):** Before writing _any_ code for a new action, **you MUST first read `rules/components/features/actions.md`** to understand the required patterns and examples.
- Use the `action` function from `@prismatic-io/spectral`.
- **Prerequisite 2 (Conditional Patterns):** _During_ implementation, assess if the action involves the following. If so, **you MUST read the specified pattern file _before_ implementing that aspect**:
  - Interacting with the target API? -> **Read `rules/components/patterns/api-client.md`**.
  - Handling potential API errors in a specific way? -> **Read `rules/components/patterns/error-handling.md`**.
  - Retrieving lists of data that might require pagination? -> **Read `rules/components/patterns/performance.md`** (specifically regarding pagination) and reference [Prismatic's Pagination Documentation](:/prismatic.io/docs/custom-connectors/pagination).
- Adhere strictly to patterns found in the prerequisite documents.
- **Always** check for existing inputs before creating new ones for a given action.
- _(Add other action-specific rules here if needed)_

### 2.2 Trigger Implementation

- **Prerequisite 1 (Core Trigger):** Before writing _any_ code for a new trigger, **you MUST first read `rules/components/features/triggers.md`**.
- Use the `trigger` function from `@prismatic-io/spectral`.
- **Prerequisite 2 (Conditional Patterns):** _During_ implementation, assess if the trigger involves the following. If so, **you MUST read the specified pattern file _before_ implementing that aspect**:
  - Interacting with the target API (e.g., polling)? -> **Read `rules/components/patterns/api-client.md`**.
  - Handling potential API errors? -> **Read `rules/components/patterns/error-handling.md`**.
  - Managing state or processing potentially large datasets? -> **Read `rules/components/patterns/performance.md`**.
- Adhere strictly to patterns found in the prerequisite documents.
- _(Add other trigger-specific rules here if needed)_

### 2.3 Data Source Implementation

- **Prerequisite 1 (Core Data Source):** Before writing _any_ code for a new data source, **you MUST first read `rules/components/features/data-sources.md`**.
- Use the `dataSource` function from `@prismatic-io/spectral`.
- **Prerequisite 2 (Conditional Patterns):** _During_ implementation, assess if the data source involves the following. If so, **you MUST read the specified pattern file _before_ implementing that aspect**:
  - Interacting with the target API? -> **Read `rules/components/patterns/api-client.md`**.
  - Handling potential API errors? -> **Read `rules/components/patterns/error-handling.md`**.
  - Retrieving lists that might require pagination? -> **Read `rules/components/patterns/performance.md`** (specifically regarding pagination).
- Adhere strictly to patterns found in the prerequisite documents.
- _(Add other data-source-specific rules here if needed)_

## 3. Common Implementation Patterns (Reference Documents)

_The documents below contain guidance for common implementation needs. They are referenced as prerequisites in Section 2 when applicable._

- **API Client:** `rules/components/patterns/api-client.md`
- **Error Handling:** `rules/components/patterns/error-handling.md`
- **Performance (incl. Pagination):** `rules/components/patterns/performance.md`
