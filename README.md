# CannyEdgeDetection-from-scratch-python
This  is an implementation of Canny Edge Detection Algorithm in Python without using any library. There are many improvements that can be made in this implementation.

Canny's Edge Detector is well known for its ability to generate single-picel thick continuous edges. It consists of four major steps, which are described below, along with interesting implementation details and outputs. The program has four inputs: Input image I, value of smoothing parameter sigma, high threshold Th and low threshold Tl.

## Step 1: Generation of Masks:

This module requires the value of sigma as an input and generates x- and y-derivative masks as output. In Canny's method, the masks used are 1st derivative of a Gaussian in x- and y-directions. The masks are also written in two text files for reference.
To generate the masks, the first step is a reasonable computation of the mask size. The mask size should not be too large compared to the lobes of the mask, otherwise it will result in unnecessary computational overhead during convolution. At the same time, the mask size should not be so small to loose the characteristics of primary lobes. We chose mask size based on analysing the Gaussian and applying a threshold T. This idea is explained in the figure.
First, size of half mask, sHalf is computed by finding the point on the curve where the Gaussian value drops below T, i.e. solving exp(-x^2/(2*sigma^2) = T. This gives sHalf = round(sqrt(-log(T) * 2 * sigma^2)). The mask size is then 2*sHalf + 1 to incorporate both positive and negative sides of the mask. We also put a lower limit on sigma of 0.5, because below that the mask size came out to be less than 3, which was not reasonable for finding fx and fy.

In our notation, x-direction is taken to be vertically downward, and y-direction horizontally rightward. This means that the lobes of x-mask are vertically aligned, while those of y-mask are horizontally aligned. Once the masks are generated, they are scaled and rounded off so that convolution is done with integer values rather than floats. The scale factor is saved, because gradient magnitude is later scaled down (after convolution) by the same factor.

## Step 2: Applying Masks to Images

The masks are applied to the images using convolution. The result is then scaled down by the same factor which was used to scale up the masks. To write output to image files, the min and max values are scaled to 0 and 255 respectively. 

The effect of increasing sigma is obvious from the convolved images: the gradients are much smoother. It should be noted that horizontally running edges are identified in fx, because the gradient change occurs when moving along the x (downward) direction. Simlarly vertical edges are seen in fy because our definition of y-axis is along the column direction.

Gradient Direction, Phi, is computed using atan2 function. The difference between atan and atan2 functions is that atan returns output range from -pi/2 to pi/2, whereas atan2 returns in the range of -pi to pi. Thus atan2 is preferred because that is the real range of the possible directions of gradient in an image. To keep things simple in our code, we converted the angle returned by atan2 function to degrees and added 180 to get an output range of 0-360 degrees.
This image is difficult to interpret, even though it can be seen that linear structures like pillars, have similar gradient direction values. Also, the ropes from which the flags are hanging show similar values of gradient direction. A better visualization technique is to view only those values of direction for which gradient magnitude is high enough. In this method, we made all values in the gradient direction image -1 where gradient magnitude was below a certain threshold (t = 10). Thus, the output is visible only at significant edges.

## Step 3: Non Maxima Suppression:

Non maxima supression step makes all edges in M one pixel thick. This is an important step in Canny's algorithm, which distinguishes it from other algorithms. The first step is to quantize gradient direction into just four directions. In our implementation, the following values were assigned during quantization

Angle Value      assigned
      
      0        0 to 22.5
              157.5 to 202.5
              337.5 to 360

      1       22.5 to 67.5
              202.5 to 247.5
          
      2       67.5 to 112.5
              247.5 to 292.5
          
      3       112.5 to 157.5
              292.5 to 337.5

The next step is to pick two neighbors of each edge point along the gradient direction. This is because gradient direction is perpendicular to the edge, and therefore, this is the direction in which we are searching for edge points. For example, for quantized direction 0 (in the table above), the gradient direction could be less than 0 degrees, meaning the edge is a horizontal edge. Therefore the two neighbors that need to be picked for comparison are the north and south neighbors, denoted in code by (r-1, c) and (r+1,c) respectively. If the edge point (r,c) is greater than both these neighbors, then it is maintained in M otherwise it is made zero.

## Step 4: Hysteresis Thresholding

The final step in Canny's edge detection algorithm is to apply two thresholds to follow edges. Since edges are to be followed recursively by looking at neighbors, we first made the border pixels zero, so that finding neighbors does not go out of bounds of the image. Next, the image is scanned from left to right, top to bottom. The first pixel in non-maxima suppressed magnitude image which is above a certain threshold, Th, is declared an edge. Then all its neighbors are recursively followed, and those above threshold, Tl, are marked as an edge. A visited map is also maintained so that recursion does not loop infinitely. Thus there are really two stopping conditions: if a neighbor is below Tl, we wont recurse on it; also, if a neighbor has already been visited, then we won't recurse on it.
