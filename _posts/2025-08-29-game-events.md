---
title: "Game Events"
author: Silas
date: 2025-08-29 22:57:46 -0700
categories: [Unity  Programming]
tags: [Unity]
render_with_liquid: false
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/kJwX7HVlFsY" frameborder="0" allowfullscreen></iframe>

# noCTRL Game Events

**Performant, extensible in-game event system for Unity.**  
Create events as assets, reference them anywhere in your project, and decouple your game logic without relying on hard-coded dependencies.

---

## ✨ Features
- **ScriptableObject-based events** for easy reusability and scene independence.
- **Strong decoupling** between event senders and listeners.
- **Lightweight & performant** runtime.
- **Custom Editor tools** for debugging and event management.
- **Sample scenes** to get started quickly.
- Fully compatible with **Unity 2021.3+**.

---

## 📜 Crediting noCTRL Studios
If you use this package, you must credit **noCTRL Studios** and include a link to its GitHub repository:  
[https://github.com/noCTRL-Studios](https://github.com/noCTRL-Studios)  
Include this in your project documentation, README, or game credits. Just as long as it is some place! 

---

## 📦 Installation

### Unity Package Manager (Git URL)
1. Open Unity → **Window** → **Package Manager**.
2. Click the **+** button → **Add package from git URL…**
3. Paste:
    ```text
    https://github.com/noCTRL-Studios/Unity-Game-Events.git
    ```

---

## 🚀 Quick Start

### 1. Create an Event Asset
1. Right-click in the **Project** window.  
2. Select **Create → Game Events → New Game Event**.  
3. Name it (e.g., `OnCoinCollected`).  

### 2. Raise an Event in Code
```csharp
using UnityEngine;

public class Coin : MonoBehaviour
{
    public GameEvent OnCoinCollected;

    public void Collect()
    {
        OnCoinCollected.RaiseAll();
        Destroy(gameObject);
    }
}
```

### 3. Listen for the Event
Use the **GameEvent Listener** component:  
1. Add the listener component to a GameObject.  
2. Drag your **GameEvent** asset into the component’s Event field.  
3. Select a response from the available prefab functions.  

---

## 🧪 Sample Scene
A ready-to-use example scene is included in:
```
Samples~/Basic Usage
```
Demonstrates:
- Creating and raising events
- Listening and responding to events
- Debugging events in the Inspector

---

## 📂 Project Structure
```
Runtime/       # Core runtime scripts (events, listeners)
Editor/        # Custom editors for workflow improvements
Samples~/      # Example scenes and scripts
Tests/         # Optional automated tests
```

---

## 👤 Author
**noCTRL Studios** – Tools and SDKs 
📧 silashafeli@noctrlstudios.com  
🌐 [https://noctrlstudios.com](https://noctrlstudios.com)