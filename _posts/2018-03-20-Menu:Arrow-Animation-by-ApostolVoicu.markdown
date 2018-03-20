---
layout: post
title: Menu/Arrow Animation by Apostol Voicu
date: 2018-03-20
description: Implementing Menu/Arrow Animation by Apostol Voicu using canvas apis in Android 
img: 2018_03_20_cover.gif
comments: true
---


Path animation is very fascinating! Today I want to share how path animation can be implemented.

[See the original Menu/Arrow Animation by Apostol Voicu][url-original]

I implemented this animation and share the full source code in [here][url-my-github]


First, let's look at the big flow of implementing pass animations.
1. Draw a path to animate.
2. Start the path animation.


## Draw a path to animate.

I have declared a member variable called `mSrcPath` which contains the information to draw on the canvas.
The `onSizeChanged()` method, which is called when the size changes, creates a path based on the view size.
The contents of the path are defined in the `makeMenuArrowPath()` method.
I then used the `onDraw()` method to draw the path.

{% highlight java %}
public class MenuArrowAnimationButtonTest extends AppCompatButton {

    private Path mSrcPath = new Path();
    private Paint mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);

     @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);

        mCanvasWidth = w;
        mCanvasHeight = h;

        makeMenuArrowPath();
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        mPaint.setStrokeWidth(10f);
        mPaint.setColor(0xff43479f);
        canvas.drawPath(mSrcPath, mPaint);
    }
}
{% endhighlight %}

![Step 1. Draw Path]({{site.baseurl}}/assets/img/2018_03_20_draw_path.png)


The implementation part of the `makeMenuArrowPath()` method is shown below. 
It's just a part of creating a path, and It's just modularization the path you want to animate.

{% highlight java %}

public class MenuArrowAnimationButtonTest extends AppCompatButton {

    ...

    private void makeMenuArrowPath() {

        final float LEFT_CIRCLE_DIAMETER  = ((float) mCanvasHeight)*(2f/3f);
        final float RIGHT_CIRCLE_DIAMETER = mCanvasHeight;

        final float LEFT_CIRCLE_RADIUS    = LEFT_CIRCLE_DIAMETER/2f;
        final float RIGHT_CIRCLE_RADIUS   = RIGHT_CIRCLE_DIAMETER/2f;

        mCX = LEFT_CIRCLE_DIAMETER + ((mCanvasWidth -LEFT_CIRCLE_DIAMETER-RIGHT_CIRCLE_DIAMETER)/2f);

        mSrcPath.moveTo(mCX, LEFT_CIRCLE_DIAMETER);
        RectF leftRound = new RectF();
        leftRound.set(0, 0, LEFT_CIRCLE_DIAMETER, LEFT_CIRCLE_DIAMETER);
        mSrcPath.arcTo(leftRound, 90, 180);

        mSrcPath.lineTo(mCanvasWidth -RIGHT_CIRCLE_DIAMETER, 0);

        leftRound.set(mCanvasWidth -RIGHT_CIRCLE_DIAMETER, 0, mCanvasWidth, mCanvasHeight);
        mSrcPath.arcTo(leftRound, 270, 180);

        mSrcPath.lineTo(mCX, mCanvasHeight);
        mSrcPath.lineTo(LEFT_CIRCLE_RADIUS, mCanvasHeight /2f);
        mSrcPath.lineTo(mCX, 0);
    }

}
{% endhighlight %}


## Start the path animation.

Animation will start when you call the `startPathAnimation()` method.
I passed the beginning of the path and the end of the path to the parameters of the `ObjectAnimator.ofFloat()` method.
Note that the start and end values are shifted backward.
The reason for this is related to the behavior of the `DashPathEffect` class, which is the heart of Path animation.
More information can be found on the [Romain Guy's blog](http://www.curious-creature.com/2013/12/21/android-recipe-4-path-tracing/).

I overriding `onDraw()` method that called `invalidate()` on the `setPhase()` method, which is called whenever the value changes.

And I calls `invalidate()` method on the `setPhase()` method, which is called whenever the value changes. 

{% highlight java %}

private float mPathLength;
private float mPhase;

public void startPathAnimation() {

    // Measure the path
    PathMeasure measure = new PathMeasure(mSrcPath, false);
    mPathLength = measure.getLength();

    Animator animator = ObjectAnimator.ofFloat(this, "phase", mPathLength, 0f);
    animator.setDuration(3000);
    animator.start();
}

public void setPhase(float phase) {
    mPhase = phase;
    invalidate();
}

public float getPhase() {
    return mPhase;
}

{% endhighlight %}



As the result, the `onDraw()` method animates the path by drawing a new value.

{% highlight java %}

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    mPaint.setStrokeWidth(10f);
    mPaint.setColor(0xff43479f);

    // Apply the dash effect
    PathEffect effect = new DashPathEffect(new float[] { mPathLength, mPathLength }, mPhase);
    mPaint.setPathEffect(effect);

    canvas.drawPath(mSrcPath, mPaint);
}

{% endhighlight %}


The result of executing the above code is the same as the video below.

<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='//www.youtube.com/embed/oC97AV3EaWE' frameborder='0' allowfullscreen></iframe></div>



Now, let's implement it as closely as possible to [Apostol Voicu's work][url-original] we have seen above.
First, let's change each corner of the path to look round.


