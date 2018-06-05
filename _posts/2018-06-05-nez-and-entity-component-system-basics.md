---
layout: post
date: 2018-06-05
category: devblog
tags: [ Nez, C#, gamedev ]
comments: true
---

This tutorial will cover Entity Component System (ECS) basics, and how Nez fits into it's design.

Full disclosure before wading into this tutorial: My understanding of ECS can be described, at best, as amateur, but I understand enough to implement it. This tutorial will describe my personal approach to ECS, and how it ties in with Nez.

ECS should be considered a guideline and Object Oriented Programming (OOP) principles can still be applied. Each programmer will find a balance that best works for them, and tutorials like this should not be seen as a definitive approach.

## Entity Component System (ECS) Basics

In a sentence, an `Entity` is a container of components, a `Component` is a container for data, and a `System` acts upon entities with specific components. There's a fantastic [key and lock analogy on Stack Exchange](https://gamedev.stackexchange.com/a/31491) that should help explain these concepts.

### Components

Components typically consists only of data, and do not contain methods for acting upon that data. They can be thought of as similar to C structs. However, I believe that simple methods for accessing, updating, and manipulating data within components are fine. 

Some examples could be:

A `Position` component containing an X and Y value, which references its position within a 2D world.

A `Sprite` component containing the texture needed to render an image to the screen.

A `CombatStats` component which may contain health, strength, and defense values. It could also have simple getters and setters, or a helper for applying damage. It could look like this:

```csharp
class CombatStats : Nez.Component {

    int _health, _maxHealth;
    public int Health {
        get { return _health; }
        set {
            // Ensure health is set between 0 and maxHealth
            _health = value > 0 ? value : 0;
            if (_health > _maxHealth)
                _health = _maxHealth;
        }
    }

    int _strength;
    public int Strength {
        get { return _strength; }
        set { _strength = value > 0 ? value : 0; }
    }

    int _defense;
    public int Defense {
        get { return _defense; }
        set { _defense = value > 0 ? value : 0; }
    }

    public CombatStats(int health, int strength, int defense) {
        _health = health;
        _maxHealth = _health;
        _strength = strength;
        _defense = defense;
    }

    /// <summary>
    /// Deal damage and returns the total damage dealt
    /// </summary>
    /// <param name="value">Incoming damage value</param>
    /// <returns>Total damage dealt</returns>
    public int DealDamage(int value) {
        // Apply defense, return if no damage was dealt
        int reducedDamage = value - Defense;
        if (reducedDamage <= 0)
            return 0;

        // Return whichever value is lower: current health or damageValue
        int damageDealt = _health < value ? _health : value;
        _health -= value;
        if (_health <= 0)
            _health = 0;
        return damageDealt;
    }
}
```

### Entities

An `Entity` is a container for components, and has a unique ID. They are typically considered only as containers and not as classes, objects, or containing any logic.

For example, a player entity could consist of a `Sprite`,  `Position`, `Velocity`, and `CombatStats` components, while a simple object like a rock entity would only have  `Sprite` and `Position` components. It is the job of a `System` to update entities based on their components. Components can also be added and removed from entities at runtime.

Creating entities is very simple, just create a new entity, then plug in whatever components you want:

```csharp
var playerEntity = new Entity("player");
playerEntity.addComponent(new Sprite());
playerEntity.addComponent(new PositionComponent(100, 150));
playerEntity.addComponent(new VelocityComponent());
playerEntity.addComponent(new CombatStatsComponent(10, 5, 1));
```

### Systems

Systems act upon entities that house certain components.  A system iterates through the list of entities and performs whatever logic necessary.

For example, a `Movement` system would act on entities have both  `Position` and  `Velocity` components. It could update the entity's position base on its velocity.

```csharp
foreach (Entity ent in entities.containing(Position, Velocity)) {
    var position = ent.getComponent<PositionComponent>();
    var velocity = ent.getComponent<VelocityComponent>();
    position.X += velocity.deltaX;
    position.Y += velocity.deltaY;
}
```

Systems operate in the order they are added to the program. A system will operate on all entities in their list, then hand off control to the next system.

For example, say we have the `Movement` system above, an `Input` system that listens for a player's input, and a `Drawing` system that draws sprites to the screen. We would want the `Input` system to handle the input first, then have the `Movement` system update the player entity's position based on that input, then have the `Drawing` system display the entity's new position to the screen.

## Nez's Scene Entity Component System

This will be a very surface level of Nez's Scene/Entity/Component System. Most of Nez revolves around an ECS. You can read about it in-depth in the [Nez's Scene/Entity/Component System FAQ](https://github.com/prime31/Nez/blob/master/FAQs/Scene-Entity-Component.md).

Basically, a Nez Scene can be thought of as parts of your game, such as menus, levels, etc. Nez Entities are added to and managed by scenes. Nez Components are added to and managed by entities and make up the bulk of a game and determine how entities behave.

### Nez Entities

Nez entities contain a series of lifecycle methods that are called by the scene, including `onAddedToScene()`, `onRemovedFromScene()`,  `update()`, and `debugRender()`. They have important properties such as `updateOrder`,  `tag`, `colliders`, and `updateOrder`.

### Nez Components

Nez components contain a series of lifecycle methods that are called by the entity. Like above, methods will be called for initializing, for adding/removing components from entities, debug rendering, and when the entity is enabled or disabled.

### Nez Scenes

A Nez `Scene` is the root of the ECS. It can be thought of as different parts of your game, such as menus, levels, etc. It manages a list of entities, renderers, and post-processors. It has many methods for finding and managing entities, a content manager, setting up screen resolutions, etc. It's something you should familiarize yourself with [here](https://github.com/prime31/Nez/blob/master/FAQs/Scene-Entity-Component.md). For the time being, we'll only cover the basics and dive into more detail in future tutorials.

### Setting up a custom Nez Scene

Within our `Game1.cs` class, we'll setup our custom Nez Scene:

```csharp
protected override void Initialize() {
    base.Initialize();

    // Create our custom scene
    Scene myScene = Scene.createWithDefaultRenderer(Color.LightGreen);

    // Set the scene so Nez can take over
    scene = myScene;
}
```

`Scene.createWithDefaultRenderer(Color.LightGreen)` creates a new scene with a `DefaultRenderer`, and fills it with the supplied color. The `DefaultRenderer` simply renders all render layers. We'll cover controlling rendering in future tutorials.

### Adding Entities to the Scene

We'll add two entities to our scene: A `Player` and an `Enemy`. While these won't have any logic attached, they will have graphics.

```csharp
protected override void Initialize() {
    base.Initialize();

    // Create our custom scene
    Scene myScene = Scene.createWithDefaultRenderer(Color.LightGreen);

    // Create the Player entity
    Entity playerEntity = myScene.createEntity("player");
    Texture2D playerTexture = myScene.content.Load<Texture2D>("images/playerTexture");
    playerEntity.addComponent(new Sprite(playerTexture));
    playerEntity.position = new Vector2(50, 50);

    // Create the Enemy entity
    Entity enemyEntity = myScene.createEntity("enemy");
    Texture2D enemyTexture = myScene.content.Load<Texture2D>("images/enemyTexture");
    enemyEntity.addComponent(new Sprite(enemyTexture));
    enemyEntity.position = new Vector2(150, 50);
    
    // Set the scene so Nez can take over
    scene = myScene;
}
```

To explain what's going on here:

`Entity playerEntity = myScene.createEntity("player");` creates a new entity, adds it to myScene, and gives it the name "player".

`Texture2D playerTexture = myScene.content.Load<Texture2D>("images/playerTexture");` loads a Texture2D object into myScene. You can read about adding content to a MonoGame project [here](http://rbwhitaker.wikidot.com/monogame-managing-content). 

`playerEntity.addComponent(new Sprite(playerTexture));` creates a new Sprite component using the previously created texture. Nez sprites are automatically drawn by the default renderer.

`playerEntity.position = new Vector2(50, 50);` simply sets the position of the entity.

Once the project is built, we'll be greeted by our new entities!

![Player and Enemy entities](\assets\nez_and_ecs_basics\PlayerEnemyEntities.png)



## Nez Entity Systems

The way to manage entities with Nez is by using `Entity System` , which function as a traditional ECS `System`.  You can read about them in depth in the [Nez's Entity Systems FAQ.](https://github.com/prime31/Nez/blob/master/FAQs/EntitySystems.md) Entity Systems are added to Nez scenes and are executed in the order they are added. Nez includes `EntityProcessingSystem`, `ProcessingSystem`, and `PassiveSystem`, the details of which can be found in the FAQ.

### Entity Processing System Example

The system we'll be covering is the `EntityProcessingSystem`. It's a basic entity processing system for processing many entities with specific components.

First we'll create a new, simple component to process, dubbed the `HealthComponent`. It'll keep track of only one thing, the entity's current health:

```csharp
namespace NezProject {
    public class HealthComponent : Nez.Component {
        public int health = 100;
    }
}
```

Next we'll implement our own custom entity processing system called, rather creatively, `HealthSystem`:

```csharp
using Nez;

namespace NezProject {
    class HealthSystem : EntityProcessingSystem {
        public HealthSystem(Matcher matcher) : base(matcher) { }

        public override void process(Entity entity) {
            // Grab the component from the entity
            var healthComponent = entity.getComponent<HealthComponent>();

            // Tick down the health
            healthComponent.health--;

            // Destory the entity once it runs out of health
            if (healthComponent.health <= 0)
                entity.destroy();
        }
    }
}
```

Now we'll add the new component to our enemy, and our system to our scene:

```csharp
enemyEntity.addComponent(new HealthComponent());

// Add the entity processor for health components
myScene.addEntityProcessor(new HealthSystem(new Matcher().all(typeof(HealthComponent))));
```

The new thing to explain here is `Matcher`, which is  covered in the FAQ. It generates a list of all entities that have a `HealthComponent`. You can pass an arbitrary number of component types to it.

Now, whenever a `HealthComponent` is attached to an entity, its health will drain over time until it is gone, and the entity itself will be destroyed.

![Enemy Death Gif](\assets\nez_and_ecs_basics\PlayerEnemy.gif)

## Conclusion

You can see the files of interested created during the tutorial on this [Github Repo](https://github.com/LeeCombs/NezTutorial-FractalPixels/tree/master/Nez_ECS_Basics).

ECS is a design architecture that you'll want to familiarize yourself with going forward. I hope that the concept will become more clear as I continue the series, but I definitely recommend reading up and practicing the concept independently.

Coming up next will be attempting to create a basic game using Nez!