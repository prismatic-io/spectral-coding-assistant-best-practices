# Input Field Implementation Rules

## Overview

This document outlines the standardized patterns for implementing input fields in Prismatic components using the Spectral SDK.

## Input Field Types

The following input field types are available:

### Basic Types

- `string`: Basic text input
- `data`: Data input (similar to string but for data)
- `text`: Multi-line text input
- `password`: Secure password input
- `boolean`: True/false input
- `code`: Code editor with syntax highlighting
- `date`: Date input
- `timestamp`: Date and time input
- `flow`: Flow reference input

### Advanced Types

- `conditional`: For conditional logic expressions
- `connection`: For component connections
- `objectSelection`: For selecting objects and their fields
- `objectFieldMap`: For mapping fields between objects
- `jsonForm`: For complex JSON form inputs
- `dynamicObjectSelection`: For dynamic object selection
- `dynamicFieldSelection`: For dynamic field selection

## Input Field Structure

### Base Input Field Properties

```typescript
interface BaseInputField {
  label: string; // Display label for the input
  placeholder?: string; // Placeholder text
  required?: boolean; // Whether input is required
  clean?: InputCleanFunction; // Function to clean/validate input and perform type coercion
  comments?: string; // Help text for the input
  default?: unknown; // Default value
  example?: string; // Example value
}
```

### Collection Types

Two collection types are available:

- `valuelist`: Array of values
- `keyvaluelist`: Array of key-value pairs

## Implementation Examples

### Basic String Input

```typescript
const nameInput = input({
  label: "Name",
  type: "string",
  required: true,
  placeholder: "Enter full name",
  comments: "The full name of the user",
  example: "John Doe",
  clean: util.types.toString,
});
```

### Password Input

```typescript
const apiKeyInput = input({
  label: "API Key",
  type: "password",
  required: true,
  comments: "Your API key from the dashboard",
  clean: util.types.toString,
});
```

### Code Input with Syntax Highlighting

```typescript
const jsonConfigInput = input({
  label: "Configuration",
  type: "code",
  language: "json",
  required: true,
  comments: "Enter JSON configuration",
  example: '{"key": "value"}',
  clean: util.types.toJSON,
});
```

### Value List Collection

```typescript
const tagsInput = input({
  label: "Tags",
  type: "string",
  collection: "valuelist",
  required: false,
  comments: "Enter multiple tags",
  example: "production,staging,dev",
});
```

### Key-Value List Collection

```typescript
const headersInput = input({
  label: "Headers",
  type: "string",
  collection: "keyvaluelist",
  required: false,
  comments: "HTTP headers to include",
  example: "Content-Type: application/json",
  clean: util.types.keyValPairListToObject,
});
```

### Connection Input

```typescript
const connectionInput = input({
  label: "Connection",
  type: "connection",
  required: true,
  comments: "Select the connection to use",
});
```

## Best Practices

1. **Type Coercion**

   - Always use type coercion utilities in clean functions:

   ```typescript
   import { util } from "@prismatic-io/spectral";

   const quantity = input({
     label: "Quantity",
     type: "string",
     default: "0",
     required: true,
     clean: util.types.toNumber,
   });
   const name = input({
     label: "Name",
     type: "string",
     required: true,
     clean: util.types.toString,
   });
   ```

2. **Default Values**

   - Provide sensible defaults when possible
   - Document default values in comments
   - Use the `default` property for simple values
   - For collections, use `default` with appropriate type:

     ```typescript
     // For valuelist
     collection: "valuelist",
     default: ["value1", "value2"]


     // For keyvaluelist
     collection: "keyvaluelist",
     default: [{ key: "header1", value: "value1" }]
     ```

3. **Required Fields**
   - Mark fields as required only when absolutely necessary
   - Provide clear error messages for missing required fields
   - Consider providing defaults for optional fields

## Error Handling

1. **Input Validation**

   - Return clear error messages
   - Use type coercion early

2. **Error Messages**
   - Be specific about what went wrong
   - Provide guidance on how to fix the error
   - Include relevant context

## Security Considerations

1. **Sensitive Data**
   - Use `password` type for sensitive inputs
   - Never log sensitive input values
   - Use connections for API credentials
   - Consider using environment variables for sensitive defaults

## Using Inputs in Actions

1. **Action Definition**

   ```typescript
   import { action, input } from "@prismatic-io/spectral";

   export const myAction = action({
     display: {
       label: "My Action",
       description: "Performs an action",
     },
     inputs: {
       name: nameInput,
       apiKey: apiKeyInput,
       headers: headersInput,
     },
     perform: async (context, { name }) => {
       // Action logic
       const result = await someApiCall(name);
       return { data: result };
     },
   });
   ```

2. **Connection Usage**

```typescript
const myActionWithConnection = action({
  display: {
    label: "Action With Connection",
    description: "Uses a connection",
  },
  inputs: {
    connection: connectionInput,
    data: dataInput,
  },
  perform: async (context, { connection, data }) => {
    const client = createClient(connection);

    return await client.process(data);
  },
});
```
