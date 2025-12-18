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
--------------------------------------------------------------------------------------------------------------------------------
<br>
<h3>Minimal Command Cheat-Sheet for getting raw live data(Bookmark This)</h3>
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
ros2 topic echo /sbg/imu_data

<br>
--------------------------------------------------------------------------------------------------------------------------------

<br>



<h2>IMU CALIBRATION</h2>
Non-zero IMU output at rest is NORMAL

This is due to:
Sensor bias
Noise
Temperature drift

BUT BUT BUT what values i am getting are not at all normal like the value of accelaration in x i am getting is 1.03 m/s^2 other values are good enough
so have to caliberate it now will continue later now signing off ~PJ

continuing -----

The SBG Ellipse-2N internally runs an Extended Kalman Filter (EKF) that fuses inertial measurements and internal models (and GNSS, if available) to produce drift-reduced state estimates.

The IMU provides:

Orientation (attitude)

Linear velocity

Position (with GNSS)

Gravity-compensated motion information

These states are consumed by:

Vehicle controllers

Localization systems

Sensor fusion pipelines

Factory Calibration

The Ellipse-2N is factory calibrated for:
Scale factors
Axis misalignment
Temperature effects
No factory recalibration is required in normal usage.

Runtime Bias Estimation (What You Did)

Even after factory calibration:

Gyroscopes have bias

Accelerometers have offsets

Bias changes with temperature

Therefore, a static bias estimation is performed at startup.

Procedure

Place IMU on a rigid, vibration-free surface

Power on and allow 5–10 minutes warm-up

Record IMU data while stationary

Compute mean gyro and accelerometer values

<h3>why accelerometer bias appears to be large?</h3>
Accelerometer bias is dominated by gravity projection error:

A 1° tilt causes ~0.17 m/s² apparent bias
This is not sensor error
Therefore:
Accelerometer bias from static data is approximate
EKF estimates it more accurately online

Do NOT Use Raw IMU for Navigation

Raw IMU topics (/sbg/imu_data) are:

Noisy

Biased

Not gravity compensated

❌ Do not integrate raw acceleration or angular velocity
❌ Do not use raw IMU for localization

**SBG EKF Outputs (What You SHOULD Use)**

The Ellipse-2N publishes EKF-estimated state variables on /sbg/ekf_* topics.

These are the only topics recommended for vehicle operation

**----------------------------------------------------------------------------------**

##### Start IMU driver
ros2 launch sbg_driver sbg_device_launch.py

##### Orientation
ros2 topic echo /sbg/ekf_quat

##### Velocity
ros2 topic echo /sbg/ekf_vel

##### Position
ros2 topic echo /sbg/ekf_pos

**------------------------------------------------------------------------------------**


