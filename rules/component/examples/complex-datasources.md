# Complex datasource and field mapping examples

## Type Definitions

```typescript
import { dataSource, input, util, Element } from "@prismatic-io/spectral";

// Common mapping types
interface FieldMappingElement extends Element {
  label: string;
  key: string;
}

interface ObjectSelection {
  object: { key: string; label: string };
  fields: { key: string; label: string }[];
}
```

## Implementation Patterns

### 1. Basic Picklist Data Source

The simplest field mapping pattern is a picklist that provides a list of options to select from:

```typescript
import { dataSource, input, Element } from "@prismatic-io/spectral";

// Define the data source to fetch a list of items from an API
export const basicPicklist = dataSource({
  display: {
    label: "Basic Picklist",
    description: "Fetch a list of options for selection",
  },
  inputs: {
    connection: input({
      label: "Connection",
      type: "connection",
      required: true,
    }),
  },
  dataSourceType: "picklist",
  perform: async (context, { connection }) => {
    // Create API client using the connection
    const client = createApiClient(connection);

    // Fetch data from the API
    const response = await client.get("/items");

    // Map API response to elements for the picklist
    const elements = response.data.items.map((item) => ({
      label: item.name,
      key: item.id,
    }));

    return { result: elements };
  },
  examplePayload: {
    result: [
      { label: "Item 1", key: "item_1" },
      { label: "Item 2", key: "item_2" },
    ],
  },
});
```

### 2. Object Selection with Metadata

This pattern allows users to select objects with additional metadata fields:

```typescript
import { dataSource, input } from "@prismatic-io/spectral";

export const objectSelectionDataSource = dataSource({
  display: {
    label: "Object Selection",
    description: "Select objects with metadata fields",
  },
  inputs: {
    connection: input({
      label: "Connection",
      type: "connection",
      required: true,
    }),
  },
  dataSourceType: "objectSelection",
  perform: async (context, { connection }) => {
    // Create API client using the connection
    const client = createApiClient(connection);

    // Fetch objects from the API
    const response = await client.get("/products");

    // Map API response to object selection format
    const objects = response.data.products.map((product) => ({
      object: { key: product.id, label: product.name },
      fields: [
        { key: product.sku, label: "SKU" },
        { key: product.price.toString(), label: "Price" },
        { key: product.quantity.toString(), label: "Quantity" },
      ],
    }));

    return { result: objects };
  },
  examplePayload: {
    result: [
      {
        object: { key: "prod_123", label: "Product A" },
        fields: [
          { key: "SKU123", label: "SKU" },
          { key: "99.99", label: "Price" },
          { key: "10", label: "Quantity" },
        ],
      },
    ],
  },
});
```

### 3. JSON Forms for Complex Mapping

For complex field mapping with nested objects and multiple selections, JSON Forms provide a flexible solution:

```typescript
import { dataSource, util } from "@prismatic-io/spectral";

export const jsonFormDataSource = dataSource({
  display: {
    label: "JSON Form Data Source",
    description: "Complex mapping with JSON Forms",
  },
  inputs: {
    connection: input({
      label: "Connection",
      type: "connection",
      required: true,
    }),
  },
  dataSourceType: "jsonForm",
  perform: async (context, { connection }) => {
    // Use client to fetch available fields from source system
    const sourceClient = createApiClient(connection);
    const { data: sourceFields } = await sourceClient.get("/fields");

    // Use client to fetch available fields from target system
    const targetClient = createApiClient(connection);
    const { data: targetFields } = await targetClient.get("/fields");

    // Create schema for JSON Form
    const schema = {
      type: "object",
      properties: {
        fieldMappings: {
          type: "array",
          items: {
            type: "object",
            properties: {
              source: {
                type: "string",
                oneOf: sourceFields.map((field) => ({
                  const: field.id,
                  title: field.name,
                })),
              },
              target: {
                type: "string",
                oneOf: targetFields.map((field) => ({
                  const: field.id,
                  title: field.name,
                })),
              },
              transform: {
                type: "string",
                oneOf: [
                  { const: "none", title: "No Transformation" },
                  { const: "uppercase", title: "Convert to Uppercase" },
                  { const: "lowercase", title: "Convert to Lowercase" },
                  { const: "number", title: "Convert to Number" },
                ],
              },
            },
          },
        },
      },
    };

    // Create UI schema for JSON Form
    const uiSchema = {
      type: "VerticalLayout",
      elements: [
        {
          type: "Control",
          scope: "#/properties/fieldMappings",
          label: "Field Mappings",
        },
      ],
    };

    // Default values for the form
    const defaultValues = {
      fieldMappings: [
        {
          source: sourceFields[0]?.id,
          target: targetFields[0]?.id,
          transform: "none",
        },
      ],
    };

    return {
      result: { schema, uiSchema, data: defaultValues },
    };
  },
});
```

### 4. Data Validation with Nested Mapping

This pattern implements field mapping with validation for a data integration scenario:

