---
layout: post
title: Gooey Effect Using Canvas API In Android
date: 2018-03-17
description: Implementing Gooey effect using canvas apis in Android # Add post description (optional)
img: 2018_03_17_cover.gif
comments: true
---
2 Years ago, I tried to implement gooey effect using a web way.

Gooey Effect in web uses 3 steps.
1. Draw Two Circles.
2. Blur the Circles.
3. Contrast the Circles.

In Android, However, It didn't work. The second step that blurs two circles is too much slow and I failed to make it.
But 2 Weeks ago, I watched an awesome YouTube [droidcon SF 2017 - Canvas Drawing for Fun and Profit][awsome-youtube] that inspired me and I try again.
On YouTube, I got some idea from Drawing doughnut icing part and Today I introduce implementing gooey effect using canvas API in android.

There are also three steps as the web version, though the way different.
1. Draw two circle paths.
2. Union two paths to one path.
2. Change the path to gooey effect path.

At first, I declared some member variables into my custom view called [GoogeyEffectView][github-url-gooey-effect].
{% highlight java %}
public class GooeyEffectView extends View {

    private Path mLeftCirclePath;
    private Path mRightCirclePath;

    private Paint mPaint;

    private int mCenterX;
    private int mCenterY;

    public GooeyEffectView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);

        mLeftCirclePath = new Path();
        mRightCirclePath = new Path();

        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.BLUE);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(10f);
    }

    ...
}
{% endhighlight %}

There are two paths `mLeftCirclePath`, `mRightCirclePath` that represent left side circle and right side circle on canvas.
`mPaint` is set as stork to see obviously each step changes.

Then I set center values in `onSizeChanged()` method to draw circles on the center of the screen.

{% highlight java %}
@Override
protected void onSizeChanged(int w, int h, int oldw, int oldh) {
    super.onSizeChanged(w, h, oldw, oldh);

    mCenterX = w/2;
    mCenterY = h/2;

    initGooeyPath();
}
{% endhighlight %}

In `initGooeyPath()` method(), Gooey Effect will be build up.


## Draw two circle paths.

In `initGooeyPath()` method(), I reset paths to redraw circle path in the center of the screen when screen size be changed and overriding `onDraw()` method to draw circles on canvas.

{% highlight java %}
public void initGooeyPath() {

    mLeftCirclePath.reset();
    mRightCirclePath.reset();

    final float RADIUS = 100f;
    final float GAP = 75f;

    mLeftCirclePath.addCircle((mCenterX - GAP), mCenterY, RADIUS, Path.Direction.CW);
    mRightCirclePath.addCircle((mCenterX + GAP), mCenterY, RADIUS, Path.Direction.CW);
}

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    canvas.drawPath(mLeftCirclePath, mPaint);
    canvas.drawPath(mRightCirclePath, mPaint);
}
{% endhighlight %}

![Step 1. Draw Two Circle]({{site.baseurl}}/assets/img/2018_03_17_gooey_step1.png)


## Union two paths to one path.

In `initGooeyPath()` method(), I use `Path.Op.UNION` enum to make two paths into one path.
And draw only one path.

{% highlight java %}
public void initGooeyPath() {

    mLeftCirclePath.reset();
    mRightCirclePath.reset();

    final float RADIUS = 100f;
    final float GAP = 75f;

    mLeftCirclePath.addCircle((mCenterX - GAP), mCenterY, RADIUS, Path.Direction.CW);
    mRightCirclePath.addCircle((mCenterX + GAP), mCenterY, RADIUS, Path.Direction.CW);
    mLeftCirclePath.op(mRightCirclePath, Path.Op.UNION);
}

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    canvas.drawPath(mLeftCirclePath, mPaint);
    //canvas.drawPath(mRightCirclePath, mPaint);
}
{% endhighlight %}

![Step 2. Make One Path]({{site.baseurl}}/assets/img/2018_03_17_gooey_step2.png)


