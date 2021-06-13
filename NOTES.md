# Raspberry Pi and USB Webcam RTSP Server

Based heavily on this amazing tutorial: [Raspberry Pi Hardware Accelerated RTSP Camera](https://codecalamity.com/raspberry-pi-hardware-accelerated-h264-webcam-security-camera/#compile-ffmpeg-with-hardware-acceleration)

A simple server to run for my DeepStream SDK application running on my NVIDIA Jetson Nano. I am using tutorials found [here](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/docs/Module%203%20-%20Develop%20and%20deploy%20Custom%20Object%20Detection%20Models%20with%20IoT%20Edge%20DeepStream%20SDK%20Module.md) to create a security system powered by AI.

## Hardware

1. Raspberry Pi 4B 4GB with WiFi
2. MicroSD card (at least 16 GB)
3. USB-C power supply (3.5A recommended [example](https://www.amazon.com/dp/B07TYQRXTK))
4. USB Webcam (mine is the [CyberTrack H4](https://www.adesso.com/product/cybertrack-h4-1080p-hd-usb-webcam-with-built-in-microphone/))
5. (Optional) Ethernet cable for headless setup, GPIO fan for temperature management

## Raspberry Pi Setup

1. Flash the recommended Raspbian OS using Raspberry Pi Imager and add a blank file (no extensions) to the <code>/boot</code> folder before ejecting the MicroSD card.
> On Linux-based systems, you can open the /boot folder in your terminal and use: <code>touch ssh</code>
2. Assuming you are using headless mode (if you connect a monitor and mouse/keyboard, the UI is very easy to follow instead), plug in an ethernet cable before connecting the power, wait for the Pi to boot for a minute, and find its IP address using your router admin page or an IP scanner. Then issue the following command:

<code>ssh pi@[IP_ADDRESS]</code> (default password is 'raspberry')

3. Update and upgrade packages using <code>sudo apt update --fix-missing && sudo apt upgrade -y</code> and then reboot.
4. Double check the Linux architecture with <code>uname -m</code>, then download the appropriate .tar from the rtsp-simple-server [releases](https://github.com/aler9/rtsp-simple-server/releases) page. This simply contains an executable and config file, and requires no dependencies.
5. Plug in the USB camera and make sure it is recognized with: <code>v4l2-ctl --list-devices</code>, and then list all available formats with: <code>v4l2 -d /dev/video0 --list-formats-ext</code>
> If this is the only USB device and camera plugged into the Pi, it should be video0, but double-check with --list-devices.

My camera has these possible outputs:

<code>pi@raspberrypi:~ $ v4l2-ctl -d /dev/video0 --list-formats-ext
ioctl: VIDIOC_ENUM_FMT
	Type: Video Capture

	[0]: 'MJPG' (Motion-JPEG, compressed)
		Size: Discrete 1920x1080
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.067s (15.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1280x960
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.067s (15.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1280x720
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.067s (15.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1024x768
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.067s (15.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 800x600
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.067s (15.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 640x480
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.040s (25.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.067s (15.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 640x360
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.067s (15.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 320x240
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.067s (15.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 320x180
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.067s (15.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
	[1]: 'YUYV' (YUYV 4:2:2)
		Size: Discrete 640x480
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.067s (15.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1920x1080
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1280x720
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)</code>

After testing a few different video formats and resolutions, I settled on MJPEG 1280x720 (720p) as the input encoded using h264_omx

6. Edit rtsp-simple-server.yml via <code>nano rtsp-simple-server.yml</code> and go to the **Path options** section. Replace the entire <code>paths</code> section with:

<code>paths:
  webcam:
    runOnInit: ffmpeg -input_format mjpeg -f video4linux2 -s 1280x720 -r 30 -i /dev/video0 -c:v h264_omx -r 30 -b:v 8M -an -f rtsp rtsp://localhost:$RTSP_PORT/$RTSP_PATH
    runOnInitRestart: yes</code>

Params to change:
- **-input_format** if you want to use different native encodings from the camera
- **-s, -r** are the size/resolution and framerate to use
- **-c:v, -b:v** are the video codec and video bitrate to use by ffmpeg (the output encoding that is streamed)
- **-an** removes audio (I didn't test with audio, so unsure if necessary)

7. Run the server! Save the config file and run: <code>./rtsp-simple-server</code>
> View the stream with VLC using: <code>vlc rtsp://[RPI_IP_ADDRESS]:8554/webcam</code>

