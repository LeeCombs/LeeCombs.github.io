---
layout: post
date: 2018-09-30
category: devblog
tags: [ Unity, C#, gamedev ]
comments: true
author: Fractal Pixels
thumbnail_link: /assets/thumbnails/unity_2D_radial_progress_bar.png
permalink: /devblog/unity-2D-radial-progress-bars

---

<div class="post-intro" markdown="1">
	
In this post I'll explain how to make a radial progress bar using Unity and C#. After going through it you should be able to create a basic radial progress bar using Unity assets, as well as adding simple functionality.

This is a continuation of a 2D Progress Bar Tutorial
<a class="out-link" href="/devblog/unity-2D-progress-bars" onclick="ga('send','event','click','internal page click','2D Bar');">
  found here.
</a>

First we'll make a radial fill that will simply be for displaying a value between a defined minimum and maximum, both as a graphical radial bar and numerical text display. Then we'll move into providing progress bar functionality to the radial fill where we can call methods once it's been filled.

At the end of the tutorial we'll wind up with a radial fill that looks like this:

![RadialBarFilling](\assets\unity_2d_radial_progress\RadialBarFilling.gif)

I'm currently teaching myself Unity and C# while making a small incremental/idle game. While I may not be currently following the best practices for coding or design, I am open to constructive feedback regarding either. If you have any recommendations or requests please feel free to share them.

Files for this tutorial are hosted on my
<a class="out-link" href="https://github.com/LeeCombs/Fractal-Pixels-Tutorials/tree/master/Unity%202D%20Radial%20Progress" onclick="ga('send','event','click','github click','2D Radial');">
  Github Page Here.
</a>

{% include toc.html %}

</div>

## [Anatomy of a Radial Fill](#anatomy-of-a-radial-fill)

The `Radial Fill` will be made up of a handful of objects. As with any Unity object, everything exists in a `GameObject`. Within that object will be the graphics, text, and logic that make up the fill bar. It can be broken down as follows:

![RadialFillAnatomy](\assets\unity_2d_radial_progress\RadialFillAnatomy.png)

## [Creating a Basic Radial Fill](#creating-a-basic-radial-fill)

First we'll create a basic radial fill by layering several images on top of each other, then adding a text display to show the fill amount numerically. Then we'll create a simple script to handle the logic. We'll start with a new, blank project using the 2D template. 

### Circle Upon Circle Upon Circle

We'll import a custom circle image which will be used by all image components. We'll be using an off-white circle called "SimpleCircle.png": 

<img src="\assets\unity_2d_radial_progress\SimpleCircle.png" width="100" height="100">

We'll start with a new, blank project using the 2D template. Unity UI elements exist on a `Canvas`, so we'll have to add one to our scene. Right click the `SampleScene` menu and add a **UI > Canvas**. Next we'll create the game object that will be our radial fill by right clicking the canvas, adding a **Create Empty**, and call it `RadialFill`. Adjust the *Width* and *Height* to **100**.

Now we'll start to create the components that make up the radial fill. First, add a **UI > Image** to the game object and name it `Background`. Now make the following adjustments:

- Source Image **= SimpleCircle**
- Color **= Grey**
- Anchor's Min X/Y **= 0**, Max X/Y **= 1**

Now we'll duplicate the `Background` and name it `Fill Image`. We duplicate it to simply keep the applied source image and anchors. Now we'll make these changes:

- Color **= Green**
- Image Type **= Filled**
- Fill Method **= Radial 360**

We can mess around with *Fill Amount* to achieve the radial loading effect:

![RadialFillAnimation](\assets\unity_2d_radial_progress\RadialFillAnimation.gif)

Duplicate the background again, and rename it `Center`. This will go over our fill image, giving the illusion of a radial fill bar. We'll make a couple of adjustments to shrink it down:

- Color **= White**
- Rect Transform's Left/Right/Top/Bottom **= 12**

Finally we'll create the text display to show off the current fill as a percentage. We'll make the text object a child of the center image. Right click `Center` and add a **UI > Text**, and name it `Display Text`. Adjustments:

- Anchor's Min X/Y **= 0**, Max X/Y **= 1**
- Rect Transform's Left/Right **= 10**, Top/Bottom **= 15**
- Text **= "0%"**
- Font Style **= Bold**
- Alignment = **Middle & Center**

With all the above we wind up with a pretty basic looking circular display. However, the text display doesn't update with the current value yet. That'll be handled in the script we'll create shortly.

![RadialFilling2](\assets\unity_2d_radial_progress\RadialFilling2.gif)

### Radial Fill Scripting

Now we'll write a script that will handle getting and setting the fill's current value, handling the fill image's total fill, and displaying the current value numerically. The script will be similar to the one created in the [fill bar tutorial here](/devblog/unity-2d-progress-bars.html#creating-a-custom-fill-bar). The main difference is how min/max and *CurrentValue* are handled. Instead of relying on slider's *minValue* and *maxValue*, we'll handle our own. We'll also calculate the running fill percentage and use that as the fill image's amount and text display.

To add a new script, right click the **Assets folder > Create > C# Script**. We'll call this one `Radiall.cs`. Be sure to attach the script to the `RadialFill` object! Now copy and paste this code into the script. For the explanation of what's going on be sure to check the "Fill Bar Scripting" section in the [fill bar tutorial here](/devblog/unity-2d-progress-bars.html#creating-a-custom-fill-bar).

```csharp
using UnityEngine;
using UnityEngine.UI;

public class RadialFill : MonoBehaviour {

    // Public UI References
    public Image fillImage;
    public Text displayText;

    // Trackers for min/max values
    protected float maxValue = 2f, minValue = 0f;

    // Create a property to handle the slider's value
    private float currentValue = 0f;
    public float CurrentValue {
        get {
            return currentValue;
        }
        set {
            // Ensure the passed value falls within min/max range
            currentValue = Mathf.Clamp(value, minValue, maxValue);

            // Calculate the current fill percentage and display it
            float fillPercentage = currentValue / maxValue;
            fillImage.fillAmount = fillPercentage;
            displayText.text = (fillPercentage * 100).ToString("0.00") + "%";
        }
    }

    void Start() {
        CurrentValue = 0f;
    }

    // Update is called once per frame
    void Update () {
        CurrentValue += 0.0086f;
    }
}
```

Now we have a functional radial fill that is handled through *CurrentValue*:

![RadialBarFilling](\assets\unity_2d_radial_progress\RadialBarFilling.gif)

## [Progress Functionality](#progress-functionality)

Now we'll move onto making the radial progress bar. It'll be built on top of our newly minted radial fill, so all we'll need to do is write a simple script which will be basically the same as the script created in the [progress bar tutorial here](/devblog/unity-2d-progress-bars.html#progress-functionality). That tutorial covers the reasoning behind the code.

Right click the **Assets folder > Create > C# Script** and call it `RadialProgress.cs`. We'll reuse our previously created fill bar by removing the existing `RadialFill` script, attaching our new `RadialProgress` script, and hooking up the fill image and display text just like we did before. 

Copy and paste this code into `RadialProgress.cs`. For more explanation refer to the tutorial linked previously.

```csharp
using UnityEngine;
using UnityEngine.Events;

public class RadialProgress : RadialFill {

    // Event to invoke when the progress bar fills up
    private UnityEvent onProgressComplete;

    // Create a property to handle the slider's value
    public new float CurrentValue {
        get {
            return base.CurrentValue;
        }
        set {
            // If the value exceeds the max fill, invoke the completion function
            if (value > maxValue)
                onProgressComplete.Invoke();

            // Remove any overfill (i.e. 105% fill -> 5% fill)
            base.CurrentValue = value % maxValue;
        }
    }

    // Use this for initialization
    void Start () {
        // Initialize onProgressComplete and set a basic callback
        if (onProgressComplete == null)
            onProgressComplete = new UnityEvent();
        onProgressComplete.AddListener(OnProgressComplete);
    }
	
    // Update is called once per frame
    void Update () {
        CurrentValue += 0.0273f;
    }

    // The method to call when the progress bar fills up
    void OnProgressComplete() {
        Debug.Log("Progress Complete");
    }
}
```

And that's everything! The progress bar will fill up to its max fill value, which is 1, then fire off our method and reset itself, which we can see by the "Progress Complete" entry within the console.

![RadialProgressFill](\assets\unity_2d_radial_progress\RadialProgressFill.gif)

A couple of things to note:

- When completing progress the Unity event is only fired off once, no matter how full it gets. To have it fire off each time it's filled, you could calculate the total number of fills, then invoke the event. Something similar to `int totalFills = (int)(value / slider.maxValue);`
- Instead of directly accessing *CurrentValue*, you may instead write methods for handling adding and removing progress to ensure that we're not adding negative progress, or attempting to set the current progress value too low.

## [Conclusion](#conclusion)

We've created a basic radial progress fill bar in Unity. While limited, it should create a good foundation to build off of.

If you have any comments or questions regarding anything covered by this tutorial, or have an idea for future posts, feel free the leave them below.

As stated before, all files generated during this tutorial can be found on my
<a class="out-link" href="https://github.com/LeeCombs/Fractal-Pixels-Tutorials/tree/master/Unity%202D%20Radial%20Progress" onclick="ga('send','event','click','github click','2D Radial');">
  Github Page Here.
</a>