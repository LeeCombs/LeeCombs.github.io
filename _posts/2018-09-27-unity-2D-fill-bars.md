---
layout: post
date: 2018-09-27
category: devblog
tags: [ Unity, C#, gamedev ]
comments: true
permalink: /devblog/unity-2D-fill-bars
---

In this post I'll explain how to make a "Fill Bar" using Unity and C#. After going through it you should be able to create a basic fill bar using Unity assets, as well as adding simple customizations and functionality.

The fill bar will simply be for displaying a value between defined minimum and maximum values, both as a graphical bar and numerical text display. Further functionality, such as creating a progress bar, having a delayed fill, or creating a radial fill bar, will be covered in a future tutorial.

I'm currently teaching myself Unity and C# while making a small incremental/idle game. While I may not be currently following the best practices for coding or design, I am open to constructive feedback regarding either. If you have any recommendations or requests please feel free to share them.

Files for this tutorial are hosted on my [Github here](https://github.com/LeeCombs/NezTutorial-FractalPixels/tree/master/Unity_Progress_Bars).

## [Anatomy of a Fill Bar](#anatomy-of-a-fill-bar)

The `Fill Bar` will be made up of a handful of objects. As with any Unity object, everything exists in a Game Object. Within that object will be the graphics, text, and logic that make up the fill bar. It can be broken down as follows:


![BarPartsTitles](\assets\unity_2d_fill_bars\BarPartsTitles.png)


## [Creating a Basic Fill Bar](#creating-a-basic-fill-bar)

First we'll create a basic fill bar using Unity assets. We'll start with a new, blank project using the 2D template. Unity UI elements exist on a `Canvas`, so we'll have to add one to our scene. Right click the `SampleScene` menu and add a **UI > Canvas**. For a basic fill bar starter, we'll use Unity's `Slider`. Right click the canvas and add a **UI > Slider**. We'll be greeted with a single, simple slider on the canvas as seen below.


![SliderCreation](\assets\unity_2d_fill_bars\SliderCreation.png)


The slider comes with several components by default: `Background`, `Fill Area`, and `Handle Slide Area`. We're only interested in the background and fill area as they will be a part of the final fill bar. The handle slide area normally handles user input, allowing them to click and drag the slider to change its value. Since we'll be disabling direct user interaction, we'll delete the handle component and set **Slider > Slider (Script) > Interactable** toggle to false, as shown below.


![InteractableToggle](\assets\unity_2d_fill_bars\InteractableToggle.png)


We'll resize the slider to fit its parent game object and appear more rectangular. To do so we'll make a few changes to each component, but note that if you're more comfortable with adjusting the anchors and dimensions by mouse, then by all means do so:

**Slider**

- *Rect Transform*'s Width **= 300**, Height **= 50**

**Background**

- *Anchor*'s Min X/Y **= 0**, Max X/Y **= 1**
- *Rect Transform*'s Top/Bottom **= 0**


![BackgroundRectTransform](\assets\unity_2d_fill_bars\BackgroundRectTransform.png)


**Fill Area**

- *Anchor*'s Min X/Y **= 0**, Max X/Y **= 0**
- *Rect Transform*'s Right **= 5**, Top/Bottom **= 0**
- *Fill*'s Color **= Green**

We can see the results of the change below:


![SliderOldToNew](\assets\unity_2d_fill_bars\SliderOldToNew.png)


With those changes we've created a very basic fill bar using Unity's prebuilt assets. We can adjust the slider's value to see it in action!


![BarPartsTitles](\assets\unity_2d_fill_bars\FillBarMovement.gif)


We have a problem however as a Value of 0 still displays a small amount of fill since its width is 10. Lowering the width causes an unsightly stretch distortion of the fill image. To ignore the issue, we can set fill's Width to **0**, and set the fill area's rect transform's Left and Right to **0**. To solve the issue we'll create a custom fill bar using our own assets and by making a couple of minor adjustments.

## [Creating a Custom Fill Bar](#creating-a-custom-fill-bar)

Now we'll customize the fill bar to contain everything described in the "Anatomy of a Fill Bar". We'll add a custom sprite that makes up our foreground border, edit the fill and background images to be a simple square, and add a text field to display the slider's value numerically.

### The Foreground Border

We'll create our custom foreground border by importing a sprite, "9-Slicing" it, and setting up a new image object that will house it. The image being used for the border will be "SimpleBorder": ![SimpleBorder](\assets\unity_2d_fill_bars\SimpleBorder.png)

Save the file locally, then drag and drop into Unity's Assets folder. We will 9-slice the sprite so that it may stretch to fit any sized fill bar we want without worrying about degrading the image with scaling. You can read more about 9-slicing by reading [Unity's Manual to 9-slicing Sprites](https://docs.unity3d.com/Manual/9SliceSprites.html). 

To slice the sprite all we'll need to do is select the "SimpleBorder" asset, select the Sprite Editor button in the inspector, then set Border's L/T/R/B values to **8**, then select Apply and Close.


![Sprite9Slice](\assets\unity_2d_fill_bars\Sprite9Slice.png)


### Fill Bar Additions and Adjustments

We'll first create a new UI `Text` field to display the slider's current value. Add the text by right clicking **Slider > UI > Text**. We'll make adjustments to it shortly.

Now we'll create the `Foreground Border` by right clicking `Background`, selecting duplicate, and naming it "Foreground Border". We duplicate the background instead of creating a new UI `Image` mainly to preserve the anchors and offsets, instead of manually adjusting each field.

Now we'll make the following adjustments:

**Background**

- Image (Script) > Source Image **= None**

**Fill Area** 

- *Rect Transform*'s Left/Right offset **= 0**
- Fill's *Rect Transform*'s Width **= 0**
- Fill > Image (Script) > Source Image **= None**

**Foreground Border**

- Image (Script) > Source Image **= SimpleBorder**
- Image (Script) > Image Type **= Sliced**

**Text**

- Rename it to **"Display Text"**
- Text (Script) > Alignment **= Middle and Center**
- Text **= "100%"**
- Size **= 24**
- Style = **Bold**

Ensure proper draw order by ordering the objects back to front. You can drag objects to adjust their draw order, just be careful not to assign them a parent object accidentally. The order should look like this:



![ObjectOrder](\assets\unity_2d_fill_bars\ObjectOrder.png)



And our new fill bar should look like this:



![FillBar2](\assets\unity_2d_fill_bars\FillBar2.png)



While nice, the bar lacks functionality, so we'll move onto creating the script.

### Fill Bar Scripting

Now we'll write a script that will handle getting and setting the slider's current value, the fill area's width, and displaying the current value numerically. To add a new script, right click the **Assets folder > Create > C# Script**. We'll call this one "FillBar". Be sure to attach the script to the `Slider` object!

First we'll add a reference to Unity's UI package so that we can access the Slider and Text classes.

```csharp
using UnityEngine.UI;
```

Now we'll add public references to the slider and text objects that belong to the fill bar:

```csharp
public class FillBar : MonoBehaviour {
    
    // Unity UI References
    public Slider slider;
    public Text displayText;
    
    ...
}
```

To keep track of the current value we'll add a property called `CurrentValue`, with a private backing field called `currentValue`. If you're unfamiliar with C# properties you can read more about it [in the C# Programming Guide](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties). We'll keep it simple and use it for a simple getter/setter.

```csharp
// Unity UI References
...

// Create a property to handle the slider's value
private float currentValue = 0f;
public float CurrentValue {
    get {
        return currentValue;
    }
    set {
        currentValue = value;
        slider.value = currentValue;
        displayText.text = (slider.value * 100).ToString("0.00") + "%";
    }
}
```

Here we created two floats to handle the fill bar's current fill value: the private `currentValue` and public `CurrentValue`. The getter simply returns the private field, while the setter handles updating the private field, the slider's value, and the displayText's text to reflect the new value as a percentage to two decimal places.

Now the fill bar has logic to handle what it needs to, but we don't have an easy way to quickly check if it's working. To do this we'll temporarily increment CurrentValue to see the bar in action. In `Start()` we'll set the initial CurrentValue to 0, and in `Update()` we'll increment CurrentValue a little bit every frame.

```csharp
// Use this for initialization
void Start () {
    CurrentValue = 0f;
}

// Update is called once per frame
void Update () {
    CurrentValue += 0.0043f;
}
```

This is our fill now in action:

![FillBarFill](\assets\unity_2d_fill_bars\FillBarFill.gif)

And here is the script's code:

```csharp
using UnityEngine;
using UnityEngine.UI;

public class FillBar : MonoBehaviour {
    
    // Unity UI References
    public Slider slider;
    public Text displayText;
    
    // Create a property to handle the slider's value
    private float currentValue = 0f;
    public float CurrentValue {
        get {
            return currentValue;
        }
        set {
            currentValue = value;
            slider.value = currentValue;
            displayText.text = (slider.value * 100).ToString("0.00") + "%";
        }
    }

    // Use this for initialization
    void Start () {
        CurrentValue = 0f;
    }
	
    // Update is called once per frame
    void Update () {
        CurrentValue += 0.0043f;
    }
}
```

And that's it! We have a completed, basic fill bar. A couple of things to note however:

- The slider's default Min Value is **0** and Max Value is **1**. If you'd like to change these you'll want to update the displayText's txet to properly calculate the current percentage based on the new min/max values.
- If you'd prefer to not display the fill as a percentage simply set displayText's text as `slider.value.ToString();`, or any other format you prefer.
- The fill bar does nothing when it's filled since it's designed to simply reflect the slider's current value. We'll add progress functionality to it, including adding and removing progress, and toggling events when the progress bar fills up.
- Be sure to remove the temporary incrementation code we added to check if the bar functions properly.


## [Conclusion](#conclusion)

We've created a basic "Fill Bar" in Unity. While limited, it creates a good foundation to build off of. Going forward we will extend upon it's functionality and dive into Unity's systems a bit more.

As stated before, all files generated during this tutorial can be found on my [Github here](https://github.com/LeeCombs/NezTutorial-FractalPixels/tree/master/Unity_Progress_Bars).

If you have any comments or questions regarding anything covered by this tutorial, or have an idea for future posts, feel free the leave them below.

Coming up next will be Unity Progress Bars, Radial Fills, and Delayed Fill Bars, and possibly anything else I can think of that's related.