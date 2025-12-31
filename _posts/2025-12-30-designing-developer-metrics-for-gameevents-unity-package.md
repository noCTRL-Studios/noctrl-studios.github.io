---
title: "Designing Developer Metrics for GameEvents Unity Package"
author: Silas
date: 2025-12-30 14:54:05 -0800
categories: [Unity  Programming]
tags: [Unity]
render_with_liquid: false
---



## Designing Developer Metrics for the GameEvents Unity Package

> 💡 **Active Development:**  
> Follow progress and commits on the [GameEvents Dev branch](https://github.com/noCTRL-Studios/com.noctrl.game-events/tree/Dev).

Sometimes inspiration comes from weird corners. I was watching a documentary about cracking the Enigma codes when something clicked — the Allies didn’t just break messages; they analyzed the metadata around them: who sent what, how often, and from where. They could learn a ton without ever reading the contents.

That idea stuck with me right after releasing the first version of GameEvents. I started wondering: what if developers could do the same thing with their events?
Not reading the data inside — just observing how often systems talk to each other and how they behave during play.

That curiosity turned into the GameEvents Developer Metrics system.

## Why Build Metrics at All?

I wanted a way for developers to see what their systems were doing in real time — how events actually moved through their project.
The goals were simple:

Run automatically, no setup or boilerplate.

Visualize event activity without touching gameplay code.

Provide insight for debugging, performance, and design analysis.

So I focused on tracking a few key things:

When each event fires

How many listeners were active at that moment

Total invocation counts

When listeners registered and unregistered

All without changing how people use the package.


## Lessons from Building It

Digging into this surfaced some interesting corners of Unity’s editor behavior.

Playmode callbacks:
Unity’s EditorApplication.playModeStateChanged turned out to be the cleanest way to detect session boundaries. That callback is gold — it fires when entering or exiting Play Mode, which made it perfect for knowing when to start and stop data collection.

Editor-only singletons:
Keeping a persistent editor-only singleton for metrics was trickier than expected. It has to live in the Editor domain but still survive domain reloads and be safely referenced during Play Mode. I ended up wrapping it with preprocessor macros so the runtime version compiles cleanly. I can already see future footguns if something references it while the editor object’s been unloaded.

Writable cache paths:
Git-based Unity packages are read-only, which means you can’t store session data in the package folder.
The workaround: write to the Library directory — Unity’s intended scratch space.

```c#
    string path = Path.Combine(Application.dataPath, "../Library/<Cache_Folder>");
```
That path is guaranteed writable and keeps metrics local per project.


## What It Looks Like So Far

The current editor dashboard exposes two simple control groups:

Event Metric Controls — for clearing, viewing, or exporting session data

Game Event Controls — for creating and monitoring active events

Here’s a look at the early build of the dashboard UI:

![Dashboard](assets/images/GameEvents_Dashboard_UI.png)


Right now, metrics are being collected and exported to JSON whenever Unity disables objects. That’s functional but not ideal — I want to serialize when exiting Play Mode instead. Moving that logic into the editor callbacks is next.

The output looks like this:
    
```json

{
  "session": {
    "startTime": "2025-12-21T22:14:05.1234567Z",
    "endTime": "2025-12-21T22:16:42.9876543Z",
    "durationSeconds": 157.864,
    "unityVersion": "2022.3.12f1",
    "gameEventsVersion": "1.2.0",
    "projectName": "MyCoolGame"
  },

  "events": [
    {
      "eventId": "PlayerJump",
      "createdAt": "2025-12-21T22:14:05.1234567Z",
      "totalInvocations": 42,
      "uniqueListeners": 3
    }
  ],

  "invocations": [
    {
      "eventId": "PlayerJump",
      "timestamp": "2025-12-21T22:14:06.0001234Z",
      "listenerCount": 2,
      "invocationIndex": 0
    }
  ],

  "listeners": [
    {
      "listenerId": "PlayerController",
      "eventId": "PlayerJump",
      "registeredAt": "2025-12-21T22:14:05.2000000Z",
      "unregisteredAt": "2025-12-21T22:16:00.0000000Z",
      "lifetimeSeconds": 114.8
    }
  ]
}


```
It’s basic, but already enough to start seeing patterns in how events flow.

## Next Steps

There’s a lot of room to grow from here. My short list:

Visualize event activity in real time
(think timeline graphs or heatmaps per event)

Track listener lifetime accurately
Domain reloads and prefab listeners are messy edge cases.

Handle Play Mode boundaries cleanly
Data should finalize only once per session.

Later on, I’d like to experiment with aggregated session data — maybe show which systems communicate most, or even detect “dead” events that never fire.

## Final Thoughts

What started as curiosity turned into one of the most surprisingly useful debugging tools I’ve built. Just like the Enigma analysts learned from message patterns, I’m realizing how much you can learn from how a Unity project talks to itself.

GameEvents Developer Metrics is still early, but it’s already reshaping how I think about visibility in event-driven systems — not just in Unity, but across everything I build.