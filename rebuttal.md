# R1
## Summary:
The paper introduced a self-supervised 3D localization approach on LiDAR point clouds. The approach lifts 2D detections to 3D representations and can achieve decent results on KITTI benchmarks.

## Main Review:
### Strength:

1) The proposed pipeline makes sense, and the formulations at each step are sensible and correct.
2) Experimental results suggest that the proposed self-supervised learning approach is approaching supervised approaches in performance, and out-performs baselines.

### Weaknesses:
The description of the approach section is decent, however there are still several things I don't fully understand. 
1) It seems the algorithm is heavily bounded by the performance of 2D detection algorithms. If there is a FP detection, would the proposed approach still predict a 3D box? Or, would the uncertainty prediction for that region is large so the box won't be predicted? 
2) The multi-frame tracking step seems a bit out-of-context to me (as I thought the comparisons should be conducted at single-frame inputs), this is fine, but the description of the tracking step is a bit vague. There are several common challenging scenarios, such as occlusions across frames, and I wonder if the authors did certain tricks to make it work.
3) The paper demonstrated some failure cases in visualizations, but those seem to be "decent" failure cases. Since there is still a rather big gap between fully supervised approaches [26], I was wondering if the authors can provide more analysis about what remains to be done in order to catch up with [26]. There is one more example in the bottom left figure of the supp. material but I couldn't find visualizations that justify the performance gap.
4) The paper evaluated the full pipeline on the KITTI dataset, if people would use the proposed approach for new datasets, are there any guidelines they should follow (for example, which parameter needs to be tuned, which module needs to be carefully tuned, etc?)
5) The paper seems to be less polished: L409 has a missed reference. The last word in the caption of Fig3 "ares" seems to be a typo? there are duplicated references, such as [43] and [44]

### Limitations And Societal Impact:
I have a broader impact question (that won't affect my ratings): would a fully self-supervised autonomous vehicle application the right direction to pursue? I understand the authors' motivations for writing this paper and I appreciate authors discussing the impacts in the last section of the paper, but for safety concerns I personally feel like at least a certain amount of supervision is necessary.


Rating: 5: Marginally below the acceptance threshold\
Confidence: 4: You are confident in your assessment, but not absolutely certain. It is unlikely, but not impossible, that you did not understand some parts of the submission or that you are unfamiliar with some pieces of related work.

```
1)
2) the tracking is based on a simple comparison of the median of X,Y,Z values of lidar points in each frame for each detection, a threshold is then applied to ensure the distance travelled between frames is reasonable. Challenging scenarious were not accounted for as our interest in tracking is only needed for training time and at test time a single frame is used for predictions
3) Part of the gap in performance comes from the general nature of Mask RCNN trained on COCO instead of on a driving dataset where cars are more prominant. This was used for comprability to the existing works and indeed when Mask RCNN trained on Cityscapes is used the performance increases 
```
| data | Center aligned | Center aligned | Center aligned | 
|------|:--------------:|:--------------:|:--------------:|
| COCO | 0.9017	| 0.8766	| 0.8006 |
|Cityscapes | 0.9253  |	0.9072 |	0.7911 |
```
4)
5) 
```
---

# R2
## Summary:
This paper argues that obtaining accurate human annotations for 3D object detection in autonomous driving scenarios is expensive and time-consuming. To address this issue, the paper proposes an auto-annotator where it uses a pretrained 2D Mask-CNN to obtain 2D object locations, and then lift the 2D results to 3D by fitting a average car mesh to the projected point cloud. The paper also proposes components to discount outlier points, direct optimize for yaw and obtain multi-frame consistency. Experiments on KITTI shows that the proposed method outperforms existing methods [18, 43].

## Main Review:
## Strengths:

The objective of the paper is clear and the method is easy to follow.
Code and detailed implementation is provided, making the results convincing.
The paper has novel elements. Using a template mesh and regress the pose of the vehicle is intuitive. The yaw prediction and tail removal modules are well motivated.
## Major Weaknesses:

1) Motivation is not clear. The paper uses an 2D object detector pre-trained offline on a huge dataset MS-COCO, to obtain 3D object detection. However, the paper also assume to have lidar point cloud of all the video frames. Then, I wonder why not use a 3D object detector pre-trained on some existing dataset like Waymo and Nuscenes to get the 3D auto-labels. The paper did not compare with this baseline (either in accuracy or time/space complexity), and did not provide a concrete reason for not doing so.
2) The multi-frame consistency greatly limited the driving scenario where this method can be used. E.g., If the data is captured on any type of crossing, the method breaks. Thus, although reported improvement with ablation study, this hinders the robustness of the algorithm.
3) Missing experiments on whether these auto-generated annotations can help improve 3D detection model learning on limited human annotation. The objective of the paper is to obtain 3D annotation. However, the overall results on KITTI 3D detection validation set cannot even 77 AP, which is nowhere near the human annotation. I cannot think of a scenario where the detection results of this method can be used. If training with both limited KITTI real annotation and large amount of auto-generated annotations from this paper actually improves the overall performance, the paper should have presented such results.
4) The object is restricted to car. However, both kitti and MS-COCO provide classes like 'truck', 'cyclist', etc. Since training the method does not take a lot of time, why not test the method on these classes as well.

### Minor Weaknesses:
1) The components of the methods are mostly combination of and incremental over existing methods [7, 11, 18, 20, 28].
2) Paper has many grammar issues and typos. E.g. 51 "form", etc.
3) Abstract is poorly written. The objective is mentioned but not the motivation. The high-level summary is missing, and the test involves too many details, making it hard to follow. Line 7, what is "this space". Line 12 "these", etc.
4) In related work, "Supervised" is a bad, unfinished heading. Same goes for "Weakly Supervised".
5) Table 3 is not clear. The columns are not aligned, and I do not why "our" method has two lines of results.
Limitations And Societal Impact:
The authors pointed out the limitations and potential genative societal impact of the work.

