---
layout: post
title: Installing Open cv2 on Raspian Lite
date: 2020-8-15
image: OpenCV+Raspian_Cover.png
---

I'm going to be showing you how to install opencv2 on a raspberry pi running Raspbian Buster Lite. I know a lot of tutorials exist for this already but many are outdated and are also not for Raspbian Lite, which doesn't have a GUI. I've tested this on the 3B+ and 4 with both the normal and contrib versions of cv2.

I'm going to assume that you already have Raspbian Lite setup on your pi.

Being by installing pip3

`sudo apt-get install python-pip3`

Now, we're going to install [open cv2](https://pypi.org/project/opencv-python-headless/). We're going to install the headless version since Raspian Lite doesn't have a GUI. 

`pip3 install opencv-contrib-python-headless`

If you don't need the extra packages that come with contrib simply install `opencv-python-headless` instead. 

Note that all the opencv packages use the same namespace, so if you installed `opencv-python` or `opencv-contrib-python` previously uninstall them.

If this comand seems to hang on `Running setup.py bdist_wheel for numpy`, don't worry about it...it just takes a while.  

Now we get to have fun installing the rest of the dependencies.

	sudo apt-get install libhdf5-dev libharfbuzz-dev liblapack-dev libatlas-base-dev libwebp-dev 	libtiff-dev libjasper-dev libilmbase23 openexr 	libavcodec-dev libavformat-dev libswscale-dev 	libgtk-3-dev
	
If this fails try running `sudo apt-get update` and then trying again.	
	
This completes the installation. If you can sucessfully import opencv2, great!

However, if you get an `undefined symbol: __atomic_fetch_add_8` error, you will need to run your script using `LD_PRELOAD=/usr/lib/arm-linux-gnueabihf/libatomic.so.1 python3 scriptName.py`. This is a known [issue](https://github.com/piwheels/packages/issues/59) and there doesn't really seem to be a better solution. I'll continue to update this and this problem will hopefully be fixed in future releases.