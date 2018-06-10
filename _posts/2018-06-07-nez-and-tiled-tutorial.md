---
layout: post
date: 2018-06-07
category: devblog
tags: [ Nez, C#, gamedev ]
comments: true
permalink: /devblog/nez-and-tiled-tutorial
---

This tutorial will go over importing and loading Tiled maps with Nez!

Nez's Pipeline Importers include support for Tiled maps. It covers tile, image, and object layers and rendering with full culling support built-in along with optimized collider generation. 

Maps generated for this project are created with the [Tiled Map Editor](https://www.mapeditor.org/), which is a free, 2D level editor that is great for developing maps for your games. 

All of the files used during this tutorial can be located in the [Github Repo here](https://github.com/LeeCombs/NezTutorial-FractalPixels/tree/master/Nez_and_Tiled).

## [Some Initial Setup](#some-initial-setup)

Before continuing with this tutorial, please ensure that the Nez.PipelineImporter is set up. See [Setting up Nez Pipeline Importer tutorial](/devblog/nez-installation-tutorial.html#adding-nez-pipeline-importer).

Also, to help manage render layers for upcoming tutorials, we'll add a `RenderLayer` enum to keep everything in one place.

```csharp
enum RenderLayer {
    AboveDetailShadow = -2,
    AboveDetail = -1,
    Player = 1,
    TileMap = 10
}
```



## [The Tiled Map](#the-tiled-map)

This is the map we'll be working with, called `Map_1.tmx`:

![Tile Map](\assets\tiled\TiledMap.png)

Here's an explanation of the map's layers:

 - `Collision` contains tiles that will be collidable. If a space does not contain a tile in this layer, there will be no collision body created in that square.
 - `Above_Detail` contains tiles that will be rendered above the player, such as tree leaves or roof overhang.
 - `Detail` contains tiles like flowers, bushes, or signs that are there for extra detail on top of the Tile map.
 - `Tile` contains the base tile map, which is mostly grass, sand, and walls.
 - `Background` contains background tile detail, which is currently just water.

For the purposes of this tutorial, all layers, excluding collision, will have the same render layer. We'll break these up to achieve certain effects, such as drawing the shadow of the player behind objects, in future tutorials.

## [Adding Tiled Maps to the Project](#adding-tiled-maps-to-the-project)

As with managing any content with MonoGame projects, we have to add our Tiled maps to the `Content.mgcb`. Within the Pipeline Tool click Content > Add > Existing Item... , then select `Map_1.tmx`.

![Adding content](\assets\tiled\Adding_Tiled_Map.png)

We also have to add the tile set graphic, dubbed `Tileset_Ivan_Voirol.png`. The tile set was created by Ivan Voirol, and can be found here on [OpenGameArt.org](https://opengameart.org/content/basic-map-32x32-by-silver-iv).

After adding the map and the tile set graphic, make sure to build the project so that we can actually access the map!

![Rebuilding MonoGame Content](\assets\tiled\Assets_Building.png)

## [Importing and Loading Tiled Maps](#importing-and-loading-tiled-maps)

Assuming everything's properly set up, we can import and load the map by adding these lines in our scene's `Initialize()` method:

```csharp
// Load the Tiled map, and render it below everything
var tiledMap = myScene.content.Load<TiledMap>("maps/Map_1");
var tiledEntity = myScene.createEntity("tiled-map");
var tiledMapComponent = tiledEntity.addComponent(new TiledMapComponent(tiledMap, "Collision"));
tiledMapComponent.setLayersToRender(new string[] { "Above_Detail", "Detail", "Tile", "Background" });
tiledMapComponent.renderLayer = (int)RenderLayer.TileMap;
```

That's all it takes! If we build the solution, we'll be greeted with our loaded tile map.

![TiledMap](\assets\tiled\FirstTiledBuild.png)

However, to explain what's going on above:

`var tiledMap = myScene.content.Load<TiledMap>("maps/Map_1");` loads our tiled map into our scene's content. By adding content to the scene directly, whenever we change scenes, the content will be cleaned up for us.

`var tiledEntity = myScene.createEntity("tiled-map");` as with anything ECS, our tile map will be an entity, and the `TiledMapComponent` will be doing the heavy lifting.

`TiledMapComponent(tiledMap, "Collision"));` creates the component, and the optional layer name "Collision" tells the constructor which layer will be used for collisions. Once the component is added to the entity, it will create collision rectangles based on this layer automatically. 

To have the collision rectangles drawn to the screen, we can set `debugRenderEnabled = true;` in the project, or by typing `debug-render` in the console during runtime. Some are faint, but can be noted by a red rectangle with a red dot in their center:

![Collision Rectangles](\assets\tiled\Collision_Rects.png)

I've adjusted the colors in an attempt to highlight the collision rectangles below. Tiles with their original color (water, cactuses, the house, walls...) are where collision exists:

![Collision Highlight](\assets\tiled\Collision_Rects_Highlight.png)

`tiledMapComponent.setLayersToRender(new string[] { "Above_Detail", "Detail", "Tile", "Background" });` tells the component the layers of the map we want to be drawn on screen. Layers like collision or triggers would be left out.

`tiledMapComponent.renderLayer = (int)RenderLayer.TileMap;`  sets the render layer of the entire tiled map component.

## [Conclusion](#conclusion)

As you can see, adding Tiled maps with Nez's Pipeline Importer is a breeze. In future tutorials we'll cover how to manage render layers, adding a player that collides with the tile map, and adding a player shadow for when the player is obscured by the map!

You can see the files of interested created during the tutorial on this [Github Repo](https://github.com/LeeCombs/NezTutorial-FractalPixels/tree/master/Nez_and_Tiled).