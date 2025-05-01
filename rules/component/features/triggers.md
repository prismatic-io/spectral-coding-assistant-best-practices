# Trigger Implementation Guide

## What is a Trigger?

A trigger is an entry point that determines when and how a Prismatic integration flow should execute. Triggers can:

- Respond to webhook requests (webhook triggers)
- Run on a schedule (scheduled triggers)
- Poll for changes (polling triggers)

## Trigger Lifecycle

Every trigger has three main lifecycle events:

1. **Deploy** (`onInstanceDeploy`)

   - Called when an integration instance is deployed
   - Used to set up resources (webhooks)
   - Can store instance-specific state

2. **Execute** (`perform`)

   - Called when trigger conditions are met
   - Processes incoming data
   - Can branch to different flow paths
   - Returns data for downstream steps

3. **Delete** (`onInstanceDelete`)
   - Called when an integration instance is removed
   - Cleans up resources (webhooks, connections, etc.)
   - Should handle cleanup failures gracefully

## Basic Trigger Structure

```typescript
import { trigger, input } from "@prismatic-io/spectral";

export const basicTrigger = trigger({
  // Display information
  display: {
    label: "Basic Trigger",
    description: "Description of what the trigger does",
  },

  // Input configuration
  inputs: {
    exampleInput: input({
      label: "Example Input",
      type: "string",
      required: true,
      comments: "Description of the input",
    }),
  },

  // Execution configuration
  synchronousResponseSupport: true, // Returns response synchronously
  scheduleSupport: "invalid", // Does not run on schedule
  allowsBranching: false, // Does not support multiple flow paths

  // Main execution function
  perform: async (context, payload, inputs) => {
    // Process trigger event
    return {
      payload: {
        ...payload,
        body: {
          /* processed data */
        },
      },
    };
  },

  // Deploy handler
  onInstanceDeploy: async (context, inputs) => {
    // Setup resources
  },

  // Delete handler
  onInstanceDelete: async (context, inputs) => {
    // Cleanup resources
  },
});
```

## Trigger Types

### 1. Webhook Triggers

- Respond to HTTP requests
- Can return synchronous responses
- Often require authentication/verification
- See `webhook-patterns.md` for detailed implementations

### 2. Polling Triggers

- Check for changes periodically
- Maintain state between executions
- Handle pagination and rate limiting
- See `polling-triggers.md` for detailed implementations

### 3. Scheduled Triggers

- Run at specified intervals
- Don't respond to external events
- Can't return synchronous responses

## Common Properties

1. **Execution Control**

   - `synchronousResponseSupport`: Can return immediate response
   - `scheduleSupport`: Can run on schedule
   - `allowsBranching`: Supports multiple flow paths

2. **Context**

   - `context.instanceState`: Persistent instance data
   - `context.webhookUrls`: Available webhook endpoints
   - `context.logger`: Logging utilities

3. **Payload**
   - `payload.headers`: Request headers
   - `payload.body`: Processed request body
   - `payload.rawBody`: Raw request data

## Best Practices

1. **Error Handling**

   - Handle all lifecycle event errors
   - Provide meaningful error messages
   - Clean up resources on failure

2. **State Management**

   - Use instance state sparingly
   - Clean up old state data
   - Handle missing state gracefully

3. **Resource Management**
   - Create resources in `onInstanceDeploy`
   - Clean up in `onInstanceDelete`
   - Handle partial setup/cleanup

## See Also

- [Webhook Implementation Patterns](webhook-patterns.md)
- [Polling Implementation Patterns](polling-triggers.md)
- [Prismatic Triggers Documentation](https://prismatic.io/docs/custom-connectors/triggers)
