# opencv-mobile

![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg?style=for-the-badge)
![build](https://img.shields.io/github/actions/workflow/status/rabiloo/opencv-mobile/release.yml?style=for-the-badge)
![download](https://img.shields.io/github/downloads/rabiloo/opencv-mobile/total.svg?style=for-the-badge)

![Android](https://img.shields.io/badge/Android-3DDC84?style=for-the-badge&logo=android&logoColor=white)
![ARM Linux](https://img.shields.io/badge/ARM_Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Firefox](https://img.shields.io/badge/Firefox-FF7139?style=for-the-badge&logo=Firefox-Browser&logoColor=white)
![Chrome](https://img.shields.io/badge/Chrome-4285F4?style=for-the-badge&logo=Google-chrome&logoColor=white)

✔️ This project provides the minimal build of opencv library for the **Android** and **ARM Linux** platforms.

✔️ Packages for **Linux** and **WebAssembly** are available now.

✔️ We provide prebuild binary packages for opencv **3.4.18**.

✔️ All the binaries are compiled from source on github action, **no virus**, **no backdoor**, **no secret code**.

# Download

## Android

(armeabi-v7a, arm64-v8a, x86, x86_64) build with ndk r25b and android api 24.

* [opencv-mobile-3.4.18-android.zip](https://github.com/rabiloo/opencv-mobile/releases/download/v17/opencv-mobile-3.4.18-android.zip)


## ARM Linux

(arm-linux-gnueabi, arm-linux-gnueabihf, aarch64-linux-gnu) build with ubuntu cross compiler.

* [opencv-mobile-3.4.18-armlinux.zip](https://github.com/rabiloo/opencv-mobile/releases/download/v17/opencv-mobile-3.4.18-armlinux.zip)

## Linux

(x86_64) build on Ubuntu-18.04, 20.04 and 22.04.

* [opencv-mobile-3.4.18-ubuntu-1804.zip](https://github.com/rabiloo/opencv-mobile/releases/download/v17/opencv-mobile-3.4.18-ubuntu-1804.zip)
* [opencv-mobile-3.4.18-ubuntu-2004.zip](https://github.com/rabiloo/opencv-mobile/releases/download/v17/opencv-mobile-3.4.18-ubuntu-2004.zip)
* [opencv-mobile-3.4.18-ubuntu-2204.zip](https://github.com/rabiloo/opencv-mobile/releases/download/v17/opencv-mobile-3.4.18-ubuntu-2204.zip)

## WebAssembly

(basic, simd, threads, simd+threads) build with Emscripten 2.0.8.

* [opencv-mobile-3.4.18-webassembly.zip](https://github.com/rabiloo/opencv-mobile/releases/download/v17/opencv-mobile-3.4.18-webassembly.zip)

# Usage Android

1. Extract archive to ```<project dir>/app/src/main/jni/```
2. Modify ```<project dir>/app/src/main/jni/CMakeListst.txt``` to find and link opencv

```cmake
set(OpenCV_DIR ${CMAKE_SOURCE_DIR}/opencv-mobile-3.4.18-android/sdk/native/jni)
find_package(OpenCV REQUIRED)

target_link_libraries(your_jni_target ${OpenCV_LIBS})
```

# Usage ARM Linux, Linux, WebAssembly

1. Extract archive to ```<project dir>/```
2. Modify ```<project dir>/CMakeListst.txt``` to find and link opencv

```cmake
set(OpenCV_DIR ${CMAKE_SOURCE_DIR}/opencv-mobile-3.4.18-armlinux/arm-linux-gnueabihf/lib/cmake/opencv4)
find_package(OpenCV REQUIRED)

target_link_libraries(your_target ${OpenCV_LIBS})
```

# How-to-build your custom package

**step 1. download opencv source**
```shell
wget -q https://github.com/opencv/opencv/archive/3.4.18.zip -O opencv-3.4.18.zip
unzip -q opencv-3.4.18.zip
cd opencv-3.4.18
```

**step 2. strip zlib dependency and use stb-based highgui implementation (optional)**
```shell
patch -p1 -i ../opencv-3.4.18-no-zlib.patch
truncate -s 0 cmake/OpenCVFindLibsGrfmt.cmake
rm -rf modules/gapi
rm -rf modules/highgui
cp -r ../highgui modules/
```

**step 3. patch opencv source for no-rtti build (optional)**
```shell
patch -p1 -i ../opencv-3.4.18-no-rtti.patch
```

**step 4. apply your opencv options to opencv3_cmake_options.txt**

**step 5. build your opencv package with cmake**
```shell
mkdir -p build
cd build
cmake -DCMAKE_INSTALL_PREFIX=install \
-DCMAKE_BUILD_TYPE=Release \
`cat ../../opencv3_cmake_options.txt` \
-DBUILD_opencv_world=OFF ..
```

**step 6. make a package**
```shell
zip -r -y opencv-mobile-3.4.18.zip install
```

# Some notes

* The minimal opencv build contains most basic opencv operators and common image processing functions, with some handy additions like keypoint feature extraction and matching, image inpainting and opticalflow estimation.

* Many computer vision algorithms that reside in dedicated modules are discarded, such as face detection etc. [You could try deep-learning based algorithms with nerual network inference library optimized for mobile.](https://github.com/Tencent/ncnn)

* Image IO functions in highgui module, like ```cv::imread``` and ```cv::imwrite```, are re-implemented using [stb](https://github.com/nothings/stb) for smaller code size. GUI functions, like ```cv::imshow```, are discarded.

* cuda and opencl are disabled because there is no cuda on mobile, no opencl on ios, and opencl on android is slow. opencv on gpu is not suitable for real productions. Write metal on ios and opengles/vulkan on android if you need good gpu acceleration.

* C++ RTTI and exceptions are disabled for minimal build on mobile platforms and webassembly build. Be careful when you write ```cv::Mat roi = image(roirect);```  :P

# opencv modules included

|module|comment|
|---|---|
|opencv_core|Mat, matrix operations, etc|
|opencv_imgproc|resize, cvtColor, warpAffine, etc|
|opencv_highgui|imread, imwrite|
|opencv_features2d|keypoint feature and matcher, etc (not included in opencv 2.x package)|
|opencv_photo|inpaint, etc|
|opencv_video|opticalflow, etc|
|opencv_calib3d|camera calibration, etc|

# opencv modules discarded

|module|comment|
|---|---|
|opencv_androidcamera|use android Camera api instead|
|opencv_contrib|experimental functions, build part of the source externally if you need|
|opencv_dnn|very slow on mobile, try ncnn for nerual network inference on mobile|
|opencv_dynamicuda|no cuda on mobile|
|opencv_flann|feature matching, rare uses on mobile, build the source externally if you need|
|opencv_gapi|graph based image processing, little gain on mobile|
|opencv_gpu|no cuda/opencl on mobile|
|opencv_imgcodecs|link with opencv_highgui instead|
|opencv_java|wrap your c++ code with jni|
|opencv_js|write native code on mobile|
|opencv_lagacy|various good-old cv routines, build part of the source externally if you need|
|opencv_ml|train your ML algorithm on powerful pc or server|
|opencv_nonfree|the SIFT and SURF, use ORB which is faster and better|
|opencv_objdetect|HOG, cascade detector, use deep learning detector which is faster and better|
|opencv_ocl|no opencl on mobile|
|opencv_python|no python on mobile|
|opencv_shape|shape matching, rare uses on mobile, build the source externally if you need|
|opencv_stitching|image stitching, rare uses on mobile, build the source externally if you need|
|opencv_superres|do video super-resolution on powerful pc or server|
|opencv_ts|test modules, useless in production anyway|
|opencv_videoio|use android MediaCodec or ios AVFoundation api instead|
|opencv_videostab|do video stablization on powerful pc or server|
|opencv_viz|vtk is not available on mobile, write your own data visualization routines|


