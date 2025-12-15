---
title: "My Game Engine Development: A Three-Year Journey"
author: Silas
date: 2025-12-12 21:18:17 -0800
categories: [Game Development, Engine Architecture, Software Modernization, C++ Programming, Graphics Programming]
tags: [Game Development, Engine Architecture, Software Modernization, C++ Programming]
render_with_liquid: false
---

<div style="text-align:center; margin-bottom: 1.5rem;">
  <iframe width="560" height="315"
  src="https://www.youtube.com/embed/V6lfcq1Xk7g"
  title="Three Years of Engine Work"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen></iframe>
</div>


## Year 1: SFML Beginnings

My journey started in the classroom, with a 2D engine and a simple **Space Invaders clone** built in SFML. It was my first real attempt at designing an engine, and while the scope was small, it gave me a taste of how rendering, input, and game logic could come together.

You can see the project here:  
[Star Wars Space Invaders Clone (SFML)](https://github.com/Shafeli/Star-Wars-Space-invaders-clone)

The code is rough but it has good bones. It taught me the fundamentals of game loops, entity management, and rendering pipelines, and it gave me the confidence to tackle bigger challenges in later semesters.

---

## Year 2: SDL Rewrite and Architectural Lessons

The following semester, I enrolled in a **Game Engine Architecture** course. Almost immediately, I realized I had to throw out two systems I thought I had “finished”: **rendering and input**. The course required using **SDL2**, which meant rewriting both subsystems from scratch.

At the time, it felt like a major setback. In hindsight, it was one of the most valuable lessons of the entire project.

Rewriting these systems forced me to think about **clean system boundaries and modularity** rather than hard-coding functionality directly into the engine. I spent a significant amount of time studying **C++17 templates and containers** which led to the creation of a **type-agnostic subsystem manager** that became the backbone of the engine.

I reused this pattern across multiple areas:
- Core engine systems
- An ECS-style container
- etc...

This approach gave the engine room to grow without collapsing under its own complexity.

This semester was intense — I was regularly putting in **40+ hour weeks**, attending instructor office hours, and dedicating nearly all of my spare time to the engine. It became the largest and most demanding project I had ever worked on, and through that grind I learned how to balance persistence with architectural clarity.

You can explore the code here:  
[BrokkrEngine (SDL2)](https://github.com/Shafeli/BrokkrEngine)

### Core System Management Example

```cpp
namespace Brokkr
{
    class CoreSystems
    {
        inline static Logger m_fileLog{ "CoreSystemLog" };
        std::vector<std::unique_ptr<CoreSubsystem>> m_pCoreSubsystems;

    public:

        bool Init(const char* pGameTitle, int screenWidth, int screenHeight); // For when system is building

        // For init() of runtime systems ...TODO: Tagged for re-design
        template <typename System, typename... Args>
        void RuntimeInit(Args&&... args)
        {
            // Initialize each with the given arguments
            for (auto& system : m_pCoreSubsystems)
            {
                if (System* targetSystem = dynamic_cast<System*>(system.get()))
                {
                    targetSystem->Init(std::forward<Args>(args)...);
                }
            }
        }

        template <typename CoreSubsystem>
        CoreSubsystem* GetCoreSystem()
        {
            // Iterate through all components in the vector
            for (auto& subsystem : m_pCoreSubsystems)
            {
                // If the cast is successful
                if (CoreSubsystem* target = dynamic_cast<CoreSubsystem*>(subsystem.get()))
                {
                    return target;
                }
            }

            const std::string error = "Error Core System Failed to : Get a Subsystem!";
            m_fileLog.Log(Logger::LogLevel::kError, error);
            return nullptr; // If no system of type is found
        }

        template <typename CoreSubsystem, typename ... Args>
        CoreSubsystem* AddCoreSystem(Args&&... args)
        {
            std::unique_ptr<CoreSubsystem> newCoreSubsystem = std::make_unique<CoreSubsystem>(this, std::forward<Args>(args)...);
            newCoreSubsystem->AddRef(); // increment the reference count

            CoreSubsystem* result = newCoreSubsystem.get(); // Get a raw pointer to the component
            m_pCoreSubsystems.emplace_back(std::move(newCoreSubsystem)); // Add the system to the vector

            if (!result)
            {
                const std::string error = "Error Core System Failed to : Construct a Subsystem!";
                m_fileLog.Log(Logger::LogLevel::kError, error);
            }
            return result; // Return a pointer
        }

        template<typename CoreSubsystem>
        void RemoveCoreSystem(CoreSubsystem* System)
        {
            // Iterate through all components in the vector
            for (size_t i = 0; i < m_pCoreSubsystems.size(); ++i)
            {
                // If the cast is successful
                if (CoreSubsystem* target = dynamic_cast<CoreSubsystem*>(m_pCoreSubsystems[i].get()))
                {
                    if (target == System) // If the found type is the pointer we already have then
                    {
                        const std::string error = "Core System : Removed a Subsystem!";
                        m_fileLog.Log(Logger::LogLevel::kInfo, error);

                        const int remainingRefs = target->Release(); // relase returns a number == the remaining Refs to this system

                        if (remainingRefs <= 0) // If no more ref remove the ptr
                        {
                            target->Destroy();
                            // Swap and pop the element at index i
                            std::swap(m_pCoreSubsystems[i], m_pCoreSubsystems.back());
                            m_pCoreSubsystems.pop_back();
                            break; // Stop searching
                        }
                    }
                }
            }
        }
    };
}
```
## Engine Core Systems

Before attempting another game project, I focused on improving the **engine itself**. Since I had invested so much effort into making systems modular and plug‑in friendly, this was the perfect time to refactor and expand.

- **Event Manager**  
  Built a centralized event manager with payload support. After learning more about plug‑in architecture, this system became a common communication layer across the engine. Out of all the systems I built during this phase, it was without a doubt my favorite to work on — and I think that shows in the design.

You can explore the code here:  
[BrokkrEngine (EventManager)](https://github.com/Shafeli/BrokkrEngine/tree/main/BrokkrEngine/EventManager)

```cpp
    namespace Brokkr
{

    class EventManager final : public CoreSubsystem
    {
    public:
        // event handler functions, including a priority value
        using EventHandler = std::pair<int, std::function<void(const Event&)>>;

        // Comparison functor for event handler functions
        struct EventHandlerComparer
        {
            // takes two EventHandler objects and returns a bool
            bool operator()(const EventHandler& left, const EventHandler& right) const;
        };

        // Comparison functor for events based on their enum value
        struct EventComparer
        {
            // Definition of operator() that takes two Event objects and returns a bool
            bool operator()(const Event& left, const Event& right) const;
        };

    private:
        // Map of event types to a set of event handlers, sorted using the custom comparator
        std::unordered_map<uint32_t, std::set<EventHandler, EventHandlerComparer>> m_handlers;

        // Priority queue of events waiting to be processed, sorted by their enum value
        std::priority_queue<Event, std::vector<Event>, EventComparer> m_eventQueue;

    public:
        explicit EventManager(CoreSystems* pCoreManager): CoreSubsystem(pCoreManager) { }

        // Add a handler for an event by string
        void AddHandler(const char* eventTypeString, const EventHandler& handler);

        // Remove a handler for an event by string
        void RemoveHandler(const char* eventTypeString, const EventHandler& handler);

        // Add an event handler for a given event type with a specified priority
        void AddHandler(uint32_t eventHash, const EventHandler& handler);

        // Remove an event handler for a given event type
        void RemoveHandler(uint32_t eventHash, const EventHandler& handler);

        // Add an event to the event queue
        void PushEvent(const Event& event);

        // Process all events currently in the event queue
        void ProcessEvents();

        virtual void Destroy() override;
    };
}
```
## GameObject ECS System

Another derivative of the work I did in **CoreSystems** was the **GameObject ECS System**. Every `GameEntity` is essentially a container of components — pretty standard by now — but the way I structured it made creating and managing game objects much easier for me a solo dev.  

By reusing the subsystem patterns, I was able to design entities that could dynamically attach, detach, enable, disable, and serialize components. Later, when I integrated the XML parser, this system made adding new components incredibly fast and flexible. It became the backbone of how gameplay objects were defined and extended.

[BrokkrEngine (GameEntity)](https://github.com/Shafeli/BrokkrEngine/blob/main/BrokkrEngine/GameEntity)

```cpp
namespace Brokkr
{
    class GameEntity
    {
        int m_id;
        std::vector<std::unique_ptr<Component>> m_pComponents;
    public:
        GameEntity() : m_id(IDGenerator::GenerateUniqueID()){}
        ~GameEntity();

        [[nodiscard]] int GetId() const { return m_id; }

        void Update() const;
        void Render() const;

        template<typename ComponentType>
        ComponentType* GetComponent();

        template<typename ComponentType>
        void CallAttachOnComponent();

        template<typename ComponentType>
        void CallDetachOnComponent();

        template<typename ComponentType>
        void CallEnableOnComponent();

        template<typename ComponentType>
        void CallDisableOnComponent();

        template<typename ComponentType, typename... Args>
        ComponentType* AddComponent(Args&&... args);
    };

    template <typename ComponentType>
    ComponentType* GameEntity::GetComponent()
    {
        for (auto& component : m_pComponents)
        {
            if (auto* target = dynamic_cast<ComponentType*>(component.get()))
                return target;
        }
        return nullptr;
    }

    template <typename ComponentType, typename ... Args>
    ComponentType* GameEntity::AddComponent(Args&&... args)
    {
        auto newComponent = std::make_unique<ComponentType>(this, std::forward<Args>(args)...);
        ComponentType* result = newComponent.get();
        m_pComponents.emplace_back(std::move(newComponent));
        return result;
    }
}
```
This ECS design gave me a clean way to manage game entities and their behaviors. Combined with the XML parsing system, it allowed me to define entities declaratively and extend them without touching core engine code — a huge productivity boost when building out gameplay later.

## Asset Management and XML Parsing

One of the unusual design decisions I made was to embed the **XML parser** directly into the `AssetManagerSystem` rather than making it a standalone subsystem. At the time, it felt natural to group everything under one umbrella, but looking back I would never build another asset manager like this.  

The **AssetManager** became massive — a home for things like the scripting manager, XML parsing, and texture handling. Many of these subsystems could have (and probably should have) been independent. It’s honestly a wonder I never had time to add proper sound or music support, given how much the asset manager absorbed.

[BrokkrEngine (AssetManager)](https://github.com/Shafeli/BrokkrEngine/tree/main/BrokkrEngine)

```cpp
#pragma once
#include <memory>
#include <vector>
#include "../Application/CoreSystems/CoreSystems.h"
#include "../Application/CoreSystems/CoreSubsystem/CoreSubsystem.h"
#include "../Logging/Logger.h"

namespace Brokkr
{
    class AssetManager;

    class AssetSubsystem
    {
    public:

        AssetSubsystem(AssetManager* assetManager) : m_pAssetManager(assetManager) {}
        virtual ~AssetSubsystem() = default;

        virtual void Destroy() = 0;

    protected:
        AssetManager* m_pAssetManager;
    };

    class CoreSystems;

    class AssetManager final : public CoreSubsystem
    {
        inline static Logger m_fileLog{ "AssetSystemManagerLog" };
        std::vector<std::unique_ptr<AssetSubsystem>> m_pAssetSystems{ };

    public:
        explicit AssetManager(CoreSystems* engineSystems) : CoreSubsystem(engineSystems) { }
        void Init();

        template <typename AssetSystem>
        AssetSystem* GetAssetSystem()
        {
            // Iterate through all components in the vector
            for (auto& component : m_pAssetSystems)
            {
                // If the cast is successful
                if (AssetSystem* target = dynamic_cast<AssetSystem*>(component.get()))
                {
                    return target;
                }
            }
            return nullptr; // If no system of type is found
        }

        template <typename AssetSystem, typename ... Args>
        AssetSystem* AddAssetSystem(Args&&... args)
        {
            // Create a instance of the component type passing in the current AssetSystem pointer
            std::unique_ptr<AssetSystem> newComponent = std::make_unique<AssetSystem>(this, std::forward<Args>(args)...);

            AssetSystem* result = newComponent.get(); // Get a raw pointer to the component

            m_pAssetSystems.emplace_back(std::move(newComponent)); // Add the system to the vector 
            return result; // Return a pointer
        }

        virtual void Destroy() override;
    };

}
```
This design gave me a centralized way to manage assets and data pipelines, but it also highlighted the trade‑offs of overloading a single system. While it worked, it reinforced the importance of separation of concerns — a lesson I carried forward into later iterations of the engine.

- **XML Integration**
    When making the design choice to embed XML parsing into the asset manager, I leaned on the external library **tinyXML2** to save time instead of writing my own parser. At the time, it felt natural to treat XML as just another “asset” rather than a core engine system. In hindsight, this was a questionable choice — it led to a lot of pointer traversal and caching workarounds. Still, because nearly everything in the engine was component‑based, the design wasn’t too limiting.

[BrokkrEngine (XMLManager)](https://github.com/Shafeli/BrokkrEngine/tree/main/BrokkrEngine/BrokkrEngine/AssetManager/XMLManager)
```cpp
#pragma once
#include <memory>
#include <vector>
#include "../../AssetManager/AssetManager.h"

namespace tinyxml2
{
    class XMLDocument;
}

namespace Brokkr
{
    class CoreSystems;

    class XMLParser 
    {
    public:
        XMLParser(CoreSystems* pCoreSystems)
            :m_pCoreSystems(pCoreSystems)
        {
            //
        }
        virtual ~XMLParser() = default;
        virtual bool Parse(tinyxml2::XMLDocument& doc) = 0;

    protected:
        CoreSystems* m_pCoreSystems = nullptr;
        inline static Logger m_log{ "XMLParserLog" };
    };

    class XMLManager final : public AssetSubsystem
    {
        CoreSystems* m_pCoreSystems = nullptr;
        std::vector<std::unique_ptr<XMLParser>> m_parsers;

    public:
        XMLManager(AssetManager* assetManager) : AssetSubsystem(assetManager) {}
        void Init(CoreSystems* coreSystems);
        void Load(const char* fileName) const;
        virtual void Destroy() override;

        template <typename ParserType, typename ... Args>
        ParserType* AddParser(Args&&... args)
        {
            // Create a instance of the Parser Type
            std::unique_ptr<ParserType> newParserType = std::make_unique<ParserType>(std::forward<Args>(args)...);

            // Get a raw pointer to the parser
            ParserType* result = newParserType.get();

            // Add the parser to the vector
            m_parsers.emplace_back(std::move(newParserType));

            // Return a pointer
            return result;
        }

        template <typename ParserType>
        ParserType* GetParser()
        {
            // Iterate through all components in the vector
            for (auto& parser : m_parsers)
            {
                // If the cast is successful
                if (ParserType* target = dynamic_cast<ParserType*>(parser.get()))
                {
                    return target;
                }
            }

            // If no type is found
            return nullptr;
        }
    };
}
```
- **Prefab System**
    The prefab system in the engine was little more than an XML parser and at the time, that was perfect. With the tight development schedule it did exactly what it needed to.

    If I had more time I would have loved to expand it with an abstract factory so components could register their own creation methods. That would have reduced recompiles and made adding new components much cleaner.

    Even with its limitations, this was the little system that could. Without it, creating new entities would have been a slow, manual, and error-prone process. Since it mostly loaded data during scene transitions, it never needed to be overly complex, just reliable enough to keep production moving.

[BrokkrEngine (EntityXMLParser)](https://github.com/Shafeli/BrokkrEngine/tree/main/BrokkrEngine/BrokkrEngine/AssetManager/XMLManager/Parsers/EntityXMLParser)

```cpp
    
bool Brokkr::EntityXMLParser::Parse(tinyxml2::XMLDocument& doc)
{
    auto* root = doc.RootElement();
    if (!root || strcmp(root->Name(), "Entity") != 0)
        return false;

    for (auto* entity = root->FirstChildElement("GameEntity");
         entity; entity = entity->NextSiblingElement("GameEntity"))
    {
        GameEntity* pEntity = 
            m_pCoreSystems->GetCoreSystem<GameEntityManager>()->GetNextEntityAvailable();

        for (auto* comp = entity->FirstChildElement(); comp; comp = comp->NextSiblingElement())
        {
            std::string name = comp->Name();

            if (name == "SpriteComponent")
                pEntity->AddComponent<SpriteComponent>(comp->Attribute("texture"), m_pCoreSystems);

            else if (name == "TransformComponent")
                pEntity->AddComponent<TransformComponent>(
                    m_pCoreSystems,
                    Brokkr::Rect<float>(
                        { comp->FloatAttribute("x"), comp->FloatAttribute("y") },
                        { comp->FloatAttribute("width"), comp->FloatAttribute("height") }));

            else if (name == "ColliderComponent")
                BuildColliderComponent(pEntity, comp);

            else if (name == "TriggerComponent")
                BuildTriggerComponent(pEntity, comp);

            // other components as needed...
        }
        pEntity->Init();
    }
    return true;
}
```

Because of this system, I was able to build complete gameplay objects like Pac-Man and pretty much anything else directly from XML. Entities, components, and behaviors could all be defined without touching a single line of engine code. 

```xml
    <Entity>

    <GameEntity name="Texture Wall">
        <TransformComponent x="460" y="100" height="50" width="50" />
        <SpriteComponent texture="Test" />
        <ColliderComponent overlap="dynamic" passable="no"/>
        <HealthComponent />
    </GameEntity>

    <GameEntity name="Random Color Change wall">
        <TransformComponent x="400" y="100" height="50" width="50" />
        <SpriteComponent texture="Square" />
        <ColliderComponent overlap="dynamic" passable="no"/>
        <RandomColorComponent />
    </GameEntity>

    <GameEntity name="Random Color Change Pad">
        <TransformComponent x="340" y="100" height="50" width="50" />
        <SpriteComponent texture="Square" />
        <DefaultControllerComponent up="Up" down="Down" left="Left" right="Right"/>
        <ColliderComponent overlap="dynamic" passable="yes"/>
        <HealthComponent/>
        <RandomColorComponent />
    </GameEntity>

    <GameEntity name="Trap">
        <TransformComponent x="550" y="100" width="25" height="25" />
        <SpriteComponent texture="Square"/>
        <RandomColorComponent />
        <TriggerComponent x="550" y="100" width="25" height="25" eventStr="Kill" target="overlap"/>
    </GameEntity>

    <GameEntity name="Hero">
        <TransformComponent x="400" y="170" height="50" width="50" />
        <SpriteComponent texture="Square" />
        <DefaultControllerComponent up='W' down="S" left="A" right="D"/>
        <ColliderComponent overlap="dynamic" passable="yes"/>
        <RandomColorComponent />
    </GameEntity>

    </Entity>
```

## Physics System  

The physics system was meant to be the center point for all movement in the engine, but I **underestimated how complex that really is**. Rolling your own collision and movement logic is a full-time job, and I learned that the hard way.  

Thankfully, I had instructors to pester at all hours and a very active student Discord where we all suffered through the same problems together. Between that and a heavy dose of **Handmade Hero** and **Handmade Con** talks (Casey Muratori was a huge influence — I grew up modding *Dungeon Siege* and it felt full-circle), I absorbed everything I could about low-level systems.  

### System Design  

My goal was to make the physics layer feel like an ecosystem:  
- Components would request a **collider handle**.  
- When an object wanted to move, it would pass the collider pointer and desired `Vector2` displacement to the physics system.  
- The system would then handle collision detection and correction automatically.  

That last part — *“handle the rest”* — turned out to be the mountain.  

I had to create custom math primitives (`Vector2`, `Rect`) and then build a full **collision-detection pipeline**. After taking a low-level optimization class that covered spatial acceleration, I wanted to implement a **quadtree** instead of a simple cell grid. The goal was faster collision checks and, later, the ability to run spatial queries for gameplay logic:  

  ```cpp
        [[nodiscard]] std::vector<ObjectID> QueryAreaDynamics(const Rect<float>& area) const;
        [[nodiscard]] std::vector<ObjectID> QueryAreaStatics(const Rect<float>& area) const;
        [[nodiscard]] std::vector<ObjectID> QueryAreaAll(const Rect<float>& area) const;
  ```
### Challenges & Solutions  

My first version of the quadtree had no limit on depth — the tree just kept subdividing until everything broke.  
After reading more, I learned the two common approaches:  
1. Cap the minimum node size to the smallest collider dimension.  
2. Cap the **tree depth** itself.  

I went with the second option.  

Another early mistake was treating the quadtree like a **cell-based grid**, constantly removing and reinserting colliders each frame. After a lot of trial and error, I realized I was just trying to rebuild the dynamic collider data every tick. The solution was simple but effective:  
- Maintain **two trees** — one for static colliders, one for dynamic.  
- Rebuild only the dynamic tree each frame while keeping the static tree persistent.  

It wasn’t elegant, but it worked — and I built it in roughly two weeks under a brutal deadline. That project taught me how to recognize when a system needs to be “good enough for now” instead of perfect.  

[BrokkrEngine (PhysicsManager)](https://github.com/Shafeli/BrokkrEngine/tree/main/BrokkrEngine/BrokkrEngine/PhysicsManager)
  ```cpp
 namespace Brokkr
{
  class Collider 
  {
    Rect<float> m_collider;
    std::vector<Vector2<float>> m_displacements;
    std::vector<Vector2<float>> m_corrections;
    int m_ownerID = -1;
    bool m_moveable = false;

    void Update(const Event& event) 
    {
        if (!m_moveable) return;

        for (auto& d : m_displacements)
        { 
            m_collider.Adjust(d); 
        }

        for (auto& c : m_corrections)
        { 
            m_collider.Adjust(c);
        }
        m_displacements.clear();
        m_corrections.clear();
    }
};

class PhysicsManager : public CoreSubsystem 
{
    QuadTree m_staticColliderRoot;
    QuadTree m_dynamicColliderRoot;
    void RefreshDynamicTree();
    void RefreshStaticTree();
    std::vector<ObjectID> QueryAreaAll(const Rect<float>& area) const;
};
}
  ```

### Reflection  

This system taught me more about **scoping, iteration, and algorithm design** than any other part of the engine. It forced me to think in terms of data ownership, spatial partitioning, and performance trade-offs — skills that carried directly into how I design systems today.  

## Lua Scripting Integration  

The scripting system I implemented used **Lua** — not entirely by choice, but because it was a project requirement. That said, I already had some experience with Lua from earlier work: building a small game in **LÖVE2D** and experimenting with **World of Warcraft UI mods**.  

Those experiences gave me a practical feel for Lua as a scripting language. I understood how it worked from the *modder’s* side — how scripts interact with a host application — and that perspective shaped how I approached integrating it into my own engine.  

One decision I made early on was **not** to use a DLL for Lua. Instead, I built it directly into the solution as a native project. The goal was to eventually **customize and strip** parts of the Lua runtime — just like Blizzard does. (For example, they disabled `require()` in the WoW client to sandbox scripts — a fascinating design choice I wanted to explore.)  

### System Design  

The engine’s Lua system was made up of three main parts:  
1. **Script Asset Object** — Modeled Lua scripts as asset types within the asset manager. This made runtime compilation and reloading possible later on.  
2. **Script System (Core Subsystem)** — Integrated Lua into the core system architecture, paving the way for potentially adding other scripting languages in the future.  
3. **Lua System Implementation** — Managed the Lua state, stack, and C++/Lua type conversions through template specializations.  

One cool side effect of how modular my engine was: Lua could be **added or removed without breaking anything else**.  
Because it was a self-contained subsystem, you could literally delete the Lua files, recompile the engine, and everything would still run — like ripping out the oil pump and having the car start anyway. Not practical, but pretty funny to see work.  

Here’s a small snippet from the utility registration layer — where C++ functions were pushed into Lua for runtime access:  

```cpp
void Brokkr::LuaSystem::LoadUtilities()
{
    // Register a C++ function with Lua that maybe considered basic like logging

    auto BrokkrLogDebug = [this](lua_State* L) -> int
    {
        const char* message = luaL_checkstring(L, 1); // Retrieve the string

        m_fileLog.Log(Logger::LogLevel::kDebug, message); // Log the custom message
        return 1;
    };

    PushCFunction(m_pLuaState, BrokkrLogDebug, "BrokkrLogDebug");

    auto BrokkrLogInfo = [this](lua_State* L) -> int
    {
        const char* message = luaL_checkstring(L, 1); // Retrieve the string

        m_fileLog.Log(Logger::LogLevel::kInfo, message); // Log the custom message
        return 1;
    };

    PushCFunction(m_pLuaState , BrokkrLogInfo, "BrokkrLogInfo");

    auto BrokkrLogWarning = [this](lua_State* L) -> int
    {
        const char* message = luaL_checkstring(L, 1); // Retrieve the string

        m_fileLog.Log(Logger::LogLevel::kWarning, message); // Log the custom message
        return 1;
    };

    PushCFunction(m_pLuaState, BrokkrLogWarning, "BrokkrLogWarning");

    auto BrokkrLogError = [this](lua_State* L) -> int
    {
        const char* message = luaL_checkstring(L, 1); // Retrieve the string

        m_fileLog.Log(Logger::LogLevel::kError, message); // Log the custom message
        return 1;
    };

    PushCFunction(m_pLuaState, BrokkrLogError, "BrokkrLogError");
}
```
This was a huge undertaking and one of the most educational parts of the engine. It forced me to deeply understand stack management, type specialization, and the boundaries between host and script execution.

You can check out the Lua system code here:

[BrokkrEngine (ScriptAssets)](https://github.com/Shafeli/BrokkrEngine/tree/main/BrokkrEngine/BrokkrEngine/AssetManager/ScriptManager/ScriptAssets)
[BrokkrEngine (ScriptSystem)](https://github.com/Shafeli/BrokkrEngine/tree/main/BrokkrEngine/BrokkrEngine/ScriptSystemManager/ScriptSystem)
```cpp
namespace Brokkr
{
    class CoreSystems;
    class ScriptAssetLoader;

    class LuaSystem final : public ScriptSystem
    {
        inline static Logger m_fileLog{ "LuaSystemLog" };

        lua_State* m_pLuaState = nullptr;
        ScriptAssetLoader* m_scriptAssets = nullptr;

        std::vector<std::unique_ptr<std::function<int(lua_State*)>>> m_functions;

    public:
        explicit LuaSystem(ScriptSystemManager* pSystemManager, CoreSystems* pCoreSystems);

        virtual ~LuaSystem() override;
        virtual void Init() override;
        virtual void Destroy() override;

        //TODO: Still need to add array and table support for lua
        // Get / SetArrayElement(...);
        // Get / SetTableElement(...);

        [[nodiscard]] lua_State* GetActiveState()
        {
            if (!m_pLuaState)
                Init();

            return m_pLuaState;
        } // currently not sure if there's a benefit to having more then one state

        // Getting Variable
        template<typename Type>
        Type GetGlobalVariable(lua_State* pLuaState, const std::string& name);

        // Template specializations
        template <>
        int GetGlobalVariable<int>(lua_State* pLuaState, const std::string& name);

        template <>
        bool GetGlobalVariable<bool>(lua_State* pLuaState, const std::string& name);

        template <>
        std::string GetGlobalVariable<std::string>(lua_State* pLuaState, const std::string& name);

        template <>
        float GetGlobalVariable<float>(lua_State* pLuaState, const std::string& name);

        // Setting Variable
        template<typename Type>
        void SetGlobalVariable(const std::string& name, const Type& value);

        // Template specializations
        template <>
        void SetGlobalVariable<int>(const std::string& name, const int& value);

        template <>
        void SetGlobalVariable<bool>(const std::string& name, const bool& value);

        template <>
        void SetGlobalVariable<std::string>(const std::string& name, const std::string& value);

        template <>
        void SetGlobalVariable<float>(const std::string& name, const float& value);

        // PushCFunction(...); // Push a C function so Lua scripts can call it. std::fuction object
        void PushCFunction(lua_State* pLuaState, const std::function<int(lua_State*)>& func, const std::string& funcName);

        // PushCFunction(...); // Push a C function so Lua scripts can call it. pointer no args
        template <class FuncType>
        void PushCFunction(lua_State* pLuaState, FuncType func, const std::string& funcName);

        // PushCFunction(...); // Push a C function so Lua scripts can call it. pointer with args
        template <class FuncType, class ... Args>
        void PushCFunction(lua_State* pLuaState, FuncType func, const std::string& funcName, Args&& ... args);

        bool ExecuteScript(const char* scriptName);
        [[nodiscard]] std::vector<std::string> GetScriptAssetNames() const;

        template<typename... Args>
        void CallLuaVoidFunction(lua_State* pLuaState, const char* functionName, Args... args);

        // Template specializations
        template<>
        void CallLuaVoidFunction(lua_State* pLuaState, const char* functionName);

        template <typename ReturnType, typename... Args>
        ReturnType CallLuaFunctionWithReturn(lua_State* pLuaState, const char* functionName, Args... args);

    private:
        void LoadUtilities();

        void DumpStack();
    };
}
```
Integrating Lua wasn’t just a technical exercise as it completely changed how I thought about data flow, scripting boundaries, and live iteration.
Compared to building games in LÖVE2D or modding World of Warcraft, writing a scripting layer from scratch forced me to understand what actually happens underneath those systems.

It ended up being one of the hardest and most rewarding systems I’ve ever built — not because of how polished or critical it was (it honestly didn’t see much use), but because of how much I learned while building it.

## Tiled Integration (Sort Of)
I used Tiled more like a lightweight level editor than a full TMX integration.
All I really cared about was the layer name (to decide which prefab type to spawn) and each tile’s position and size from the CSV data. That was enough to load levels visually while keeping the workflow minimal.

The long-term goal was to replace Tiled entirely with my own IMGUI-based editor, but it worked so well that it stayed in the pipeline much longer than I expected. At the end of the day, the entire integration boiled down to a single XML parser that read map data directly into the engine’s runtime.

[BrokkrEngine (MapXMLParser)](https://github.com/Shafeli/BrokkrEngine/tree/main/BrokkrEngine/BrokkrEngine/AssetManager/XMLManager/Parsers/MapXMLParser)
```cpp
bool Brokkr::MapXMLParser::Parse(tinyxml2::XMLDocument& doc)
{
    auto* root = doc.RootElement();
    if (!root || strcmp(root->Name(), "map") != 0)
        return false;

    const int tileWidth  = root->IntAttribute("tilewidth");
    const int tileHeight = root->IntAttribute("tileheight");
    const int mapWidth   = root->IntAttribute("width");
    const int mapHeight  = root->IntAttribute("height");

    auto* data = root->FirstChildElement("layer")->FirstChildElement("data");
    if (!data)
        return false;

    // Parse CSV tile data
    std::istringstream stream(data->GetText());
    std::vector<int> tiles;
    std::string value;
    while (std::getline(stream, value, ','))
        tiles.push_back(std::stoi(value));

    // Configure world bounds
    m_pCoreSystems->GetCoreSystem<PhysicsManager>()
        ->SetWorldSize({ mapWidth * tileWidth, mapHeight * tileHeight });

    // Build world tiles
    auto worldTileManager = m_pCoreSystems->GetCoreSystem<WorldTileManager>();
    for (int y = 0; y < mapHeight; ++y)
    for (int x = 0; x < mapWidth; ++x)
    {
        int tile = tiles[y * mapWidth + x];
        if (tile == 0) continue;

        auto tileEntity = worldTileManager->GetAvailable();
        auto transform  = tileEntity->AddComponent<TileTransformComponent>(tileEntity, m_pCoreSystems);
        transform->SetPosition({ x * tileWidth, y * tileHeight });
        transform->SetSize({ tileWidth, tileHeight });

        auto renderer = tileEntity->AddComponent<TileRenderComponent>(tileEntity, m_pCoreSystems);
        renderer->SetTexture("Test");
        tileEntity->Init();
    }

    return true;
}
```

Together, these systems transformed the engine from a course project into a practical, extensible development tool; something I could actually build games with, not just study.

---

## Year 3: Real Use of The Engine Pac-Man

After all the system work I’d done on the engine, the real payoff came in the following year when I finally had to build a full game with it.

I chose Pac-Man, and it ended up being surprisingly straightforward — just a handful of components, a few game systems, and some core engine pieces tied together. It was the first time the modular design philosophy I’d been chasing actually paid off: adding new mechanics or tweaking behaviors was fast and clean.

For layout, I used Tiled again to layer object positions, exported the data, and loaded it straight into the engine. Since the system was built to support both Lua and C++ scripting, I stayed in C++ — I was most comfortable there, and Lua support was still minimal.

Here’s the main function that bootstrapped the game:

```cpp
int main()
{
    Brokkr::ApplicationManager application;

    application.Init("Pac Man", 588, 768);

    auto engineSystems = application.GetEngineSystems();

    auto sceneManager = engineSystems->GetCoreSystem<Brokkr::SceneManager>();
    sceneManager->AddState("PacManGame", std::make_unique<PacManGameScene>(engineSystems));
    sceneManager->SetActiveState("PacManGame");

    auto scriptLoader = engineSystems
        ->GetCoreSystem<Brokkr::AssetManager>()
        ->GetAssetSystem<Brokkr::ScriptAssetLoader>();

    scriptLoader->AddAssetsFromFolder<Brokkr::LuaScript>(
        "../BrokkrEngine/assets/Scripts/LuaScripts", ".lua"
    );

    application.Run();
    application.Clean();

    return 0;
}
```

It’s not the most complex game, but it proved something more important — the engine wasn’t just theoretical anymore. It could actually ship something playable.