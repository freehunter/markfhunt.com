---
layout: post
title:  "WHY NOT MITRE TTPS?"
---

# WHY NOT MITRE TTPS?

MITRE ATT&CK has become very popular in cybersecurity over the the last few years, with most vendors rushing to support MITRE’s Tactics, Techniques, and Procedures (TTPs) and ATT&CK Navigator. MITRE even has the MITRE ATT&CK Defender (MAD) certification to learn in-depth knowledge of the framework. In fact I am an active holder of the MAD certification.

With all of the breathless support of MITRE ATT&CK TTPs, I started thinking of the opposite side of the coin. A devil’s advocate argument AGAINST using TTPs.

## Attackers are tricky

TTPs are reactive by nature. They describe behaviors attackers have performed without much focus on the possibilities. Attackers who come in with brand new techniques, brand new tactics, brand new procedures, could be completely missed. It takes time for MITRE to incorporate brand new TTPs, and even longer for vendors to integrate those changes into their software. In short, ATT&CK can only describe attacks that are already known.

Also, attackers are often human. And humans are intensely creative, especially hackers. How often have you looked at your false positive alerts and wondered how to tell benign uses from malicious uses when they look and act exactly the same? Living off the Land often looks exactly like what your Operations team is already doing hundreds of times a day, every day. It’s difficult to detect an attacker using T1136.001 Create Account: Local Account when your own admins are routinely doing this same activity.

MITRE provides the building blocks for you to perform your own analysis, and those building blocks are often only valuable when combined with other techniques. And too often, they’re required to be in a very specific order with very specific timing. MITRE ATT&CK does not help provide this context, leaving it entirely to the team who is implementing.

## Resource Intensive

MITRE ATT&CK is big. Complex. Difficult to understand at a glance. It’s a massive framework and no one can memorize all of the TTPs and sub-techniques. It’s not always immediately obvious if you’re actually covering the TTPs you should be covering because the process for comparing your coverage to the framework is so arcane. And even once you get there, what does “good” look like? ATT&CK isn’t designed to tell you, you need to determine that yourself. And that is a skill that is not widely available.

I’ll give you an example. Last week my CTI team handed me a list of 3 threat actors to watch out for. I looked up all of their known TTPs and there were over 400 techniques and sub-techniques. I don’t have the time, energy, or resources to even look in my security tools to see if there are detections pre-built for 400+ TTPs, let alone actually build them out and tune them to be useful. I’m going to have to be selective in which ones I implement… but which ones do I need to focus on? Again, that’s an exercise left exclusively to the reader. Which means the real world implementation is going to vary dramatically between detection engineers based on skill, experience, and honestly free time, patience, and frustration tolerance.

And forget about keeping this up to date as things change. It’s massive enough and self-guided enough that it’s often better to just start the analysis from scratch rather than try to reconcile your customized TTPs with the latest update from MITRE or your software vendors.

## Limited Scope

MITRE’s approach is that TTPs are the best and all other IoCs are inherently less valuable. That might be true to a company who has A+ cybersecurity across the board, but that’s not how it works in real life. In the real world my SOC analysts are struggling to deal with a Denial of Service right now and need solutions right now. In this scenario, blocking the IP address is often the right call. It stops the bleeding so you can triage the impact appropriately. If I’m being hit by ransomware right now I need those file hashes and domain names. I don’t need to know the TTP number of ransomware and I don’t need to know what tool the attacker is using and I don’t need to know which attack groups are using this, I need to stop the attack ASAGDMFP.

I work with a tool that pulls in threat intel, and one vendor exclusively shares TTPs and not IoCs. That’s entirely useless to this tool because TTPs are not actionable. IoCs are. Yes they can change quickly so IoCs are not a long term defense strategy, especially against APTs and nation-state threat actors. But that does not mean IoCs are not useful. They are critical to reducing Mean Time To Resolution, while TTPs are more useful for Mean Time To Detect.

## Environments are different

Is creating a local user account suspicious? Yes… if your organization does not use local user accounts. If it does, welcome to False Positive land. Or a really awful example that I have actually seen in the real world with my own eyes, is reading /etc/shadow suspicious? You would think so! But I’ve run into actual operational workflows where part of the automation script was to (sigh) read the /etc/shadow file in order to ensure the necessary user accounts existed on the machine. This ran on every server across the entire company every day. Idiosyncrasies like this make it really difficult to deploy universal detection policies like MITRE ATT&CK is often used for.

## Conclusion

After all of this, what is my opinion on MITRE ATTA&CK? I like it. I do, and I will keep using it. But we need to keep in mind that MITRE compliance is not the end goal. ATT&CK is a tool, a framework, and a very high level framework at that. Ultimately, ATT&CK is a language. If we keep that in mind, we will be better off. We should be using MITRE ATT&CK to describe our detections and attacker behaviors, but not basing our entire cyber defense strategy based on how third party vendors implement a high level framework. Nor should we look at ATT&CK as the final say in the cybersecurity world.

MITRE ATT&CK is a valuable framework to talk about and help plan your detections and defenses. But it will not define those for you and it will not ensure your security. That is still entirely on your organization.
