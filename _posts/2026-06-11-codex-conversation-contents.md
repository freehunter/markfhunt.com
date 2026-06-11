---
layout: post
title:  "Observing Codex Conversation Contents in Defender Telemetry"
category: security
author: "Mark F Hunt"
summary: >
  When computer-use is enabled, OpenAI Codex writes the conversation history to the command line. This can be logged and stored by EDR tools like Defender, raising risk and privacy implications.
---

In my previous dives into Claude Code telemetry as seen through Microsoft Defender for Endpoint, I've noted that Claude logs its intent when it uses tools:

```json
{
  "command": "cf restart-app-instance my-worker 3 2>&1",
  "description": "Restart instance #3 (disk full at 8G)"
}
```

This is a huge win for blue teams. During a recent analysis of OpenAI Codex telemetry, I observed something far more surprising: complete AI conversations appearing directly in Defender process telemetry.

Initially, I believed this behavior was produced by Codex itself. However, further investigation suggests the conversation contents are specifically associated with Codex's computer-use component rather than all Codex activity.

In every example I observed where the full conversation appeared in Defender telemetry, the originating process was either `SkyComputerUseClient` or `codex-computer-use.exe`. In my own testing, Codex activity was visible in Defender logs, but conversation contents were not always present. The sessions where conversation contents were observed were associated with `SkyComputerUseClient` or c`odex-computer-use.exe`, suggesting the behavior may be tied to computer-use functionality.

When the behavior occurs, the computer-use client emits a `turn-ended` command at the completion of an agent turn. The full JSON payload is passed as an argument in `ProcessCommandLine`. That payload includes:

* **`input-messages`**: an array containing the full text of every message the user sent during the session
* **`last-assistant-message`**: the complete text of the AI's most recent response
* **`thread-id`** and **`turn-id`**: unique identifiers enabling precise session and turn correlation
* **`cwd`**: the working directory for the session
* **`client`**: the Codex client variant in use


The command looks roughly like:

```
SkyComputerUseClient turn-ended '{"type":"agent-turn-complete","thread-id":"<uuid>","turn-id":"<uuid>","cwd":"/path/to/project","client":"Codex Desktop","input-messages":[...],"last-assistant-message":"..."}'
```

### Security Implications

When this telemetry is present, a person with access to Defender logs can reconstruct the full conversation entirely from process logs. This includes the user's questions, any data the user pasted into the session, URLs the user shared, and the AI's complete final reply. Each `turn-ended` event is cumulative: the `input-messages` array grows with each turn, causing conversation contents to be duplicated across multiple telemetry events over the lifetime of a session.

This is a fundamentally different information surface than anything produced by Claude Code's telemetry from my previous analysis. It shifts the question from "what commands did the AI run?" to "what was the user asking the AI to do, and what did the AI conclude?" The AI's reasoning about a problem, including references to internal systems, data it was shown, and its own analysis of that data, appears as plaintext in `ProcessCommandLine`.

In comparison, Claude's conversation is stored in a local `.jsonl` transcript file under `~/.claude/projects/`. The path to that file may appear in process telemetry (for example, in hook-related events), but the content does not. Recovering the conversation requires reading that file directly. Claude Code's approach keeps conversation content out of telemetry pipelines (and the DLP risk that entails), while Codex's computer-use telemetry makes user intent directly observable from logs alone.

### Why This Matters

When this telemetry is present, the implications extend beyond detection engineering.

Traditional endpoint telemetry is generally expected to contain process execution details, command lines, file paths, and network activity. In these observed cases, however, the telemetry also contained user-generated conversation content and AI-generated analysis. That changes the effective scope of data collection. Information entered into an AI session may no longer be confined to the AI platform itself. The same content can be duplicated into endpoint telemetry, SIEM platforms, long-term log retention systems, data lakes, backup archives, exported investigations, and analyst workflows.

The cumulative nature of the turn-ended events amplifies this effect. Because each event contains the growing conversation history, the same content may be recorded repeatedly throughout a session.

For defenders, this creates both an opportunity and a risk. The visibility can be invaluable for understanding user intent, reconstructing agent behavior, and investigating incidents. At the same time, organizations may be collecting significantly more user-generated content than they realize, raising questions around retention, privacy, and data loss prevention programs.