# Best Practice to Setup OpenVINO with NCS2 on Raspberry Pi
1. Download OS img of [Raspbian](https://www.raspberrypi.org/downloads/raspbian/)
   - e.g.   Choose **Raspbian Buster with desktop** if it's your first time (**Lite** = no desktop login)
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
	- then, copy-paste following text at the end of lines
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
	- auto open a terminal everytime you login (only for Raspbian with desktop)
		```
		sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
		```
		- should not be empty. If empty (for older version of Raspbian), try this
		```
		sudo nano /home/user/.config/lxsession/LXDE-pi/autostart
		```
		- then, copy-paste the following line at the end of lines
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
	
## TO BE CONTINUED	
Step5: 安裝opencv
pip3 install opencv-python (請對應好python 版本, 建議用 python3.5/pip3)
(4.1.1.26若遇到”undefined symbol: __atomic_fetch_add_8” 改成: pip install opencv-contrib-python==4.1.0.25)
sudo apt install -y libatlas-base-dev libhdf5-dev libjasper-dev libqtgui4 libqt4-test
Step6: 安裝respberry 的camera (ref)
排線插法: 一頭藍標與鏡頭同向, 另一頭藍標與網路孔同向
修改設定啟用 camera: sudo raspi-config
測試相機: raspistill -d
Step7: 安裝python API for respberry 的camera
pip3 install "picamera[array]"
如何與 opencv 串請參照 (ref)
(在Zero上可以直接cv.VideoCapture(0) 拿到影像)
在 command line cannot connect to X server:
export DISPLAY=:0
Step8. (For NCS2) 安裝OpenVino for Raspbian (link)
(官方只提供 ARMv7, ARMv6要自己build)
下載(.tgz): https://download.01.org/opencv/2020/openvinotoolkit/
sudo mkdir -p /opt/intel/openvino
sudo tar -xf l_openvino_toolkit_raspbi_p_<version>.tgz --strip 1 -C /opt/intel/openvino
sudo sed -i "s|<INSTALLDIR>|/opt/intel/openvino|" /opt/intel/openvino/bin/setupvars.sh
sh /opt/intel/openvino/install_dependencies/install_NCS_udev_rules.sh
sudo nano ~/.bashrc
在最後面貼上:
source /opt/intel/openvino/bin/setupvars.sh
存檔, 離開
以後開 terminal 都會自動設好環境變數
sudo usermod -a -G users "$(whoami)"
以後開 terminal 都會自動設好環境變數和USB設定