Rating: 4: Ok but not good enough - rejection\
Confidence: 4: You are confident in your assessment, but not absolutely certain. It is unlikely, but not impossible, that you did not understand some parts of the submission or that you are unfamiliar with some pieces of related work.

```
1) In this work we ask the question of whether or not we can use 2D information to better understand the world in 3D. Datasets such as COCO provide annotations for a wide variety of object types in 2D and while other datasets exist for detection in 3D many objects that are well understood in the image domain are not represented in 3D datasets. To this end we take a genericly trained 2D detector and explore it's usefulness in training a network that takes as input 3D information (in this case point clouds)
2) It is true that multi-frame methods are incapable of reasoning in static camera scenarious, however we can remove such sequences as we have access to the camera extrinsics, a trick which is also employed in monocular depth papers, the Eigen-Zhou split of KITTI for example removes frames where the ego-vehicle is stationary which would cause such methods to fail. In our case teh ego cameras lack of motion is not so much an impedement, as the removal of ego motion from the locaiton difference means we just check that the predicted location has not changed significantly with the fluctuation in the LiDAR points locations between frames.
```
### 3) train supervised method on auto-labels, perhaps expand training set with GT + auto-labels on other frames
### 4) train on other object classes?
---

# R3
## Summary:
This paper proposes to convert 2D mask by running MASK RCNN and raw LIDAR point clouds to 3D bounding box. In response to existing problems, a series of solutions have been proposed. All algorithms have been carefully designed.

## Main Review:
1) This paper proposes to convert 2D mask by running MASK RCNN and raw LIDAR point clouds to 3D bounding box. However, the process of predicting the 2D mask itself will introduce a lot of outliers, because the 2D mask is not so close to the object. Because the mask is not like a segmented area, it is always inevitable that outliers will be introduced. However, if you use the segmented area and perform the following process, you may avoid the problems raised in the paper. Moreover, in the network structure proposed in the text, the step of implementing point cloud segmentation is yet to come. This step can be removed after the segmentation of the image domain is introduced. Therefore, the reasoning process proposed in the article of using a 2D mask, projecting to a 3D space, and then designing related algorithms to remove outliers is inherently problematic.
2) One of the motivations of the method proposed in this paper is that the labeling process in the collection of data sets is time-consuming and laborious, and a large application of the method in this paper is obviously also in the data labeling process. So what is the result of applying the method in this paper to data annotation? Compared with the equivalent human labeling, what are the relevant results? And we know that in the process of data labeling, there are often errors in the labeling. There are also errors in Figure 1 and Figure 2 in the supplementary materials. So how to deal with this error situation? Discussion on this part should be given.
3) Experimental part. First, in Table 3, the article shows the results on the KITTI verification set. For the KITTI data set, the more convincing result should be the test set. Why don't you conduct experiments on the test set? Second, there are too few comparison methods on the one hand. On the other hand, the annotation resources used in the experiments and comparison methods in this article are not so consistent, so direct comparison of the results is not so fair.
4) The data set that the method proposed in this paper relies on is largely a video data set with continuous frames. We know that in the application of autonomous driving, sometimes it is not possible to obtain very continuous video data, often in very short segments, so what is the minimum number of frames required for the method in this article to work? In addition, for the autonomous driving data set, the number of Waymo data sets is many times that of KITTI. So what is the result of the method in this paper on a large data set like Waymo? This paper only does an experiment on the KITTI data set, which is obviously not enough.



Rating: 4: Ok but not good enough - rejection\
Confidence: 5: You are absolutely certain about your assessment. You are very familiar with the related work and checked the math/other details carefully.


```
1) if the reviewer refers to segmentation in the image then one would expect that Mask RCNN outputs would perform similar to a semantic segmentation network while also having instance labels which are important for our task. If the segmentation mentioned is of the 3D points then we do not know of any methods that can segment pointclouds without supervision from a human annotated 3D box. In place of segmentation we introduce uncertanty estimates for each lidar point where points which are on the surface of the target car should recieve a low values and those far away a high value. In papers such as [18] the LiDAR points are segemnted based on their proximity to an anchor but in this work the anchors are 3D boxes all oriented parallel to the Z axis of the camera, making them less useful to precisely predict the location in 3D as many points are lost in the case of a car with very different orientation, they require yaw supervision to overcome this issue which we do not.
```
### 2) again train FPN on the generated labels
```
3) The validation set is used for comprability to existing weakly-supervised works [18,43]. The mentioned papers are currently the only existing works on weakly-supervising 3D object detection on the KITTI dataset. We agree that the differing level of supervision makes comparison of this paper to existing methods difficult, we think however that it it noteworthy that our method requires the fewest supervision signals and achieves often vastly superior results to [18,43] who require knowledge of yaw in both cases and a wide variety of car models and synthesic scenery in [43]
4) In our work we track for up to 5 frames which gives us several thousand detected vehicles with an often wide baseline between frames as the car is typically moving at a moderate pace. As KITTI only has front facing cameras getting views of an object from wildly different viewpoints is unlikely as many move out of frame relatively quickly. 
(1,2,3) below 
```


1) Offboard 3D Object Detection from Point Cloud Sequences
2) Auto4D: Learning to Label 4D Objects from Sequential Point Clouds
3) Weakly Supervised 3D Object Detection from Lidar Point Cloud
