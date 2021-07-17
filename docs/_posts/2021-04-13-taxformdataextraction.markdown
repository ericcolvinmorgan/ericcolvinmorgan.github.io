---
layout: post
title:  "Tax Form Data Extraction - Environment Setup"
date:   2021-04-13 19:55:33 -0700
categories: projects
tags: python tesseract opencv poppler ocr
---
This post details steps in setting up an environment used for extracting text from tax forms.  Today I'll be walking through the steps to install the necessary tools needed going forward.  The environment in which I will be working will be Windows 10 with WSL2 configured and running Ubuntu 20.04 (WSL installation instructions [here](https://docs.microsoft.com/en-us/windows/wsl/install-win10)).  It is possible to install all of the dependencies mentioned today directly in a Windows environment, but it will require a lot more setup; I strongly suggest embracing WSL.  It shouldn't take more than a couple hours to get everything up and running.

We start by updating our package manager:
```
sudo apt-get update # Update package source list
sudo apt-get upgrade # Update installed packages
```

Next, install Python3.  The following command will show you if Python is installed.
```
python3 --version
```

If the command was not installed then install Python3 with the following: 
```
sudo apt-get install python3
```

You should also see `/usr/bin/python3` as the output if you type `which python3`.  If you see something different there were other versions installed on your machine at another time, so these instructions may change. 

Next, we'll install development headers
```
sudo apt-get install python3-dev
```

I like to make use of virutal environments to better manage packages while developing, so we will need to setup our dependencies for these:
```
sudo apt-get install python3-pip # Install pip3
sudo pip3 install virtualenv # Install virtualenv
```

From here, you can create a virtual environment for your project as follows:
```
cd $PROJECT_IRECTORY_HERE # Navigate to project directory
virtualenv .venv # Create virtual environment
source .venv/bin/activate # Activate environment
```

You can deactivate your environment by simply running `deactivate`.  We won't start our environment now, but we will come back to this.

Open CV
-------
Preinstall Note - There may be packages available via pip that streamline a lot of this installation.  Most likely target older versions of Open CV and/or not have all features included.

