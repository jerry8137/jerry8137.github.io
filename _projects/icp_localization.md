---
layout: page
title: Self-Driving Car LiDAR Localization
description: Midterm Project of Self-Driving Car Course
img: assets/img/localize.png
importance: 1
# gh-repo: daattali/beautiful-jekyll
# gh-badge: [star, fork, follow]
category: academic
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/CsiVsvgvNqI?si=pSkhtjEzkfGH2GIK" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/2p4rqpfgzKw?si=y8z-y0asHj6vKI9Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/HZM-01POrGw?si=yXaS5YMNeGAVDvIs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>



## Pipeline: 

The main approach used is ICP, where GPS is used as the initial guess for the first alignment, and initial yaw is scanned 360 degrees to find a good alignment score. Then, the previous result is used as the initial guess, and the main difference from the previous work is the use of `pcl::passthroughfilter` to remove the floor and faraway points, which can also speed up the computation.

## Contribution:

Use `pcl::passthroughfilter` or ProgressiveMorphologicalFilter to remove unnecessary features from the floor.
In Competition 2, when ICP is not easy to align on a long straight line, use passthrough filter to remove the useless points in the front and back y-direction.
Set the max correspondence of ICP to a smaller value, especially in Competition 1, where some points on the right side of the car exceed the map and cause the overall alignment to deviate. After setting the parameter to a smaller value, ICP will ignore these points.

## Problem and solution:

- In Competition 1, some points on the right side of the car exceed the map and cause the overall alignment to deviate. Setting the max correspondence to a smaller value, such as 1, can solve this problem.

- In Competition 2, there are too few features in the y-direction, and the car keeps going in the wrong direction. Turning off the voxel filter and using passthrough filter to cut off the front and back points can leave more features on the left and right walls.
