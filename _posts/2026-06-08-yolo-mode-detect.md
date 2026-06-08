---
layout: post
title:  "Detecting YOLO mode"
category: security
author: "Mark F Hunt"
type: note
---
Try looking for `InitiatingProcessCommandLine="*--permission-mode bypassPermissions*"` in your Defender logs. Also might be interesting to see what correlates with `(ProcessCommandLine="*vercel --prod*" OR ProcessCommandLine="*cf push*" OR ProcessCommandLine="*git push origin main*")`, or whatever deploy command your company uses.

It's always worth checking how many developers are letting Claude Code push to prod without human oversight.