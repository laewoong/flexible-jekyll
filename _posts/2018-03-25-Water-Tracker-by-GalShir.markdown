---
layout: post
title: Water Tracker by Gal Shir
date: 2018-03-25
description: Implementing Water Tracker by Gal Shir using canvas apis in Android 
img: 2018_03_25_cover.gif
comments: true
---


In this post, I want to share a wave motion implementation.
The original url of the motion to be implemented today is as follows.

[See the original Water Tracker by Gal Shir][url-original]

I implemented this motion and share the full source code in [here][url-my-github]

Wave motion is simpler than you might think. First draw the wave path and then draw the path by moving the starting point to the right.



# Draw Wave Path.

In the `WaterViewBasic` class, I declared the required member variables to draw the wave path.
When `the onSizeChanged()` method is called, the `makeWavePath()` method creates a wave path.

{% highlight java %}
public class WaterViewBasic extends View {

    private static final int VELOCITY = 30;

    private Path    mPath;
    private Paint   mPaint;

    private int mCanvasWidth;
    private int mCanvasHeight;
    
    public WaterViewBasic(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    private void init() {
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.BLUE);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(10f);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        
        mCanvasWidth = w;
        mCanvasHeight = h;

        makeWavePath();
    }
}
{% endhighlight %}


In Android, curves can be drawn with three methods:
* Path.arcTo()
* Path.cubicTo()
* Path.quadTo()


In this post, I will draw a wave path using the second `Path.cubicTo()` method.

{% highlight java %}
private void makeWavePath() {
    int halfWidth = mCanvasWidth/2;
    int halfHeight = mCanvasHeight/2;

    mPath = new Path();
    mPath.moveTo(0, halfHeight);
    float offset = 200f;
    mPath.cubicTo(halfWidth, halfHeight-offset,
            halfWidth, halfHeight+offset,
            mCanvasWidth, halfHeight);

    mPath.lineTo(mCanvasWidth, mCanvasHeight);
    mPath.lineTo(0, mCanvasHeight);
    mPath.close();
}

{% endhighlight %}


Draw the path you created above in the `onDraw()` method.

{% highlight java %}
@Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        canvas.drawPath(mPath, mPaint);
    }
{% endhighlight %}

When you execute the above code, you can see that the wave path is drawn as below.

![Draw Wave Path]({{site.baseurl}}/assets/img/2018_03_25_wave_path.png)

# Add motion to Wave.

Now let's put the motion in Wave pass.

Here's how I used it to get motion:
1. Draw the path from the starting point.
2. Move the starting point by adding speed to the starting point.
3. Draw the path from the moved starting point.
4. Draw the path at the beginning as long as it is cut behind.


To implement the above behavior, declare a variable that stores the starting point of the path.
And the `onDraw()` method moves the starting point and animates.

{% highlight java %}
public class WaterViewBasic extends View {

    ...
    private static final int VELOCITY = 30;
    private int mStartX;

    private void init() {
        
        ...
    
        mStartX = 0;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        float offset = mCanvasWidth - mStartX;
        mPath.offset(-offset, 0);
        canvas.drawPath(mPath, mPaint);
        mPath.offset(mCanvasWidth, 0);
        canvas.drawPath(mPath, mPaint);
        mPath.offset(-mStartX, 0);

        mStartX = mStartX +VELOCITY;

        if(mStartX > mCanvasWidth) {
            mStartX = 0;
        }

        invalidate();
    }
{% endhighlight %}

If you execute the above code, you can implement the same motion as the below video.

<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='//www.youtube.com/embed/BrY_yFDEDu4' frameborder='0' allowfullscreen></iframe></div>


Using this basic motion, I have implemented [the original work][url-original] as shown below.

<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='//www.youtube.com/embed/yiwvacWQyd0' frameborder='0' allowfullscreen></iframe></div>


The Basic wave motion source is in `WaterViewBasic.java`,
The source that implements [the original work][url-original] can be found in `WaterView.java`.

You can get the full source code of `Water Tracker by Gal Shir` on [my GitHub][url-my-github]

Thanks for reading the article.

[url-original]: https://dribbble.com/shots/2396543-Water-Tracker
[url-my-github]: https://github.com/laewoong/WaterTracker