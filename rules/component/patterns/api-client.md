# API Client Implementation Patterns

## Overview

This document outlines standardized patterns for implementing API clients in Prismatic components, ensuring consistent, type-safe, and maintainable API integrations.

## HTTP Client Implementation

### 1. Basic HTTP Client Setup

```typescript
import { Connection, ConnectionError } from "@prismatic-io/spectral";
import {
  createClient,
  HttpClient,
} from "@prismatic-io/spectral/dist/clients/http";

export const createApiClient = async (
  connection: Connection,
): Promise<HttpClient> => {
  const apiClient = createClient({
    baseUrl: "https://api.example.com/v1",
    headers: {
      authorization: `Bearer ${connection?.token?.access_token || connection?.fields?.apiKey}`,
    },
  });
  return apiClient;
};
```

## Authentication Patterns

### 1. API Key Authentication

```typescript
export const createApiKeyClient = (connection: Connection): HttpClient => {
  if (!connection?.fields?.apiKey) {
    throw new ConnectionError(connection, "API Key is required");
  }

  return createClient({
    baseUrl: connection.fields.baseUrl || "https://api.default.com/v1",
    headers: {
      "x-api-key": connection.fields.apiKey,
    },
  });
};
```

### 2. OAuth 2.0 Authentication

```typescript
export const createOAuthClient = (connection: Connection): HttpClient => {
  if (!connection?.token?.access_token) {
    throw new ConnectionError(connection, "Access token is required");
  }

  return createClient({
    baseUrl: connection.fields.baseUrl || "https://api.default.com/v1",
    headers: {
      Authorization: `Bearer ${connection.token.access_token}`,
    },
  });
};
```

### 3. Basic Authentication

```typescript
export const createBasicAuthClient = (connection: Connection): HttpClient => {
  if (!connection?.fields?.username || !connection?.fields?.password) {
    throw new ConnectionError(connection, "Username and password are required");
  }

  const basicAuthToken = Buffer.from(
    `${connection.fields.username}:${connection.fields.password}`,
  ).toString("base64");

  return createClient({
    baseUrl: connection.fields.baseUrl || "https://api.default.com/v1",
    headers: {
      Authorization: `Basic ${basicAuthToken}`,
    },
  });
};
```

### Example Client Usage in an Action

```typescript
import { action, input, util } from "@prismatic-io/spectral";

export const myAction = action({
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
  },

  perform: async (context, { connection, data }) => {
    try {
      //Fetch client from factory method
      const client = getClient(connection);
      const result = await client.post(data);
      return { data: result };
    } catch (error) {
      handleActionError(error);
    }
  },
});
```

## Best Practices

### 1. Client Organization

- Create separate client files for different API domains
- Implement domain-specific client factories
- Use dependency injection for client instances

### 2. Logging and Debugging

- Implement debug logging for API requests
- Sanitize sensitive information in logs

## Common Anti-Patterns to Avoid

1. Creating a new client for each request
2. Hard-coding authentication credentials
3. Missing proper error handling
4. Using generic error types
5. Hard-coding base URLs
6. Missing content-type headers

## External References

1. [Prismatic Spectral HTTP Client Documentation](https://prismatic.io/docs/spectral/clients/http)
2. [REST API Best Practices](https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design)
3. [API Error Handling Patterns](https://blog.restcase.com/rest-api-error-codes-101)
