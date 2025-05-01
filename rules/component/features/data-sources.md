# Data Source Implementation Patterns

## Overview

This document defines standardized patterns for implementing data sources in Prismatic components, enabling dynamic configuration options and form fields.

## Data Source Types

### 1. Picklist Data Source

```typescript
import { dataSource, input } from "@prismatic-io/spectral";

export const resourcePicklist = dataSource({
  display: {
    label: "Select Resource",
    description: "Choose from available resources",
  },

  inputs: {
    connection: input({
      label: "API Connection",
      type: "connection",
      required: true,
    }),
    filter: input({
      label: "Filter Term",
      type: "string",
      required: false,
      comments: "Optional search term to filter results",
    }),
  },

  perform: async (context, { connection, filter }) => {
    const client = createApiClient(connection);

    try {
      const response = await client.get("/resources", {
        params: { search: filter },
      });

      return {
        result: response.data.map((item) => ({
          label: item.name,
          value: item.id,
        })),
      };
    } catch (error) {
      handleDataSourceError(error);
    }
  },

  // Example of expected results
  examplePayload: {
    result: [
      { label: "Resource 1", value: "1" },
      { label: "Resource 2", value: "2" },
    ],
  },
});
```

### 2. Dynamic Form Data Source

```typescript
import { dataSource, input } from "@prismatic-io/spectral";

export const dynamicFormFields = dataSource({
  display: {
    label: "Form Configuration",
    description: "Get dynamic form configuration",
  },

  inputs: {
    connection: input({
      label: "API Connection",
      type: "connection",
      required: true,
    }),
    formType: input({
      label: "Form Type",
      type: "string",
      required: true,
      model: [
        { label: "Basic", value: "basic" },
        { label: "Advanced", value: "advanced" },
      ],
    }),
  },

  perform: async (context, { connection, formType }) => {
    const client = createApiClient(connection);

    try {
      const response = await client.get(`/forms/${formType}/schema`);

      return {
        result: {
          jsonSchema: response.data.schema,
          uiSchema: response.data.uiSchema,
        },
      };
    } catch (error) {
      handleDataSourceError(error);
    }
  },

  // Example of form configuration
  examplePayload: {
    result: {
      jsonSchema: {
        type: "object",
        properties: {
          name: { type: "string", title: "Name" },
          age: { type: "number", title: "Age" },
        },
        required: ["name"],
      },
      uiSchema: {
        name: { "ui:autofocus": true },
        age: { "ui:widget": "updown" },
      },
    },
  },
});
```

## Implementation Patterns

### 1. Caching Pattern

```typescript
interface CacheConfig {
  ttl: number;
  key: string;
}

async function withCache<T>(
  context: Context,
  config: CacheConfig,
  operation: () => Promise<T>,
): Promise<T> {
  const cacheKey = `datasource:${config.key}`;
  const cached = await context.cache.get(cacheKey);

  if (cached) {
    return JSON.parse(cached);
  }

  const result = await operation();
  await context.cache.set(cacheKey, JSON.stringify(result), config.ttl);

  return result;
}

// Usage in data source
perform: async (context, inputs) => {
  return withCache(
    context,
    {
      ttl: 300, // 5 minutes
      key: `resources:${inputs.filter || "all"}`,
    },
    async () => {
      const client = createApiClient(inputs.connection);
      const response = await client.get("/resources");
      return {
        result: response.data.map(mapToPicklistItem),
      };
    },
  );
};
```

### 2. Pagination Pattern

```typescript
async function fetchAllPages(
  client: ApiClient,
  path: string,
  params: Record<string, unknown>
): Promise<Array<unknown>> {
  let allResults = [];
  let page = 1;

  while (true) {
    const response = await client.get(path, {
      params: { ...params, page, limit: 100 },
    });

    allResults = allResults.concat(response.data.items);

    if (!response.data.hasNextPage) {
      break;
    }

    page++;
  }

  return allResults;
}

// Usage in data source
const myDataSource = dataSource({
  // ...
  perform: async (context, inputs) => {
  const client = createApiClient(inputs.connection);
  const allItems = await fetchAllPages(client, "/resources", {
    filter: inputs.filter,
  });

  return {
    result: allItems.map(mapToPicklistItem),
  };
});
```

## Best Practices

### 1. User Experience

- Provide clear labels and descriptions
- Include helpful examples
- Implement proper filtering

### 2. Error Handling

- Provide clear error messages
- Handle network failures
- Validate response data
- Log appropriate details

## References

- [Prismatic Data Sources Documentation](https://prismatic.io/docs)
- [JSON Schema](https://json-schema.org)
- [React JSONSchema Form](https://rjsf-team.github.io/react-jsonschema-form)
