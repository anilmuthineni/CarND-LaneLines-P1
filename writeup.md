# **Finding Lane Lines on the Road** 


The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


### Reflection

My pipeline consisted of the following steps. Output from each step is input to the next step. Input to the first step is the RGB numpy image and output from the last step is the image with lane lines drawn on it.

1. Convert the image into HLS. It is easy to find lane line colors in HLS format.
```python
  	hls_image = cv2.cvtColor(image, cv2.COLOR_RGB2HLS)
```
2. Extract mask for yellow and while colors in the image
```python 
    yellow_mask_image = cv2.inRange(hls_image, np.array([15, 38, 115], np.uint8),
                                    np.array([35, 204, 255], np.uint8))
    white_mask_image = cv2.inRange(hls_image, np.array([0, 200, 0], np.uint8),
                                   np.array([180, 255, 255], np.uint8))
    lanes_mask_image = cv2.bitwise_or(yellow_mask_image, white_mask_image)
```
3. Extract pixels in mask from original image
```python 
	original_with_mask = cv2.bitwise_and(image, image, mask=lanes_mask_image)
```
4. Convert image into gray scale
```python 
    gray_image = cv2.cvtColor(original_with_mask, cv2.COLOR_BGR2GRAY)
```
5. Use gaussian blur to smoothen the image
```python 
   gaussian_blur_image = cv2.GaussianBlur(gray_image, (5, 5), 0)
```
6. Use canny edge detection to identify pixels on the edges
```python 
	canny_image = cv2.Canny(gaussian_blur_image, 50, 150)
```
7. Mask out pixels outside region of interest
```python 
	height = image.shape[0]
    width = image.shape[1]
    interested_region_polygon = np.array([[0, height],
                                          [width/2, height/2],
                                          [width, height]],
                                         np.int32)
    
    roi_image = region_of_interest(canny_image, [interested_region_polygon])
```
8. Detect lines in the image using cv2.HoughLinesP
```python 
lines = cv2.HoughLinesP(roi_image, rho=2, theta=((np.pi/180) * 1), threshold=50, minLineLength=50,
                    maxLineGap=200)
```
9. Detect amd merge close by lines. Ideally this should output two lines. If there are more than two lines, then out lane detection algorithm has erred and detected more than 2 lines. We will take first two lines in that case.
```python 
	lines_after_merge = extend_and_merge(lines, image.shape[0])
    if len(lines_after_merge) != 2:
        lines_after_merge = lines_after_merge[:2]
    for line in lines_after_merge:
        x1, y1, x2, y2 = line
        cv2.line(image_copy, (x1, y1), (x2, y2), [255, 0, 0], 10)
```
10. Create an image with lane lines overlayed
```python 
image_with_lines = weighted_img(image, image_copy, 0.5, 0.5)
```
In order to draw a single line on the left and right lanes, I wrote the function extend_and_merge. It first extends each line till the bottom of the image (maximum y) and finds out groups of lines with closely spaced slopes and x coordinate at y max.


### 2. Identify potential shortcomings with your current pipeline
One potential shortcoming is that lanes are represented as lines, so we cannot identify curved lanes on curved roads.

### 3. Suggest possible improvements to your pipeline

A possible improvement would be to be able to identify curved lane lines.
