---
layout: post
date: 2018-06-02
category: devblog
tags: [ Nez, C#, gamedev ]
comments: true
author: Fractal Pixels
thumbnail_link: /assets/thumbnails/nez_logo.png
permalink: /devblog/nez-installation-tutorial
---

<div class="post-intro" markdown="1">

This tutorial will go over the basics of setting up a new Nez project.

Nez is a lightweight, 2D framework built upon MonoGame/FNA. You can read more about what Nez is and it's features [in the official documentation](https://prime31.github.io/Nez/). It uses a Scene/Entity/Component system with Component render layer tracking and optional entity systems, which will be covered in future tutorials. However, if you're unfamiliar with Entity Component Systems, be sure to [familiarize yourself with them](https://en.wikipedia.org/wiki/Entity%E2%80%93component%E2%80%93system).

This tutorial uses Windows 10, Visual Studio 2017, and MonoGame 3.6.

{% include toc.html %}

</div>

## [MonoGame Setup](#monogame-setup)

Nez requires a standard MonoGame project to be set up. You can follow the official documentation to do this [here](http://www.monogame.net/documentation/?page=creating_a_new_project_vs), but basically all you need to do is:

1. Download [Monogame for VisualStudio.](http://www.monogame.net/downloads/)

2. Create a new MonoGame Cross Platform Desktop Project. We'll call it "NezProject". The reason I've chosen this type of project is that is uses OpenGL, which Nez supports out of the box. Using DirectX instead may lead to complications down the road.

   ![New Monogame Project](\assets\nez_installation_pics\NewMonoGameProject.png)

3. Ensure everything works by building the project. You should be greeted by a nice cornflower blue window.

   ![Monogame Works!](\assets\nez_installation_pics\MonoGameWorks.png)



## [Adding Nez](#adding-nez)

Now we'll add Nez to our solution. The official documentation for this process can be found [here](https://prime31.github.io/Nez/documentation/setup/installation).

1. Clone or download the [Nez repository](https://github.com/prime31/Nez).

2. Add the Nez project to the solution by right clicking the NezProject solution > Add > Existing Project...

   ![Adding a project](\assets\nez_installation_pics\AddingAProject.png)

3. Navigate to the Nez repository previously downloaded and open `Nez/Nez.Portable/Nez.csproj`.

   ![Selecting Nez Project](\assets\nez_installation_pics\AddingNez.png)

4. Add a reference to the Nez project by selecting NezProject > Add > Add Reference... , select the Projects tab then select Nez.

   ![Adding Nez Reference](\assets\nez_installation_pics\AddingNezReference.png)

   It should be noted that Nez's project references may need to be re-linked at this point. If we come across build issues after setup, this is a place to check.



## [Adding Nez Pipeline Importer](#adding-nez-pipeline-importer)

Here we'll add the Nez Pipeline Importer to our solution. This step is optional, but recommended. As per documentation:

> "Nez provides a plethora of Pipeline Tool importers out of the box.  Importers take data such as Tiled maps and convert them into a binary  format that is much faster and more efficient to use at runtime."

Adding the Pipeline Importer is similar to adding the Nez project above.

1. Add the Nez.PipelineImporter project by right clicking our solution > Add > Existing Project... , navigating to and adding `Nez/Nez.PipelineImporter/Nez.PipelineImporter.csproj`.

2. Open the Nez.PipelineImporter's references and add a reference to the Nez Project.

3. Build the Nez.PipelineImporter project.

   ![Building Nez Pipeline](\assets\nez_installation_pics\BuildNezPipeline.png)

4. Open the Pipeline tool by navigating to our project's root folder > NezProject > Content, then opening the `Content.mgcb` file.

   ![Content.mgcb](\assets\nez_installation_pics\Content.png)

5. Add references to `PipelineImporter.dll`, `Ionic.ZLib.dll`, `Newtonsoft.Json.dll` and `Nez.dll`.

   ![Content References](\assets\nez_installation_pics\ContentReferences.png)

Now we're set up! Nez supports many importers out of the box. You can read more about it in the [Pipeline Importers FAQ](https://github.com/prime31/Nez/blob/master/FAQs/PipelineImporters.md).

## [Setting up the Game class](#setting-up-the-game-class)

Now that we have Nez set up, we can move into hooking it up to our previously created MonoGame project. 

We'll start by sub-classing `Game1.cs` into `Nez.Core`. You can read more about [Nez Core here](https://github.com/prime31/Nez/blob/master/FAQs/Nez-Core.md).

We'll then clean up extra code by removing references to `GraphicsDeviceManager graphics` and `SpriteBatch spriteBatch`. For flavor, I've changed the background fill color to LightGreen.

`Game1.cs` will end up looking like this:

```csharp
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;

namespace NezProject  {
    
    public class Game1 : Nez.Core {
        
        public Game1() {
            Content.RootDirectory = "Content";
        }
        
        protected override void Initialize() {
            base.Initialize();
        }
        
        protected override void LoadContent() {
            // TODO: use this.Content to load your game content here
        }
        
        protected override void UnloadContent() {
            // TODO: Unload any non ContentManager content here
        }

        protected override void Update(GameTime gameTime) {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed || Keyboard.GetState().IsKeyDown(Keys.Escape))
                Exit();

            // TODO: Add your update logic here

            base.Update(gameTime);
        }

        protected override void Draw(GameTime gameTime) {
            GraphicsDevice.Clear(Color.LightGreen);

            // TODO: Add your drawing code here

            base.Draw(gameTime);
        }
    }
}

```

Building the project should display a nice, green window.

![Nez Build Success](\assets\nez_installation_pics\NezWorks.png)

## [Conclusion](#conclusion)

That's it! Setting up Nez is pretty straightforward. We'll use this as the base of tutorials going forward as we learn how to make games using Nez!

You can see the results of the tutorial on this [Github Repo](https://github.com/LeeCombs/NezTutorial-FractalPixels/tree/master/Nez_Installation).

Coming up next will be Entity Component System basics, and how Nez fits into the puzzle. We'll set up our first Nez Scene and add a simple player to it.