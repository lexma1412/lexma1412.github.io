---
layout: default
title:  "Image Gradient"
---

# Image Gradient
Gradient vector in 2D image is a vector with 2 element is derivative in x and y direction
![Gradient Vector]((https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/ImageGradient/GradientVector.png?raw=true))

Its magnitude can show the information about how much the intensity or color at particular pixel is changed and orientation can express the direction or angel (in radian) that intensity is mostly changed, therefore, gradient vector is usually used to find edge in the image.
![Edge change]((https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/ImageGradient/Egde_Change.png?raw=true))

In digital image processing, to calculate Gradient, we can use use kernel (e.g: Sobel) and perform convolution with result is Gradient in x or y direction.<br/>
Refer to this post for more detail about Sobel kernel: [Link to a post](./_posts/2024-05-25-ImageFilter.md)


# Reference
[1] https://en.wikipedia.org/wiki/Image_gradient