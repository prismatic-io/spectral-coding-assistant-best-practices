# 01: Component Structure for Prismatic Components

This rule defines the standard directory structure and file organization for Prismatic custom components.

## 1. Standard Directory Structure

A typical component should follow this structure. For simpler components with few actions/triggers, single files like `src/actions.ts` may suffice. However, for components with multiple features, use the directory structure below:

```
my-component/
├── src/
│   ├── actions/             # Action implementations
│   ├── triggers/            # Trigger implementations
│   ├── dataSources/         # Data Source implementations
│   ├── inputs.ts            # Reusable input definitions
│   ├── connections.ts       # Connection definitions
│   ├── client.ts            # Shared API client logic
|   ├── index.ts             # Main component definition
│   └── util/                # Generic helpers, types, constants
├── test/
│   ├── harness.ts           # Testing harness setup
└── [Other project files]
```

**Key Organization Principles:**

- **Modularity:** For complex components, group features in directories (`src/actions/`, `src/triggers/`, `src/dataSources/`) with `index.ts` files to re-export implementations. Simpler components might use single files (`src/actions.ts`).
- **Reusable Inputs:** Define shared inputs in `src/inputs.ts` for consistency.
- **API Client:** Implement shared client logic in `src/client.ts`.
- **Utilities:** Place domain-specific helper functions, types, or constants in `src/util/`.
- **Refactoring to Directories:** When transitioning from a single implementation file (e.g., `src/actions.ts`) to a directory structure (e.g., `src/actions/`), the original file **must** be either:
  a) **Moved and renamed** to serve as the index file within the new directory (e.g., move `src/actions.ts` to `src/actions/index.ts`).
  b) **Replaced** by a new index file (e.g., `src/actions/index.ts`) that correctly imports and exports the contents of the modules within the directory. Ensure the previous file is removed or its contents fully migrated.

## 2. Core Component File Implementations

### `src/index.ts` (Component Entry Point)

```typescript
import { component } from "@prismatic-io/spectral";
import connections from "./connections"; // If exists
import actions from "./actions";
import triggers from "./triggers";
import dataSources from "./dataSources";

export default component({
  key: "myComponent", // **`key`:** Must be unique and typically uses camelCase. Should match the name in `README.md`.
  public: false, // NEVER use public: true
  display: {
    label: "My Component",
    description: "A brief description of what this component does.",
    iconPath: "icon.png",
  },
  connections: connections ? [connections] : undefined,
  actions,
  triggers,
  dataSources,
});
```

-

### `inputs/` or `input.ts`

- **Prerequisite:** Before writing _any_ code to develop an input, **you MUST first read `rules/component/features/inputs.md`** to understand the required patterns and examples.
- **Purpose:** Centralize input definitions for reusability.
- **Import:** Use these in the `inputs` block of actions/triggers/dataSources.

### `client.ts`

- **Prerequisite:** Before writing _any_ code to develop a client factory, **you MUST first read `rules/component/patterns/api-client.md`** to understand the required patterns and examples.
- **Purpose:** Encapsulate API client configuration and authentication.
- **Usage:** Call within action/trigger `perform` functions that need API access.
- **Always** favor using spectrals createHttpClient function over external libraries like axios, unless explicitly told otherwise

## 3. Tooling Guidance

- **Preqrequisite:** Before making any environment changes, check for existing files. If they exist, do not proceed with changes.
- Use `webpack` to bundle (build) the component into `dist`. Use `webpack.config.js` with `copy-webpack-plugin` and `ts-loader` to bundle to `commonjs2`. Copy any files in `assets` to root of `dist`. There are no `externals`.
- Use `@biomejs/biome` for `lint` and `format` scripts.

## 4. Placement Guidance Summary

This table shows where different component elements should be placed:

| Element              | Location                                            |
| -------------------- | --------------------------------------------------- |
| Actions              | `src/actions/`                                      |
| Triggers             | `src/triggers/`                                     |
| Data Sources         | `src/dataSources/`                                  |
| Connections          | `src/connections.ts`                                |
| Reusable Inputs      | `src/inputs.ts`                                     |
| Shared API Client    | `src/client.ts`                                     |
| Utilities            | `src/util/`                                         |
| Tests                | `test/`                                             |
| Component Definition | `src/index.ts` (root)                               |
| Types/Interfaces     | Near where they're used or in `src/util/` if shared |

See [Rule 02: Component Feature Implementation](./02-component-feature-implementation) for details on implementing the specific features within these files.