## Change the path to gooey effect path.

Finally, I use `PathEffect` to make a gooey effect.
* First, Split the circle path into a specific number of a segment using `DiscretePathEffect`.
* Second, Make the segmented circle path into a rounded circle path using `CornerPathEffect`.
* Third, Combine two effects to one effect using `ComposePathEffect`

And set the effect to the `mPaint` using `setPathEffect()` method.

{% highlight java %}
public void initGooeyPath() {

    mLeftCirclePath.reset();
    mRightCirclePath.reset();

    final float RADIUS = 100f;
    final float GAP = 75f;

    mLeftCirclePath.addCircle((mCenterX - GAP), mCenterY, RADIUS, Path.Direction.CW);
    mRightCirclePath.addCircle((mCenterX + GAP), mCenterY, RADIUS, Path.Direction.CW);
    mLeftCirclePath.op(mRightCirclePath, Path.Op.UNION);

    final float SEGMENT_NUM = 20f;
    PathMeasure pm = new PathMeasure(mLeftCirclePath, true);
    DiscretePathEffect discretePathEffect= new DiscretePathEffect(pm.getLength()/SEGMENT_NUM, 0);

    CornerPathEffect cornerPathEffect = new CornerPathEffect(50);

    ComposePathEffect pathEffect = new ComposePathEffect(cornerPathEffect, discretePathEffect);

    mPaint.setPathEffect( pathEffect);
}
{% endhighlight %}

![Step 3. Make Gooey Path]({{site.baseurl}}/assets/img/2018_03_17_gooey_step3.png)


And This is final code.

{% highlight java %}
public class GooeyEffectView extends View {

    private Path mLeftCirclePath;
    private Path mRightCirclePath;

    private Paint mPaint;

    private int mCenterX;
    private int mCenterY;

    public GooeyEffectView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);

        mLeftCirclePath = new Path();
        mRightCirclePath = new Path();

        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.BLUE);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(10f);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);

        mCenterX = w/2;
        mCenterY = h/2;

        initGooeyPath();
    }

    public void initGooeyPath() {

        mLeftCirclePath.reset();
        mRightCirclePath.reset();

        final float RADIUS = 100f;
        final float GAP = 75f;

        mLeftCirclePath.addCircle((mCenterX - GAP), mCenterY, RADIUS, Path.Direction.CW);
        mRightCirclePath.addCircle((mCenterX + GAP), mCenterY, RADIUS, Path.Direction.CW);
        mLeftCirclePath.op(mRightCirclePath, Path.Op.UNION);

        final float SEGMENT_NUM = 20f;
        PathMeasure pm = new PathMeasure(mLeftCirclePath, true);
        DiscretePathEffect discretePathEffect= new DiscretePathEffect(pm.getLength()/SEGMENT_NUM, 0);

        CornerPathEffect cornerPathEffect = new CornerPathEffect(50);

        ComposePathEffect pathEffect = new ComposePathEffect(cornerPathEffect, discretePathEffect);

        mPaint.setPathEffect( pathEffect);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        canvas.drawPath(mLeftCirclePath, mPaint);
    }
}
{% endhighlight %}


More Interactive, You just add Animation such as moving the x position of circles like below video.
[![Gooey effect video](https://img.youtube.com/vi/jcDgF22p3kw/0.jpg)](https://youtu.be/jcDgF22p3kw)

Or you can add resizing and an elastic `Interpolator` to make fascinating like below video.
[![Gooey effect video](https://img.youtube.com/vi/cF_gzy395TU/0.jpg)](https://youtu.be/cF_gzy395TU)


You can get the full source code of `GooeyEffectView` on [my GitHub][github-url-gooey-effect]

Thanks for reading the article.

[awsome-youtube]: https://youtu.be/H05mF0qrBVA
[github-url-gooey-effect]: https://github.com/laewoong/GooeyEffectView