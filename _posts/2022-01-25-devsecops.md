---
layout: post
title:  "YOU’RE PROBABLY NOT DOING DEVSECOPS SO STOP SAYING YOU ARE"
tags: security
---

# YOU’RE PROBABLY NOT DOING DEVSECOPS SO STOP SAYING YOU ARE

I was a security consultant for 6 years, and I’ve been working in security for 10 years and if there’s one thing I’ve learned its this: IT and the rest of the business really doesn’t like or even understand security. We get in their way and we demand things of them that only help us, and the business never sees value from it. The value exists, for sure, but the business doesn’t see it.

I saw this constantly when I was a SIEM consultant and we had to ask Ops to install a new agent. They already had Tanium and Chef and McAfee and their monitoring tools and SCCM and sysmon and audit.d and all these other agents running, and we need one more. Just for security. No you can’t look at it or touch it.

## DevSecOps implies a relationship that probably doesn’t exist

(I want to point out that the amazing [Kelly Shortridge](https://twitter.com/swagitda_) inspired this line of thinking with her article on [Security Obstructionism](https://swagitda.com/blog/posts/the-security-obstructionism-secobs-market/)

We like to say “DevSecOps” to latch onto the wildly successful DevOps principles but what does it really mean? Why not DevSecQAOps? Is QA not important? Why not DevSecQASalesOps? Is sales not important? DevSecQASalesMarketingOps. DevSecQASalesMarketingFinanceOps. DevSecQASalesMarketingFinanceRecruitingOps. I could keep going or we could admit that it’s a silly term.

If we want to latch on to successful IT practices (and we should), we should start by recognizing your team likely provides no visible value to the business. And security tooling is expensive. And from a dev/ops perspective it doesn’t work. And it takes a lot of expensive security engineers to keep it running and it doesn’t work.

When a breach happens, all IT sees is security panicking over something security failed to stop in the first place. So the next time we come to Ops and ask them to help us expand our Splunk instance… why? Or to install yet another agent on their already over-taxed servers… why?

## Because security thinks we’re different (we’re not)

The DevOps movement saw developers managing a limited amount of infrastructure and operations writing a limited amount of code. They met each other in the middle and found places where they overlap, then helped create a seamless transition between their two workflows.

Then security came in and said “oh yeah we want that too!” but refused to write code. Refused to be responsible for server management. Refused to use DevOps tools that already existed. Demanded that Ops spend their time installing agents, then locked Ops out of the tools those agents fed data to. Declared a security incident, shut down servers and services, and kept Ops in the dark because “this incident is need-to-know only”.

And now we can’t find any security engineers to hire because most people don’t know our obscure toolset and can’t afford the million dollar price tags to learn it on their own. But developers train for jobs on their own because Javascript is free and there are a thousand tutorials written every day. Operations can train for jobs on their own because Linux and Kubernetes are free and only cost a few hours watching YouTube videos.

We gatekeep our industry with mystery and magic and million dollar price tags, can’t hire anyone to do the work, then insist we’re an integral part of the business? It’s nonsense.

## The Proposal

First off, stop treating security like a magic trick. It’s not, we just pretend it is because none of us actually know what we’re doing so we hide it behind the magic curtain and don’t let anyone see it.

Secondly, maybe we should use some of the DevOps tools. They’re quite good. Do we need Tanium, or is Chef good enough (and already deployed)? Do we need QRadar/ArcSight/AlertLogic/SecureWorks/etc, or is the Splunk your DevOps teams already use good enough? We’ve relegated ourselves to an “advisory” role so we can hide any and all responsibility we have behind the shield of “we just make recommendations”. Well, if we want to be part of DevOps so bad, why don’t we write the gold Terraform/CloudFormation templates instead of just auditing them after the servers have been built?

Third, the entire company goes through security training every year. Does security go through “how to operate a server” training? Does security go through “how to write secure code” training? Of course not. So why is security “everyone’s responsibility” but the business isn’t security’s responsibility?

## We all want the same thing

Dev wants to write code that works right. They’re not putting bugs and vulnerabilities in there just for fun. Ops wants to keep their servers running. Taking prod offline to re-image a server isn’t part of that goal. We’re all working for the same team.

Dev has tools to help audit their code. Ops has tools to watch for servers behaving unusually. DevOps has tools to watch code at runtime and make sure everything is up to spec, applications are running normally, users aren’t experiencing too many errors, etc. With just a tiny bit more guardrails, this starts to look exactly the same as any security tool you might want, except it’s something Dev and Ops can use too. And not just can use, but something they want to use because it provides value to them, too!

What’s an easier sell: convincing Ops to install a new agent and build VMs and modify firewall rules and configure logging destinations? Or convincing Ops to add another line to their existing tooling that checks for out-of-spec behavior they hadn’t already been looking for? I guarantee they want to know that out-of-spec behavior too, but they don’t know what to look for. That’s where we should be helping! Instead we steal their data and hide it away and declare an incident without involving them. That’s nuts.

## It’s time security grows up and becomes part of the business

Dev and Ops and the rest of the business just wants to keep their servers running and data flowing. Security wants the same thing too. So why do we refuse to integrate? Why do we position ourselves as an “other”, sitting outside the business and outside the rest of IT, just heads-down “doing security”?

Grow up, drop the million dollar ineffective single-use tools, and start working. You want to be in the middle of Dev and Ops? Then act like it.
