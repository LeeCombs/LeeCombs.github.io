---
layout: post
date: 2018-09-28
category: devblog
tags: [ Unity, C#, gamedev ]
comments: true
permalink: /devblog/unity-2D-progress-bars
---

In this post I'll explain how to make a progress bar using Unity and C#. After going through it you should be able to create a basic progress bar using Unity assets, as well as adding simple customizations and functionality.

First we'll make a fill bar that will simply be for displaying a value between a defined minimum and maximum, both as a graphical bar and numerical text display. Then we'll move into providing progress bar functionality to the fill bar where we can call methods once it's been filled.

At the end of the tutorial we'll wind up with a radial fill that looks like this:

![ProgressFill](\assets\unity_2d_progress_bars\FillBarFill.gif)

I'm currently teaching myself Unity and C# while making a small incremental/idle game. While I may not be currently following the best practices for coding or design, I am open to constructive feedback regarding either. If you have any recommendations or requests please feel free to share them.

Files for this tutorial are hosted on my [Github here](https://github.com/LeeCombs/Fractal-Pixels-Tutorials/tree/master/Unity%202D%20Progress%20Bars).

## [Anatomy of a Fill Bar](#anatomy-of-a-fill-bar)

The `Fill Bar` will be made up of a handful of objects. As with any Unity object, everything exists in a `GameObject`. Within that object will be the graphics, text, and logic that make up the fill bar. It can be broken down as follows:


![BarPartsTitles](\assets\unity_2d_progress_bars\BarPartsTitles.png)


## [Creating a Basic Fill Bar](#creating-a-basic-fill-bar)

First we'll create a basic fill bar using Unity assets. We'll start with a new, blank project using the 2D template. Unity UI elements exist on a `Canvas`, so we'll have to add one to our scene. Right click the `SampleScene` menu and add a **UI > Canvas**. For a basic fill bar starter we'll use Unity's `Slider`. Right click the canvas and add a **UI > Slider**. We'll be greeted with a single, simple slider on the canvas as seen below.


![SliderCreation](\assets\unity_2d_progress_bars\SliderCreation.png)


The slider comes with several components by default: `Background`, `Fill Area`, and `Handle Slide Area`. We're only interested in the background and fill area as they will be a part of the final fill bar. The handle slide area normally handles user input, allowing them to click and drag the slider to change its value. Since we'll be disabling direct user interaction, we'll delete the handle component and set **Slider > Slider (Script) > Interactable** toggle to false, as shown below.


![InteractableToggle](\assets\unity_2d_progress_bars\InteractableToggle.png)


We'll resize the slider to fit its parent game object and appear more rectangular. To do so we'll make a few changes to each component, but note that if you're more comfortable with adjusting the anchors and dimensions by mouse, then by all means do so:

**Slider**

- *Rect Transform*'s Width **= 300**, Height **= 50**

**Background**

- *Anchor*'s Min X/Y **= 0**, Max X/Y **= 1**
- *Rect Transform*'s Top/Bottom **= 0**


![BackgroundRectTransform](\assets\unity_2d_progress_bars\BackgroundRectTransform.png)

**Fill Area**

- *Anchor*'s Min X/Y **= 0**, Max X/Y **= 1**
- *Rect Transform*'s Right **= 5**, Top/Bottom **= 0**
- *Fill*'s Color **= Green**

We can see the results of the change below:


![SliderOldToNew](\assets\unity_2d_progress_bars\SliderOldToNew.png)


With those changes we've created a very basic fill bar using Unity's prebuilt assets. We can adjust the slider's value to see it in action!


![BarPartsTitles](\assets\unity_2d_progress_bars\FillBarMovement.gif)


We have a problem however as a Value of 0 still displays a small amount of fill since its width is 10. Lowering the width causes an unsightly stretch distortion of the fill image. To ignore the issue, we can set fill's Width to **0**, and set the fill area's rect transform's Left and Right to **0**. To solve the issue we'll create a custom fill bar using our own assets and by making a couple of minor adjustments.

## [Creating a Custom Fill Bar](#creating-a-custom-fill-bar)

Now we'll customize the fill bar to contain everything described in the "Anatomy of a Fill Bar". We'll add a custom sprite that makes up our foreground border, edit the fill and background images to be a simple square, and add a text field to display the slider's value numerically.

### The Foreground Border

We'll create our custom foreground border by importing a sprite, "9-Slicing" it, and setting up a new image object that will house it. The image being used for the border will be "SimpleBorder": ![SimpleBorder](\assets\unity_2d_progress_bars\SimpleBorder.png)

Save the file locally, then drag and drop into Unity's Assets folder. We will 9-slice the sprite so that it may stretch to fit any sized fill bar we want without worrying about degrading the image with scaling. You can read more about 9-slicing by reading [Unity's Manual to 9-slicing Sprites](https://docs.unity3d.com/Manual/9SliceSprites.html). 

To slice the sprite all we'll need to do is select the "SimpleBorder" asset, select the Sprite Editor button in the inspector, then set Border's L/T/R/B values to **8**, then select Apply and Close.


![Sprite9Slice](\assets\unity_2d_progress_bars\Sprite9Slice.png)


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



![ObjectOrder](\assets\unity_2d_progress_bars\ObjectOrder.png)



And our new fill bar should look like this:



![FillBar2](\assets\unity_2d_progress_bars\FillBar2.png)



While nice, the bar lacks functionality, so we'll move onto creating the script.

### Fill Bar Scripting

Now we'll write a script that will handle getting and setting the slider's current value, the fill area's width, and displaying the current value numerically. To add a new script, right click the **Assets folder > Create > C# Script**. We'll call this one `FillBar.cs`. Be sure to attach the script to the `Slider` object!

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

This is our fill bar now in action:

![FillBarFill](\assets\unity_2d_progress_bars\FillBarFill.gif)

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

## [Progress Functionality](#progress-functionality)

Now we'll move onto making the progress bar. It'll be built on top of our newly minted fill bar, so all we'll need to do is write a simple script. Right click the **Assets folder > Create > C# Script** and call it `ProgressBar.cs`.

We'll reuse our previously created fill bar by removing the `FillBar` script, attaching our new `ProgressBar` script, and hooking up the slider and display text just like we did before.

We'll be importing `UnityEngine.Events` to leverage `UnityEvent`s. Why? I'm not entirely sure how I determined to use it, but they appear pretty useful. You can read more about them in [Unity's documentation here](https://docs.unity3d.com/560/Documentation/Manual/UnityEvents.html).

```csharp
using UnityEngine.Events;
```

We'll add our `UnityEvent` variable and build on top of the base `CurrentValue` property so that when we set the new value it will trigger our `UnityEvent`.

```csharp
// Event to invoke when the progress bar fills up
private UnityEvent onProgressComplete;

// Create a property to handle the slider's value
public new float CurrentValue {
    get {
        return base.CurrentValue;
    }
    set {
        // If the value exceeds the max fill, invoke the completion function
        if (value > slider.maxValue)
            onProgressComplete.Invoke();

        // Remove any overfill (i.e. 105% fill -> 5% fill)
        base.CurrentValue = value % slider.maxValue;
    }
}
```

And we'll write a method to be called when the progress bar fills up. This will be added to our `UnityEvent` shortly.

```csharp
// The method to call when the progress bar fills up
void OnProgressComplete() {
    Debug.Log("Progress Complete");
}
```

Now we'll initialize the event, and set up our tried-and-true `CurrentValue` incrementation to see the progress bar in action.

```csharp
void Start () {
    // Initialize onProgressComplete and set a basic callback
    if (onProgressComplete == null)
        onProgressComplete = new UnityEvent();
    onProgressComplete.AddListener(OnProgressComplete);
}

void Update () {
    CurrentValue += 0.0153f;
}
```

And that's everything! The progress bar will fill up to its max fill value, which is 1, then fire off our method and reset itself, which we can see by the "Progress Complete" entry within the console. 

![ProgressFill](\assets\unity_2d_progress_bars\ProgressFill.gif)

A couple of things to note:

- When completing progress the Unity event is only fired off once, no matter how full it gets. To have it fire off each time it's filled, you could calculate the total number of fills, then invoke the event. Something similar to `int totalFills = (int)(value / slider.maxValue);`
- Instead of directly accessing *CurrentValue*, you may instead write methods for handling adding and removing progress to ensure that we're not adding negative progress, or attempting to set the current progress value too low.

Lastly, here is our `ProgressBar.cs` code in its entirety:

```csharp
using UnityEngine;
using UnityEngine.Events;

public class ProgressBar : FillBar {

    // Event to invoke when the progress bar fills up
    private UnityEvent onProgressComplete;

    // Create a property to handle the slider's value
    public new float CurrentValue {
        get {
            return base.CurrentValue;
        }
        set {
            // If the value exceeds the max fill, invoke the completion function
            if (value > slider.maxValue)
                onProgressComplete.Invoke();

            // Remove any overfill (i.e. 105% fill -> 5% fill)
            base.CurrentValue = value % slider.maxValue;
        }
    }
    
    void Start () {
        // Initialize onProgressComplete and set a basic callback
        if (onProgressComplete == null)
            onProgressComplete = new UnityEvent();
        onProgressComplete.AddListener(OnProgressComplete);
    }
	
    void Update () {
        CurrentValue += 0.0153f;
    }

    // The method to call when the progress bar fills up
    void OnProgressComplete() {
        Debug.Log("Progress Complete");
    }
}
```



## [Conclusion](#conclusion)

We've created a basic progress bar in Unity. While limited, it creates a good foundation to build off of.

If you have any comments or questions regarding anything covered by this tutorial, or have an idea for future posts, feel free the leave them below.

As stated before, all files generated during this tutorial can be found on my [Github here](https://github.com/LeeCombs/Fractal-Pixels-Tutorials/tree/master/Unity%202D%20Progress%20Bars).

Coming up will be Unity Radial Fills and Delayed Fill Bars, and possibly anything else I can think of that's related.