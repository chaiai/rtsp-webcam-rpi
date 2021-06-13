# RTSP Webcam Server on a Raspberry Pi

A simple RTSP server that uses a USB webcam and a Raspberry Pi 4B to stream via RTSP to my NVIDIA Jetson Nano running DeepStream applications.

## Requirements:
- Raspberry Pi 4B 4GB with WiFi (unsure how models older than 4 will do)
- MicroSD card (at least 16GB)
- USB-C power supply (3.5A recommended [example](https://www.amazon.com/dp/B07TYQRXTK))
- USB Webcam (mine is the [CyberTrack H4](https://www.adesso.com/product/cybertrack-h4-1080p-hd-usb-webcam-with-built-in-microphone/))
- (Optional) Ethernet cable for headless setup, GPIO fan for temperature management

## Raspberry Pi Setup
1. Flash the recommended Raspbian OS using Raspberry Pi Imager and add a blank file (no extensions) to the <code>/boot</code> folder before ejecting the MicroSD card.
> On Linux-based systems, you can open the /boot folder in your terminal and use: <code>touch ssh</code>
2. Assuming you are using headless mode (if you connect a monitor and mouse/keyboard, the UI is very easy to follow instead), plug in an ethernet cable before connecting the power, wait for the Pi to boot for a minute, and find its IP address using your router admin page or an IP scanner. Then issue the following command:

        ssh pi@[IP_ADDRESS] (default password is 'raspberry')

3. Update and upgrade packages using <code>sudo apt update --fix-missing && sudo apt upgrade -y</code> and then reboot.

Assuming you are using a Raspberry Pi with armv7 architecture like I did, you can just clone this repo and modify the config file as needed

        git clone https://github.com/chaiai/rtsp-webcam-rpi.git
        cd rtsp-webcam-rpi/rtsp
        chmod +x rtsp-simple-server
        (Optional) nano rtsp-simple-server.yml
        ./rtsp-webcam-rpi
        
 ### View the stream with VLC using: <code>vlc rtsp://[RPI_IP_ADDRESS]:8554/webcam</code>
 
 #### To see config options and how to install/setup from scratch on the Pi, see <code>INSTALL.md</code>
