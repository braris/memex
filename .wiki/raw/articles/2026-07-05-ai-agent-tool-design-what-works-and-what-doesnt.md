---
title: "AI Agent Tool Design: What Works and What Doesnt"
source: "https://machinelearningmastery.com/ai-agent-tool-design-what-works-and-what-doesnt/"
type: articles
ingested: 2026-07-05
tags: [ai-agents, tool-design, mcp, schemas, error-handling, agent-reliability]
summary: "MachineLearningMastery article arguing that many AI agent failures are caused by poor tool interface design, with patterns for single-responsibility tools, tight schemas, scoped descriptions, structured errors, idempotency, dynamic tool loading, and confirmation gates."
fetched: 2026-07-05
---

[[ReadItLater]] [[Article]]

# [AI Agent Tool Design: What Works and What Doesnâ€™t - MachineLearningMastery.com](https://machinelearningmastery.com/ai-agent-tool-design-what-works-and-what-doesnt/)

In this article, you will learn how tool design â€” not model capability â€” is the root cause of most AI agent failures, and what concrete design patterns you can apply to fix it.

Topics we will cover include:

-   Tool design practices that improve agent reliability, including single-responsibility tools, tight schemas, and structured error returns.
-   Common failure modes such as unfiltered API exposure, silent partial success, and overlapping tool names that break real-world workloads.
-   Schema and error handling patterns that reduce hallucination and unreliable behavior at the tool boundary.

Letâ€™s get into it.

