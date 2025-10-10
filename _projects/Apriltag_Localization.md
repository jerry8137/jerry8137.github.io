---
layout: page
title: Object Detection and Localization by Apriltag
description: NYCU Human Centric Computing Course Final Competition
img: assets/img/lidar.jpg
importance: 3
# gh-repo: KaiYin77/Localization_by_Apriltag
category: academic
---

# Localization by Apriltag

## [Demo Video](https://youtu.be/BF3L9FirbP4)

<iframe width="560" height="315" src="https://www.youtube.com/embed/BF3L9FirbP4?si=HAzF6W5fGLqNlKhb" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

===

Team MOLA's Members
---

- 0710809 侯俊宇(ECE) [Github](https://github.com/jerry8137)
- 0710715 呂宗翰(ECE) [Github](https://github.com/Lu-Zong-Han)
- 0710747 李謙熙(ECE) [Github](https://github.com/Jackie890621)
- 0710851 洪愷尹(ECE) [Github](https://github.com/KaiYin77)

Main Method: Local Outlier Factor
---
1. **Calibrate** the choosing point from the bounding box center, and **filter out** the low possibility detection from YOLO.
2. Record the position of the specified objects. 
3. **Remove the outliers** and take **average** of the remaining as result.

We use the Novelty detection with Local Outlier Factor provided by scikit-learn in order to remove the outliers.
[scikit-learn](https://scikit-learn.org/stable/modules/outlier_detection.html)

> using the Local Outlier Factor

```python=
clf = LocalOutlierFactor(n_neighbors=(int)(len(a)/10)+2)
a_pred = clf.fit_predict(a)
```

## Engineering Problems and Solving Techniques
### Time error of global transformation
Here we **only depend on AprilTag** for localization. Thus, while the camera cannot detect AprilTag, the localization will fail.
The position of object we get is wrong if localization fails.

We take advantage of transform time and local time.

>In apriltag_localization.cpp
>Record the time in header of the global transform
```python=
trans_odem.header.stamp = min_distance_trans.stamp_;
```
> time error = local time - transform time
> Check time error is small enough

```python=
if abs(local_time - transform_time) < 1 and transform_time != 0:
     #implement object detection......
```

### Debugging by RVIZ
**Publishing the ground truth** in example bag and current robot position helps us observe the data. \\
<iframe width="560" height="315" src="https://www.youtube.com/embed/BF3L9FirbP4?si=OHeGIuDme7xUDiY6" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
> Publish robot (camera_link) position

```python=
# publish camera pos in origin frame
v1 = np.array([0,0,0,1])
org = np.matmul(global_transform, v1)
point_message = PointStamped()
point_message.header = depth_img.header
point_message.header.frame_id = "origin"
point_message.point.x = org[0]
point_message.point.y = org[1]
point_message.point.z = org[2]
pub1.publish(point_message)
print("pub camera")
```

> Publish object ground truth


```python=
# put the ground truth here for rviz
point_message.point.x = 0
point_message.point.y = 3.924
point_message.point.z = 0.17
pub2.publish(point_message)
```

Other Methods we've tried: 
---
> Since we found that object detection from yolov2-tiny(pre-train) is really unstable like false classification and miss detection so on

Using moving average to get the global position
* Pros: Real time processing
* Cons: In terms of the bad detection, the average of wrong candidate positions is also the wrong position and if the scenario have the same class objects the outcome is also wrong.
```python=
 if len(Chair_avg) >= 3: # !=0
             Chair[0] = 0
             Chair[1] = 0
             Chair[2] = 0
             for i in range(3):
                 Chair[0] = Chair[0] + Chair_avg[-i-1][0]
                 Chair[1] = Chair[1] + Chair_avg[-i-1][1]
                 Chair[2] = Chair[2] + Chair_avg[-i-1][2]

             Chair[0] = Chair[0] / 3.0
             Chair[1] = Chair[1] / 3.0
             Chair[2] = Chair[2] / 3.0
```
---

> Found that the chair case's ground truth position is in the cushion center. 
> However,the depth camera can only caluculate the depth of chair back, there has error over 20cm.
* Pros: Indeed, it solved this scenario.
* Cons: Not the general solution, for example, if the chair is back of the depth camera, it won't work.
```python=
zc = cv_depthimage2[int(y_chair)][int(x_chair)] - 200
```
---

> Found that in umbrella case when camera is too close to umbrella the Yolo detection bounding box won't include the whole umbrella, then the position of the umbrella will have an offset.
> Solution is to adopt the detection outcome by specific depth range. 
* Pros: Indeed, it solved this scenario.
* Cons: Not the general solution, for example, if change to another bag, we should watch the bag on rviz to tune the parameter again.
```python=
if Dist_Cam_Umbrella < 2.3 and Dist_Cam_Umbrella > 1.4 :
    publish_object_location(object_position, depth_img, org, Umbrella)
```
