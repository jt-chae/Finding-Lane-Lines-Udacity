# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 6 steps. Step 1, I converted the image to grayscale using the grayscale function so that my image would only have 1 color channel. Step 2, I applied the Gaussian Noise kernel to blur the image so that I can redude the noise for my next function. Step 3, I applied the Canny transform on the the image to to find the edges of the lane lines. Step 4, I applied a mask to an image to only keep my region of interest defined by the polygon formed from my selected vertices, and the rest of the image is set to black. Step 5, I use OpenCVs Hough Transform to detect line segments in my image by transforming (x1,y1,x2,y2) into m,b pairs. Step 6, I draw the line by calling the draw_lines() function inside my Hough Transform function.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by first splitting the image into left side and right side. I created arrays to store each x and y value on each side of the image. I then collected all the x and y values on both sides and put them as pairs of two points in the corresponding arrays that I created. To plot the line on the left side of the image, I ran the poly.fit() function on the x-array and y-array, which returns the coefficients m,b of the 1 degree polynomial y = mx^1 + bx^0 for the best fit. After finding the slope of the line, I can then draw the straight line using the max value of x in the x-array, and using the slope and the formula y = mx + b to calculate the y value at that point. Applying the same codes, I can draw the line for the right side also.


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be when there is a sharp turn instead of a straight line lane. Since we use the best fitted 1 degree polinomial y = mx + b, the draw_line() function will return a straight line.

Another shortcoming could be that if another car cuts in the front camera suddenly, meaning an obstacle enters the region of interest, our image would not be filtered correctly.


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to increase the degree of the polinomial, instead of 1 degree, it can be 2 degree for curves, y = Ax^2 + Bx + C.

Another potential improvement could be to have a back up program to output a different region of interest when an obstable enters the image to take in account the space taken by the obstacle and adjust the car's speed to go back to the programmed region of interest.
