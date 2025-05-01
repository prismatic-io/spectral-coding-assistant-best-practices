# Polling Trigger Patterns

## Spectral SDK Definitions

```typescript
import { pollingTrigger, input, util } from "@prismatic-io/spectral";

// Common interfaces for polling triggers
interface PollState {
  lastPoll?: string;
  cursor?: string | number;
  nextToken?: string;
}

interface ApiResponse<T> {
  data: T[];
  nextPageToken?: string;
  hasMore?: boolean;
}
```

## Implementation Patterns

### 1. Basic Polling Trigger

The standard pattern for a polling trigger that uses cursor-based pagination:

```typescript
import { pollingTrigger, input, util } from "@prismatic-io/spectral";

export const basicPollingTrigger = pollingTrigger({
  display: {
    label: "Poll for Changes",
    description: "Check for new or updated records",
  },

  inputs: {
    connection: input({
      label: "API Connection",
      type: "connection",
      required: true,
    }),
    pollInterval: input({
      label: "Poll Interval",
      type: "string",
      required: true,
      default: "300",
      comments: "Polling interval in seconds",
      clean: util.types.toInt,
    }),
  },

  perform: async (context, payload, inputs) => {
    try {
      const client = createApiClient(inputs.connection);
      const state = (await context.polling.getState()) as PollState;

      // Get last poll timestamp with fallback
      const lastPoll = state?.lastPoll || new Date(0).toISOString();

      // Fetch changes since last poll
      const changes = await client.get("/changes", {
        params: { since: lastPoll },
      });

      // Update state with current timestamp
      await context.polling.setState({
        lastPoll: new Date().toISOString(),
      });

      // Return if no changes
      if (!changes.data.length) {
        return { payload, polledNoChanges: true };
      }

      // Return changes for processing
      return {
        payload: {
          ...payload,
          body: { data: changes.data },
        },
        polledNoChanges: false,
      };
    } catch (error) {
      handleTriggerError(error);
    }
  },
});
```

### 2. Advanced Polling with Token Pagination

Example of a polling trigger that handles token-based pagination and rate limiting:

```typescript
interface TokenPollState extends PollState {
  retryCount?: number;
}

export const tokenBasedPollingTrigger = pollingTrigger({
  display: {
    label: "Poll with Pagination",
    description: "Fetch paginated changes with rate limiting",
  },

  inputs: {
    connection: input({
      label: "API Connection",
      type: "connection",
      required: true,
    }),
  },

  perform: async (context, payload, inputs) => {
    const state = context.polling.getState() as TokenPollState;
    const client = createRateLimitedClient(inputs.connection);

    const response = await client.get("/changes", {
      params: {
        pageToken: state.nextToken,
        since: state.lastPoll,
      },
    });

    await context.polling.setState({
      nextToken: response.data.nextPageToken,
      lastPoll: new Date().toISOString(),
    });

    return {
      payload: { ...payload, body: { data: response.data.items } },
      polledNoChanges: !response.data.items.length,
    };
  },
});
```

## Best Practices

1. **Efficient Cursor Management**

   - Always store and use cursors for pagination
   - Use the most efficient cursor type for the API (timestamps, IDs, etc.)
   - Handle cursor edge cases (e.g., cursor expiration)
   - Implement cursor cleanup for long-running instances

2. **Error Resilience**

   - Implement proper error handling with specific error types
   - Log meaningful error information
   - Use exponential backoff for rate limits
   - Track retry attempts in state

3. **Performance Optimization**

   - Use appropriate batch sizes
   - Process only what has changed
   - Avoid unnecessary data transformation

4. **State Management**
   - Keep poll state minimal and relevant
   - Handle state persistence edge cases
   - Consider state cleanup for long-running instances
   - Use TypeScript interfaces for type safety
   - Handle state initialization properly

## Common Edge Cases

1. **Data Consistency**

   - Handle duplicate records
   - Deal with out-of-order updates
   - Manage data conflicts
   - Validate data integrity

2. **Error Recovery**

   - Handle network timeouts
   - Manage API versioning changes
   - Recover from partial failures
   - Implement proper error logging
   - Use typed error handling

3. **Authentication**
   - Validate connection status

## Error Handling Example

```typescript
interface ErrorResponse {
  code: string;
  message: string;
  details?: unknown;
}

function handleTriggerError(error: unknown): never {
  if (error instanceof Error) {
    // Handle standard errors
    throw new Error(`Trigger failed: ${error.message}`);
  }

  const response = error as ErrorResponse;
  switch (response.code) {
    case "RATE_LIMIT":
      throw new Error(
        `Rate limit exceeded. Try again in ${response.details} seconds`,
      );
    case "AUTHENTICATION":
      throw new Error("Authentication failed. Please check your credentials.");
    default:
      throw new Error(`Unknown error: ${JSON.stringify(response)}`);
  }
}
```

## External References

- [Prismatic Polling Triggers Documentation](https://prismatic.io/docs/custom-connectors/triggers)
- [API Polling Best Practices](https://developers.google.com/drive/api/guides/best-practices#polling)
