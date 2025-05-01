# Performance Optimization Patterns

## Overview

This document outlines performance optimization patterns for Prismatic components, focusing on efficient resource usage and pagination handling.

## Pagination Handling

- **Default to Single-Page Fetching:** Unless the maximum number of records is known to be very small (e.g., < 200), actions **must** fetch only a single page of results per execution. Avoid internal loops (`do...while`) that fetch all pages.
- **Pagination Inputs:** The action **must** include inputs for the specific pagination mechanism used by the API (e.g., `cursor`, `pageNumber`, `pageSize`, `offset`). Define these using `input()`. **Never** guess at the pagination inputs. Always confirm with documentation first. If documentation is unavailable, ask the user for input.
- **Pagination Outputs:** The `perform` function's return value (`{ data: ... }`) **must** include necessary information for the _integration flow_ to request the _next_ page. Return the full API response for use in next iteration of the _integration flow_.

### 1. Cursor-Based Pagination

```typescript
/**
 * Implements cursor-based pagination for large dataset retrieval
 */
const fetchPaginatedData = async (
  connection: Connection,
  options: {
    cursor?: string;
    limit: number;
    filters?: Record<string, unknown>;
  },
): Promise<PaginatedResponse> => {
  const response = await apiClient.get("/resources", {
    params: {
      cursor: options.cursor,
      limit: options.limit,
      ...options.filters,
    },
  });

  return {
    data: response.data.items,
    nextCursor: response.data.nextCursor,
    hasMore: Boolean(response.data.nextCursor),
  };
};

/**
 * Example action implementation using pagination
 */
const listResourcesAction: Action = {
  display: {
    label: "List Resources",
    description: "Retrieves a paginated list of resources",
  },
  inputs: {
    connection: input.connection("connection"),
    pageSize: input.number("pageSize", {
      label: "Page Size",
      default: 100,
      comments: "Number of items per page",
    }),
    maxItems: input.number("maxItems", {
      label: "Max Items",
      comments: "Maximum number of items to retrieve (0 for all)",
      default: 0,
    }),
  },
  async perform(context, { connection, pageSize, maxItems }) {
    const results = [];
    let cursor: string | undefined;
    let totalItems = 0;

    do {
      const page = await fetchPaginatedData(connection, {
        cursor,
        limit: pageSize,
      });

      results.push(...page.data);
      cursor = page.nextCursor;
      totalItems += page.data.length;

      // Check if we've reached the maximum items
      if (maxItems > 0 && totalItems >= maxItems) {
        results.splice(maxItems);
        break;
      }
    } while (cursor);

    return { data: results };
  },
};
```

### 2. Offset-Based Pagination

```typescript
/**
 * Implements offset-based pagination with automatic retries
 */
const fetchWithOffset = async (
  connection: Connection,
  options: {
    offset: number;
    limit: number;
    retries?: number;
  },
): Promise<OffsetPaginatedResponse> => {
  const maxRetries = options.retries ?? 3;
  let attempt = 0;

  while (attempt < maxRetries) {
    try {
      const response = await apiClient.get("/resources", {
        params: {
          offset: options.offset,
          limit: options.limit,
        },
      });

      return {
        data: response.data.items,
        total: response.data.total,
        hasMore:
          options.offset + response.data.items.length < response.data.total,
      };
    } catch (error) {
      if (attempt === maxRetries - 1) throw error;
      attempt++;
      await new Promise((resolve) => setTimeout(resolve, 1000 * attempt));
    }
  }

  throw new Error("Failed to fetch data after multiple retries");
};
```
