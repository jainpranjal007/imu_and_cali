# imu_and_cali


<h2>by the way I am using zsh shell instead of bash so keep that in mind</h2>
<br>
also using ros2 humble and sbg systems Ellipse2-N-G4A3-B2 using serial usb connection
<br>
you can easily get the explanation of code using chatgpt or any ai let's start with the correct order of things and not waste our time 
<br>
<h3>workspace setup</h3>
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
git clone https://github.com/SBG-Systems/sbg_ros2_driver.git
<br>
<h3>install dependencies and build</h3>
cd ~/ros2_ws
rosdep update
rosdep install --from-paths src --ignore-src -r -y
source /opt/ros/humble/setup.zsh
cd ~/ros2_ws
colcon build --symlink-install
<br>
<h3>Environment Sourcing (Critical Rule) really important if you don't you might die</h3>
source /opt/ros/humble/setup.zsh
source ~/ros2_ws/install/setup.zsh
<br>
<h3>Driver Verification</h3>

Confirm driver and executable exist:

ros2 pkg list | grep sbg
ros2 pkg executables sbg_driver


Expected:

sbg_driver   sbg_device


sbg_driver → ROS 2 package

sbg_device → C++ node executable

<h3>launch the node to get live data</h3>
ros2 launch sbg_driver sbg_device_launch.py
!!check if the file name is correct
<br>

<h3>node and topic verification</h3>
ros2 node list 
expected output /sbg_device
<br>
ros2 topic list | grep sbg
expected /sbg/imu_data and others like that ....
<br>

<h3>Echoing Live IMU Data</h3>
ros2 topic echo /sbg/imu_data

for single check:
ros2 topic echo /sbg/imu_data --once
<br>

<h3>message type validation</h3>
ros2 interface show sbg_driver/msg/SbgImuData
<br>
------------------------------------------------------------------------------------------------------------------------------------------------------------
<br>
**"<h3>Minimal Command Cheat-Sheet for getting raw live data(Bookmark This)</h3>
#### Source environment
source /opt/ros/humble/setup.zsh
source ~/ros2_ws/install/setup.zsh

#### Run IMU driver (LIVE DATA)
ros2 launch sbg_driver sbg_device_launch.py

#### Check node
ros2 node list

#### Check topics
ros2 topic list | grep sbg

#### Echo live IMU data
ros2 topic echo /sbg/imu_data"**

<br>



<h2>IMU CALIBRATION</h2>
Non-zero IMU output at rest is NORMAL

This is due to:
Sensor bias
Noise
Temperature drift

BUT BUT BUT what values i am getting are not at all normal like the value of accelaration in x i am getting is 1.03 m/s^2 other values are good enough
so have to caliberate it now will continue later now signing off ~PJ
