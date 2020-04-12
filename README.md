# Best Practice: Python + OpenVINO + NCS2 on Raspberry Pi Zero
1. Download OS img of [Raspbian](https://www.raspberrypi.org/downloads/raspbian/)
   - e.g.   Choose **Raspbian Buster with desktop** if you're a newbie (**Lite** = no desktop login)
2. Use [Win32DiskImager](https://sourceforge.net/projects/win32diskimager/) to write OS img into your SD card
3. Insert SD card into Raspberry Pi
4. Plug in mouse, keyboard, ethernet
5. Plug in power supply (power-on=boot, power-off=shutdown)
6. After login to desktop, setup your password, locale and language
7. Open terminal
8. Update Raspbian then upgrade
	```
	sudo apt update
	sudo apt upgrade
	```
9. Configure Raspbian:
	```
	sudo raspi-config
	```
   Goto Interface => enable SSH, enable camera
10. Install remote desktop (only for Raspbian with desktop)
	```
	sudo apt install xrdp
	``` 
	- then, check your IP address
		```
		ifconfig
		```
	- now, un-plug your mouse and keyboard. You can remote login Raspberry Pi from your PC by [putty](https://www.putty.org/) or Windows Remote Desktop.
	
13. Install samba
	```
	sudo apt install samba
	sudo nano /etc/samba/smb.conf
	```
	- then, copy-paste following text to the end of lines
		```
		[yourfoldername] # e.g. [share]
		path = yourfolderpath # e.g. /home/pi/share
		writable = yes
		guest account = root
		force user = root
		public = yes
		force group = root
		```
	- then, ```CTRL+O``` save, ```CTRL+X``` exit
14. Make Raspberry Pi report IP address after reboot
	- write a python script to polling network connection status and report IP address once on-line
	- auto execute this script everytime you open a terminal
		```
		sudo nano ~/.bashrc
		```
	- add following command at the end of lines
		```
		python /home/pi/share/whereami.py
		```
	- automatically open a terminal everytime you login (only for Raspbian with desktop)
		```
		sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
		```
		
		- should not be empty. If empty (for older version of Raspbian), try another path like this
			```
			sudo nano /home/user/.config/lxsession/LXDE-pi/autostart
			```
		
	- then, copy-paste the following line to the end of lines
		``` 
		@lxterminal
		```
15. Disable screensaver (only for Raspbian with desktop)
	- install xscreensaver
		```
		sudo apt install xscreensaver
		```
	- then, goto desktop **Menu** (left top corner) => **preference** => **screensaver**
	- goto the **mode** drop-down menu, select **disable screensaver** then close
	- reboot your Raspberry Pi.
16. Install OpenCV
	- install dependencies
		```
		sudo apt install -y libatlas-base-dev libhdf5-dev libjasper-dev libqtgui4 libqt4-test
		```
	- install opencv
		```
		python3 -m pip install opencv-python
		```
		- if you encounter error: ```undefined symbol: __atomic_fetch_add_8``` try this
			``` 
			pip install opencv-contrib-python==4.1.0.25
			```
17. Install picamera
	- test your picamera (if you have one and have it plugged into)
		```
		raspistill -d
		```
		- if it's setup correctly, you will see a captured video for seconds
	- now, you can get video frames from opencv 
		```
		capture = cv.VideoCapture(0)
		ret, frame = capture.read()
		```
		- if not, try [PiCamera module](https://stackoverflow.com/questions/34026097/using-a-pi-camera-module-with-opencv-python)
			```
			python3 -m pip install "picamera[array]"
			```
18. Download and install OpenVINO
	- only [ARMv7 package](https://download.01.org/opencv/2020/openvinotoolkit) is officially available now (for RPi3, RPi4)
  - for ARMv7 RPi, just follow [official guide] to install
	- for ARMv6 RPi, we will build OpenVINO by ourself in the next step
	
		- but you still need the script in this package to add USB rules for NCS2
			```
			sudo usermod -a -G users "$(whoami)"
			bash /opt/intel/openvino/install_dependencies/install_NCS_udev_rules.sh
			```
19. Build OpenVINO natively on **ARMv6** RPi (e.g. Zero, Zero W and One)
	- install build tool and dependencies:
		```
		sudo apt update && apt upgrade -y
		sudo apt install build-essential
		sudo apt install -y git cmake libusb-1.0-0-dev
		sudo apt install cython
		```
	- close the terminal then open a new one
	- increase swpap space temporailty
		```
		free -h
		sudo fallocate -l 1G /swapfile
		sudo chmod 600 /swapfile
		ls -lh /swapfile
		sudo mkswap /swapfile
		sudo swapon /swapfile
		sudo swapon -show
		free -h
		```
	- git clone **dldt**(Deep Learning Deployment Toolkit=OpenVINO) from source
		```
		mkdir dldt
		git clone https://github.com/opencv/dldt.git
		```
	- build dldt
		```
		cd ~/dldt
		git submodule init
		git submodule update -recursive
		mkdir build && cd build
		sudo cmake -DCMAKE_BUILD_TYPE=Release -DENABLE_MKL_DNN=OFF -DENABLE_CLDNN=OFF -DENABLE_SSE42=OFF -DTHREADING=SEQ -DENABLE_GNA=OFF  -DENABLE_OPENCV=OFF -DENABLE_PYTHON=ON -DPYTHON_EXECUTABLE=/usr/bin/python3.7 -DPYTHON_LIBRARY=/usr/lib/arm-linux-gnueabihf/libpython3.7m.so -DPYTHON_INCLUDE_DIR=/usr/include/python3.7 ..
		sudo make
		```
		- build should be successful, if you encounter error: ```symbol '_ZN2cv6String10deallocateEv'```, it's a version confliction caused by by opencv, try this
			```
			sudo apt-get autoremove libopencv-dev
			```
- next, automatically setup environment once terminal open
	```
	sudo nano ~/.bashrc
	```
	- add the following to the end of lines (this is to add the path of openvino to your environment variables)
	  - change ```<path to where you build openvino>``` to fit your case
	  - for my example: ```~/share/dldt/bin/armv6l/Release/lib/python_api/python3.7```
		```
		export PYTHONPATH=$PYTHONPATH:<path to where you build openvino>
		export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:<path to where you build openvino>
		```


		




