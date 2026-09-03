---
layout: post
title:  "Attribution for Agentic AI"
category: security
author: "Mark F Hunt"
summary: >
  Attribution is a major issue for agentic AI use. Who authorized it? Who approved it? Was the user aware that this action was happening?
---

In a previous post I wrote about an instance of Claude Code breaking a CAPTCHA on a website. Through the logging I had available during that incidence, one question I was not able to answer was 
> Who authorized that action?

My Defender for Endpoint telemetry did not have the data necessary to answer the question. I know Claude was operating in autonomous mode so it did not need explicit authorization to perform that action. But did the user prompt Claude to bypass this technical control?

Imagine reading the incident report three different ways.

**Version A**

> User instructed Claude to bypass the CAPTCHA.

That's one problem.

**Version B**
> Claude proposed bypassing the CAPTCHA. User explicitly approved it.

That's a different problem.

**Version C**

> User requested data collection. Claude independently concluded bypassing the CAPTCHA was necessary.

That is an entirely different problem all over again.

Traditional automation executes predefined logic. Agentic AI executes *objectives* while exercising discretion about how to achieve them. Which is exactly why attribution becomes difficult. If a developer schedules a cron job, every action ultimately comes from code they wrote. If the work is delegated to Claude, that developer is delegating *judgment*, not just execution.

## Decision Provenance  

I think security will eventually need something like decision provenance: the ability to reconstruct not only what an AI agent did, but how authority for each significant decision was established. This could become one of the more important questions in agentic AI security: *what authority was delegated to the AI at the time it made this decision?*

Each of the above options imply different accountability, different policies, different detections, and may ultimately have different legal implications. Without being able to reconstruct how a decision was authorized, these scenarios become indistinguishable after the fact.

Today, organizations often seem to fall toward one of two extremes. Either every action taken by an agent is treated as if the employee performed it directly, or the agent is viewed as exercising meaningful autonomy within delegated authority. Neither extreme scales particularly well once agents begin operating for hours at a time. Reality will probably end up somewhere between those positions, but distinguishing them requires provenance that we often don't have.

If Claude deployed fifty times, did the employee make fifty decisions? Or just one?

## Incident Response

This will likely matter most often to IR teams. If an incident happened and an agentic AI was involved, was the decision explicitly made by the user, explicitly authorized by the user, or made by the agent while operating within authority previously delegated by the user? At this point, at least in my experience, we currently lack sufficient telemetry to distinguish between user-directed, user-authorized, and delegated autonomous AI actions. In practice, this often leaves incident responders unable to distinguish between user intent, user approval, and agent autonomy. Organizations may default to treating the actions as attributable to the user simply because they lack evidence to support a more nuanced conclusion.

Unfortunately, I have also see incidences where the user was able to say "I didn't approve that" or "I didn't expect that" and the incident was closed. It is definitely not a solved issue.

## Detections

Attribution also matters for building proper detections logic. In fact, it can sometimes be the difference between a policy violation and a security incident.

Imagine three detections:

| Telemetry shows                                      | What it might indicate             |
| ---------------------------------------------------- | ---------------------------------- |
| User repeatedly instructs agents to disable controls | Malicious or negligent user        |
| User repeatedly approves risky recommendations       | Human decision-making problem      |
| Agent repeatedly proposes or executes risky actions  | Agent safety or governance problem |

These are three different security problems. If the telemetry collapses them into "Claude ran Python," you've lost the ability to build targeted detections.

## Definitions

I'd love to see some standard language around this. Something like:

- user-directed actions
- user-authorized actions
- delegated autonomous actions

User-directed actions would be what the user explicitly prompted the AI to do. This is the user's direct intent. "If you encounter a CAPTCHA or other rate-limiting, figure out a way to bypass it." This is the most actionable attribution since the user knew and authorized the action ahead of time. 

User-authorized actions would be when the AI returns to the user and asks permission. "I've encountered a CAPTCHA and written code to solve the challenge. Would you like me to run this? Yes/No/Other". This is something the user may not have anticipated, and the AI is giving them the option. Clicking yes does not always mean the user was aware of the implications (no one reads every Claude python script just like no one reads the T&Cs), but it does indicate the user had the option to say no.

Delegated autonomous actions is the tricker one, and is the situation I witnessed in the CAPTCHA incident. Claude had broad authority to choose its own sequence of actions in pursuit of the user's objective. From the telemetry I had available, I could not determine whether bypassing the CAPTCHA reflected explicit user intent, an approved recommendation, or a decision the agent made within its delegated authority. That distinction matters, but today it's often invisible.

Attribution is no longer just about identifying who executed an action. It's increasingly about identifying who decided the action should be taken.