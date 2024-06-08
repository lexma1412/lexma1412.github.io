---
layout: default
title:  "Image Moment"
---

# Image Moment
This post introduces briefly about moment in image processing. Moment demonstrates area (total intensity), centroid, and information about orientation of objects in image (after segmentation).<br />

# Moment
In grayscale image, moment is defined as below:
![moment formula](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/ImageMoment/Moment_formula.png?raw=true)<br/>
with $Mij$ is moment (i+j) order of image, $I(x,y)$ is intensity of pixel at position (x,y)<br/>

$M00$ is zero order moment, it provides the **area** value of considered object in image.To calculate it, we just need to sum all intensity of pixels of image or of interested region (object region) <br/>
* $M00 =  \sum I(x,y)$

$M10$ and $M01$ is first order moment, base on that, the **centroid** of considered object is calculated as $(\overline{x},\overline{y}) =(M10/M00;M01/M00)$ with: 
* $M10 = \sum_{x}\sum_{y}xI(x,y)$
* $M01 = \sum_{x}\sum_{y}yI(x,y)$

## Central Moment
Next, we will introduce about **central moment $\mu$**, it provides crucial information bout the shape and structure of objects in an image, independent of their location. **central moment** in image is defined as:

* $\mu_{pq} = \sum_{x}\sum_{y}(x-\overline{x})^p(y-\overline{y})^qI(x,y)$

The second order central moment:
* $\mu_{20} = M20 - \overline{x}M10$
* $\mu_{02} = M02 - \overline{y}M01$
* $\mu_{11} = M11 - \overline{x}M01 =  M11 - \overline{y}M10$<br/>

and so on...Base on $\mu_{20}, \mu_{02}, \mu_{11}$, covariant matrix of image is known with elements are $\mu_{20}^{'}, \mu_{02}^{'}, \mu_{11}^{'}$

* $\mu_{20}^{'}  = \mu_{20}/\mu_{00} = M20/M00 - \overline{x}^2$
* $\mu_{02}^{'} = \mu_{02}/\mu_{00} = M02/M00 - \overline{y}^2$
* $\mu_{11}^{'} = \mu_{11}/\mu_{00} =  M11/M00 - \overline{x} \overline{y}$<br/>

where ($\overline{x},\overline{y}$) is centroid of object<br/>
Central moment is **invariant translation**, it means that when object moving in the image, its central moment does not change (independence from its location).<br/>

The angle of the eigenvector of covariant matrix provides information about the angle of object:

* $\theta = 1/2 tan^{-1}({2\mu_{11}}/{(\mu_{20} - \mu_{02})})$<br/>

Let go with example: TBD

## Moment invariants
Base on the central moment concept, Hu [3] proposes Translation/Scale/Rotation invariants that are invariant from translation, scaling and rotation. So it is called moment invariants or Hu moment invariants.<br/>

**Scaling invariant** $\eta$ is defined as:

* $\eta_{ij} = \mu_{ij}/(\mu_{00}^{1+(i+j)/2})$<br/>

**Rotation invariant** $I$ is defined as:
![Hu moment formula](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/ImageMoment/Hu_moment.png?raw=true)<br/>


## Reference
[1] Wiki: https://en.wikipedia.org/wiki/Image_moment <br/>
[3] M. K. Hu, "Visual Pattern Recognition by Moment Invariants"











