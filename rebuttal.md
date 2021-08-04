# R1
Summary:
The paper introduced a self-supervised 3D localization approach on LiDAR point clouds. The approach lifts 2D detections to 3D representations and can achieve decent results on KITTI benchmarks.

Main Review:
Strength:

The proposed pipeline makes sense, and the formulations at each step are sensible and correct.
Experimental results suggest that the proposed self-supervised learning approach is approaching supervised approaches in performance, and out-performs baselines.
Weaknesses:

The description of the approach section is decent, however there are still several things I don't fully understand. 1) It seems the algorithm is heavily bounded by the performance of 2D detection algorithms. If there is a FP detection, would the proposed approach still predict a 3D box? Or, would the uncertainty prediction for that region is large so the box won't be predicted? 2) The multi-frame tracking step seems a bit out-of-context to me (as I thought the comparisons should be conducted at single-frame inputs), this is fine, but the description of the tracking step is a bit vague. There are several common challenging scenarios, such as occlusions across frames, and I wonder if the authors did certain tricks to make it work.
The paper demonstrated some failure cases in visualizations, but those seem to be "decent" failure cases. Since there is still a rather big gap between fully supervised approaches [26], I was wondering if the authors can provide more analysis about what remains to be done in order to catch up with [26]. There is one more example in the bottom left figure of the supp. material but I couldn't find visualizations that justify the performance gap.
The paper evaluated the full pipeline on the KITTI dataset, if people would use the proposed approach for new datasets, are there any guidelines they should follow (for example, which parameter needs to be tuned, which module needs to be carefully tuned, etc?)
The paper seems to be less polished: L409 has a missed reference. The last word in the caption of Fig3 "ares" seems to be a typo? there are duplicated references, such as [43] and [44]
Limitations And Societal Impact:
I have a broader impact question (that won't affect my ratings): would a fully self-supervised autonomous vehicle application the right direction to pursue? I understand the authors' motivations for writing this paper and I appreciate authors discussing the impacts in the last section of the paper, but for safety concerns I personally feel like at least a certain amount of supervision is necessary.


Rating: 5: Marginally below the acceptance threshold
Confidence: 4: You are confident in your assessment, but not absolutely certain. It is unlikely, but not impossible, that you did not understand some parts of the submission or that you are unfamiliar with some pieces of related work.




# R2
Summary:
This paper argues that obtaining accurate human annotations for 3D object detection in autonomous driving scenarios is expensive and time-consuming. To address this issue, the paper proposes an auto-annotator where it uses a pretrained 2D Mask-CNN to obtain 2D object locations, and then lift the 2D results to 3D by fitting a average car mesh to the projected point cloud. The paper also proposes components to discount outlier points, direct optimize for yaw and obtain multi-frame consistency. Experiments on KITTI shows that the proposed method outperforms existing methods [18, 43].

Main Review:
Strengths:

The objective of the paper is clear and the method is easy to follow.
Code and detailed implementation is provided, making the results convincing.
The paper has novel elements. Using a template mesh and regress the pose of the vehicle is intuitive. The yaw prediction and tail removal modules are well motivated.
Major Weaknesses:

Motivation is not clear. The paper uses an 2D object detector pre-trained offline on a huge dataset MS-COCO, to obtain 3D object detection. However, the paper also assume to have lidar point cloud of all the video frames. Then, I wonder why not use a 3D object detector pre-trained on some existing dataset like Waymo and Nuscenes to get the 3D auto-labels. The paper did not compare with this baseline (either in accuracy or time/space complexity), and did not provide a concrete reason for not doing so.
The multi-frame consistency greatly limited the driving scenario where this method can be used. E.g., If the data is captured on any type of crossing, the method breaks. Thus, although reported improvement with ablation study, this hinders the robustness of the algorithm.
Missing experiments on whether these auto-generated annotations can help improve 3D detection model learning on limited human annotation. The objective of the paper is to obtain 3D annotation. However, the overall results on KITTI 3D detection validation set cannot even 77 AP, which is nowhere near the human annotation. I cannot think of a scenario where the detection results of this method can be used. If training with both limited KITTI real annotation and large amount of auto-generated annotations from this paper actually improves the overall performance, the paper should have presented such results.
The object is restricted to car. However, both kitti and MS-COCO provide classes like 'truck', 'cyclist', etc. Since training the method does not take a lot of time, why not test the method on these classes as well.
Minor Weaknesses:

The components of the methods are mostly combination of and incremental over existing methods [7, 11, 18, 20, 28].
Paper has many grammar issues and typos. E.g. 51 "form", etc.
Abstract is poorly written. The objective is mentioned but not the motivation. The high-level summary is missing, and the test involves too many details, making it hard to follow. Line 7, what is "this space". Line 12 "these", etc.
In related work, "Supervised" is a bad, unfinished heading. Same goes for "Weakly Supervised".
Table 3 is not clear. The columns are not aligned, and I do not why "our" method has two lines of results.
Limitations And Societal Impact:
The authors pointed out the limitations and potential genative societal impact of the work.

Rating: 4: Ok but not good enough - rejection
Confidence: 4: You are confident in your assessment, but not absolutely certain. It is unlikely, but not impossible, that you did not understand some parts of the submission or that you are unfamiliar with some pieces of related work.




# R3
Summary:
This paper proposes to convert 2D mask by running MASK RCNN and raw LIDAR point clouds to 3D bounding box. In response to existing problems, a series of solutions have been proposed. All algorithms have been carefully designed.

Main Review:
This paper proposes to convert 2D mask by running MASK RCNN and raw LIDAR point clouds to 3D bounding box. However, the process of predicting the 2D mask itself will introduce a lot of outliers, because the 2D mask is not so close to the object. Because the mask is not like a segmented area, it is always inevitable that outliers will be introduced. However, if you use the segmented area and perform the following process, you may avoid the problems raised in the paper. Moreover, in the network structure proposed in the text, the step of implementing point cloud segmentation is yet to come. This step can be removed after the segmentation of the image domain is introduced. Therefore, the reasoning process proposed in the article of using a 2D mask, projecting to a 3D space, and then designing related algorithms to remove outliers is inherently problematic.
One of the motivations of the method proposed in this paper is that the labeling process in the collection of data sets is time-consuming and laborious, and a large application of the method in this paper is obviously also in the data labeling process. So what is the result of applying the method in this paper to data annotation? Compared with the equivalent human labeling, what are the relevant results? And we know that in the process of data labeling, there are often errors in the labeling. There are also errors in Figure 1 and Figure 2 in the supplementary materials. So how to deal with this error situation? Discussion on this part should be given.
Experimental part. First, in Table 3, the article shows the results on the KITTI verification set. For the KITTI data set, the more convincing result should be the test set. Why don't you conduct experiments on the test set? Second, there are too few comparison methods on the one hand. On the other hand, the annotation resources used in the experiments and comparison methods in this article are not so consistent, so direct comparison of the results is not so fair.
The data set that the method proposed in this paper relies on is largely a video data set with continuous frames. We know that in the application of autonomous driving, sometimes it is not possible to obtain very continuous video data, often in very short segments, so what is the minimum number of frames required for the method in this article to work? In addition, for the autonomous driving data set, the number of Waymo data sets is many times that of KITTI. So what is the result of the method in this paper on a large data set like Waymo? This paper only does an experiment on the KITTI data set, which is obviously not enough.
Limitations And Societal Impact:
The authors have claimed the potential negative societal impact of their work.


Rating: 4: Ok but not good enough - rejection
Confidence: 5: You are absolutely certain about your assessment. You are very familiar with the related work and checked the math/other details carefully.
