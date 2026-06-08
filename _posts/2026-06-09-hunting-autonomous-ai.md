---
layout: post
title:  "Hunting Autonomous AI Workflows with Defender and Splunk"
category: security
author: "Mark F Hunt"
summary: >
  Autonomous AI agents create observable decision loops. 
  The pattern of Observe -> Diagnose -> Act is a reliable method of identifying autonomous AI workflows.
  Based on real SPL and real Defender telemetry.
---


**Why this matters**:
- AI agents create observable decision loops
- Traditional detections often focus on outcomes rather than decision cycles
- Agent telemetry may expose intent and reasoning that human operators do not

## Introduction

I recently had a reason to look for when Claude Code is being used as a fully autonomous agent. When Claude is configured with `bypassPermissions` mode, it can write code, deploy it to cloud platforms, read application logs, diagnose failures, and iterate on fixes with no human-in-the-loop. Some organizations allow this, others don't. Either way it's good to know how to track it.

This creates a new class of activity that looks superficially like a developer, but carries a very different risk profile: Claude never read your employee handbook, can't be held responsible for policy violations, its decisions are made at machine speed, and the full capability of a cloud CLI is available to the agent.

While this research focuses on Claude Code, the broader lesson is that autonomous agents create observable decision loops. The specific artifacts discussed here may change across tools, but the pattern of observe -> diagnose -> act is likely to appear across many agentic systems.


## The Pattern: What Autonomous Claude Looks Like

An autonomous Claude workflow running against cloud infrastructure produces a repeating cycle:

```
Check logs -> Diagnose -> Act (restart / scale / modify code) -> Wait -> Check logs again
```

The full loop can repeat dozens of times per hour, unattended, for hours or days. The key forensic artifact is that **every action is wrapped in a Claude hook**, which means each shell command carries a JSON payload containing Claude's reasoning. The `description` field is Claude narrating *why* it's doing something.

---

## Telemetry Source

The events described here come from **Microsoft Defender for Endpoint**, specifically the `DeviceProcessEvents` table, `ActionType = ProcessCreated`. 

In my situation, I was watching a Javascript workflow running on CloudFoundry. The child process was always `node`, running either a pre-built script or an inline `node -e '...'` payload. This would be different if you were watching a Python developer (look for `eval python3` or `PYEOF`). 

**What the telemetry looks like:**

```json
{
  "session_id": "6bac43dd-...",
  "permission_mode": "bypassPermissions",
  "hook_event_name": "PostToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "cf logs my-app --recent 2>&1 | tail -10",
    "description": "Check raw application logs"
  }
}
```

**Basic SPL:**

```spl
index=your_edr_index sourcetype=mdatpall ActionType=ProcessCreated
| where match(ProcessCommandLine, "permission_mode.*bypassPermissions")
```

---

## Observe -> Diagnose -> Act

When an AI is working autonomously on deploying code, it will follow a predictable pattern. Read the logs, determine what's wrong, make a change, then observe again.

### Observe

The AI will run `cf logs`, `kubectl logs`, or similar platform log commands with targeted `grep` queries to understand the state of a running application. The `description` field tells you exactly what diagnostic question Claude is trying to answer.

```json
{
  "command": "cf logs my-worker-app --recent 2>&1 | grep -iE 'error|OOM|crash' | tail -20"
}
```

### Diagnose

The incredible thing is Claude labels each command with its intent. The `description` field provides a machine-generated explanation of intent. This is invaluable for IR and threat hunting because it tells you what Claude believed was wrong at each point in time.

(Seriously, when do we ever get the chance to see *why* a user performed a command? Huge win here.)

```json
{
  "command": "cf restart-app-instance my-worker 3 2>&1",
  "description": "Restart instance #3 (disk full at 8G)"
}
```

These descriptions reveal that Claude was making categorized diagnoses: it distinguished between "disk full" failures and "process stuck" failures, and applied different remediation strategies accordingly.

### Act

When Claude identifies a bug or inadequacy in the running code, it modifies source files (`Edit` or `Write` tool calls) and then restarts the application to apply the changes. The Edit/Write tool calls do not create OS-level processes and are therefore invisible to EDR. But **the restart that follows them reveals they happened**.