[Open Source Computer Vision (Open CV)](https://www.opencv.org/) is a library that contains a large number of computer vision algorithms.  Why do we need this to extract detail from a tax form?  At the most basic level we'll likely need to clean and manipulate a PDF document to get it into a state that it is ready to run OCR across it.  OCR works best on documents that are clean, and OpenCV contains a number of tools that will help us with pre-processing.  We can also use OpenCV to help us define structure in a document.

We start by updating our package manager:
```
sudo apt-get update # Update package source list
sudo apt-get upgrade # Update installed packages
```

Next, we start our installation of OpenCV.  The instructions below are adapted from the instructions [here](https://docs.opencv.org/4.4.0/d7/d9f/tutorial_linux_install.html).
```
sudo apt-get install build-essential # Required compiler dependencies
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev # Required packages
```

We can install additional optional dependencies next.  These are optional for OpenCV, but we'll be using them for our projects so go ahead and install:
```
sudo apt-get install python3-dev python3-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev # Optional packages
```

These are as follows:

    - python3-dev # Python development headers
    - python3-numpy # Numpy for Python3
    - libtbb2 libtbb-dev # Parallelism libraries for C++
    - libjpeg-dev libpng-dev libtiff-dev # Libraries for various media

We will need to build OpenCV, so we'll be downloading the latest version of OpenCV and OpenCV's extra modules next:
```
cd ~
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.5.2.zip
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.5.2.zip
unzip opencv.zip
unzip opencv_contrib.zip
```

After everything has been unzipped, we can build OpenCV.  This process may take some time.  OpenCV is an open source library, and utilizes a number of algorithms that are free for personal or academic purposes, but are not for commercial purposes.  Depending on your use case you can turn these patented algorithms off using the `OPENCV_ENABLE_NONFREE` flag below.
```
cd opencv-4.5.2 # Open unzipped directory
mkdir build # Make build directory
cd build # Open build directory
cmake -D CMAKE_BUILD_TYPE=RELEASE \
	-D CMAKE_INSTALL_PREFIX=/usr/local \
	-D INSTALL_C_EXAMPLES=ON \
	-D INSTALL_PYTHON_EXAMPLES=ON \
	-D OPENCV_GENERATE_PKGCONFIG=ON \
	-D OPENCV_ENABLE_NONFREE=OFF \
	-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-4.5.2/modules \
	-D BUILD_EXAMPLES=ON ..
```

You should see output similar to the below when successful:
```
-- General configuration for OpenCV 4.5.2 =====================================
--   Version control:               unknown
-- 
--   Extra modules:
--     Location (extra):            /home/eric/opencv_contrib-4.5.2/modules
--     Version control (extra):     unknown
-- 
--   Platform:
--     Timestamp:                   2021-04-13T21:53:55Z
--     Host:                        Linux 4.19.104-microsoft-standard x86_64
--     CMake:                       3.16.3
--     CMake generator:             Unix Makefiles
--     CMake build tool:            /usr/bin/make
--     Configuration:               RELEASE
-- 
--   CPU/HW features:
--     Baseline:                    SSE SSE2 SSE3
--       requested:                 SSE3
--     Dispatched code generation:  SSE4_1 SSE4_2 FP16 AVX AVX2 AVX512_SKX
--       requested:                 SSE4_1 SSE4_2 AVX FP16 AVX2 AVX512_SKX
--       SSE4_1 (17 files):         + SSSE3 SSE4_1
--       SSE4_2 (2 files):          + SSSE3 SSE4_1 POPCNT SSE4_2
--       FP16 (1 files):            + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 AVX
--       AVX (5 files):             + SSSE3 SSE4_1 POPCNT SSE4_2 AVX
--       AVX2 (31 files):           + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2
--       AVX512_SKX (7 files):      + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2 AVX_512F AVX512_COMMON AVX512_SKX
-- 
--   C/C++:
--     Built as dynamic libs?:      YES
--     C++ standard:                11
--     C++ Compiler:                /usr/bin/c++  (ver 9.3.0)
--     C++ flags (Release):         -fsigned-char -W -Wall -Werror=return-type -Werror=non-virtual-dtor -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Wsuggest-override -Wno-delete-non-virtual-dtor -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -fvisibility-inlines-hidden -O3 -DNDEBUG  -DNDEBUG
--     C++ flags (Debug):           -fsigned-char -W -Wall -Werror=return-type -Werror=non-virtual-dtor -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Wsuggest-override -Wno-delete-non-virtual-dtor -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -fvisibility-inlines-hidden -g  -O0 -DDEBUG -D_DEBUG
--     C Compiler:                  /usr/bin/cc
--     C flags (Release):           -fsigned-char -W -Wall -Werror=return-type -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -O3 -DNDEBUG  -DNDEBUG
--     C flags (Debug):             -fsigned-char -W -Wall -Werror=return-type -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -g  -O0 -DDEBUG -D_DEBUG
--     Linker flags (Release):      -Wl,--exclude-libs,libippicv.a -Wl,--exclude-libs,libippiw.a   -Wl,--gc-sections -Wl,--as-needed  
--     Linker flags (Debug):        -Wl,--exclude-libs,libippicv.a -Wl,--exclude-libs,libippiw.a   -Wl,--gc-sections -Wl,--as-needed  
--     ccache:                      NO
--     Precompiled headers:         NO
--     Extra dependencies:          dl m pthread rt
--     3rdparty dependencies:
-- 
--   OpenCV modules:
--     To be built:                 aruco bgsegm bioinspired calib3d ccalib core datasets dnn dnn_objdetect dnn_superres dpm face features2d flann freetype fuzzy gapi hfs highgui img_hash imgcodecs imgproc intensity_transform line_descriptor mcc ml objdetect optflow phase_unwrapping photo plot python3 quality rapid reg rgbd saliency shape stereo stitching structured_light superres surface_matching text tracking ts video videoio videostab wechat_qrcode xfeatures2d ximgproc xobjdetect xphoto
--     Disabled:                    world
--     Disabled by dependency:      -
--     Unavailable:                 alphamat cnn_3dobj cudaarithm cudabgsegm cudacodec cudafeatures2d cudafilters cudaimgproc cudalegacy cudaobjdetect cudaoptflow cudastereo cudawarping cudev cvv hdf java julia matlab ovis python2 sfm viz
--     Applications:                tests perf_tests examples apps
--     Documentation:               NO
--     Non-free algorithms:         NO
-- 
--   GUI: 
--     GTK+:                        YES (ver 2.24.32)
--       GThread :                  YES (ver 2.64.6)
--       GtkGlExt:                  NO
--     VTK support:                 NO
-- 
--   Media I/O: 
--     ZLib:                        /usr/lib/x86_64-linux-gnu/libz.so (ver 1.2.11)
--     JPEG:                        /usr/lib/x86_64-linux-gnu/libjpeg.so (ver 80)
--     WEBP:                        build (ver encoder: 0x020f)
--     PNG:                         /usr/lib/x86_64-linux-gnu/libpng.so (ver 1.6.37)
--     TIFF:                        /usr/lib/x86_64-linux-gnu/libtiff.so (ver 42 / 4.1.0)
--     JPEG 2000:                   build (ver 2.4.0)
--     OpenEXR:                     build (ver 2.3.0)
--     HDR:                         YES
--     SUNRASTER:                   YES
--     PXM:                         YES
--     PFM:                         YES
-- 
--   Video I/O:
--     DC1394:                      NO
--     FFMPEG:                      YES
--       avcodec:                   YES (58.54.100)
--       avformat:                  YES (58.29.100)
--       avutil:                    YES (56.31.100)
--       swscale:                   YES (5.5.100)
--       avresample:                NO
--     GStreamer:                   NO
--     v4l/v4l2:                    YES (linux/videodev2.h)
-- 
--   Parallel framework:            pthreads
-- 
--   Trace:                         YES (with Intel ITT)
-- 
--   Other third-party libraries:
--     Intel IPP:                   2020.0.0 Gold [2020.0.0]
--            at:                   /home/eric/opencv-4.5.2/build/3rdparty/ippicv/ippicv_lnx/icv
--     Intel IPP IW:                sources (2020.0.0)
--               at:                /home/eric/opencv-4.5.2/build/3rdparty/ippicv/ippicv_lnx/iw
--     VA:                          NO
--     Lapack:                      NO
--     Eigen:                       NO
--     Custom HAL:                  NO
--     Protobuf:                    build (3.5.1)
-- 
--   OpenCL:                        YES (no extra features)
--     Include path:                /home/eric/opencv-4.5.2/3rdparty/include/opencl/1.2
--     Link libraries:              Dynamic load
-- 
--   Python 3:
--     Interpreter:                 /usr/bin/python3 (ver 3.8.5)
--     Libraries:                   /usr/lib/x86_64-linux-gnu/libpython3.8.so (ver 3.8.5)
--     numpy:                       /usr/lib/python3/dist-packages/numpy/core/include (ver 1.17.4)
--     install path:                lib/python3.8/dist-packages/cv2/python-3.8
-- 
--   Python (for build):            /usr/bin/python2.7
-- 
--   Java:                          
--     ant:                         NO
--     JNI:                         NO
--     Java wrappers:               NO
--     Java tests:                  NO
-- 
--   Install to:                    /usr/local
-- -----------------------------------------------------------------
-- 
-- Configuring done
-- Generating done
-- Build files have been written to: /home/eric/opencv-4.5.2/build
```

After you have produced the make files, it's time to build and install.  Type `make -j5` (or just `make`) into the command line and step away for awhile.  This process will take 30 - 1 hour.

Finally, run `sudo make install` to complete the installation.

Installation can be verified using the following.  If succesful you will see the applicable version number for both of the commands below:
```
python3 -c "import cv2; print(cv2.__version__)" # Verify Python success
pkg-config --modversion opencv4 # Verify Open CV is available to pkg-config (not applicable for just Python users)
```

To enable cv2 in a Python environment run the following command to copy the necessary package files.
```
cp /usr/local/lib/python3.8/dist-packages/cv2 $ENVIRONMENT_PATH/.venv/lib/python3.8/site-packages
```

Tesseract
---------
[Tesseract](https://tesseract-ocr.github.io/tessdoc/Home.html) is an open source Optical Character Recognition (OCR) solution.  This library will be used to extract text from image files.  Installation here will be significantly easier than the previous OpenCV installation.

```
sudo apt-get install tesseract-ocr # Installs Tesseract
tesseract -v # Verify Tesseract has been installed
```

If successful you should see something similar to the following.  This lists the tesseract version and supported image libraries.
```
tesseract 4.1.1
 leptonica-1.79.0
  libgif 5.1.4 : libjpeg 8d (libjpeg-turbo 2.0.3) : libpng 1.6.37 : libtiff 4.1.0 : zlib 1.2.11 : libwebp 0.6.1 : libopenjp2 2.3.1
```

At this point, typing `tesseract $IMAGE_PATH stdout` should display output text from an image directly to the command line.  You should see the text found in the image.


You can install a tesseract wrapper for Python by activating your environment and installing pytesseract.  You can verify it works by running the below command.  You should see the same output as the `tesseract $IMAGE_PATH stdout` command run above.
```
pip3 install pytesseract
python3 -c "import pytesseract; from PIL import Image; print(pytesseract.image_to_string(Image.open('$IMAGE_PATH')))" # Verify Python succes
```

Poppler
-------
[Poppler](https://poppler.freedesktop.org/) is a utility that can convert PDFs to image files, and can also extract text embedded in PDFs directly.  As embedded text is definitely preferred over having to OCR a document to extract text, we'll use this as our first pass on a tax form.  Install the tools needed using the following:

```
sudo apt-get install poppler-utils libpoppler-cpp-dev
```

You can confirm installation by typing `pdftotext -v` to view version details.

After Poppler has been successfully installed, you can activate your Python environment and install the Python wrapper:
```
pip install python-poppler
```

That's it!  You should now be good to go for each of the above libraries.