
**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.png
[image3]: ./examples/windows1.png
[image4]: ./examples/windows2.png
[image5]: ./examples/windows3.png
[image6]: ./examples/windows4.png
[image7]: ./examples/windows_all.png
[image8]: ./examples/windows_vehicle_detect.png
[image9]: ./examples/heatmap.png
[image10]: ./examples/heatmap_threshold.png
[image11]: ./examples/labels_map.png
[image12]: ./examples/boxes.png
[image13]: ./examples/test_images_boxes.png
[image14]: ./examples/vehicle_detect.png
[video1]: ./project_video_out.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

All of the code for this project is in the Jupyter notebook VehicleDetectionandTracking.ipynb

###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the third and fourth code cell of the IPython notebook.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I extracted HOG features with get_hog_features function. The image below shows the HOG features for a car and non-car image:

![alt text][image2]


####2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters for colorspace, orientations, pixels per cell, cells per block, and HOG channel. After experimenting with each and using SVM classifier accuracy and run time as performance criteria, I chose YUV cspace, 11 orientations, 16 pixels per cell, 2 cells per block and ALL channles of the cspace. This gave me an accuracy of around 98.03%

####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM in the section titled 'Train Classifier" The SVM was trained with the default paramers and only using HOG features. The data set was shuffled and split into training and test sets before being inputted into the SVM. I experimented with the spatial intensity and channel intensity, however these increased the computational time and had only a decrease in performance 84.94 with (16,16) spatial size and 16 histogram bins. 

###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

After training the classifier, the following section titled "Detect Cars on Image" has a function called find_cars that was taken from the lectures. It uses HOG feature extraction and sliding windows. It extracts HOG features for the entire image (as suggested by the lectures) and then is subsmapled according to the size of each sliding window before being fed into the SVM. It performs classification prediction on the features in each window and returns a list of bounding boxes for any window that is classfies as a car.

This method was first tested by manually selecting windows that are near the cars on a sample image:

![alt text][image14]

Then, as suggested in the lecture, windows of different positions, sizes, and overlaps depending on what the expected size of the car would be were fit to the bottom portion of the image. The following images show different window sizes:

![alt text][image3]
![alt text][image4]
![alt text][image5]
![alt text][image6]

Finally, all of these windows(and more) were used to detect vehicles in an image:

![alt text][image7]

As you can see from above (and again suggested by the lectures) each car will be correctly identified by a number of windows (and some windows will have false positives where there is no car). To deal with this, a heatmap and threshold are applied to the windows. The add_heat function sums the pixel value, or heat, for every positively identified window that the pixel is in. So if a pixel is in multiple windows, it will appear hotter (brighter color) while the pixels that are in no windows remain black:

![alt text][image8]
![alt text][image9]

As previously mentioned, a threshold is applied to remove any false positives. The threshold I selected was 1:

![alt text][image10]

Next the scipy.ndimage,measurements.label() function uses spatial data from the heatmap and assigns each "cluster" a label:

![alt text][image11]

From these labels the extreme bounding boxes can be taken to give us a final detection area.

####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I ended up only using the HOG features from the 3 YUV channels with 11 orientations, 16 pixels per cell, and 2 cells per block. Here are some example images:

![alt text][image13]

---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_out.mp4)


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

The same find_car function used for single images was applied to the video. However, each detection was stored from the last 10 frames using a class Vehicle_Detect and adding rectagles to it. Instead of using the previous signle frame heatmap/threshold/label/bounding box step, the detection for the last 10 frames are passed to the heatmap and the threshold is set to 1+len(prev_rect)//2.


---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Some potential problems could be detection accuracy for oncoming traffic. Since I chose a lower window overlap, if vehicles are moving quickly on the image they might not be caught in subsequent windows and may not be identified. Also, any "vehicle" bus, motorcycle, etc that isnt in the training set will not be identified. 

