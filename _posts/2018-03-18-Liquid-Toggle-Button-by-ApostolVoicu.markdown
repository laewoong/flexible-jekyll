---
layout: post
title: Liquid Toggle Button by Apostol Voicu
date: 2018-03-18
description: Implementing Liquid toggle button by Apostol Voicu using canvas apis in Android 
img: 2018_03_18_cover.gif
comments: true
---


In dribble, There are many things that inspire me.
I am going to introduce one simple but very interesting design today.

[See the Liquid Toggle Button by Apostol Voicu](https://dribbble.com/shots/2853092-Liquid-Toggle-Button)

And I implemented the button and share the full source code in [here][github-url-liguid-toggle-button]

There were two things that were difficult to implement.
1. Punching a hole in button.
2. What a Thumb part disappear when applied the gooey effect.


## Punching a hole in button.

I originally tried to clip canvas as a size of a thumb. But that way could not apply 'Gooey Effect' [in my knowledge][github-url-gooey-effect]. 
So I changed original code as following code that just makes two paths. 

{% highlight java %}
private void makeTogglePath() {

    // draw rounded rect track
    mFinalPath.reset();

    RectF leftRound = new RectF();
    leftRound.set(0, 0, canvasHeight, canvasHeight);
    mFinalPath.arcTo(leftRound, 90, 180);
    mFinalPath.lineTo(canvasWidth-TRACK_RADIUS, 0);

    float lineWidth = canvasWidth-(TRACK_RADIUS*2);
    leftRound.set(lineWidth, 0, canvasWidth, canvasHeight);
    mFinalPath.arcTo(leftRound, 270, 180);
    mFinalPath.lineTo(TRACK_RADIUS, canvasHeight);
    mFinalPath.close();

    // rotate for animation
    Matrix tractMatrix = new Matrix();
    tractMatrix.postRotate(-mThumbRotation, canvasWidth/2f, canvasHeight/2f);
    mFinalPath.transform(tractMatrix);

    // draw thumb
    mThumbPath.reset();

    float thumbCenter = 2 * THUMB_RADIUS;
    mThumbPath.addCircle(thumbCenter, thumbCenter, mCurThumbRadius, Path.Direction.CW);
    
    // rotate for animation
    Matrix thumbMatrix = new Matrix();
    thumbMatrix.postRotate(mThumbRotation, canvasWidth/2,canvasHeight/2);
    mThumbPath.transform(thumbMatrix);

    ...
}
{% endhighlight %}

The problem was that I only want to fill a track part except for a thumb part.
To make this, I added a code.

{% highlight java %}
private void makeTogglePath() {

    ...

    // make one path
    mFinalPath.op(mThumbPath, Path.Op.DIFFERENCE);

    ...
}
{% endhighlight %}


`Path.op()` method() makes it possible to fill only a track part that I wanted.
The second parameter will only fills the original path except for a path that be passed at first parameter when I fill the path on canvas.

<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='//www.youtube.com/embed/gczvrvHb9JU' frameborder='0' allowfullscreen></iframe></div>


You can think that the design has been implemented somewhat well with the code up to here.
But my goal is to do almost exactly the same thing.

So I applied [the Gooey Effect][github-url-gooey-effect] to make a thumb and a track seem to be connected together.

<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='//www.youtube.com/embed/t9fNRcPKBp0' frameborder='0' allowfullscreen></iframe></div>


## What a Thumb part disappear when applied the gooey effect.

During the animation, The thumb was disappeared when the thumb is fully in the track.
Through debugging, I found that `DiscretePathEffect` path effect caused this result.
So I started to search a solution that thumb is not disappearing when the thumb is fully in the track.
And My solution is that use `PathMeasure.nextContour()` method().
This method returns `true` When a `Path` has no other independent paths.
So I only applied gooey effect when a path does not include other independent paths.

{% highlight java %}
private void makeTogglePath() {

    ...

    PathMeasure pm = new PathMeasure(mFinalPath, true);
    final float SEGMENT_NUM = 20f;

    // If you delete or move under line, pm.nextContour() method doesn't work I intended.
    final float SEGMENT_LENGHT = pm.getLength()/SEGMENT_NUM;

    if(pm.nextContour()) {

        mTrackPaint.setPathEffect( null);
    }
    else {

        DiscretePathEffect discretePathEffect= new DiscretePathEffect(SEGMENT_LENGHT, 0);
        CornerPathEffect cornerPathEffect = new CornerPathEffect(50);
        ComposePathEffect pathEffect = new ComposePathEffect(cornerPathEffect, discretePathEffect);

        mTrackPaint.setPathEffect( pathEffect);
    }
}
{% endhighlight %}

When the `PathMeasure.nextContour()` return true, I set `null` to the path effect to remove the previous path effect.
With this solution, I was able to implement the original design in much the same way.


You can look demo video below.

<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='//www.youtube.com/embed/0YN6adajz-U' frameborder='0' allowfullscreen></iframe></div>


You can get the full source code of `Liquid Toggle Button by Apostol Voicu` on [my GitHub][github-url-liguid-toggle-button]

Thanks for reading the article.

[awsome-youtube]: https://youtu.be/H05mF0qrBVA
[github-url-gooey-effect]: https://github.com/laewoong/GooeyEffectView
[github-url-liguid-toggle-button]: https://github.com/laewoong/LiquidToggleButton