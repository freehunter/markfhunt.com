---
layout: post
title:  "Vague Direction in the Age of AI"
category: security
author: "Mark F Hunt"
---

Vague direction from stakeholders is annoying. It's difficult to follow a directive that someone hasn't fully thought out. A task that lacks clear requirements or even a well-defined ask quickly turns into a time sink. This becomes especially true once generative AI is involved.

Working at a tech company, my team has been strongly encouraged to use generative AI at work. In most situations we use this tool to automate well-defined tasks that we have already been completing, not for net-new work. We also maintain internal documentation for our team's inputs and outputs, which is written by hand. When we started this initiative, we were given the documentation template and expectations for the standard look and feel of the documentation, and it has gone well. Every team's documentation looks and behaves the same. Predictable, functional, usable.

But like most internal technical documentation, the material is dry and drab. And then we were encouraged to "make it prettier". This is dangerous, and I'll explain why.

## Vague Instructions are Dangerous

At an AI-first company, on engineering teams with no front-end design experience, the first thing any team member is going to do is open the AI and copy the stakeholder's instructions. In this case, the prompt is "make this page prettier". And the AI will comply as AI always will, without hesitation or follow-up questions, working only with the context it has been given. The AI follows the instructions it was prompted with, the engineer follows the instructions *they* were prompted with, and if the initial prompt was vague, the output will invariably be vague as well.

In this situation, the output was JavaScript loading several external dependencies from third party libraries. The output was infinite CSS animations, looping at 60FPS dozens of times on one page. The output was one thousand lines of front-end web design written by an AI, without anyone on the team having the knowledge or skills to read, understand, or maintain it. The output was "pretty". The reality was not.

Repeat this over half a dozen teams, and quickly the documentation look and feel began to drift. None of the pages looked the same, worked the same, followed the same layout. You needed to learn every team's design language and directory layout. And since the AI only has the context given in the prompt, every page quickly began to look different, even on the same team’s site.

Making it look "pretty" ended up making it pretty ugly.

## Vague Instructions are Unacceptable

If the engineers have access to generative AI to do their work, so does everyone at the company. Including the person giving the instructions. If visually unappealing technical documentation is not acceptable, how is a vague direction any more acceptable? If an engineer can ask Claude "make this page prettier", could a stakeholder not ask Claude "how would I direct an engineer to make this page prettier"? 

You can start with:  
`I want this documentation to look prettier and more professional.`

but maybe follow it up with:  
`Turn this into a clear directive for an engineering team. Include success criteria and examples.`

That's better. Maybe push it further:   
`Add specific, observable success criteria and remove ambiguity.`

And then have the AI translate for you:   
`Rewrite this as a ticket or project brief for an engineering team.`

And suddenly "make it prettier" turns into:  

### Example Final Directive
**Objective:**  
Improve documentation usability and presentation quality for internal and external cybersecurity stakeholders.


**Scope:**  
- Standardize formatting across all documentation
- Improve readability and navigation
- Add visual aids where helpful


**Success Criteria:**  
- All documents follow consistent heading structure
- No sections exceed X lines without formatting (lists, diagrams, etc.)
- Architecture sections include diagrams
- Documents are understandable by a non-engineer stakeholder


**Out of Scope:**
- No changes to underlying technical content
- No tooling migration unless necessary

## The Fine Line Between Being Prescriptive and Being Vague

A high level stakeholder doesn't want to tell an engineer how to do their job. That's a good thing. That's why they have engineers and why they trust their engineers. But the direction needs to be explicit, and you can be explicit about *outcomes* without being explicit about *methods*. "Prettier" isn't an outcome, it's an opinion, and it's non-actionable. It's the job of leadership to compress ambiguity and force clarity before execution.

If engineers have access to generative AI and are expected to use it to make their job faster and more efficient, it's the job of leadership to do the same. No one is expecting every stakeholder to be a technical leader. That's what we have AI for. 

So use it.
