---
layout: post
category: web micro log
tags:
---

I'm currently taking the Georgia Tech course on computational photography and I thought I'll give some of the lecture materials a go, more specifically the sections on blending images.

In brief, if we convert all the image channels (RGB) to be in the range between \\(0\\) and \\(1\\), we can then can apply the following blending approaches to images \\(a\\) and \\(b\\).

- Divide the two images (brightens)
- Addition (brightens, but adds too many whites)
- Subtract (darkens, but too many blacks)
- Darken, \\(f(a,b) = min(a,b)\\)
- Lighten, \\(f(a,b) = max(a,b)\\)
- Multiply (brightens)
- Screen \\(f(a,b) = 1-(1-a)(1-b)\\)
- Overlay \\(f(a,b) = 2ab\\), if \\(a < 0.5\\) and \\(f(a,b) = 1-2(1-a)(1-b)\\) if \\(a \geq 0.5\\)

How would we implement this using [OpenCV](http://www.opencv.org/) and Python?

Without using the implementation of `cv2` library in Python, but using pure `numpy` to understand what `cv2` does behind the scenes, means that we have to take into consideration that the minimum and maximum colours may extend over our "range" of colours, so we have to apply minimums and maximums to whatever operation we perform.

Starting with these images,

![keyfob](/img/opencv-blending/keyfob_orig.png)

![duck](/img/opencv-blending/duck_orig.png)

We can combine them together using the algorithms to yield images (where "keyfob" is the first source image and the "duck" image is the second source image) as follows:

![div](/img/opencv-blending/division.jpg)

![add](/img/opencv-blending/addition.jpg)

![sub](/img/opencv-blending/subtract1.jpg)

![dark](/img/opencv-blending/darken.jpg)

![lighten](/img/opencv-blending/lighten.jpg)

![multiply](/img/opencv-blending/multiply.jpg)

![screen](/img/opencv-blending/screen.jpg)

![overlay](/img/opencv-blending/overlay1.jpg)

In this scenario for some of the images, order does matter! Think about which one of these that would be the case.

The Python code used looks like the following.

```py
import numpy as np  # NumPy: for direct image (matrix) manipulation
import cv2  # OpenCV 2.x.x: for more advanced functions, utilities

img1 = cv2.imread('keyfob_orig.png')
img2 = cv2.imread('duck_orig.png')

img1 = cv2.imread('keyfob_orig.png')
img2 = cv2.imread('duck_orig.png')

img_white = np.ones(img1.shape, np.float32) * 255.0
img_black = np.zeros(img1.shape, np.float32)

im1 = np.divide(img1.astype(np.float32), img_white)
im2 = np.divide(img2.astype(np.float32), img_white)

img_add = np.minimum((np.add(im1, im2)*255.0), img_white).astype(np.uint8)

img_mul = np.maximum(np.minimum((np.multiply(im1, im2)*255.0), img_white), img_black).astype(np.uint8)

img_sub1 = np.maximum(np.minimum((np.subtract(im1, im2)*255.0), img_white), img_black).astype(np.uint8)

img_sub2 = np.maximum(np.minimum((np.subtract(im2, im1)*255.0), img_white), img_black).astype(np.uint8)

img_div = np.maximum(np.minimum((np.divide(im1, im2)*255.0), img_white), img_black).astype(np.uint8)

img_darken = np.maximum(np.minimum((np.minimum(im1, im2)*255.0), img_white), img_black).astype(np.uint8)

img_lighten = np.maximum(np.minimum((np.maximum(im1, im2)*255.0), img_white), img_black).astype(np.uint8)

img_screen = np.maximum(np.minimum((1-np.multiply(1-im1, 1-im2))*255.0, img_white), img_black).astype(np.uint8)

im_overlay = np.zeros(img1.shape, np.float32)
im_overlay[im1 < 0.5] += 2*np.multiply(im1[im1 < 0.5], im2[im1 < 0.5])
im_overlay[im1 >= 0.5] += 1-2*np.multiply(1-im1[im1 >= 0.5], 1-im2[im1 >= 0.5])

img_overlay1 = np.maximum(np.minimum(im_overlay*255.0, img_white), img_black).astype(np.uint8)

im_overlay = np.zeros(img1.shape, np.float32)
im_overlay[im2 < 0.5] += 2*np.multiply(im1[im2 < 0.5], im2[im2 < 0.5])
im_overlay[im2 >= 0.5] += 1-2*np.multiply(1-im1[im2 >= 0.5], 1-im2[im2 >= 0.5])

img_overlay2 = np.maximum(np.minimum(im_overlay*255.0, img_white), img_black).astype(np.uint8)

```
