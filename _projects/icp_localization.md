---
layout: page
title: Self-Driving Car LiDAR Localization
description: Midterm Project of Self-Driving Car Course
img: /assets/img/localize.png
# gh-repo: daattali/beautiful-jekyll
# gh-badge: [star, fork, follow]
category: work
---
# SDC Localization Competition Report


[Scene1 Demo](https://youtu.be/CsiVsvgvNqI)\\
[Scene2 Demo](https://youtu.be/2p4rqpfgzKw)\\
[Scene3 Demo](https://youtu.be/HZM-01POrGw)

1. Pipeline
主要利用ICP,用GPS當第一次alignment的initial guess ,initial yaw 用掃描360度的方式去
找alignment score 好的,接著用上一次的結果當initial guess ,與上一次作業主要不同的是
有使用pcl::passthroughfilter 可以把地板和太遠的點去掉,也可以加速運算。
2. Contribution
    1. 用pcl::passthroughfilter 或ProgressiveMorphologicalFilter 去除地板不必要的feature
    2. Competition 2 在長直線上ICP 不容易align 時,用passthrough filter 去掉前後y方向沒有用的
點
    3. ICP 的 setmaxcorrespondence 要設小一點,尤其是competition 1 中間的時候,車右邊有些
點超出了map ,會把整體拉偏,參數設小之後ICP會忽略這些點。
3. Problem and solution
    1. Competition1 中間車右邊有些點超出了map ,會把整體拉偏,將setmaxcorrespondence設
小至1即可解決。
    1. Competition2 y方向feature 太少,一開始一直走錯方向,將voxel filter關掉,且用
passthrough filter 切掉前後的點,留下較多左右,牆壁上的features。