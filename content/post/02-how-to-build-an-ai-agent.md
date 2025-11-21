+++
date = "2025-07-26T09:00:00-05:00"
title = "How to Build an AI Agent Like an Engineer — Not a Prompt Hobbyist"
tags = ["AI Agents", "Workflow Automation", "Tooling", "LLM Engineering", "Observability"]
draft = false
description = "Planner + executor loops, tool schemas, guardrails, and telemetry are what make AI agents shippable—here's the engineering playbook."
+++

#### Introduction

Most "AI agents" you see online aren't agents. They're chatbots with cosplay ambitions.

Real agents — the kind that can plan, act, recover from errors, and operate reliably — require architecture, not prompt tricks.

The fundamental misunderstanding is this:

> LLMs aren't agents. They're planners inside an agent loop.

Engineers build the loop. LLMs choose the next step.

This article teaches the engineering approach: how to design agent loops, action schemas, state stores, tool interfaces, error-handling, and evaluation so your agent is repeatable, debuggable, and aligned.

This isn't "make the model role-play as Jarvis." This is build an actual system.

Let's break it down.

#### 1. The Agent Pattern: The Simplest Possible Definition

An agent is not magic. It is a structured loop:

```
state = initial_state()

while not goal_reached(state):
    action = LLM(plan using state + tools)
    result = execute(action)
    state = update(state, result)
```

That's it. Everything else — memory, tool libraries, retries, guardrails — is optimization and reliability engineering.

This framework is called:

**Observe → Plan → Act → Update (OPAU)**

This is how all agent systems (LangGraph, AutoGPT, CrewAI, Haystack agents, etc.) ultimately work.

- LLM = Plan
- Your code = Observe / Act / Update

If you forget this separation, your agent will be untestable and fragile.

#### 2. The Five Components of an Engineer-Grade Agent System

Every robust agent has five technical components:

1. Goal & Task Spec
2. State Management
3. Tool Interface Layer
4. Planner (LLM)
5. Execution Loop

Each of these requires engineering discipline. Let's go through them.

#### 3. Component 1 — Goal & Task Specification

Most agent failures start here. A goal must be:

- unambiguous
- measurable
- decomposable
- actionable

**Bad goal:**

> "Analyze this dataset and tell me insights."

**Good goal:**

> "Produce a JSON spec listing:
> - top 5 significant correlations (> 0.3 absolute),
> - missing values %,
> - outlier rows,
> using the provided dataset and the run_pandas_code tool."

LLMs perform much better when the goal is expressed as:

- inputs
- outputs
- constraints
- success conditions

You must specify:

- the shape of the desired output
- the tools allowed
- the error boundaries

This is the contract your agent executes under.

#### 4. Component 2 — State Management: The "Memory" That Actually Matters

Agents need different types of state:

##### 1. Ephemeral State

Internal scratchpad:

- intermediate tool outputs
- temporary planning notes
- parsed data

##### 2. Task State

The running context of the task:

- current step
- what has been done
- what remains
- errors encountered

##### 3. User State

Profile, preferences, restrictions.

##### 4. World State (External)

Retrieved documents, updates from APIs, DB results.

The LLM must NEVER store critical state purely in its own continuation. You must serialize and pass it back each loop.

**Example state structure:**

```json
{
  "goal": "...",
  "steps_completed": [],
  "current_step": "",
  "pending_items": [],
  "variables": {},
  "error_history": []
}
```

This is how you make an agent deterministic and debuggable.

#### 5. Component 3 — Tool Interfaces: Your Agent's "Hands and Eyes"

Tools are how an agent interacts with the world:

- run SQL
- query APIs
- fetch URLs
- call internal services
- write files
- trigger workflows
- run Python
- parse PDFs
- extract data
- send emails (with guardrails)

The #1 mistake: **tools are under-specified.**

**Bad tool:**

```json
{
  "name": "query_db",
  "input_schema": {"type": "string"}
}
```

This is a hallucination breeding ground.

**Good tool:**

```json
{
  "name": "query_sales_db",
  "description": "Run a safe, read-only analytics query.",
  "input_schema": {
    "type": "object",
    "properties": {
      "table": { "type": "string", "enum": ["orders", "customers", "invoices"] },
      "filters": { "type": "object" },
      "metrics": { "type": "array", "items": { "type": "string" } }
    },
    "required": ["table"]
  }
}
```

Why so strict? Because constrained tools = constrained behavior.

Tools are your safety rails, your policy enforcement, and your data correctness layer.

#### 6. Component 4 — Planner (LLM): What the Model Actually Does

The LLM's responsibilities in an agent:

- Interpret the goal
- Break it into steps
- Choose the next action
- Choose the correct tool
- Handle edge cases
- Update task plans
- Reason about failures

The LLM is not executing the steps. It is selecting the next step.

This is closer to symbolic planners than chatbots.

