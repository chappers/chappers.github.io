---
layout: post
category: web micro log
tags:
---

I have always [been interested in the Scene Completion using Millions of Photographs](http://graphics.cs.cmu.edu/projects/scene-completion/) paper and I was fortunate enough to learn enough computation photography to be able to replicate it in a rather naive way during my final project within my Georgia Tech course.

# Pipeline

What exactly does "scene completion" try to accomplish? It can be broken down innto the following three points:

1.  Select a suitable scene with the hole boundary from the original image. This is called the "Local Context"
2.  Search for the best scene from the semantically similar image based on alignment error.
3.  Match the two scenes together using graph cut seams, and blended using poisson blending.

![the-pipeline](/img/scene-completion/pipeline_scene-completion.png)

## Local Context

To compute the local context, we simply need to take the edges of our maks and dilate the edges to be fit our local context.

In Using the Opencv library, we can easily extract the local context information through the use of the dilate function.

![overlay-mask](/img/scene-completion/overlay-mask.png)

## Alignment Error

Alignment error can be as simple or complex depending on the metric used. here is mostly just the sum of squares (SSD)

```py
ssd = np.sum(np.square(imageA[imageB > 0] - imageB[imageB > 0]))
```

![best-scene-using-ssd](/img/scene-completion/best-scene-using-ssd.png)

## Graph Cut Seam Finding

The graph cut is implemented using SciPy Images library which implements the minimum cost path. The implementation and its use can be found in the [official documentation](http://scikit-image.org/docs/dev/api/skimage.graph.html#mcp). To find the optimal cut, we would have to simulate all possible cuts. In this report, I will only use four randomly selected cuts to be determine the optimal cut.

```py
costMCP = skimage.graph.MCP(cost_matrix, fully_connected=True)
cumpath, trcb = costMCP.find_costs(starts=start_pixel, ends=end_pixel)
path = costMCP.traceback(mask_point)
```

![graph-cut](/img/scene-completion/graph-cut.png)

### Poisson Blending

Blending was completed using the poisson blending which is implemented in OpenCV under “seamless cloning” with some simple alpha blending. This function and its parameters are explained in the OpenCV documentation and the [Learn OpenCV](http://www.learnopencv.com/seamless-cloning-using-opencv-python-cpp/) tutorial.

```py
output = cv2.seamlessClone(match_scene, orig_scene, mask, center, cv2.NORMAL_CLONE)
```

![poisson-blending](/img/scene-completion/poisson-blending.png)

## Result

![final-blend](/img/scene-completion/final-blend.png)
