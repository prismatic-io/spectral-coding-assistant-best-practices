# Error Handling Patterns

## Overview

This document outlines standardized patterns for implementing error handling in Prismatic components, ensuring consistent, predictable, and user-friendly error management across all component types.

## Error Handling Patterns

### 1. API Response Error Handling

For components interacting heavily with an API, it's beneficial to create a dedicated error handling utility to standardize logging and error re-throwing for common API client errors.

**Example Utility (`src/util/errorHandling.ts`):**

```typescript
import { util } from "@prismatic-io/spectral"; // Or appropriate type for logger

/**
 * Handles common API errors from the HTTP client.
 * Logs the error details and throws a standardized error message.
 *
 * @param error The error object caught.
 * @param logger The Spectral logger instance (typically context.logger).
 * @param actionName The name of the action/context where the error occurred.
 * @throws Throws a new Error with a formatted message.
 */
import { ActionLogger } from "@prismatic-io/spectral";

export const handleError = (
  error: unknown,
  logger: ActionLogger,
  actionName: string,
): never => {
  let errorMessage = "An unknown error occurred";
  if (error instanceof Error) {
    errorMessage = error.message;
  }

  let statusCode: number | undefined = undefined;
  let responseData: unknown = undefined;

  logger.error(`${actionName} Error: ${errorMessage}`, {
    originalError: error,
  });

  throw new Error(
    `Failed to ${actionName.toLowerCase()}${statusCode ? ` (Status ${statusCode})` : ""}: ${errorMessage}${
      responseData ? ` - Details: ${JSON.stringify(responseData)}` : ""
    }`,
  );
};
```

**Usage within an Action (`src/actions/someAction.ts`):**

```typescript
import { action /* ... */ } from "@prismatic-io/spectral";
import { createGoToWebinarClient } from "../client";
import { handleError } from "../util/errorHandling";

export const someAction = action({
  // ... display, inputs ...
  perform: async (context, { connection /* ... inputs */ }) => {
    const client = await createGoToWebinarClient(connection);
    const actionContext = "Some Action"; // e.g., "List Webinars", "Get Webinar (123)"

    try {
      const response = await client.get("/some-endpoint");

      // Optional: Validate response with Zod
      // const validatedData = SomeSchema.parse(response.data);
      // return { data: validatedData };

      return { data: response.data };
    } catch (error: unknown) {
      handleError(error, context.logger, actionContext);
    }
  },
});
```

## Best Practices

### 1. Error Handling Hierarchy

- Use specific error types for different error scenarios
- Provide clear error messages with actionable information
- Include contextual details to aid troubleshooting
- Maintain consistent error structure across components

### 2. Error Presentation

- Make error messages user-friendly
- Use appropriate error codes for classification
- Sanitize sensitive information in error details

### 3. Error Recovery

- Include proper resource cleanup in error paths

### 4. Error Documentation

- Document common error codes and causes
- Include troubleshooting steps in component documentation
- Provide examples of error handling in action documentation

## External References

1. @Prismatic Error Handling Documentation
2. @Node.js Error Handling Best Practices
3. @HTTP Status Codes
