---
layout: post
category : 
tags : 
tagline: 
---

Inspired by Dracula's geometric color palette, here is my attempt at creating a similar variation with Monokai:

```py
from PIL import Image, ImageColor, ImageDraw
import colorsys
import math
import numpy as np

if __name__ == "__main__":

    im = Image.new("RGB", (300, 300))
    radius = min(im.size) / 2.0
    cx, cy = im.size[0] / 2, im.size[1] / 2
    pix = im.load()
    small_radius = radius / 10
    draw = ImageDraw.Draw(im)

    for x in range(im.width):
        for y in range(im.height):
            rx = x - cx
            ry = y - cy
            s = (rx ** 2.0 + ry ** 2.0) ** 0.5 / radius
            if s <= 1.0:
                h = ((math.atan2(ry, rx) / math.pi) + 1.0) / 2.0
                rgb = colorsys.hsv_to_rgb(h, s, 1.0)
                pix[x, y] = tuple([int(round(c * 255.0)) for c in rgb])

    # draw circle around areas of the palette...
    colors = [
        ImageColor.getcolor("#e6db74", "HSV"),  # yellow
        ImageColor.getcolor("#fd971f", "HSV"),  # orange
        ImageColor.getcolor("#f92672", "HSV"),  # red
        # ImageColor.getcolor("#fd5ff0", "HSV"),  # magenta
        ImageColor.getcolor("#ae81ff", "HSV"),  # violet
        ImageColor.getcolor("#66d9ef", "HSV"),  # blue
        ImageColor.getcolor("#a1efe4", "HSV"),  # cyan
        ImageColor.getcolor("#a6e22e", "HSV"),  # green
    ]

    colors = [
        ImageColor.getcolor("#dede71", "HSV"),  # yellow
        ImageColor.getcolor("#e6ad6b", "HSV"),  # orange
        ImageColor.getcolor("#ef7c66", "HSV"),  # red
        ImageColor.getcolor("#E674B8", "HSV"),  # magenta
        ImageColor.getcolor("#c166ef", "HSV"),  # violet
        ImageColor.getcolor("#6abbec", "HSV"),  # blue # 747FE6, 66d9ef
        ImageColor.getcolor("#66EFC1", "HSV"),  # cyan
        ImageColor.getcolor("#94ef66", "HSV"),  # green
    ]

    def draw_circle(pt_list):
        draw.ellipse(pt_list, width=5, outline=(255, 255, 255, 255))
        draw.ellipse(pt_list, width=3, outline=(0, 0, 0, 255))

    def draw_at_point(col):
        col_np = np.array(list(col))
        curr_err = None
        tol = 0.1

        # find min...
        for x in range(im.width):
            for y in range(im.height):
                rx = x - cx
                ry = y - cy
                s = (rx ** 2.0 + ry ** 2.0) ** 0.5 / radius
                if s <= 1.0:
                    h = ((math.atan2(ry, rx) / math.pi) + 1.0) / 2.0
                    rgb = colorsys.hsv_to_rgb(h, s, 1.0)
                    curr_color = np.array([int(round(c * 255.0)) for c in rgb])

                    err = np.mean((col_np - curr_color) ** 2)
                    if curr_err is None:
                        curr_err = err
                    elif curr_err > err:
                        curr_err = min(err, curr_err)
        print(f"best err: {curr_err}")
        for x in range(im.width):
            for y in range(im.height):
                rx = x - cx
                ry = y - cy
                s = (rx ** 2.0 + ry ** 2.0) ** 0.5 / radius
                if s <= 1.0:
                    h = ((math.atan2(ry, rx) / math.pi) + 1.0) / 2.0
                    rgb = colorsys.hsv_to_rgb(h, s, 1.0)
                    curr_color = np.array([int(round(c * 255.0)) for c in rgb])

                    err = np.mean((col_np - curr_color) ** 2)
                    if err < curr_err + tol:
                        pt = [
                            (x - small_radius, y - small_radius),
                            (x + small_radius, y + small_radius),
                        ]
                        draw_circle(pt)
                        return

    draw.ellipse([(0, 0), (im.width, im.height)], width=5, outline=(255, 255, 255, 255))
    for col in colors:
        draw_at_point(col)

    im.show()

```

Now would I actually use this? I'm not too sure. Sticking with the defaults rather than moving things around is generally a wise idea. Though some things just have "bad" defaults (like the default macOS terminal). I might play around and see how far I get, but deviating too far from monokai defaults might be more pain than its worth, especially when continually resetting machines etc. 