![AI Agent Tool Design: What Works and What Doesn't](https://machinelearningmastery.com/wp-content/uploads/2026/06/mlm-agent-tool-design.png)

AI Agent Tool Design: What Works and What Doesnâ€™t  

## Introduction

Most [AI agent](https://www.ibm.com/think/topics/ai-agents) failures look like model mistakes: choosing the wrong tool, passing bad arguments, or mishandling errors. But in practice, the model is usually working with the interface it was given. The underlying issue is often the **tool design** itself.

A model can only reason from the information exposed through the tool interface: the tool name, its description, the parameter schema, and the parameter descriptions. Those details shape how the model interprets intent, plans actions, and executes tasks. When the tool design is unclear, incomplete, or loosely structured, failures become predictable rather than accidental.

Problems like vague naming, ambiguous instructions, inconsistent schemas, weak parameter definitions, and poor error handling all increase the likelihood of failures. Stronger models can reduce some mistakes, but they cannot reliably compensate for a flawed interface. This article covers:

-   Tool design practices that improve reliability
-   Failure modes that look fine in demos but break under real workloads
-   Schema and error design that reduces hallucination at the tool boundary

Each pattern is paired with its failure counterpart, because understanding *why* a design fails is as important as knowing *what* to replace it with.

## What Works in AI Agent Tool Design

### 1\. One Tool, One Responsibility

In most agent systems, **a tool should represent a single, clear operation**. When one tool handles multiple behaviors through an `action` parameter, the model must first figure out which mode to invoke before it can solve the actual task.

The difference becomes clearer when comparing a multi-action tool against dedicated single-purpose tools:

|   1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27     |   \# Avoid: action-based multi-behavior tool  @tool  def manage\_customer(  Â Â Â Â action: str,  Â Â Â Â customer\_id: str \| None \= None,  Â Â Â Â data: dict \| None \= None  ):  Â Â Â Â """  Â Â Â Â action: create \| get \| update \| delete \| suspend  Â Â Â Â """  Â Â Â Â ...  \# Prefer: single-responsibility tools  @tool  def create\_customer(data: CustomerInput) \-> Customer:  Â Â Â Â """Create a new customer record."""  Â Â Â Â ...  @tool  def get\_customer(customer\_id: str) \-> Customer:  Â Â Â Â """Retrieve a customer by ID."""  Â Â Â Â ...  @tool  def suspend\_customer(customer\_id: str, reason: str) \-> SuspensionResult:  Â Â Â Â """Suspend a customer account."""  Â Â Â Â ...     |
| --- | --- |

![One Tool, One Responsibility](https://machinelearningmastery.com/wp-content/uploads/2026/06/srp-tool-design.png)

One Tool, One Responsibility  

  
Single-responsibility tools give the model an unambiguous function and give you cleaner error handling and easier [observability](https://www.ibm.com/think/insights/ai-agent-observability).

**âš ï¸ Note**: This is a useful default rather than a universal rule. Some domains â€” such as shell, filesystem, browser, or calendar tools â€” may benefit from a constrained multi-action interface because the action space itself is part of the underlying abstraction.

### 2\. Schemas That Make Invalid States Impossible

In [tool-calling agents](https://machinelearningmastery.com/the-roadmap-to-mastering-tool-calling-in-ai-agents/), the model constructs tool call arguments by reasoning from your schema.

-   A loose schema means the model guesses at constraints.
-   A tight schema encodes those constraints so no guessing is needed.

Hereâ€™s an example:

|   1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22     |   from pydantic import BaseModel, Field  from enum import Enum  class Priority(str, Enum):  Â Â Â Â LOW \= "low"  Â Â Â Â MEDIUM \= "medium"  Â Â Â Â HIGH \= "high"  class CreateTaskInput(BaseModel):  Â Â Â Â title: str \= Field(  Â Â Â Â Â Â Â Â description\="Short, actionable task title. Use imperative form: 'Review PR', not 'PR Review'.",  Â Â Â Â Â Â Â Â min\_length\=5,  Â Â Â Â Â Â Â Â max\_length\=100  Â Â Â Â )  Â Â Â Â priority: Priority \= Field(  Â Â Â Â Â Â Â Â description\="Task priority. Use HIGH only for blockers affecting other work.",  Â Â Â Â Â Â Â Â default\=Priority.MEDIUM  Â Â Â Â )  Â Â Â Â due\_date: str \= Field(  Â Â Â Â Â Â Â Â description\="Due date in ISO 8601 format: YYYY-MM-DD. Must be a future date.",  Â Â Â Â Â Â Â Â pattern\=r"^\\d{4}-\\d{2}-\\d{2}$"  Â Â Â Â )     |
| --- | --- |

Enums are particularly useful for fields with a small set of valid values because they eliminate a class of plausible-but-invalid outputs. Validation failures surface at the tool boundary rather than as cryptic downstream errors.

### 3\. Descriptions That Define Scope, Not Just Purpose

[Tool descriptions](https://www.anthropic.com/engineering/writing-tools-for-agents#prompt-engineering-your-tool-descriptions) are model-facing documentation. They need to do two things: **explain when to use the tool, and explain when not to**. Most descriptions only do the first.

|  |   \# Weak: explains what it does, not when not to use it  """Search for documents in the knowledge base."""  \# Strong: defines purpose, scope, and boundaries  """  Search the internal knowledge base for documents, policies, and reference material.  Use this when the user asks about company procedures, product specs, or documented workflows.  Do NOT use this for real-time data (prices, availability, current status) â€” use get\_live\_data() instead.  Returns up to 5 results ranked by relevance. If no results are returned, the information is not in the knowledge base.  """     |
| --- | --- |

Without the disambiguation, the model infers scope from the tool name alone, which is often a reliable source of selection errors at scale. A good tool definition includes clear boundaries from other tools, not just usage instructions.

### 4\. Structured, Actionable Error Returns

When a tool fails, the model reads the error and decides what to do next. An unhandled exception or stack trace produces noise-driven follow-up behavior. A structured error gives the model something to branch on.

Structured errors should not only report what failed but also help the agent decide what to do next. A good error format makes retry behavior explicit and gives the model a clear recovery path:

|   1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21     |   class ToolError(BaseModel):  Â Â Â Â error\_code: strÂ Â Â Â Â Â  \# machine-readable, for the model to branch on  Â Â Â Â message: strÂ Â Â Â Â Â Â Â Â Â \# human-readable description  Â Â Â Â recoverable: boolÂ Â Â Â  \# can the agent retry?  Â Â Â Â suggested\_action: str \# what the agent should do next  \# Record not found: retryable  return ToolError(  Â Â Â Â error\_code\="RECORD\_NOT\_FOUND",  Â Â Â Â message\="No user record found with ID 'usr\_123'.",  Â Â Â Â recoverable\=True,  Â Â Â Â suggested\_action\="Use list\_users() to get valid user IDs before calling get\_user()."  )  \# Quota exceeded: not retryable  return ToolError(  Â Â Â Â error\_code\="QUOTA\_EXCEEDED",  Â Â Â Â message\="API quota for this tool has been reached for today.",  Â Â Â Â recoverable\=False,  Â Â Â Â suggested\_action\="Notify the user and stop. Do not retry this tool today."  )     |
| --- | --- |

The `recoverable` flag and `suggested_action` field are what change agent behavior. Without them, models retry non-retryable errors or abandon recoverable ones.

### 5\. Idempotent State-Changing Operations

Every tool that mutates state â€” creates a record, sends a message, transfers funds â€” must be safe to call twice. In practice, agents retry, networks fail, and the LLM loop may issue a second call because confirmation of the first never arrived.

A simple way to prevent duplicate side effects is to require an idempotency key for every write operation:

|   1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17     |   @tool  def send\_email(  Â Â Â Â to: str,  Â Â Â Â subject: str,  Â Â Â Â body: str,  Â Â Â Â idempotency\_key: str \= Field(  Â Â Â Â Â Â Â Â description\="Unique key for this send operation. Use a hash of recipient + subject + timestamp. "  Â Â Â Â Â Â Â Â Â Â Â Â Â Â Â Â Â Â Â Â "Same key on retry returns the original result without re-sending."  Â Â Â Â )  ) \-> dict:  Â Â Â Â """Send an email. Idempotent: the same idempotency\_key will not trigger a second send."""  Â Â Â Â existing \= idempotency\_store.get(idempotency\_key)  Â Â Â Â if existing:  Â Â Â Â Â Â Â Â return existing  Â Â Â Â result \= email\_service.send(to\=to, subject\=subject, body\=body)  Â Â Â Â idempotency\_store.set(idempotency\_key, result, ttl\=86400)  Â Â Â Â return result     |
| --- | --- |

Without idempotency guarantees, transient failures can easily turn into duplicate actions.

## What Doesnâ€™t Work in AI Agent Tool Design

### 1\. Thin Wrappers Around Unfiltered APIs

Pointing an agent at a REST API and surfacing it as a tool is the most common shortcut and the most common source of production failures. [APIs built for developers often expose far more detail than agents actually need](https://www.anthropic.com/engineering/writing-tools-for-agents#choosing-the-right-tools-for-agents). Responses come packed with hundreds of fields, even when only a handful are relevant. They rely on pagination, use opaque internal IDs with little contextual meaning, and return error codes that require deep domain knowledge to interpret.

A purpose-built wrapper handles pagination internally, projects only the fields the agent needs, and maps API errors to the structured ToolError format discussed above. The agent never constructs API paths or manages pages; it receives typed objects it can reason about.

That said, over-wrapping can also be harmful. If every endpoint becomes a separate, narrowly defined tool with no shared structure, the tool surface can become fragmented and harder for the model to navigate. The goal is not maximal abstraction, but a consistent, agent-friendly abstraction layer.

### 2\. Loading All Tools Into Every Context

Accuracy degrades as the tool catalog grows. [LongFuncEval](https://arxiv.org/abs/2505.10570), a 2025 study on tool-calling performance across long contexts, found [performance drops substantially as the tool catalog size increased](https://arxiv.org/pdf/2505.10570) â€” even in models with 128K context windows. Loading every tool into every system prompt compounds this by consuming token budget before any task content is processed.

[Dynamic tool loading](https://www.lunar.dev/post/why-dynamic-tool-discovery-solves-the-context-management-problem) addresses both problems. Determine which tools are relevant to the current step and include only those:

|  |   STEP\_TOOL\_MAP \= {  Â Â Â Â "research": \["search\_documents", "search\_web", "get\_url\_content"\],  Â Â Â Â "write":Â Â Â Â \["create\_document", "update\_document", "format\_text"\],  Â Â Â Â "send":Â Â Â Â  \["send\_email", "post\_to\_slack", "create\_calendar\_event"\],  }  def get\_tools\_for\_step(step\_type: str, available\_tools: list) \-> list:  Â Â Â Â relevant\_names \= STEP\_TOOL\_MAP.get(step\_type, \[\])  Â Â Â Â return \[t for t in available\_tools if t.name in relevant\_names\]     |
| --- | --- |

![Dynamic Tool Loading](https://machinelearningmastery.com/wp-content/uploads/2026/06/dynamic-tool-loading.png)

Dynamic Tool Loading  

  
Exposing only a small, relevant subset of tools at each step â€” rather than the full toolset â€” generally improves selection accuracy and reduces per-call token cost.

### 3\. Silent Partial Success

Partial success becomes a problem when a tool completes only part of the requested work but returns a response that looks fully successful. The agent continues execution with an incomplete or misleading view of the system state.

This usually happens when tools suppress internal failures and return only the successful portion of the result:

|   1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27     |   \# This version silently misleads the agent  @tool  def bulk\_create\_tasks(tasks: list) \-> dict:  Â Â Â Â created \= \[\]  Â Â Â Â for task in tasks:  Â Â Â Â Â Â Â Â try:  Â Â Â Â Â Â Â Â Â Â Â Â result \= task\_api.create(task)  Â Â Â Â Â Â Â Â Â Â Â Â created.append(result.id)  Â Â Â Â Â Â Â Â except Exception:  Â Â Â Â Â Â Â Â Â Â Â Â passÂ Â \# silent failure: this is the bug  Â Â Â Â return {"created": created}  \# This version makes partial success explicit  @tool  def bulk\_create\_tasks(tasks: list) \-> BulkCreateResult:  Â Â Â Â created, failed \= \[\], \[\]  Â Â Â Â for task in tasks:  Â Â Â Â Â Â Â Â try:  Â Â Â Â Â Â Â Â Â Â Â Â created.append(task\_api.create(task).id)  Â Â Â Â Â Â Â Â except TaskCreationError as e:  Â Â Â Â Â Â Â Â Â Â Â Â failed.append({"input": task.title, "reason": str(e)})  Â Â Â Â return BulkCreateResult(  Â Â Â Â Â Â Â Â created\_ids\=created,  Â Â Â Â Â Â Â Â failed\_items\=failed,  Â Â Â Â Â Â Â Â success\=len(failed) \== 0,  Â Â Â Â Â Â Â Â partial\_success\=len(created) \> 0 and len(failed) \> 0  Â Â Â Â )     |
| --- | --- |

The `partial_success` flag gives the model something to branch on: retry the failed items, surface the partial result to the user, or halt the workflow.

### 4\. Overlapping Tool Names and Descriptions

When two tools do *similar* things, the model reasons about which to use on every call. That reasoning costs tokens and introduces errors. Some common examples include:

-   `search_documents` and `find_documents` with identical purpose
-   `get_user` and `fetch_user_profile` with unclear differences
-   `create_task`, `add_task`, and `new_task` as three tools for one operation

In such cases, renaming alone isnâ€™t the fix. Every tool needs a purpose that can be described without reference to other tools in the set. If a description requires â€œunlike X, this oneâ€¦â€ to make sense, thatâ€™s a design problem. Tool sprawl â€” too many tools with overlapping scope â€” is a source of unreliable agent behavior in enterprise deployments.

### 5\. Destructive Actions Without a Confirmation Gate

Any tool that takes an irreversible action â€” deleting records, messaging real users, executing financial transactions â€” needs a structural two-step confirmation, not an in-prompt â€œare you sure?â€ A staged approach introduces an explicit confirmation boundary that reduces the risk of accidental or unauthorized execution.

The safest pattern is to separate staging from execution and require a short-lived confirmation token between the two steps:

|  |   @tool  def stage\_deletion(record\_ids: list\[str\], reason: str) \-> StagedDeletion:  Â Â Â Â """Stage records for deletion. Does NOT delete anything.  Â Â Â Â Returns a confirmation token that expires in 60 seconds.  Â Â Â Â Call confirm\_deletion() with this token to proceed."""  Â Â Â Â token \= generate\_deletion\_token(record\_ids)  Â Â Â Â staged\_deletions\[token\] \= {"ids": record\_ids, "expires": now() + 60}  Â Â Â Â return StagedDeletion(token\=token, records\_to\_delete\=len(record\_ids), expires\_in\_seconds\=60)  @tool  def confirm\_deletion(token: str) \-> DeletionResult:  Â Â Â Â """Execute a staged deletion. IRREVERSIBLE. Confirm only after explicit user approval."""  Â Â Â Â staged \= staged\_deletions.get(token)  Â Â Â Â if not staged or staged\["expires"\] < now():  Â Â Â Â Â Â Â Â raise ValueError("Token invalid or expired. Stage the deletion again.")  Â Â Â Â \# proceed     |
| --- | --- |

![Destructive Actions Without a Confirmation Gate](https://machinelearningmastery.com/wp-content/uploads/2026/06/no-single-step-destruction.png)

Destructive Actions Without a Confirmation Gate  

  
Two distinct tool calls mean the model cannot complete a destructive operation in a single reasoning step, which is the point.

**âš ï¸ Note**: Two-step safety flows, however, are often **not sufficient on their own** in many systems. Even when staging and confirmation are used, additional safeguards â€” such as short-lived, single-use tokens, strict session binding, and replay protection â€” are necessary to prevent token reuse, leakage, or cross-session execution that can bypass the intended safety boundary.

## AI Agent Tool Design Decisions at a Glance

Every row represents a key decision in AI agent tool design:

| Design Area | Works | Doesnâ€™t Work |
| --- | --- | --- |
| Tool Scope | Single responsibility per tool | Action-parameter tools like `manage_database(action="create")` |
| Schema | Tight: enums, validators, typed fields | Loose: free strings, untyped dicts |
| Descriptions | Include scope boundaries and when *not to use* | Happy path only |
| Write Operations | Idempotent with idempotency keys | Fire-and-forget, no retry safety |
| Error Returns | Structured: `error_code`, `recoverable`, `suggested_action` | Unhandled exceptions or untyped strings |
| Tool Count | Dynamic loading per step | All tools in every context |
| API Wrapping | Purpose-built wrapper with agent-facing schema | Unfiltered API exposure |
| Partial Success | Explicit `partial_success` field in return | Silent exception swallowing |
| Destructive Actions | Two-step staging + confirmation | Single-call delete/send/execute |
| Tool Overlap | Semantically distinct, audited before deploy | Similar names and descriptions competing |

[Writing effective tools for AI agents â€” using AI agents from Anthropic](https://www.anthropic.com/engineering/writing-tools-for-agents) is a useful reference on tool design.

##### No comments yet.