```typescript
import { dataSource, input, util } from "@prismatic-io/spectral";

export const validatedMappingDataSource = dataSource({
  display: {
    label: "Validated Field Mapping",
    description: "Map fields with validation",
  },
  inputs: {
    connection: input({
      label: "Connection",
      type: "connection",
      required: true,
    }),
  },
  dataSourceType: "jsonForm",
  perform: async (context, { connection }) => {
    // Fetch available fields from API
    const client = createApiClient(connection);
    const { data: fields } = await client.get("/schema");

    // Define required fields for the target system
    const requiredFields = ["firstName", "lastName", "email"];

    // Create schema for JSON Form with validation
    const schema = {
      type: "object",
      properties: {
        mappings: {
          type: "object",
          properties: {},
          required: requiredFields,
        },
      },
    };

    // Add properties for each required field
    fields.forEach((field) => {
      schema.properties.mappings.properties[field.id] = {
        type: "string",
        title: field.name,
        description: field.description,
      };

      // Add validation patterns if available
      if (field.validation) {
        schema.properties.mappings.properties[field.id].pattern =
          field.validation;
      }
    });

    // Create UI schema
    const uiSchema = {
      type: "VerticalLayout",
      elements: [
        {
          type: "Control",
          scope: "#/properties/mappings",
          label: "Field Mappings",
        },
      ],
    };

    return {
      result: { schema, uiSchema },
    };
  },
});
```

## Best Practices

### Data Source Type Selection

- Use `picklist` for simple selection from a list of options
- Use `objectSelection` when you need to include metadata with selectable objects
- Use `jsonForm` for complex, nested, or conditional field mapping scenarios

### Field Mapping Design

1. **Descriptive Labels**: Always provide clear labels and descriptions for fields
2. **Validation**: Include validation rules where applicable
3. **Defaults**: Provide sensible defaults when possible
4. **Grouping**: Group related fields together in a logical structure
5. **Nested Fields**: Use nested structures for complex data models

### Error Handling

1. Always handle API errors gracefully with informative error messages
2. Validate inputs before making API calls
3. Provide fallback options when external services are unavailable

### Performance Considerations

1. Cache results when appropriate to reduce API calls
2. Implement pagination for large datasets
3. Use appropriate throttling and rate limiting

## Example Implementation

Here's a complete implementation example of a field mapping data source:

```typescript
import { dataSource, input, util } from "@prismatic-io/spectral";
import { createClient } from "@prismatic-io/spectral/dist/clients/http";

// Helper function to create an API client
const createApiClient = (connection) => {
  return createClient({
    baseURL: connection.fields.baseUrl,
    headers: {
      Authorization: `Bearer ${connection.token.access_token}`,
      "Content-Type": "application/json",
    },
  });
};

export const customerFieldMapping = dataSource({
  display: {
    label: "Customer Field Mapping",
    description: "Map customer fields from your system to the target system",
  },
  inputs: {
    connection: input({
      label: "Connection",
      type: "connection",
      required: true,
    }),
  },
  dataSourceType: "jsonForm",
  perform: async (context, { connection }) => {
    try {
      const client = createApiClient(connection);

      // Fetch available customer fields
      const { data: customerFields } = await client.get("/customer-fields");

      // Define the schema with required and optional fields
      const schema = {
        type: "object",
        properties: {
          required: {
            type: "object",
            properties: {
              id: {
                type: "string",
                oneOf: customerFields
                  .filter((field) => field.type === "id")
                  .map((field) => ({
                    const: field.id,
                    title: field.name,
                  })),
              },
              name: {
                type: "string",
                oneOf: customerFields
                  .filter((field) => field.type === "string")
                  .map((field) => ({
                    const: field.id,
                    title: field.name,
                  })),
              },
              email: {
                type: "string",
                oneOf: customerFields
                  .filter((field) => field.type === "email")
                  .map((field) => ({
                    const: field.id,
                    title: field.name,
                  })),
              },
            },
            required: ["id", "name", "email"],
          },
          optional: {
            type: "object",
            properties: {
              phone: {
                type: "string",
                oneOf: customerFields
                  .filter((field) => field.type === "phone")
                  .map((field) => ({
                    const: field.id,
                    title: field.name,
                  })),
              },
              address: {
                type: "string",
                oneOf: customerFields
                  .filter((field) => field.type === "address")
                  .map((field) => ({
                    const: field.id,
                    title: field.name,
                  })),
              },
            },
          },
        },
      };

      // Define the UI layout
      const uiSchema = {
        type: "VerticalLayout",
        elements: [
          {
            type: "Group",
            label: "Required Fields",
            elements: [
              {
                type: "Control",
                scope: "#/properties/required/properties/id",
                label: "Customer ID Field",
              },
              {
                type: "Control",
                scope: "#/properties/required/properties/name",
                label: "Customer Name Field",
              },
              {
                type: "Control",
                scope: "#/properties/required/properties/email",
                label: "Customer Email Field",
              },
            ],
          },
          {
            type: "Group",
            label: "Optional Fields",
            elements: [
              {
                type: "Control",
                scope: "#/properties/optional/properties/phone",
                label: "Customer Phone Field",
              },
              {
                type: "Control",
                scope: "#/properties/optional/properties/address",
                label: "Customer Address Field",
              },
            ],
          },
        ],
      };

      // Define default values if fields are available
      const defaults = {
        required: {
          id: customerFields.find((f) => f.type === "id")?.id,
          name: customerFields.find((f) => f.type === "string")?.id,
          email: customerFields.find((f) => f.type === "email")?.id,
        },
        optional: {},
      };

      return {
        result: {
          schema,
          uiSchema,
          data: defaults,
        },
      };
    } catch (error) {
      // Handle errors appropriately
      context.logger.error("Error fetching customer fields", error);
      throw new Error(`Failed to load field mapping options: ${error.message}`);
    }
  },
});
```
