##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/car_example.png
[image2]: ./examples/notcar_example.png
[image3]: ./examples/hog.png
[image4]: ./examples/window1.png
[image5]: ./examples/window2.png
[image6]: ./examples/window3.png
[image7]: ./examples/labeled.png
[image8]: ./examples/heat.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one. 


### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for HOG fucntion is implemented during this lesson. It is named "lesson_functions.py".

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1] ![image2]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `RGB` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image3]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and find using "YCrCb" and "YUV" color space and ALL hog channel get better result. So at last I combine the color feature form "YCrCb" color space and hog features from all channels as input to train the SVM classifier. 

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

The features are normalized with "StandardScaler()" before training.
I trained a linear SVM using 80% of the training data, and 20% for testing. The training and test sets are randomized. The Test accuracy is 0.99, I think it is good enough to be applied to the video and images.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The slide window search function is implemented in "find_cars()" function. Using the linear SVM I just trained before, and search step is 2 cells, meas 75% overlap. I set the scale to 1.0, 1.5 and 2.0 to search different size of windows. 
Three different scales are used to detect vehicles with different distances. The bunding boxes found are a little more accurate than using only scale=1.5.
75% overlaping is a good choice regarding the balance of accuracy and processing speed. 

And this combinition get very stable detection result, but the searching speed slowed down significantly because it searchs on the image for three times.


#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

I try to set the start and stop position to cut out the unnessary searching area to speed up the searching process. The searching area on y axis is from 400 to 650, on x axis is from 600 to 1280. So the search area is about one forth of the image.


Ultimately I searched on three scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]![image5]![image6]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./lane_video_output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

I used history information to help detect vehicles and exclude false positives. the "history_frames" queue stores detected boxes from recent 10 frames. I sumed them up to generate a heatmap, and use the number of history_frames (10 for the most time) as the threshold. This effectively reduced the false positive detections.

Here is an example of a heatmap image and the corresponding labed image:

![image7]![image8]





---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Finally the pipline is applied on the lane detection video from last project, and using only one scale (1.5) the processing speed is about 5.2 frames per second. The result video is "lane_video_out.mp4".
When using three scale searching, the processing speed reduced to 1.7 frames per second on my laptop. These can be improved by choose different searching area for different scales.

The bunding boxes on detected vehicles are not stable, the exact area can be calculated because we already know the position of vehicles. 

Further, I think sliding window method is a preliminary step for vehicle dections, because it requires a lot of calculation on each frame of video. CNN could work better for detecting cars on the image.

The fixed searching area on the image is not robust. When the car drive to a cross road, slop it will fail to detect cars and cause danger.






