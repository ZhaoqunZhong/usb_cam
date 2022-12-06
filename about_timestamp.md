1. According to line 1242 in usb_cam.cpp
```
  msg->header.stamp = ros::Time::now();
```
the image timestamps comming from this ros node are the driver's aquisition times of the images(not the start of exposure), from the same clock source, no matter the number and kind of the cameras connected.


2. According to [v4l spec](https://linuxtv.org/downloads/legacy/video4linux/API/V4L2_API/spec/ch03s05.html)

Nominally timestamps refer to the first data byte transmitted. In practice however the wide range of hardware covered by the V4L2 API limits timestamp accuracy. Often an interrupt routine will sample the system clock shortly after the field or frame was stored completely in memory. So applications must expect a constant difference up to one field or frame period plus a small (few scan lines) random error. The delay and error can be much larger due to compression or transmission over an external bus when the frames are not properly stamped by the sender. This is frequently the case with USB cameras. Here timestamps refer to the instant the field or frame was received by the driver, not the capture time. These devices identify by not enumerating any video standards, see Section 1.7, “Video Standards”.

Special rules apply to USB cameras where the notion of video standards makes little sense. More generally any capture device, output devices accordingly, which is

- incapable of capturing fields or frames at the nominal rate of the video standard, or

- where timestamps refer to the instant the field or frame was received by the driver, not the capture time, or

- where sequence numbers refer to the frames received by the driver, not the captured frames.

Here the driver shall set the std field of struct v4l2_input and struct v4l2_output to zero, the VIDIOC_G_STD, VIDIOC_S_STD, VIDIOC_QUERYSTD and VIDIOC_ENUMSTD ioctls shall return the EINVAL error code.[8]

3. I added 3 functions to check, list and set video standards of the current connected camera. 

	If the current code prints 
	```
	Current input Camera Terminal supports:
	VIDIOC_ENUMSTD: Inappropriate ioctl for device
	```
	in the terminal, then it means the current camera falls under the special rules of USB camera listed above. 

4. I checked realsense d435i and Baidu fisheye cam, they are both "USB cams" when launched using this ros node. 