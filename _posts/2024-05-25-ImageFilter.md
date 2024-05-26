---
layout: default
title:  "Image Filter"
---

# Image Filter
In image processing, filter is important step to pre-process input image to remove noise in input data. There are many filter techniques, this post will summarize some of them that supported in opencv library

## Gaussian Filter
Gaussian Filter also called Gaussian blur in opencv,  is a low pass filter used for reducing noise. First, let see  gaussian function definition:<br />
![Gaussian Filter formula](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/ImageFilter/Gaussian_formula.png?raw=true)


- From this definition, we can contribute a 2-D matrix that called Gaussian kernel and size is alway odd (e.g: 3x3. 5x5). E.g. we have below 3x3 gaussian kernel with sigma = 1:<br />
![Gaussian Filter formula](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/ImageFilter/3x3GaussianKernel.png?raw=true)

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
- If you want, you can create a Gaussian kernel with the function, cv.getGaussianKernel().

## Sobel Filter
Sobel filter is  High-pass filter, it will specify the **derivative (or gradient)**  Gx, GY in X and Y direction by convolute with corresponding Sobel kernel.
- 3x3 Sobel kernel for X direction:<br />
<p>-1 0 1<br />
-2 0 2<br />
-1 0 1</p>

- 3x3 Sobel kernel for X direction:<br />
<p>-1 -2 -1<br />
0 0 0<br />
1 2 1</p>

- Magnitude of gradient vector can be calculated as: sqrt(Gx^2+Gy*2)

```python
cv.Sobel(src, ddepth, dx, dy[, dst[, ksize[, scale[, delta[, borderType]]]]]) ->dst
```


## Laplacian Filter
Laplacian filter is  High-pass filter, it will specify the 2nd order derivative.

- In opencv, it specifies 2 approaches to calculate 2nd order derivative
* (1)Using below Laplacian kernel as a filter kernel and convolute 1 time only:
<p>0 -1 0<br />
-1 4 -1<br />
0 -1 0</p>
- Other kind of Laplacian kernel you can found when searching is :
<p>-1 -1 -1<br />
-1 8 -1<br />
-1 -1 -1</p>

* (2)Using Sobel kernel to calculate 1st and also for 2nd derivative.

```python
cv.Laplacian(src, ddepth[, dst[, ksize[, scale[, delta[, borderType]]]]]) ->dst
```