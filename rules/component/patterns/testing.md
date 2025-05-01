# Component Testing Implementation Rules for AI Agents

## Overview

This document provides rules and patterns for AI agents to follow when implementing tests for Prismatic components. These guidelines ensure consistent, reliable, and maintainable test suites across components.

## Implementation Rules

### Rule 1: Test File Structure

- Create test files with the `.test.ts` extension
- Place test files alongside the code they test
- Follow the naming pattern: if code is in `actions.ts`, tests should be in `actions.test.ts`
- Always separate action and trigger tests into different files

### Rule 2: Required Test Setup

```typescript
// Required imports for every test file. Do not use these outside of the test context.
import {
  createConnection,
  createHarness,
} from "@prismatic-io/spectral/dist/testing";
import component from "."; // Import the component being tested

// Required harness setup
const harness = createHarness(component);
```

### Rule 3: Connection Configuration

When creating test connections, use environment variables instead of hard-coded variables:

```typescript
// CORRECT: Use environment variables for sensitive data.
const testConnection = createConnection(connectionDefinition, {
  baseUrl: process.env.BASE_URL,
  apiKey: process.env.API_KEY,
});

// INCORRECT: Never hardcode credentials
const badConnection = createConnection(connectionDefinition, {
  baseUrl: "https://api.example.com",
  apiKey: "1234567890", // Don't do this
});
```

### Rule 4: Action Test Implementation

For each action in the component:

1. Test happy path with valid inputs
2. Test error scenarios with invalid inputs
3. Test edge cases for input validation

```typescript
// Required structure for action tests
describe("Action: {actionName}", () => {
  test("successfully processes valid inputs", async () => {
    const result = await harness.action("actionName", {
      connection: testConnection,
      requiredInput: "value",
    });
    expect(result.data).toMatchObject(expectedShape);
  });

  test("handles invalid inputs appropriately", async () => {
    await expect(async () => {
      await harness.action("actionName", {
        connection: testConnection,
        requiredInput: invalidValue,
      });
    }).rejects.toThrow();
  });
});
```

### Rule 5: Trigger Test Implementation

For each trigger in the component:

1. Test payload processing
2. Test authentication/validation
3. Test error handling

```typescript
// Required structure for trigger tests
describe("Trigger: {triggerName}", () => {
  test("processes valid webhook payload", async () => {
    const payload = defaultTriggerPayload();
    payload.rawBody.data = JSON.stringify(validData);

    const result = await harness.trigger("triggerName", payload, {
      connection: testConnection,
    });
    expect(result.payload).toBeDefined();
  });

  test("validates webhook authentication", async () => {
    const payload = defaultTriggerPayload();
    payload.headers["x-signature"] = invalidSignature;

    await expect(async () => {
      await harness.trigger("triggerName", payload, {
        connection: testConnection,
      });
    }).rejects.toThrow();
  });
});
```

### Rule 6: Test Data Management

When creating test data:

1. Define expected data as constants at the top of the test file
2. Use type definitions for expected shapes
3. Never include sensitive information in test data

```typescript
// CORRECT: Define expected data as typed constants
const expectedItems: ItemType[] = [{ id: 1, name: "Test Item", quantity: 10 }];

// INCORRECT: Inline data without types
const badData = [{ id: 1, name: "Test" }]; // Don't do this
```

### Rule 7: Error Testing Requirements

For each error scenario:

1. Test specific error messages
2. Test error types
3. Test error handling behavior

```typescript
// Required error test pattern
test("handles specific error condition", async () => {
  await expect(async () => {
    await harness.action("actionName", invalidInputs);
  }).rejects.toThrow("Expected error message");
});
```

## Validation Checklist

AI agents must verify these points before completing test implementation:

1. Test Coverage:

   - [ ] All actions have basic success case tests
   - [ ] All actions have error case tests
   - [ ] All triggers have payload processing tests
   - [ ] All triggers have authentication tests

2. Code Quality:

   - [ ] No hardcoded credentials
   - [ ] All test data is properly typed
   - [ ] Clear test descriptions
   - [ ] Proper use of Jest matchers

3. Security:
   - [ ] Sensitive data uses environment variables
   - [ ] No exposure of production credentials
   - [ ] Proper handling of authentication tests

## Testing Tools Reference

### Required Jest Matchers

- `toMatchObject`: For partial result validation
- `toStrictEqual`: For exact result validation
- `toThrow`: For error validation
- `rejects`: For async error validation

### Required Prismatic Testing Utilities

- `createHarness`: Component test environment setup
- `createConnection`: Test connection configuration
- `defaultTriggerPayload`: Webhook payload creation
- `invoke`: Direct action testing

## Implementation Notes

1. Always implement tests in TypeScript
2. Always use async/await for asynchronous tests
3. Always group related tests using describe blocks
4. Never skip error scenario testing
5. Never use production APIs in tests without explicit approval

## External References

1. [Prismatic Unit Testing Documentation](https://prismatic.io/docs/custom-connectors/unit-testing/)
2. [Jest Testing Framework](https://jestjs.io/docs/getting-started)
3. [TypeScript Testing Best Practices](https://github.com/goldbergyoni/javascript-testing-best-practices)
