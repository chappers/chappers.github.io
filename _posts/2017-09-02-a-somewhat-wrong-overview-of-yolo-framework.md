---
layout: post
category:
tags:
tagline:
---

The [YOLO (you only look once) framework](https://pjreddie.com/darknet/yolo/) is a cutting edge approach to object detection in images. In this post I thought I'll go through in high level detail how it works, and how we might build our own YOLO architecture (if we wish). The aim of this post is not to provide a comprehensive view, but rather demonstrate some of the ideas that might be different for users who don't have a background in object detection.

At a high level YOLO is relatively simple. There are two steps in the training process:

1.  Train the network for classification
2.  Train the network for detection

These two steps are sequential, and the idea is that we would first train the network for classification, and then transfer the learning into a detection pipeline. As this can still all reside in a single neural network by the end of the second step - this will allow a single neural network to perform both classification and detection, without running multiple passes over the same image which many other implementations need to do.

## Training for Classificaton

In the first step, the neural network is constructed in the same way that many other image classification frameworks are constructed to detect images.

Convolution layers are used to create feature maps which are then mapped to dense layers which will perform and provide the relevant predictions.

That is the ouput of the neural network is:

- Probability of each class being classified

## Training for Detection

To build a model for detection, we simply take the classification model that was first created and resume training from the convolution layers. From this step we can add more convolution layers if we desire, with the prediction goal being the prediction of the coordinates of a bounding box and the relevant probabilities of the classes.

The output of this neural network would be:

- center of the image (the x, y coordinates)
- Width and height of the image
- Confidence of prediction

This is done for the number of bounding boxes we decide to predict in our neural network.

In order to speed up this process, the YOLO framework purposesly splits an image into grid cells - this helps to also determine the probabilities and weights for each region.

From here each grid returns one set of class probabilities (irrespective of number of bounding boxes we create). Then the grid can be used to inform what the final bounding boxes on the "global" image would look like.

To understand what this looks like consider the diagram on the original [YOLO paper](https://pjreddie.com/darknet/yolov1/).
