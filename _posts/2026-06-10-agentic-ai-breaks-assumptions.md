---
layout: post
title:  "How Agentic AI Breaks Traditional Detection Assumptions"
category: security
author: "Mark F Hunt"
summary: >
  Agentic AI tools like Claude Code create a new category of endpoint actor: autonomous, fast-moving, credential-bearing, and difficult to attribute with traditional process telemetry alone.
---

## 1. Background & Case Summary

I was investigating agentic AI use at work and found an interesting session. I want to highlight what this session implies for cybersecurity defenders and how our assumptions are quickly shifting. What I found was:

- A ~2.5 hour autonomous session on a managed macOS endpoint
- 208 Defender events, primarily `ProcessCreated` action types
- 59 unique inline Python scripts executed via a zsh wrapper process
- All scripts authenticated to and modified a third-party SaaS API using a long-lived API credential
- The credential appeared in plaintext in `ProcessCommandLine` for every single event
- No script files were written to disk. All code was passed inline via shell heredoc

**Key Observation**

To a traditional detection rule, this session presents as a credential-bearing script executing repeated authenticated API calls to an external service over 2.5 hours, which is a pattern commonly associated with credential abuse or data exfiltration. In reality it was an AI agent completing a legitimate configuration task on behalf of a user.

## 2. How Claude Code Operates: The Agentic Loop

Understanding the *mechanics* of how Claude Code works is essential to building appropriate detection logic. The user interacts via natural language. The AI autonomously translates goals into executable actions. Traditional security assumes the actor, the credential owner, and the process owner are all the same entity. Agentic AI separates those roles.


| Phase | Description |
| --- | --- |
| 1. User Prompt | One or a few natural-language instructions. The user describes *what* they want, not *how*. |
| 2. AI Plans | Claude decomposes the goal into sub-tasks and decides autonomously what tools, such as Bash and file reads/writes, to use. |
| 3. Execute | A short-lived child process runs, such as a Python script. Output is returned to the AI. |
| 4. Reflect | Claude reads stdout/stderr. If it failed or was unexpected, it generates a corrected script. |
| 5. Repeat | Steps 3-4 loop autonomously until the sub-task succeeds or Claude asks the user a clarifying question. |

**The user does not approve each iteration.** A single prompt can authorize dozens of distinct system actions. In this case, approximately 59 API operations were performed from what was likely a handful of user messages, including mutations that deleted records, modified dashboards, and uploaded files.

Agentic AI simultaneously breaks two foundational assumptions: attribution and proportionality. We assume we can identify who performed an action, and we assume the observable activity roughly reflects the amount of user activity that occurred. Neither assumption remains reliable once an AI agent is operating autonomously.

## 3. Why Traditional Detection Rules Break Down

| Traditional Assumption | How Agentic AI Breaks It |
| --- | --- |
| **One human action = one process event** | One user prompt can produce 50+ process creation events across an hour or more. Volume-based anomaly detection fires on legitimate activity. |
| **Attribution: process owner = actor** | All 59 scripts run under the same user account. But the *decision-maker* is the AI. The human may not know what specific API calls were made. |
| **Credentials in CLI args = exfiltration or abuse** | Claude Code embeds credentials directly in command-line arguments by design and they appear in every `ProcessCommandLine` event, even for fully authorized use. |
| **Script-on-disk as indicator** | Claude Code never writes scripts to disk. All code is passed as inline heredoc strings. File-based detections do not apply. |
| **Repeated similar commands = automated attack tool** | The AI's iterative "Observe -> Decide -> Act" pattern looks identical to scripted credential stuffing or API enumeration from a Defender perspective. |
| **Scope of access = scope of the prompt** | The AI dynamically expands its own scope by discovering API endpoints at runtime. The user said "set up the workspace" and the AI decided on its own to call deletion, upload, and filter-modification endpoints. Traditional automation executes a predefined sequence of actions. Agentic AI determines the sequence of actions **while it is executing**. |

## 4. Security Implications & Risk Areas

### 4.1 Credential Exposure in Process Telemetry

In this case, a third-party SaaS API token was passed directly within the `ProcessCommandLine` field:

```text
API_TOKEN = "[REDACTED]"  # visible in Defender telemetry for all 59 child processes
```

This is not a user error. Even when used exactly as intended, credentials supplied to the agent may be propagated into command-line telemetry during execution. The user likely did not know the token would appear verbatim in endpoint telemetry. This represents a systemic credential hygiene risk that increases proportionally with AI tool adoption.

Agentic AI may fundamentally challenge the viability of long-lived API credentials. Credentials that remain valid for months become increasingly difficult to control once autonomous agents begin handling them directly. We've previously tolerated long lived access tokens because humans rarely touch them. The moment autonomous agents start handling credentials thousands of times per day, all the hidden assumptions around secret storage, telemetry exposure, and credential rotation become impossible to ignore.

### 4.2 Autonomous Scope Expansion

The AI autonomously called deletion APIs, file upload APIs, and dashboard state modification endpoints, none of which may have been individually anticipated by the user. The user's prompt authorized the *goal*, not the individual actions. Current DLP and access controls are not designed to intercept decisions made inside an LLM reasoning loop.

Traditional risk modeling asks: "what can this user access?" Agentic AI changes the question to: "what can this user's AI access, and how broadly will it interpret its mandate?" In this case, a user with access to one SaaS workspace effectively granted the AI broad authority to modify that workspace autonomously over a 2.5-hour window. The user was likely not watching for the majority of that time.

## 5. Opportunity

Agentic AI sessions produce rich, structured telemetry. You've got access to command-line content, consistent process lineage, and predictable shell patterns. This is more instrumentable than many traditional attack tools that actively evade logging. The challenge is not lack of signal, it's building detection logic that distinguishes legitimate AI-assisted work from misuse.

The opportunity is, the AI puts its thought process directly into its logs and tool calls. It tells you what assumptions it is making before it tries to act, then tells you what went right or wrong during the next step in its process. Its process patterns are highly consistent and it rarely tries evasion. Human adversaries actively try to hide; AI highlights everything it does in neon.

This creates an unusual defensive advantage. Instead of detecting malicious payloads, defenders can often detect the decision-making process itself. Repeated observe-decide-act loops, tool invocation patterns, autonomous retries, and iterative error correction all create behavioral signatures that are largely independent of the specific task being performed.

## 6. Summary

Agentic AI tools like Claude Code represent a new category of endpoint actor: one that operates autonomously, moves fast, embeds credentials in process arguments, and takes actions the user may not have individually approved. Our current detection and attribution models were designed for a world where one user action produces one observable event. That assumption no longer holds.

The good news is that these tools leave distinctive, consistent telemetry fingerprints. With targeted use-case development, like credential scanning in command lines, agentic session baselining, and external SaaS contact monitoring, etc, we can build effective coverage without generating excessive noise against legitimate use.

The deeper strategic shift is in how we think about **attribution and authorization**. Going forward, a security incident involving an AI agent will require us to answer not just "who ran this process?" but "what did the user ask for, what did the AI decide to do, and where is the boundary between them?" That is a question our current tooling cannot yet answer. Closing that gap should be a priority.
