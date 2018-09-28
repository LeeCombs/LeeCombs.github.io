---
layout: post
date: 2019-09-30
category: devblog
tags: [ Unity, C#, gamedev ]
comments: true
permalink: /devblog/unity-2D-radial-progress-bars

---

Intro blurb

I'm currently teaching myself Unity and C# while making a small incremental/idle game. While I may not be currently following the best practices for coding or design, I am open to constructive feedback regarding either. If you have any recommendations or requests please feel free to share them.

Files for this tutorial are hosted on my [Github here](https://github.com/LeeCombs/NezTutorial-FractalPixels/tree/master/Unity_Progress_Bars).

## [Progress Bar Functionality](#progress-bar-functionality)

Build upon previously created fill bar, link the tutorial

Script changes and unity discussion.

## [Radial Fill Bar](#radial-fill-bar)

Import SimpleCircle

Empty Game Object

- Width / Height = 100

UI > Image "Background"

- Source Image = Simple Circle
- Color = Grey

- Anchor Adjust (?)

Duplicate Background > "Fill Image"

- Color = Green

- Type = Filled
- Fill Method = Radial 360

Mess around with Fill Amount to achieve the effect (image)

Duplicate Background > "Center"

- Change color
- Anchors and Offsets (12)

Center > UI > Text "Display Text"

- Anchors 0/1
- Offsets L/R 10, T/D 15
- Text Bold

Scripting

- import Unity ui

- CurVal property
- increment tester, like fill bar
- Link components to public references