The signal is a restart command whose `description` field references applying a change. "To pick up new entrypoint", "deploy updated config", "apply patch", etc.

**What the telemetry looks like:**

The Edit/Write events are invisible to Defender. You see only the deployment step with commands like `cf restart` `cf scale` `kubectl rollout restart` `kubectl scale` `heroku restart` `fly scale` `az webapp restart` `aws ecs update-service`.

Followed immediately by a post-deploy verification with commands like `pick up` `apply` `deploy` `new.*config` `updated` `patch` `change` `entrypoint` `restart.*after`

**The sleep-then-verify post-deploy pattern:**

The `sleep N &&` prefix seems to be a reliable signal: Claude is deliberately waiting for the app to fully restart before checking its output. This pattern `sleep -> cf logs` almost always follows a code change deployment.

---

## Broader Hunting - Is This Pattern Happening at All?

For environments where you have no prior leads, start with this broad hunt before pivoting to any specific account.

**Hunt for any process with a Claude hook in the parent chain:**

```spl
index=your_edr_index sourcetype=mdatpall ActionType=ProcessCreated
| where match(InitiatingProcessCommandLine, "\.claude/hooks/")
    OR match(ProcessCommandLine, "\.claude/hooks/")
| stats count, dc(AccountUpn) as accounts, values(AccountUpn) as account_list,
        dc(DeviceName) as hosts, earliest(Timestamp) as first_seen, latest(Timestamp) as last_seen
    by InitiatingProcessFileName
| sort -count
```

**Hunt for the inline `node -e` hook pattern specifically:**

```spl
index=your_edr_index sourcetype=mdatpall ActionType=ProcessCreated
| where match(ProcessCommandLine, "node -e '")
    AND match(ProcessCommandLine, "hook_event_name")
    AND match(ProcessCommandLine, "tool_name")
| rex field=ProcessCommandLine "\"permission_mode\":\"(?P<permission_mode>[^\"]+)\""
| stats count by permission_mode, AccountUpn, DeviceName
| sort -count
```
In Python, `node -e` is often replaced with `eval python3`. You can also look for `PYEOF`.

---

## Summary: IOBs (Indicators of Behavior)

These are not IOCs, there are no malicious hashes or suspicious domains. The signals are entirely behavioral:

| Behavior | Detection Method |
|---|---|
| Claude session in bypassPermissions mode | `permission_mode.*bypassPermissions` in ProcessCommandLine |
| Repeated log-checks against same app | `cf logs` or `kubectl logs` frequency by app name |
| Self-narrated diagnostic reasoning | Non-empty `description` field in tool_input JSON |
| Autonomous restart/scale of cloud apps | `cf restart`, `cf scale`, `kubectl rollout` in bash commands |
| Code change implied by restart description | Descriptions containing "pick up", "apply", "new entrypoint", etc. |
| Post-deploy wait-and-verify | `sleep N && cf logs` pattern, N > 10 |
| Deployed app solving CAPTCHAs | CAPTCHA/OCR strings in `stdout` of log-check events |
| Long-running autonomous session | Same `session_id` spanning > 1 hour of continuous activity |

---

## Notes From My Research

**Tuning guidance:** In my observation, the `description` field was the highest-fidelity signal because it is AI-generated natural language, not a command string that could vary syntactically.

**False positive risk:** Developers who use Claude Code interactively in `bypassPermissions` mode (a common configuration for trusted personal machines) will generate the same hook structure. The distinguishing factor is session duration and action velocity. An interactive human session will have gaps, human-paced actions, and a shorter window; an autonomous session runs continuously for hours with consistent cadence.

**Blind spots:** Edit and Write tool calls are invisible to EDR because they don't spawn processes. You can only infer that code was modified by the `description` of the subsequent restart. If you have file integrity monitoring on the developer's workstation or VCS webhooks, correlate those events with the restart timestamp to confirm.

**Bonus: Isn't This Just Automation?**

Many of these patterns resemble the automation loops defenders already hunt in malware and cloud intrusions. The difference is not the behavior per se, but the authorization and business context surrounding it. This is automation with a tool that is capable of observing, reasoning, and rewriting the code. It is non-deterministic, and operations staff is not writing rules for it to follow. The AI is trying to determine user intent from a natural language prompt then perform actions autonomously based on that, for extended durations of time.
