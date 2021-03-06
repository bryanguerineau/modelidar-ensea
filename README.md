# :pickup_truck: ENSEA Modelidar Robot

![Licence](https://img.shields.io/github/license/bryanguerineau/modelidar-ensea?style=plastic)

Modelidar is a student project at ENSEA (Ecole Nationale Supérieure de l'Electronique et de ses Applications). The objective of the project is the creation of a radio-controlled mobile platform, carrying a LIDAR of the Velodyme company allowing to **model in 3D the interior of the buildings**.

To realize this project, we were a team of ten students supervised by the professor C.Barès. In order to carry out this project, different tasks had to be realized, and teams worked in a polyvalent way.

![presentation](https://user-images.githubusercontent.com/96022647/173254152-1d3c968e-84c0-44fe-88cd-a94229d2f3c5.png)

The tasks were as follows: 
1. Electromechanics part
   - Power electronics
   - Mechanical realization
2. Computer Science part
   - Installation of the ROS Environment
   - Using the Microsoft Sidewinder Joystick with ROS
   - Using the Raspberry Joystick with ROS
   - Control of the motors
3. Control / Servoing

## :desktop_computer: Computer Science Part

With my team-mate, we worked on the computer part. I had to master and install the ROS environment on Raspberry Pi 3/4, create ROS nodes and perform the STM32->ROS translation of the frames coming from the Joystick and ROS->STM32 for the motor control. The language used was **Python 3.8.10**.

### :question: What is ROS?

The Robot Operating System (ROS) is a set of software libraries and tools that help you build robot applications. From drivers to state-of-the-art algorithms, and with powerful developer tools, ROS has what you need for your next robotics project. And it's all open source.

### :hammer_and_wrench: ROS installation for Raspberry Pi3 or Pi4

The first step is to install Pi OS on the Raspberry you are going to use.

#### Installation of Pi OS on the Raspberry
1. Download [Raspberry Pi OS](https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit) 64bit Desktop version
2. Flash the USB stick with [BalenaEtcher](https://www.balena.io/etcher/)
3. Setup on Raspberry
   - Remount the SD card boot partition
   - Activate SSH server: create an empty ssh file in the root of the partition \boot\
   - Wifi connection (optional): Create a wpa_supplicant.conf file at the root of the partition \boot\

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=FR

network={
  ssid="<Name of your wireless LAN>"
  psk="<Password for your wireless LAN>"
}
```
5. Remove the sd card, install it in the raspberry and start it up.
6. Access raspberry with ssh:
```
ssh pi@<name_or_raspberry_ip>
```
***Default password is raspberry***

7. Raspberry Update
```
sudo apt update && sudo apt upgrade && apt clean
```
8. Utilities installation:
```
sudo apt install vim curl
```

#### ROS/noetic installation
The second step is to install ROS noetic on Pi OS. The details of this installation can be found [here](http://wiki.ros.org/noetic/Installation/Ubuntu), moreover a quick guide is presented below: 
1. Adding debian repositories (buster):
```
sudo sh -c 'echo "deb http://deb.debian.org/debian buster main" > /etc/apt/sources.list.d/buster.list'
```
2. Setup of the ROS repository:
```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu buster main" > /etc/apt/sources.list.d/ros-latest.list'
```
3. Install ROS mainteners key:
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
```
4. Install of ROS/noetic:
```
sudo apt update && sudo apt install ros-noetic-desktop && sudo apt install ros-desktop-full
```
5. Update .bashrc:
```
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
```
6. Verification of correct installation:
```
rospack find roscpp
```
If this command returns this, the installation is successful :white_check_mark: :
```
/opt/ros/noetic/share/roscpp
```

#### ROS use
1. Creation of a ROS space on Raspberry Pi
```
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
catkin_init_workspace
cd ~/catkin_ws
catkin_make
source devel/setup.bash
```

It is necessary to have two Raspberry Pi for this project. The first one which is used to link with the Joystick will be used as a sending node. The second Raspberry Pi will be connected to the STM32 controlling the motors.

2. Creation of the Talker package (Raspberry Joystick):
```
cd src/ && catkin_create_pkg modelidar-joystick-talker std_msgs sensor_msgs  rospy roscpp
```
We create here the ROS package composed of the libraries *std_msgs* (send and receive messages) and *sensor_msgs* (contains the Joy module for the joysticks).

3. Creation of the python files listener.py and talker.py:
In the folder ~/catkin_ws/src/modelidar-joystick-talker/scripts/, you can create two files: 
   - talker.py
   - listener.py
```
cd ~/catkin_ws/src/modelidar-joystick-talker/
mkdir scirpts
cd scripts
>talker.py && >listener.py
```

4. Editing *package.xml* and *CMakeLists.txt* files
It is useful to fill in the comments in the *package.xml* file, so that other developers can contact you in case of problems. 

Concerning the *CMakeLists.txt* file, we have to add our python files in the Install part: 
```
catkin_install_python(PROGRAMS scripts/talker.py scripts/listener.py
	DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
```
5. Editing *listener.py* and *talker.py* files
These two files are completed and detailed above. You can download and copy them in your /scripts folder.

Then you have to compile the package:
```
cd ~/catkin_ws/src
catkin_make
```

### :green_circle: ROS execution
Finally you must run in three different consoles: 
**Console 1 : *roscore* ** : 
```
cd ~/catkin_ws/src
roscore
```
**Console 2 : *talker.py* ** : 
```
cd ~/catkin_ws/src/modelidar-joystick-talker/scripts
python3 talker.py
```
**Console 3 *listener.py* ** : 
```
cd ~/catkin_ws/src/modelidar-joystick-talker/scripts
python3 talker.py
```
Now, if the joystick is correctly connected, and you have specified the correct serial_port in talker.py, you should see the joystick frames between -100 and 100 being sent and displayed on the listener console.