Good planner prompts include:

- goal
- state
- available tools
- last tool result
- constraints
- safety requirements
- execution rules
- expected output format

This transforms the LLM from "talker" to "decision maker."

#### 7. Component 5 — The Execution Loop: The Agent's Nervous System

This is where most agent libraries differ. But the core loop is simple:

```python
while True:
    let plan = LLM(state + tools)
    if plan.action == "finish":
        break
    result = execute_tool(plan)
    state = update(state, result)
```

The key is not letting the LLM run uncontrolled.

**Engineering rules:**

- Hard step limit
- Timeouts
- Retry policy
- Dead loop detection
- Deterministic logs
- State validation

Agents break the moment you treat LLMs like infallible executors. They are planners — nothing more.

#### 8. Building a Minimal Production-Grade Agent

Let's build a realistic example.

**Goal:**

> "Generate a 7-day workout plan using the user's profile, verify intensity against heart rate zones, and output a clean JSON schedule."

**Tools Needed:**

- fetch_user_profile
- get_zone2_range
- validate_workout_intensity
- write_output_json

**State Example:**

```json
{
  "goal": "create 7-day plan",
  "profile": null,
  "days_completed": 0,
  "draft_plan": [],
  "errors": []
}
```

##### Loop Execution Walkthrough

**Step 1: LLM Planning**

LLM decides:

- fetch profile → tool call
- store profile in state
- begin with Day 1 plan

**Step 2: Tool Execution**

Tool returns profile:

```json
{
  "age": 30,
  "resting_hr": 55,
  "max_hr": 185,
  "available_days": ["Mon","Tue","Wed","Thu","Fri"]
}
```

**Step 3: Update State**

State becomes:

```json
{
  "profile": {...},
  "days_completed": 0,
  "draft_plan": []
}
```

**Step 4: LLM Creates Day 1 Plan**

- proposes Z2 run
- intensity checked via validate_workout_intensity tool

**Step 5: Repeat**

Once 7 days are built → agent calls write_output_json.

This structure scales. You can replace tools, upgrade planning prompts, and debug every step. No "role-play as a fitness coach" prompt nonsense needed.

#### 9. Tooling Patterns Engineers Should Use

Every production agent benefits from these patterns.

##### Pattern 1: Schema-First Tooling (JSON Schema)

Use strict schemas so the LLM cannot abuse tools.

Schemas prevent:

- SQL injection
- invalid parameters
- out-of-range values

##### Pattern 2: Auto-Retry with Exponential Backoff

Agents should recover from transient failures:

- API rate limits
- timeouts
- partial data

Retries improve robustness dramatically.

##### Pattern 3: Synthetic Step Compression

Sometimes you want fewer LLM calls. Let the model:

- plan multiple steps
- but execute one
- then re-evaluate

This balances efficiency and control.

##### Pattern 4: Dual LLM System (Planner + Verifier)

Use small LLMs to:

- verify JSON
- check tool-call correctness
- validate schema
- inspect reasoning

This boosts reliability.

##### Pattern 5: Deterministic Observability Layer

Every step logged:

- state
- tool result
- LLM reasoning (summarized)
- errors
- retries

This is crucial for debugging.

#### 10. Error Handling: The Heart of Agent Reliability

Agents must be built to fail gracefully.

**LLM failure types:**

- invalid tool call
- bad JSON
- contradictory plan
- hallucinated parameters
- infinite loops
- premature "finish" signals

**Engineer solutions:**

- JSON schema validation
- repair prompts
- fallback prompts
- auto-correction loops
- tool safety wrappers
- execution caps

##### Add Failure Recovery Steps

If tool call fails:

- add to state.error_history
- LLM re-plans based on error
- tool retry (max N)
- fallback path if still failing

This is how real agents survive.

#### 11. Testing & Evaluation for Agents

You cannot ship an agent without tests.

**Must-have tests:**

- tool schema tests
- state mutation tests
- LLM planning tests
- negative tests (missing data)
- loop termination tests
- data consistency tests

Agents are not just "prompts" — they're systems.

#### 12. The Mindset Shift: Engineers Build Agents. Hobbyists Build Characters.

**Prompt hobbyists write:**

> "You are an agent. Think step-by-step."

**Engineers build:**

- state machines
- tool contracts
- retrieval pipelines
- validation layers
- deterministic loops

The difference is the difference between:

- a toy demo
- and a real automation system

Agents are engineering challenges, not storytelling exercises.

#### Conclusion

An LLM alone is not an agent. A prompt alone is not an agent. Role-play alone is not an agent.

An agent is a system composed of:

- Explicit goals
- Structured state
- Strict tools
- A reasoning model
- A tightly controlled execution loop

If you get these five right, you build agents that are:

- reliable
- explainable
- safe
- efficient
- maintainable

And — most importantly — aligned with engineering reality instead of hype.
