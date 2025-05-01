# Webhook Integration Patterns

## Type Definitions

```typescript
import {
  HttpResponse,
  TriggerPayload,
  trigger,
  ActionContext,
  util,
} from "@prismatic-io/spectral";

// Common webhook response type
type WebhookResponse = HttpResponse & {
  statusCode: number;
  contentType: string;
  body: string | Record<string, unknown>;
  headers?: Record<string, string>;
};

// Common webhook event types
interface WebhookEvent {
  type: string;
  data: Record<string, unknown>;
  timestamp: string;
}

// Webhook verification types
interface WebhookVerification {
  webhookSecret?: string;
  signature?: string;
  signatureHeader?: string;
}
```

## Implementation Patterns

### 1. Webhook with Initial Verification

Many APIs (like Asana) require a verification step when setting up webhooks:

```typescript
export const verifiedWebhook = trigger({
  display: {
    label: "Verified Webhook",
    description: "Handles initial webhook verification and ongoing events",
  },

  allowsBranching: true,
  staticBranchNames: ["Verification", "Event"],

  inputs: {
    connection: input({
      label: "API Connection",
      type: "connection",
      required: true,
    }),
  },

  perform: async (context, payload) => {
    const headers = util.types.lowerCaseHeaders(payload.headers);
    const webhookSecret = headers["x-hook-secret"];

    // Handle initial verification
    if (webhookSecret) {
      return {
        payload,
        response: {
          statusCode: 200,
          headers: {
            "X-Hook-Secret": webhookSecret,
          },
          contentType: "text/plain",
        },
        branch: "Verification",
        instanceState: { webhookSecret }, // Store for future validation
      };
    }

    // Validate ongoing requests
    validateHmac(payload, headers["x-hook-signature"], [
      context.instanceState.webhookSecret,
    ]);

    // Process the event
    return {
      payload: await processWebhookData(context, payload),
      branch: "Event",
    };
  },

  // Set up webhook on deploy
  onInstanceDeploy: async (context, inputs) => {
    const endpoint = context.webhookUrls[context.flow.name];
    await createWebhook({
      connection: inputs.connection,
      endpoint,
      filters: getEventFilters(inputs),
    });
  },

  // Clean up webhook on delete
  onInstanceDelete: async (context, inputs) => {
    const endpoint = context.webhookUrls[context.flow.name];
    await deleteWebhook({
      connection: inputs.connection,
      endpoint,
    });
  },

  synchronousResponseSupport: true,
  scheduleSupport: "invalid",
});
```

### 2. Event-Filtered Webhook

Pattern for handling specific event types with configurable filters:

```typescript
export const filteredWebhook = trigger({
  display: {
    label: "Filtered Webhook",
    description: "Process specific event types from webhooks",
  },

  inputs: {
    triggerOnCreate: input({
      label: "Trigger on Create",
      type: "boolean",
      required: true,
      default: true,
    }),
    triggerOnUpdate: input({
      label: "Trigger on Update",
      type: "boolean",
      required: true,
      default: true,
    }),
    triggerOnDelete: input({
      label: "Trigger on Delete",
      type: "boolean",
      required: true,
      default: false,
    }),
  },

  perform: async (context, payload, inputs) => {
    const event = payload.body as WebhookEvent;

    // Check if event type matches filters
    if (
      (event.type === "create" && inputs.triggerOnCreate) ||
      (event.type === "update" && inputs.triggerOnUpdate) ||
      (event.type === "delete" && inputs.triggerOnDelete)
    ) {
      return {
        payload: {
          ...payload,
          body: { data: event.data },
        },
      };
    }

    // Event filtered out
    return { payload };
  },
});
```

### 3. CSV Webhook (Data Transformation)

Example from Prismatic docs for handling non-JSON data:

```typescript
import papaparse from "papaparse";

export const csvWebhook = trigger({
  display: {
    label: "CSV Webhook",
    description: "Accept CSV data and return parsed JSON",
  },

  inputs: {
    hasHeader: input({
      label: "CSV Has Header",
      type: "boolean",
      default: "false",
    }),
  },

  perform: async (context, payload, { hasHeader }) => {
    // Echo back confirmation header if required
    const response: HttpResponse = {
      statusCode: 200,
      contentType: "text/plain; charset=utf-8",
      body: payload.headers["x-confirmation-code"],
    };

    // Parse CSV data
    const parseResult = papaparse.parse(
      util.types.toString(payload.rawBody.data),
      {
        header: util.types.toBool(hasHeader),
      },
    );

    // Return both response and parsed data
    return {
      payload: {
        ...payload,
        body: { data: parseResult.data },
      },
      response,
    };
  },

  synchronousResponseSupport: true,
  scheduleSupport: "invalid",
});
```

### 4. Heartbeat/Health Check Handler

Pattern for handling health check or heartbeat events:

```typescript
export const heartbeatWebhook = trigger({
  display: {
    label: "Heartbeat Webhook",
    description: "Handle both heartbeat and regular events",
  },

  perform: async (context, payload) => {
    // Check for heartbeat event
    if (isHeartbeatEvent(payload.body)) {
      context.logger.debug("Heartbeat received");
      return {
        payload,
        response: {
          statusCode: 200,
          contentType: "application/json",
          body: { status: "healthy" },
        },
      };
    }

    // Process regular event
    return {
      payload: await processWebhookData(context, payload),
    };
  },
});
```

## Security Best Practices

1. **Webhook Verification**

   - Always verify webhook signatures when available
   - Use constant-time comparison for signatures
   - Implement proper error handling for verification failures

2. **Request Validation**

   - Validate payload structure and content
   - Check required headers
   - Verify content types
   - Validate data integrity

3. **Response Security**

   - Return minimal information in responses
   - Use appropriate status codes
   - Don't expose internal errors
   - Implement rate limiting

4. **Error Handling**
   - Handle verification failures gracefully
   - Log security-related errors
   - Implement proper retry logic
   - Clean up resources on failures

## Helper Functions

```typescript
// Signature validation
function validateHmac(
  payload: TriggerPayload,
  signature: string,
  secrets: string[],
): void {
  if (!signature) {
    throw new Error("Missing signature");
  }

  const isValid = secrets.some((secret) => {
    const computed = crypto
      .createHmac("sha256", secret)
      .update(payload.rawBody.data)
      .digest("hex");
    return crypto.timingSafeEqual(
      Buffer.from(signature),
      Buffer.from(computed),
    );
  });

  if (!isValid) {
    throw new Error("Invalid signature");
  }
}

// Webhook resource management
async function createWebhook(params: {
  connection: Connection;
  endpoint: string;
  filters: Record<string, boolean>;
}): Promise<void> {
  const client = createApiClient(params.connection);
  await client.post("/webhooks", {
    url: params.endpoint,
    events: Object.entries(params.filters)
      .filter(([_, enabled]) => enabled)
      .map(([event]) => event),
  });
}

async function deleteWebhook(params: {
  connection: Connection;
  endpoint: string;
}): Promise<void> {
  const client = createApiClient(params.connection);
  await client.delete("/webhooks", {
    params: { url: params.endpoint },
  });
}
```

## External References

- [Prismatic Webhook Documentation](https://prismatic.io/docs/custom-connectors/triggers)
- [Webhook Security Best Practices](https://webhooks.fyi/security)
- [Asana Webhooks Documentation](https://developers.asana.com/docs/webhooks)
