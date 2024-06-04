# **ECSE 415 Final Project: Traffic Counting**

Course Final Project to count the number of moving cars, parked cars, and pedestrians passed from two separate dashcam videos, where we needed to come up with our own solution based on libraries and approaches shown in class. 

Our approach uses YOLO (Ultralytics) and Optical Flow, details outlined below. (Grade: 93%)

## Implementation Summary

Our solution consist of 2 separate tasks:

### **Identifying/Counting passed cars and people**

We keep track of the id's of any cars and people that we passed.

We assume that we pass an object according to:

*Assumption 1: Any person whose bounding box center is anywhere in the bottom 40% of the screen will be considered as passed, and for cars it is 26.5%.*

 From our testing, this helped with situations where objects might only be detected in the few frames before they are passed, and reduce instances where occlusion in the environment caused by moving cars and our moving perspective removes the chance of detecting an object. Tighter thresholds would perhaps be more rigorous but greatly reduce the window to detect objects. This threshold also helps reduce the times we recount the same object twice, and count objects near the end of the video that we would not have passed yet. We also assume that any person detected is a pedestrian, which would also include any bikers.

### **Classifying passed cars as moving/parked**

The classification consists of 2 stages:

1.   **Initial Guess:**
  Based on the observation that moving cars are generally driving in the middle of the road (following a certain path), while parked cars are on the side of the road, we make the assumption that every car (we pass) that passes through a certain region of the screen is a moving car. We thus classify every car that passes through a certain line as moving and every other car as parked. This gives us a usable first guess for the classification.

2.   **Refinement Stage:**
This stage is based on the flow of the cars. We observed that the flow magnitudes have the general ordering
```
flow(moving_same_dir) < flow(parked) < flow(moving_opposite_dir)
```

which follows the idea that the speed of the cars relative to the dashcam (which follow above ordering) are reflected in the flow.

We use this observation along with :

*Assumption 2: All moving cars that we pass move in the same direction as each other*

which is a fair assumption when driving in the city (due to the restricted size of the streets). We then use K-means clustering with K=2 and our previous guess as initial labels in order to identify cars that the previous method may have misclassified. The resulting labels are our final classification.

**Note:** We define the flow magnitude of a car as the maximum of all mean flow magnitudes (excluding 0 flows) in the bounding boxes (for every frame) of a car. This is the flow-based feature we found that reflects above ordering the most consistently. When including other features such as flow-angle or the standard deviation in flow-angle, the results got worse.

## Software Packages Used
**YOLO (Ultralytics)**
You Only Look Once (YOLO), is a popular object detection and image segmentation model, originating from a paper called "You Only Look Once: Unified, Real-Time Object Detection" by Joseph Redmon et al. from the University of Washington. The original YOLO implementation uses a single CNN to predict bounding boxes and class probabilities directly from images in one evaluation. It first divides the image into a SxS grid, where it tries to find a suitable bounding box in each cell and the corresponding class confidence. Ultralytics provides an improved version of YOLO capable of providing fast, meaningful results and has a variety of extra features. Some features include detection, segmentation, pose estimation, tracking, and classification. For our purposes, we use a YOLOv8 model, which is pre-trained on the COCO dataset, for object classification, tracking, bounding box generation, and object counting.

**OpenCV**
Open Source Computer Vision Library (OpenCV) is a very popular computer vision library that provides many essential functionalities such as image processing, and feature extraction. In this project, OpenCV is used for
* reading the frames of the video
* preprocessing the frames
* running calcOpticalFlowFarneback to find the optical flow
* K-means Clustering

## Results

**Results:**

St. Catherine
* Total Number of Passed Moving Cars: 4
* Total Number of Passed Parked Cars: 50
* Total Number of Pedestrians: 61

McGill Drive
* Total Number of Passed Moving Cars: 23
* Total Number of Passed Parked Cars: 10
* Total Number of Pedestrians: 21


**Ground Truth:**

St. Catherine
* Total Number of Passed Moving Cars: 2
* Total Number of Passed Parked Cars: 55
* Total Number of Pedestrians: 104

McGill Drive
* Total Number of Passed Moving Cars: 24
* Total Number of Passed Parked Cars: 10
* Total Number of Pedestrians: 35

---------------------------------------

Looking at True Positive Counts in each video for all classes relative to ground truth we find:

* Pedestrian Detected: (St. Cath) 56.7% and (McGill Drive) 54.3%
* Moving Cars Detected: (St. Cath) 50% and (McGill Drive) 50%
* Parked Cars Detected: (St. Cath) 91% and (McGill Drive) 10%

In terms of program performance, we detected 50-60% of all pedestrians in each video. Any false positives were due to bikers, misclassifications, and repeat counts are kept to a minimum (2 per video). 

For moving cars, the program on the St. Catherine video had detected 3 false positives cases of moving cars, whereas it counted 10 false positives in McGill Drive. Similarly, almost all parked cars were mostly correctly determined in the St. Catherine video, but 9 detected parked cars in the McGill drive video were false positives causing it to perform very poorly. Overall, the program was effective in determining the correct total number of cars passed. In distinguishing the number of moving and parked cars, our method seems to incorrectly classify these two classes on the busier street scene in the McGill Drive but has better success on the one-way road of St. Catherine.