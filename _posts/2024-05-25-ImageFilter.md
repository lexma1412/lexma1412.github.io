---
layout: default
title:  "Image Filter"
---

# Image Filter
In image processing, filter is important step to pre-process input image to remove noise in input data. There are many filter techniques, this post will summarize some of them that supported in opencv library

## Gaussian Filter
Gaussian Filter also called Gaussian blur in opencv,  is a low pass filter used for reducing noise. First, let come with gaussian function definition:<br />
![Gaussian Filter formula](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/ImageFilter/Gaussian_formula.png?raw=true)

- From this definition, we can contribute a 2-D matrix that called Gaussian kernel and size is alway odd (e.g: 3x3. 5x5). E.g. we have below 3x3 gaussian kernel with sigma = 1, 

- For step by step to create this kernel, let consider one random pixel and coordinate around this pixel with sigma = 1 
<p> (−1,−1) (−1,0) (−1,1)<br />
(0,−1) <strong>(0,0)</strong> (0,1) <br />
(1,−1) (1,0) (1,1)</p>

- Calculate each element of kernel
<p>G(0,0)=1/(2*pi)=0.159<br />
G(1,0)=G(−1,0)=G(0,1)=G(0,−1)=1/(2*pi)e^(-0.5)=0.097<br />
G(1,1)=G(1,−1)=G(−1,1)=G(−1,−1)=1/(2*pi)e^(-1)=0.059</p>

- Next step is normalize the kernel by diving each element with sum. After normalization, the values approximate to:
<p>0.075 0.123 0.075<br />
0.123 0.203 0.123<br />
0.075 0.123 0.075</p>

- For example code, let use function provided in python opencv
```python
# Gaussian filter function
cv2.GaussianBlur(src, ksize, sigmaX[, dst[, sigmaY[, borderType]]]) ->	dst
```