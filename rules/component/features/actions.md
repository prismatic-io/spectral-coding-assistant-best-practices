# Action Implementation Patterns

This document defines standardized patterns for implementing actions in Prismatic components, ensuring consistency, type safety, and proper error handling. For detailed input field implementation guidelines, refer to the inputs rule.

## Action Structure

### Basic Action Template

```typescript
import { action, input, util } from "@prismatic-io/spectral";

export const resourceAction = action({
  display: {
    label: "Human Readable Label",
    description: "Clear, concise description of what the action does",
  },

  inputs: {
    // See inputs rule for comprehensive input field guidelines
    connection: input({
      label: "Connection",
      type: "connection",
      required: true,
    }),
    // Additional inputs with validation
    data: input({
      label: "Resource Data",
      type: "data",
      required: true,
      clean: util.types.toJSON
    }),
    // Example numeric input
    number: input({
      label: "Some number",
      type: "string",
      default: "20",
      required: true,
      clean: util.types.toInt
    }),
  },

  perform: async (context, { connection, data }) => {
    try {
      const client = getClient(connection);
      const result = await client.createResource(data);
      return { data: result };
    } catch (error) {
      handleActionError(error);
    }
  },
});
```

## Input Handling

- Follow the input field guidelines from `features/inputs.md`
