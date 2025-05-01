# Prismatic Spectral Coding Assistant Best Practices

- [Getting Started](#getting-started)
- [Effective Prompting](#effective-prompting)
  - [Expect Conversation](#expect-conversation)
  - [Respect the Context Window](#respect-the-context-window)
  - [What, not How](#what-not-how)
- [Troubleshooting](#troubleshooting)
  - [The AI didn't do what I meant or expected](#the-ai-didnt-do-what-i-meant-or-expected)
  - [Cursor keeps doing the same thing in the chat](#cursor-keeps-doing-the-same-thing-in-the-chat)
  - [Cursor is stuck "Generating..." for extended periods of time](#cursor-is-stuck-generating-for-extended-periods-of-time)
- [What is Prismatic?](#what-is-prismatic)
- [Who uses Prismatic?](#who-uses-prismatic)
- [What kind of integrations can you build using Prismatic?](#what-kind-of-integrations-can-you-build-using-prismatic)
- [License](#license)

This repo contains rules and guidelines to enable coding assistants to build out well-structured custom component using Prismatic's [Spectral library](https://github.com/prismatic-io/spectral/). They guide the coding assistant you already use to follow component development best practices and are useful for both initial build out and later maintenance.

## Getting Started

To start a new component project, follow these steps:

1. Initialize the component: `prism components:init <name>`
   > [!NOTE]
   > Why use `prism` instead of letting the AI generate all files?
   >
   > This is a very valid question! It turns out that using AI to scaffold out specific files that _must be_ a specific form is a prime source of frustration. Model knowledge cutoffs routinely causes outdated versions of libraries to be used and extraordinary measures have to be taken in prompting to encourage using latest versions. Instead, we use our existing scaffolding functionality as a seed to build upon with these rules and prompting.
2. Install dependencies with: `npm install`
3. Remove the default source directory: `rm -rf <name>/src/`
4. Copy the ruleset into your new component: `cp -R rules/component <name>/rules`
5. Copy the `.cursor` config: `cp -R rules/component/.cursor <name>/.cursor`
6. Given you have a URL to documentation or an API specification, you can then prompt:

   > I want to create a component that wraps this API [paste api docs url] @start-new-component.md

   This will generate parts of the component and a plan of action in `TASK.md`. This file can then be human or AI edited to add or remove actions, data sources, etc for the component.

7. Once `TASK.md` has been reviewed/modified, you can then use the `next-task.md` prompt:
   > @next-task.md

## Effective Prompting

While prompting is more art than science right now, there are a few general guidelines that can provide better results.

### Expect Conversation

Prompt crafting tends to be easier and feel more natural in a conversational style. Specifically, this is about taking "turns" with the AI and has advantages such as smaller prompts that don't require such careful crafting. You're able to scope in the conversation over time and focus on specific human-curated tasks (in addition to those expressed through `TASK.md`). You also gain finer control over adding items to context, such as specific files or enabling Web searches (`@Web`).

### Respect the Context Window

There's only so much context a single conversation can hold. Try to use new chats for distinct tasks and consider breaking down the `TASK.md` file further (by hand or with the AI). Note that this _does not_ mean you need to have the shortest possible prompts but to be mindful of the length of the conversation and the amount of content pulled into context (particularly Web searches, folders, or large files).

The `TASK.md` pattern of breaking down tasks helps the AI by providing "save points" that allows clearing out the context window of the analysis phase of work. Subsequent prompting can be as simple as "Implement the next task".

### What, not How

You don't need to be prescriptive about _how_ something is accomplished. Focus on the _what_ and include relevant context whether that be links to documentation, pulling in prompts or other rule files, or enabling web search. For example, "Add a data source for the [X] endpoint of [api docs url]" is quite possibly sufficient prompting with this repo's rules in place to achieve a good starting point.

## Troubleshooting

### The AI didn't do what I meant or expected

Even if you get surprising results from AI-driven updates, you have many options for getting unstuck:

- Send another prompt in the existing conversation indicating the error and suggest a resolution. "Only the X, Y, and Z inputs are needed on the <name> action. Remove the rest."
- If the changes are completely off base, you can use the "Restore checkpoint" button on your prior prompt to "revert" the last round of suggested edits. You can then resubmit the prompt or edit it before resubmission.
- You can keep the conversational state (context window) the same but discard the changes using the "Reject All" button immediately above the prompt field.

### Cursor keeps doing the same thing in the chat

This most commonly occurs because the context window has been filled up. The context window keeps track of all tokens used in the entire chat conversation, including output tokens. This "death loop" likely indicates that the output tokens returned by the LLM were truncated and not included, causing effectively the same input to be supplied repeatedly. The only resolution is to do a net-new chat and re-prompt. If you're using the `TASK.md` pattern you are also able to use the `@next-task.md` prompt to continue through the unfinished tasks.

### Cursor is stuck "Generating..." for extended periods of time

We generally recommend using Cursor's Auto mode instead of selecting models but this does hide which LLM service it is attempting to use to field your particular chat session. Unfortunately these APIs aren't always performant or available and can have issues. You can either wait for the requests to complete or you are able to swap models. Click "Stop" to abort the current AI generation, change to a different model, and resend your query (you may need to click "Restore checkpoint" first before resending).

## What is Prismatic?

Prismatic is the leading embedded iPaaS, enabling B2B SaaS teams to ship product integrations faster and with less dev time. The only embedded iPaaS that empowers both developers and non-developers with tools for the complete integration lifecycle, Prismatic includes low-code and code-native building options, deployment and management tooling, and self-serve customer tools.

Prismatic's unparalleled versatility lets teams deliver any integration from simple to complex in one powerful platform. SaaS companies worldwide, from startups to Fortune 500s, trust Prismatic to help connect their products to the other products their customers use.

With Prismatic, you can:

- Build [integrations](https://prismatic.io/docs/integrations/) using our [intuitive low-code designer](https://prismatic.io/docs/integrations/low-code-integration-designer/) or [code-native](https://prismatic.io/docs/integrations/code-native/) approach in your preferred IDE
- Leverage pre-built [connectors](https://prismatic.io/docs/components/) for common integration tasks, or develop custom connectors using our TypeScript SDK
- Embed a native [integration marketplace](https://prismatic.io/docs/embed/) in your product for customer self-service
- Configure and deploy customer-specific integration instances with powerful configuration tools
- Support customers efficiently with comprehensive [logging, monitoring, and alerting](https://prismatic.io/docs/monitor-instances/)
- Run integrations in a secure, scalable infrastructure designed for B2B SaaS
- Customize the platform to fit your product, industry, and development workflows

## Who uses Prismatic?

Prismatic is built for B2B software companies that need to provide integrations to their customers. Whether you're a growing SaaS startup or an established enterprise, Prismatic's platform scales with your integration needs.

Our platform is particularly powerful for teams serving specialized vertical markets. We provide the flexibility and tools to build exactly the integrations your customers need, regardless of the systems you're connecting to or how unique your integration requirements may be.

## What kind of integrations can you build using Prismatic?

Prismatic supports integrations of any complexity - from simple data syncs to sophisticated, industry-specific solutions. Teams use it to build integrations between any type of system, whether modern SaaS or legacy with standard or custom protocols. Here are some example use cases:

- Connect your product with customers' ERPs, CRMs, and other business systems
- Process data from multiple sources with customer-specific transformation requirements
- Automate workflows with customizable triggers, actions, and schedules
- Handle complex authentication flows and data mapping scenarios

For information on the Prismatic platform, check out our [website](https://prismatic.io/) and [docs](https://prismatic.io/docs/).

## License

This repository is MIT licensed.
