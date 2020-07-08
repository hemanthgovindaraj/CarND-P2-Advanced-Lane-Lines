
## Advanced Lane Finding Project

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./test_output/undist_chessBoard.jpg "Undistorted"
[image2]: ./test_output/undist_test_images/straight_lines1.jpg "Road Transformed"
[image3]: ./test_output/binary_combo_example.jpg "Binary Example"
[image4]: ./test_output/warped_straight_lines.jpg "Warp Example"
[image5]: ./test_output/final_output.jpg "Output"
[video1]: ./test_output/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

For calibrating the camera, i started by creating an array  of object points , designed for a chess board of interior row of size 9 corners and column of 6 corners. Now this is the points of ideally where the position of the chess points are supposed to be.
Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image using the open CV findChessboardCorners function.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Chessboard undistorted][image1]

### Pipeline (single images)

#### 1. Example of a distortion-corrected image.

To check whether the coefficients function correctly, i apply the distortion correction to all of the test images like this one:
![lanelinesundistorted][image2]

#### 2. Using color transforms, gradients or other methods to create a thresholded binary image. 

Once the images are undistorted, we can now start the process of finding lanes, by splitting the images into various components like gradients or color combinations to try and figure the best combination to detect lane lines in every situation of the road.
I used the green component of the image in rgb representation and the s component of the image in hls representation to detect the lane lines out of the given image.

![Image thresholded][image3]

#### 3. Performing a perspective transform

Now that i have an image which is thresholded and contains the binaries of the lane signal, i can try to get a birds eye view of the given image using the Perspective transform technique. I start by detecting source points by manually having a look at the image and trying to find out the exact position of the lane lines, which will be my source (`src`) points. Then i can again manually pick a near rectangle coordinates to which these source points can be mapped and i call them the destination (`dst`) points.
The code for my perspective transform includes a function called 'birds_eye_view()', which does the job of calculating the Transform Matrix M which i use to warp the image to a birds eye view.  
I chose the hardcode the source and destination points in the following manner:

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 460      | 250, 0        | 
| 700, 460      | 1050,0        |
| 1060, 690     | 1060,720      |
| 230,680       | 230,720       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![transformed image][image4]

#### 4.Identifying lane-line pixels and fitting their positions with a polynomial

Now that i have a birds-eye view of the image with lane lines on it, i can now proceed to getting the line in terms of a polynomial.
For this, i start with the Sliding window technique. Using the fact that the lane lines start at the edge of the image, and looking at the histogram, i can find out the activated points and have them as the start of the image. Now, i decide a rectangle that can be drawn around this activation points and find out the next of pixel positions that are activated. I can move the centre of the rectangle to the mean positions of this activated pixel points. Now using the same technique, we build up to have a list of all the activated points for the left and the right lanes. These are basically the activation points and using the OpenCV polyfit function, i can find out the polynomial coefficients of the lane lines.

See the image in the below attachment. 

Now that we already have a fit to the lane line, it happens that there is not a huge difference in the pixels positions of the activated lane points, but there is only a gradual change in between different images. We can use this concept and develop the search algorithm which takes in the last detected coefficients of the polynomial and really only searches in a margin region around the preiously detected polynomial, and saving us a lot of time in computation. This is my `polynomial_lane_detection()` function.

Since we have the imformation of where the lane lines are at the bottom of the image in the histogram data, we know that the middle of the lane lines are basically the midpoint of the between lane lines. With this infomation and midpoint of the image, i calculated the offset of the camera, which i later use for the display.

![alt text][image5]

#### 5. Calculating the radius of curvature of the lane

Now that i have the lane line coefficients , i can find out the radius of curvature basically by using a mathematical formula.
I calculate the curvature for the left and the right lanes and then average them out to find out the final radius of curvature.

#### 6. Plotting back with the lane lines.

With the coefficients of the lane lines found and already armed with the inverse transform coefficients , i have used the OpenCV warping function again to basically unwarp the image with the lane line and draw a polygon representing the region in between the lane lines.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1.My Pipeline videp.

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I actually got stuck with the issue that after detecting the lane lines for 23 seconds, my algorithm did not work anymore. Having spent a lot of time, i spent time then trying to get some sanity checks into the code by implementing the python class as suggested only to find out that basically my lane thresholding blacked out when there is bright sunshine on the road. So, thresholding the green component of the image helped in identifying the lane lines.

Areas that can be improved

- Having seen the above problem, i am sure that there are certain whether conditions that affect the image and the algorithm might not be robust enough to capture the lane lines in all conditions.

- When the camera basically shifted position because the bump on the road, i can think the lane detection algorithm my take a hit for sometime ad will recover after the camera position recovers.

