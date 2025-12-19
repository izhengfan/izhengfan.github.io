---
layout: post
title: "OpenCV: Basic Structures"
date: 2016-01-14 12:00:00
categories: en
tags: OpenCV
---

__Contents__

* contents
{:toc}

### Mat like a Smart Pointer

The mostly used basic structure in OpenCV C++ is `Mat`. And the first thing to remember about `Mat` is to treat it like a smart pointer (like `shared_ptr`) with the addition of some meta data (like `cols` `rows` etc), instead of an ordinary struct or container (like `std::vector`). Hence, when using `=` operator to pass `Mat` like:

```cpp
using namespace cv;
Mat m1;
Mat m2 = m1;
```

You must know that apart from some meta data, `m2` and `m1` share one memory part for their internal matrix data, and any change you make to the matrix data of `m1` or `m2` will happen to another one. The reason why OpenCV implements `Mat` like this is to release users from mannual memory management. Since in computer vision applications, image data is usually stored in `Mat`, and it would be huge cost for the computer if image data was frequently copied and reallocated in memory. 

Internally, such mechanism is achieved by the internal `cv::Mat::data` pointer, which is managed by OpenCV and ordinary users do not need to care about its details.

Of course, as `Mat` is quite a good tool for matrix operation, in many cases we use it to store small matrices (like a 3 by 3 rotation matrix). For such operations, making an individual copy of `Mat` is necessary and would cost little computer resources. To create a new clone independent from `m1`, we can use `clone()` function:

```cpp
Mat m2 = m1.clone();
```

Or if we already have `m2`, use `copyTo()`:

```cpp
m1.copyTo(m2);
```



### Some Functions For Mat

#### copyTo

There are a bit more need to mention about `copyTo()` function. Here the tricky thing is, with `m1.copyTo(m2)`, what exactly happens to `m2` depends on whether `m2` has the same size as `m1`. We use a simple example here:

```cpp
Mat m1 = Mat::ones(3,3,CV_32FC1);
Mat m2 = m1;
Mat m3 = Mat::zeros(3,3,CV_32FC1);
m3.copyTo(m1);
cout << "m1 = " << endl << m1 << endl;
cout << "m2 = " << endl << m2 << endl;
```

This will give us:

    m1 = 
    [0, 0, 0;
    0, 0, 0;
    0, 0, 0]
    m2 = 
    [0, 0, 0;
    0, 0, 0;
    0, 0, 0]

The result is reasonable. Since `m2` and `m1` share the same memory location, as we copy the data of `m3` to `m1`, the same change will happen to `m2`. However, if we initialize `m3` to be

```cpp
Mat m3 = Mat::zeros(3,2,CV_32FC1);
```

The output will be 

    m1 = 
    [0, 0;
    0, 0;
    0, 0]
    m2 = 
    [1, 1, 1;
    1, 1, 1;
    1, 1, 1]

`m2` does not change like `m1`! This is because `m1` and `m3` have different size. In such situation, when copying data from `m3` to `m1`, OpenCV will allocate a new memory location for `m1`, hence `m1` and `m2` no longer share the same memory location, and are independent from each other ever since.

#### reshape

`reshape` function is to rearrange the elements in a `Mat` structure. Its format is like

```cpp
Mat Mat::reshape(int cn, int rows=0) const
```

Parameters:	

- cn – New number of channels. If the parameter is 0, the number of channels remains the same.

- rows – New number of rows. If the parameter is 0, the number of rows remains the same.

The key of using this function is to remember:

1. the number of elements in (rows * cols * numChannels) must be the same before and after reshaping

2. the elements keep an order of 'left to right, up to down'

Suppose we have 

	mat = [a b c d]

Then `mat.reshape(0,2)` gives us

	[a b; c d]
    
#### push_back

Just like what we do with `std::vector`, we can also push a `Mat` (its rows) to the back of another `Mat`. This is sometimes very convenient. For example, to stitch two images, just do:

```cpp
img1.push_back(img2);
```

It will make a new image with img2 under the original img1.

### Mat_ as Template Container

Apart from `Mat`, OpenCV also provide a template container `Mat_`, which is derived from `Mat`, for more convenient matrix operation. For example, with `Mat_`, we can fill a matrix like:

```cpp
float theta;
Mat m = (Mat_<float>(3,3) <<
        cos(theta), -sin(theta), 0,
        sin(theta),  cos(theta), 0,
        0,           0,          1);
```

Very elegant, right? And a bit like MATLAB. And the use of `<<` operator is very C++. Eigen library also support similar operation style on matrix.

BTW, the above opperation will also allocate a new memory location for `m`. Therefore, with the code below:

```cpp
Mat m1 = Mat::ones(3,3,CV_32FC1);
Mat m2 = m1;
Mat m3 = Mat::zeros(3,3,CV_32FC1);
m1 = (Mat_<float>(3,3) << 1,0,0,0,1,0,0,0,1);
cout << "m1 = " << endl << m1 << endl;
cout << "m2 = " << endl << m2 << endl;
```

The output is 

    m1 = 
    [1, 0, 0;
    0, 1, 0;
    0, 0, 1]
    m2 = 
    [1, 1, 1;
    1, 1, 1;
    1, 1, 1]

`m1` no longer points to the same location as `m2`.

For element access, `Mat_` also provides more convenient interfaces than `Mat`. One example:

```cpp
void Frame::computeBoundUn(const Mat& K, const Mat& D){
    float x = (float)img.cols;
    float y = (float)img.rows;
    if(D.at<float>(0) == 0.){
        minXUn = 0.f;
        minYUn = 0.f;
        maxXUn = x;
        maxYUn = y;
        return;
    }
    Mat_<Point2f> mat(1,4);
    mat << Point2f(0,0), Point2f(x,0), Point2f(0,y), Point2f(x,y);
    undistortPoints(mat, mat, K, D, Mat(), K);
    minXUn = std::min(mat(0).x, mat(2).x);
    minYUn = std::min(mat(0).y, mat(1).y);
    maxXUn = std::max(mat(1).x, mat(3).x);
    maxYUn = std::max(mat(2).y, mat(3).y);
}
```

See the last four lines. With `(i)` (or `(i, j)`), element access becomes much convenient than using `.at<TypeName>(i, j)` function.


### Matrices with More than Two Dimensions

We usually use `Mat` for 2D matrices. What if we want multi-dimension matrices? One method is like:

```cpp
int dims[] = {3,4,5};
Mat mat(3, dims, CV_32FC1, Scalar(0));
cout << mat.at<float>(0,0,0);
```

Initialize a multi-dimension matrix by passing in an `int` for how many dimensions to assign and an array for the depth of every dimension. For element access, use `at<TypeName>()` function, as we do for 2-D Mat.

Using a 2D `Mat` with more than one channel is also common. For example, for an RGB image, we can use `Mat(m, n, CV_8UC3)`. Every element of the matrix has three channels, each channel representing the value of Red/Green/Blue. To access the element in a specific channel of a given location `(i, j)`, use `cv::Vec3b`:

```cpp
uchar newval[] = {255, 255, 255};
image.at<cv::Vec3b>(i, j)[0] = newval[0];
image.at<cv::Vec3b>(i, j)[1] = newval[1];
image.at<cv::Vec3b>(i, j)[2] = newval[2];
```  

Although these can be used for multi-dimension matrices, strictly speaking, they are still 2-D matrices, and are different from those initialized directly with dimention number and dimension depths.
