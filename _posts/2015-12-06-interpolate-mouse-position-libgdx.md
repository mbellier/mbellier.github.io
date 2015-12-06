---
layout: post
title:  "Interpolate the mouse position in LibGDX"
date:   2016-12-06
categories: dev libgdx
---

[LibGDX](https://libgdx.badlogicgames.com/), a Java game development framework, provides an interface, [InputProcessor](https://github.com/libgdx/libgdx/wiki/Event-handling#input-processor), to handle input events. Three methods can be implemented to listen to mouse events: 

- `touchDown(int x, int y, int pointer, int button)`
- `touchUp(int x, int y, int pointer, int button)`
- `touchDragged(int x, int y, int pointer)`

Because these events are triggered punctually, the logic needs to be changed for continuous movements. Let's consider the example of a drawing tool. Simply plotting a point when the drag event is triggered only produces a sequence of dots:

![dashed draw]({{ site.baseurl }}/assets/2015-12-06-draw1.png){: .img-smaller} 

### Interpolation

In order to get a continuous line, a segment has to be drawn from the previous position of the mouse to the current position. One solution is to use [Bresenham's line algorithm](https://en.wikipedia.org/wiki/Bresenham's_line_algorithm). It revolves around the idea that, during the interpolation, one of the coordinates is represented by a decimal value, which is incremented according to the slope of the line, while the other is incremented by one. Pixels can be drawn along the line, one at a time, until the position in pixels matches the rounded decimal position. 

In addition, I wanted to add some thickness to the line, by drawing patches of pixels instead of a single one. This produces the following output:

![continous draw]({{ site.baseurl }}/assets/2015-12-06-draw2.png){: .img-smaller}

### Code example

Notations:

* `(x0; y0)` represents the previous position of the mouse,
* `(x1; y1)` the current position,
* `plot(x, y)` is a placeholder for drawing a pixel at the position (x; y).

{% highlight java %}
// draw line from (x0; y0) to (x1; y1)
public void drawLine(int x0, y0, x1, y1){
  int directionX = x1 - x0 > 0 ? 1 : -1;
  int directionY = y1 - y0 > 0 ? 1 : -1;

  double deltaX = x1 - x0;
  double deltaY = y1 - y0;

  // case of vertical line
  if (deltaX == 0) {
    for (int y = y0; Math.abs(y1 - y) > 0; y += directionY) {
      plot(x1, y);
    }
  } else {
    // the error represents the difference between 
    // the decimal position and the pixel position
    double error = 0;
  
    double slope = Math.abs(deltaY / deltaX);
  
    int y = y0;
    for (int x = x0; Math.abs(x1 - x) > 0; x += directionX) {
      plot(x, y);
      error += slope;

      // if there is enough difference to catch up,
      // draw pixel(s) along the vertical axis
      while (error >= 0.5) {
        plot(x, y);
        plot(x, y + directionY); // to add thickness
      
        y += directionY;
        error -= 1.0;
      }
    }
  }
}
{% endhighlight %}

### Drawing on a tiled map

To draw on a [tiled map](https://libgdx.badlogicgames.com/nightlies/docs/api/com/badlogic/gdx/maps/tiled/TiledMapTileLayer.html), the screen coordinates have to be transformed relatively to the camera, then mapped to a tile. These steps provide the values used in the previous example. The implementation in LibGDX's `InputProcessor` would look like this:

{% highlight java %}
int x0, y0, x1, y1;

@Override
public boolean touchDragged(int screenX, int screenY, int pointer) {
  Vector3 position = camera.unproject(position.set(screenX, screenY, 0));
  x0 = x1;
  y0 = y1;
  x1 = Math.round(position.x + TILE_OFFSET_X) / (TILE_W + TILE_MARGIN);
  y1 = Math.round(position.y + TILE_OFFSET_Y) / (TILE_H + TILE_MARGIN);
  
  drawLine(x0, y0, x1, y1);
}
{% endhighlight %}

Instead of screen coordinates, the `plot(int x, int y)` method, used in `drawLine(~)`, has to use the coordinates of a tile to change its texture.
