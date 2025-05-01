# Connection Implementation Patterns

## Connection Types

```typescript
import { connection } from "@prismatic-io/spectral";
connection({
  key: "someKey",
  display: {
    label: "Human readable label",
    description: "Human readable description",
  },
  inputs: {
    //...connection-specific inputs
  },
});
```

### 1. API Key Connection

```typescript
import { connection, input, util } from "@prismatic-io/spectral";

export const apiKeyConnection = connection({
  key: "apiKey",
  display: {
    label: "API Key Authentication",
    description: "Connect using an API key",
  },
  inputs: {
    endpoint: input({
      label: "API Endpoint",
      type: "string",
      required: true,
      comments: "The base URL for API calls",
      example: "https://api.service.com/v1",
    }),
    apiKey: input({
      label: "API Key",
      type: "password",
      required: true,
      comments: "Your API key from the service dashboard",
    }),
    // Optional configuration
    timeout: input({
      label: "Request Timeout",
      type: "string",
      required: false,
      default: "30000",
      comments: "Request timeout in milliseconds",
      clean: util.types.toInt,
    }),
  },
});
```

### 2. OAuth2 Connection

```typescript
export const oauth2Connection = connection({
  key: "oauth2",
  display: {
    label: "OAuth 2.0 Authentication",
    description: "Connect using OAuth 2.0",
  },
  inputs: {
    endpoint: input({
      label: "API Endpoint",
      type: "string",
      required: true,
    }),
    clientId: input({
      label: "Client ID",
      type: "string",
      required: true,
    }),
    clientSecret: input({
      label: "Client Secret",
      type: "password",
      required: true,
    }),
    scopes: input({
      label: "OAuth Scopes",
      type: "string",
      required: true,
      clean: (value) => value.split(",").map((s) => s.trim()),
      example: "read:users, write:data",
    }),
    authorizeUrl: {
      label: "Authorize URL",
      type: "string",
      default: "https://service.com/oauth/authorize",
      required: true,
      shown: false,
      comments: "The OAuth 2.0 Authorization URL for the API",
    },
    tokenUrl: {
      label: "Token URL",
      type: "string",
      default: "https://service.com/oauth/token",
      required: true,
      shown: false,
      comments: "The OAuth 2.0 Token URL for the API",
    },
  },
});
```

### 3. Basic Auth Connection

```typescript
export const basicAuthConnection = connection({
  key: "basicAuth",
  display: {
    label: "Basic Authentication",
    description: "Connect using username and password",
  },
  inputs: {
    endpoint: input({
      label: "API Endpoint",
      type: "string",
      required: true,
    }),
    username: input({
      label: "Username",
      type: "string",
      required: true,
    }),
    password: input({
      label: "Password",
      type: "password",
      required: true,
    }),
  },
});
```

## Connection Usage: Shared Client Factory Pattern

```typescript
import { Connection, ConnectionError } from "@prismatic-io/spectral";
import {
  createClient,
  HttpClient,
} from "@prismatic-io/spectral/dist/clients/http";

export const createApiClient = async (
  connection: Connection,
): Promise<HttpClient> => {
  const baseUrl = connection.fields.baseUrl;
  const apiKey = connection.token?.access_token || connection.fields.apiKey;

  if (!baseUrl || !apiKey) {
    throw new ConnectionError(
      connection,
      "Required connection fields are missing.",
    );
  }

  return createClient({
    baseUrl,
    headers: {
      Authorization: `Bearer ${apiKey}`,
      "Content-Type": "application/json",
    },
    responseType: "json",
  });
};
```

## Best Practices

### 1. Security

- Never log sensitive connection data
- Use password type for sensitive fields
- Handle connection errors gracefully

### 2. Configuration

- Provide clear field descriptions
- Include example values
- Use appropriate input types
- Always include the API endpoint as an input on the connection

## References

- [Prismatic Connections Documentation](https://prismatic.io/docs/custom-connectors/connections)
- [OAuth 2.0 Specification](https://oauth.net/2)
- [HTTP Client Best Practices](https://axios-http.com/docs/best_practices)
