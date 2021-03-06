# Self-Driving Car Engineer Nanodegree


## Project: **Finding Lane Lines on the Road** 
***
In this project, you will use the tools you learned about in the lesson to identify lane lines on the road.  You can develop your pipeline on a series of individual images, and later apply the result to a video stream (really just a series of images). Check out the video clip "raw-lines-example.mp4" (also contained in this repository) to see what the output should look like after using the helper functions below. 

Once you have a result that looks roughly like "raw-lines-example.mp4", you'll need to get creative and try to average and/or extrapolate the line segments you've detected to map out the full extent of the lane lines.  You can see an example of the result you're going for in the video "P1_example.mp4".  Ultimately, you would like to draw just one line for the left side of the lane, and one for the right.

In addition to implementing code, there is a brief writeup to complete. The writeup should be completed in a separate file, which can be either a markdown file or a pdf document. There is a [write up template](https://github.com/udacity/CarND-LaneLines-P1/blob/master/writeup_template.md) that can be used to guide the writing process. Completing both the code in the Ipython notebook and the writeup template will cover all of the [rubric points](https://review.udacity.com/#!/rubrics/322/view) for this project.

---
Let's have a look at our first image called 'test_images/solidWhiteRight.jpg'.  Run the 2 cells below (hit Shift-Enter or the "play" button above) to display the image.

**Note: If, at any point, you encounter frozen display windows or other confounding issues, you can always start again with a clean slate by going to the "Kernel" menu above and selecting "Restart & Clear Output".**

---

**The tools you have are color selection, region of interest selection, grayscaling, Gaussian smoothing, Canny Edge Detection and Hough Tranform line detection.  You  are also free to explore and try other techniques that were not presented in the lesson.  Your goal is piece together a pipeline to detect the line segments in the image, then average/extrapolate them and draw them onto the image for display (as below).  Once you have a working pipeline, try it out on the video stream below.**

---

<figure>
 <img src="examples/line-segments-example.jpg" width="380" alt="Combined Image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;"> Your output should look something like this (above) after detecting line segments using the helper functions below </p> 
 </figcaption>
</figure>
 <p></p> 
<figure>
 <img src="examples/laneLines_thirdPass.jpg" width="380" alt="Combined Image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;"> Your goal is to connect/average/extrapolate line segments to get output like this</p> 
 </figcaption>
</figure>

**Run the cell below to import some packages.  If you get an `import error` for a package you've already installed, try changing your kernel (select the Kernel menu above --> Change Kernel).  Still have problems?  Try relaunching Jupyter Notebook from the terminal prompt.  Also, consult the forums for more troubleshooting tips.**  

## Import Packages


```python
#importing some useful packages
import matplotlib.pyplot as plt
import matplotlib.image as mpimg
import numpy as np
import cv2
%matplotlib inline
```

## Read in an Image


```python
#reading in an image
image = mpimg.imread('test_images/solidWhiteRight.jpg')

#printing out some stats and plotting
print('This image is:', type(image), 'with dimensions:', image.shape)
plt.imshow(image)  # if you wanted to show a single color channel image called 'gray', for example, call as plt.imshow(gray, cmap='gray')

```

    This image is: <class 'numpy.ndarray'> with dimensions: (540, 960, 3)
    




    <matplotlib.image.AxesImage at 0x17f8c39d3a0>




    
![png](output_6_2.png)
    


## Ideas for Lane Detection Pipeline

**Some OpenCV functions (beyond those introduced in the lesson) that might be useful for this project are:**

`cv2.inRange()` for color selection  
`cv2.fillPoly()` for regions selection  
`cv2.line()` to draw lines on an image given endpoints  
`cv2.addWeighted()` to coadd / overlay two images  
`cv2.cvtColor()` to grayscale or change color  
`cv2.imwrite()` to output images to file  
`cv2.bitwise_and()` to apply a mask to an image

**Check out the OpenCV documentation to learn about these and discover even more awesome functionality!**

## Helper Functions

Below are some helper functions to help get you started. They should look familiar from the lesson!


```python
import math

def grayscale(img):
    """Applies the Grayscale transform
    This will return an image with only one color channel
    but NOTE: to see the returned image as grayscale
    (assuming your grayscaled image is called 'gray')
    you should call plt.imshow(gray, cmap='gray')"""
    return cv2.cvtColor(img, cv2.COLOR_RGB2GRAY)
    # Or use BGR2GRAY if you read an image with cv2.imread()
    # return cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    
def canny(img, low_threshold, high_threshold):
    """Applies the Canny transform"""
    return cv2.Canny(img, low_threshold, high_threshold)

def gaussian_blur(img, kernel_size):
    """Applies a Gaussian Noise kernel"""
    return cv2.GaussianBlur(img, (kernel_size, kernel_size), 0)

def region_of_interest(img, vertices):
    """
    Applies an image mask.
    
    Only keeps the region of the image defined by the polygon
    formed from `vertices`. The rest of the image is set to black.
    `vertices` should be a numpy array of integer points.
    """
    #defining a blank mask to start with
    mask = np.zeros_like(img)   
    
    #defining a 3 channel or 1 channel color to fill the mask with depending on the input image
    if len(img.shape) > 2:
        channel_count = img.shape[2]  # i.e. 3 or 4 depending on your image
        ignore_mask_color = (255,) * channel_count
    else:
        ignore_mask_color = 255
        
    #filling pixels inside the polygon defined by "vertices" with the fill color    
    cv2.fillPoly(mask, vertices, ignore_mask_color)
    
    #returning the image only where mask pixels are nonzero
    masked_image = cv2.bitwise_and(img, mask)
    return masked_image


def draw_lines(img, lines, color=[255, 0, 0], thickness=5):
    """
    NOTE: this is the function you might want to use as a starting point once you want to 
    average/extrapolate the line segments you detect to map out the full
    extent of the lane (going from the result shown in raw-lines-example.mp4
    to that shown in P1_example.mp4).  
    
    Think about things like separating line segments by their 
    slope ((y2-y1)/(x2-x1)) to decide which segments are part of the left
    line vs. the right line.  Then, you can average the position of each of 
    the lines and extrapolate to the top and bottom of the lane.
    
    This function draws `lines` with `color` and `thickness`.    
    Lines are drawn on the image inplace (mutates the image).
    If you want to make the lines semi-transparent, think about combining
    this function with the weighted_img() function below
    """
    x_left_array = [] #create an array to store x-values from the left side of the image
    y_left_array = [] #create an array to store y-values from the left side of the image
    
    x_right_array = [] #create an array to store x-values from the right side of the image
    y_right_array = [] #create an array to store y-values from the right side of the image
    
    shape = img.shape #returns a tuple of number of rows, columns and channels
    #identify the left and right lane lines with line segments from the region of interest
    for line in lines:
        for x1,y1,x2,y2 in line:
            if x1 < shape[1]/2 and x2 < shape[1]/2: #index by columns to split the image from left side to right side
                # Collecting all x and y pairs from left side of image and put them in the left side arrays
                x_left_array += [x1, x2]
                y_left_array += [y1, y2]
            else:
               # Collecting all x and y pairs from right side of image and put them in the right side arrays
                x_right_array += [x1, x2]
                y_right_array += [y1, y2]
    #Plot line from the left side        
    if len(x_left_array) != 0 and len(y_left_array) != 0:
        m,b = np.polyfit(x_left_array, y_left_array, 1) #returns the coefficients m,b of the 1 degree polynomial y = mx^1 +bx^0
        max_x = max(x_left_array)
        # Plot line from x1 = (max_y - b)/m, y1 = max_y to x2 = max_x, y2 = m*max_x + b
        cv2.line(img, (int((shape[0]-b)/m), shape[0]), (max_x, int(m*max_x + b)), color, thickness)
    
    #Plot line from the right side
    if len(x_right_array) != 0 and len(y_right_array) != 0:
        m,b = np.polyfit(x_right_array, y_right_array, 1) #returns the coefficients m,b of the 1 degree polynomial y = mx^1 +bx^0
        min_x = min(x_right_array)
        # Plot line from x1 = min_x, y1=(min_x*m)+b to x2=(max_y-b)/m, y2 = max_y
        cv2.line(img, (min_x, int(min_x * m + b)), (int((shape[0]-b)/m), shape[0]), color, thickness)
#    Comment out the below codes, this was for plotting the line segments
#    for line in lines:
#        for x1,y1,x2,y2 in line:
#            cv2.line(img, (x1, y1), (x2, y2), color, thickness)

def hough_lines(img, rho, theta, threshold, min_line_len, max_line_gap):
    """
    `img` should be the output of a Canny transform.
        
    Returns an image with hough lines drawn.
    """
    lines = cv2.HoughLinesP(img, rho, theta, threshold, np.array([]), minLineLength=min_line_len, maxLineGap=max_line_gap)
    line_img = np.zeros((img.shape[0], img.shape[1], 3), dtype=np.uint8)
    draw_lines(line_img, lines)
    return line_img

# Python 3 has support for cool math symbols.

def weighted_img(img, initial_img, ??=0.8, ??=1., ??=0.):
    """
    `img` is the output of the hough_lines(), An image with lines drawn on it.
    Should be a blank image (all black) with lines drawn on it.
    
    `initial_img` should be the image before any processing.
    
    The result image is computed as follows:
    
    initial_img * ?? + img * ?? + ??
    NOTE: initial_img and img must be the same shape!
    """
    return cv2.addWeighted(initial_img, ??, img, ??, ??)
```

## Test Images

Build your pipeline to work on the images in the directory "test_images"  
**You should make sure your pipeline works well on these images before you try the videos.**


```python
import os
fileNameImages = os.listdir("test_images/")
```

## Build a Lane Finding Pipeline



Build the pipeline and run your solution on all test_images. Make copies into the `test_images_output` directory, and you can use the images in your writeup report.

Try tuning the various parameters, especially the low and high Canny thresholds as well as the Hough lines parameters.


```python
# TODO: Build your pipeline that will draw lane lines on the test_images
# I used these codes to save the images after each transforms to its own transform folder
directory = "test_images_output/"
if not os.path.isdir("test_images_output"):
    os.mkdir("test_images_output")
directory2 = "test_images_output_gray/"
if not os.path.isdir("test_images_output_gray"):
    os.mkdir("test_images_output_gray")
directory3 = "test_images_output_gaussian/"
if not os.path.isdir("test_images_output_gaussian"):
    os.mkdir("test_images_output_gaussian")
directory4 = "test_images_output_canny/"
if not os.path.isdir("test_images_output_canny"):
    os.mkdir("test_images_output_canny")
directory5 = "test_images_output_hough/"
if not os.path.isdir("test_images_output_hough"):
    os.mkdir("test_images_output_hough")
for each_image in os.listdir('test_images/'):
    image = mpimg.imread('test_images/' + each_image)
    processed_img = grayscale(image)
    mpimg.imsave(directory2 + each_image , processed_img)
    processed_img = gaussian_blur(processed_img,5)
    mpimg.imsave(directory3 + each_image , processed_img)
    processed_img = canny(processed_img, 50, 150)
    mpimg.imsave(directory4 + each_image , processed_img)
    imshape = processed_img.shape
    vertices = np.array([[(0,imshape[0]),(450, 335), (540, 335), (imshape[1],imshape[0])]], dtype=np.int32)
    processed_img = region_of_interest(processed_img,vertices)
    line_image = np.copy(processed_img)*0 # creating a blank to draw lines on
    processed_img = hough_lines(processed_img,2 , np.pi/180, 10, 40, 130)
    mpimg.imsave(directory5 + each_image , processed_img)
    processed_img = weighted_img(processed_img, image, ??=0.8, ??=1., ??=0.)
    output_image = processed_img
    mpimg.imsave(directory + each_image , output_image)
    plt.imshow(output_image)

```


    
![png](output_16_0.png)
    


## Test on Videos

You know what's cooler than drawing lanes over images? Drawing lanes over video!

We can test our solution on two provided videos:

`solidWhiteRight.mp4`

`solidYellowLeft.mp4`

**Note: if you get an import error when you run the next cell, try changing your kernel (select the Kernel menu above --> Change Kernel). Still have problems? Try relaunching Jupyter Notebook from the terminal prompt. Also, consult the forums for more troubleshooting tips.**

**If you get an error that looks like this:**
```
NeedDownloadError: Need ffmpeg exe. 
You can download it by calling: 
imageio.plugins.ffmpeg.download()
```
**Follow the instructions in the error message and check out [this forum post](https://discussions.udacity.com/t/project-error-of-test-on-videos/274082) for more troubleshooting tips across operating systems.**


```python
# Import everything needed to edit/save/watch video clips
from moviepy.editor import VideoFileClip
from IPython.display import HTML
```


```python
def process_image(image):
    # NOTE: The output you return should be a color image (3 channel) for processing video below
    # TODO: put your pipeline here,
    processed_img = grayscale(image)
    processed_img = gaussian_blur(processed_img,5)
    processed_img = canny(processed_img, 50, 150)
    imshape = processed_img.shape
    vertices = np.array([[(0,imshape[0]),(450, 335), (540, 335), (imshape[1],imshape[0])]], dtype=np.int32)
    processed_img = region_of_interest(processed_img,vertices)
    line_image = np.copy(processed_img)*0 # creating a blank to draw lines on
    processed_img = hough_lines(processed_img,2 , np.pi/180, 10, 40, 130)
    processed_img = weighted_img(processed_img, image, ??=0.8, ??=1., ??=0.)
    plt.imshow(processed_img)
    # you should return the final output (image where lines are drawn on lanes)
    result = processed_img
    return result
```

Let's try the one with the solid white lane on the right first ...


```python
white_output = 'test_videos_output/solidWhiteRight.mp4'
## To speed up the testing process you may want to try your pipeline on a shorter subclip of the video
## To do so add .subclip(start_second,end_second) to the end of the line below
## Where start_second and end_second are integer values representing the start and end of the subclip
## You may also uncomment the following line for a subclip of the first 5 seconds
#clip1 = VideoFileClip("test_videos/solidWhiteRight.mp4").subclip(0,5)
clip1 = VideoFileClip("test_videos/solidWhiteRight.mp4")
white_clip = clip1.fl_image(process_image) #NOTE: this function expects color images!!
%time white_clip.write_videofile(white_output, audio=False)
```

    t:   0%|          | 0/221 [00:00<?, ?it/s, now=None]

    Moviepy - Building video test_videos_output/solidWhiteRight.mp4.
    Moviepy - Writing video test_videos_output/solidWhiteRight.mp4
    
    

                                                                  

    Moviepy - Done !
    Moviepy - video ready test_videos_output/solidWhiteRight.mp4
    Wall time: 6.87 s
    


    
![png](output_21_4.png)
    


Play the video inline, or if you prefer find the video in your filesystem (should be in the same directory) and play it in your video player of choice.


```python
HTML("""
<video width="960" height="540" controls>
  <source src="{0}">
</video>
""".format(white_output))
```





<video width="960" height="540" controls>
  <source src="test_videos_output/solidWhiteRight.mp4">
</video>




## Improve the draw_lines() function

**At this point, if you were successful with making the pipeline and tuning parameters, you probably have the Hough line segments drawn onto the road, but what about identifying the full extent of the lane and marking it clearly as in the example video (P1_example.mp4)?  Think about defining a line to run the full length of the visible lane based on the line segments you identified with the Hough Transform. As mentioned previously, try to average and/or extrapolate the line segments you've detected to map out the full extent of the lane lines. You can see an example of the result you're going for in the video "P1_example.mp4".**

**Go back and modify your draw_lines function accordingly and try re-running your pipeline. The new output should draw a single, solid line over the left lane line and a single, solid line over the right lane line. The lines should start from the bottom of the image and extend out to the top of the region of interest.**

Now for the one with the solid yellow lane on the left. This one's more tricky!


```python
yellow_output = 'test_videos_output/solidYellowLeft.mp4'
## To speed up the testing process you may want to try your pipeline on a shorter subclip of the video
## To do so add .subclip(start_second,end_second) to the end of the line below
## Where start_second and end_second are integer values representing the start and end of the subclip
## You may also uncomment the following line for a subclip of the first 5 seconds
##clip2 = VideoFileClip('test_videos/solidYellowLeft.mp4').subclip(0,5)
clip2 = VideoFileClip('test_videos/solidYellowLeft.mp4')
yellow_clip = clip2.fl_image(process_image)
%time yellow_clip.write_videofile(yellow_output, audio=False)
```

    t:   0%|          | 0/681 [00:00<?, ?it/s, now=None]

    Moviepy - Building video test_videos_output/solidYellowLeft.mp4.
    Moviepy - Writing video test_videos_output/solidYellowLeft.mp4
    
    

                                                                  

    Moviepy - Done !
    Moviepy - video ready test_videos_output/solidYellowLeft.mp4
    Wall time: 20 s
    


    
![png](output_26_4.png)
    



```python
HTML("""
<video width="960" height="540" controls>
  <source src="{0}">
</video>
""".format(yellow_output))
```





<video width="960" height="540" controls>
  <source src="test_videos_output/solidYellowLeft.mp4">
</video>




## Writeup and Submission

If you're satisfied with your video outputs, it's time to make the report writeup in a pdf or markdown file. Once you have this Ipython notebook ready along with the writeup, it's time to submit for review! Here is a [link](https://github.com/udacity/CarND-LaneLines-P1/blob/master/writeup_template.md) to the writeup template file.