* `Paint.setStrokeCap(Paint.Cap.ROUND)` method call rounds the end of the path.
* `Paint.setStrokeJoin (Paint.Join.ROUND)` method call rounds the parts of the path that are connected to each other.

{% highlight java %}

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    mPaint.setStrokeCap(Paint.Cap.ROUND);
    mPaint.setStrokeJoin(Paint.Join.ROUND);
    mPaint.setStrokeWidth(10f);
    mPaint.setColor(0xff43479f);

    // Apply the dash effect
    PathEffect effect = new DashPathEffect(new float[] { mPathLength, mPathLength }, mPhase);
    mPaint.setPathEffect(effect);

    canvas.drawPath(mSrcPath, mPaint);
}

{% endhighlight %}


Compared to the first image, you can see that the end of the path and the connected parts have changed smoothly.


![rounded_join_cap_path]({{site.baseurl}}/assets/img/2018_03_20_rounded_join_cap.png)



Now, let's look briefly at what I have implemented to implement [the original work][url-original].
[The original work][url-original] will not be animated from the beginning to the complete form as seen in the above video.
As the segments move, the part at the end is finally drawn on the screen.
In order to implement this animation, I have removed the existing method and separated the `Animator` into two.

1. `arrowAnimator` and `menuAnimator` animators to specify where the current path should be drawn.
2. `pathLengthAnimator` animator to specify the length of the current path and specifies how far to draw it.

{% highlight java %}

 private void animateMenuArrow(boolean isArrowStatus)
    {
        makeMenuArrowPath();

        AnimatorSet set = new AnimatorSet();

        if(isArrowStatus)
        {
            START_MENU_BAR_VALUE = TOTAL_PATH_LENGTH * 0.242f;
            END_MENU_BAR_VALUE = START_MENU_BAR_VALUE + MENU_OVER_LENGTH;

            final float startD = -(mOffSet*2);

            ObjectAnimator arrowAnimator = ObjectAnimator.ofFloat(this, "arrowPhase", 
                                                          startD, TOTAL_PATH_LENGTH - ARROW_LENGTH -mOffSet);
            arrowAnimator.setInterpolator(new OvershootInterpolator(TENSION));

            ObjectAnimator pathLengthAnimator = ObjectAnimator.ofFloat(this, "pathLength", 
                                                               MENU_UNDER_LENGTH *2, ARROW_LENGTH);

            set.playTogether(arrowAnimator, pathLengthAnimator);
            set.setDuration(mAnimationDuration);

        }
        else
        {
            START_MENU_BAR_VALUE = TOTAL_PATH_LENGTH * 0.27f;
            END_MENU_BAR_VALUE = START_MENU_BAR_VALUE + MENU_OVER_LENGTH;

            PathMeasure measure = new PathMeasure(mToMenuPath, false);

            float startD = measure.getLength() - ARROW_LENGTH;

            float endD = mOffSet;

            ObjectAnimator menuAnimator = ObjectAnimator.ofFloat(this, "menuPhase", startD, endD);
            menuAnimator.setInterpolator(new OvershootInterpolator(TENSION));

            ObjectAnimator pathLengthAnimator = ObjectAnimator.ofFloat(this, "pathLength", 
                                                               ARROW_LENGTH, MENU_UNDER_LENGTH);
            pathLengthAnimator.setInterpolator(new DecelerateInterpolator(1.5f));

            set.playTogether(menuAnimator, pathLengthAnimator);
            set.setDuration(mAnimationDuration);
        }

        set.start();
    }

{% endhighlight %}


The `setMenuPhase()` method, which is called whenever the value changes in the `menuAnimator`, sets the current path to be drawn.
Within this method, I use the `PathMeasure.getSegment()` method to get the path for the start and end points and save it in `mCurPath`.
And in [the original work][url-original], when drawing the menu path, the top part of the menu should be drawn as well, so I checked the status as below code and stored it in `mCurPath` too.
I then draw the `mCurPath` inside the `onDraw()` method by calling the `invalidate()` in `setMenuPhase()`method.

{% highlight java %}

public void setMenuPhase(float phase) {
    mPhase = phase;

    PathMeasure pm = new PathMeasure();
    pm.setPath(mToMenuPath, false);

    mCurPath.reset();
    pm.getSegment(phase, phase + mCurPathLength, mCurPath,true);

    if(phase < START_MENU_BAR_VALUE)
    {
        pm.getSegment(START_MENU_BAR_VALUE, END_MENU_BAR_VALUE, mCurPath,true);
    }
    else if (phase < END_MENU_BAR_VALUE)
    {
        pm.getSegment( phase, END_MENU_BAR_VALUE, mCurPath,true);
    }

    invalidate();
}

{% endhighlight %}


For reference, I gave an extra pass to the path to give it an overshooting effect like [the original][url-original].

When changing to the menu and changing to Arrow, the paths overshooting each other are different.
For convenience of calculation and maintainability, I divided the path into two.
It is helpful to refer to `MenuArrowAnimationButton.java`.

The completed animation is as follows.

<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='//www.youtube.com/embed/euGaUdCzsxo' frameborder='0' allowfullscreen></iframe></div>


You can get the full source code of `Menu/Arrow Animation by Apostol Voicu` on [my GitHub][url-my-github]

Thanks for reading the article.

[url-original]: https://dribbble.com/shots/2550799-Menu-Arrow-Animation
[url-my-github]: https://github.com/laewoong/MenuArrowAnimation