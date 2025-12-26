# imu_and_cali


<h2>by the way I am using zsh shell instead of bash so keep that in mind</h2>
<br>
also using ros2 humble and sbg systems Ellipse2-N-G4A3-B2 using serial usb connection
<br>
you can easily get the explanation of code using chatgpt or any ai let's start with the correct order of things and not waste our time 
<br>

you can list the usb port using ls /dev/ttyUSB* -> generally it will show /dev/ttyUSB0

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



**message type validation**
# SBG Ellipse Messages
std_msgs/Header header
	builtin_interfaces/Time stamp
		int32 sec
		uint32 nanosec
	string frame_id

# Time since sensor is powered up [us]
uint32 time_stamp

# IMU Status
SbgImuStatus imu_status
	bool imu_com
	bool imu_status
	bool imu_accel_x
	bool imu_accel_y
	bool imu_accel_z
	bool imu_gyro_x
	bool imu_gyro_y
	bool imu_gyro_z
	bool imu_accels_in_range
	bool imu_gyros_in_range
	bool imu_gyros_use_high_scale

# Filtered Accelerometer [m/s^2]
#
# NED convention:
#   x: X axis of the device frame
#   y: Y axis of the device frame
#   z: Z axis of the device frame
#
# ENU convention:
#   x: Y axis of the device frame
#   y: X axis of the device frame
#   z: -Z axis of the device frame
geometry_msgs/Vector3 accel
	float64 x
	float64 y
	float64 z

# Filtered Gyroscope [rad/s]
#
# NED convention:
#   x: X axis of the device frame
#   y: Y axis of the device frame
#   z: Z axis of the device frame
#
# ENU convention:
#   x: Y axis of the device frame
#   y: X axis of the device frame
#   z: -Z axis of the device frame
geometry_msgs/Vector3 gyro
	float64 x
	float64 y
	float64 z

# Internal Temperature [°C]
float32 temp

# Sculling output [m/s2]
#
# NED convention:
#   x: X axis of the device frame
#   y: Y axis of the device frame
#   z: Z axis of the device frame
#
# ENU convention:
#   x: Y axis of the device frame
#   y: X axis of the device frame
#   z: -Z axis of the device frame
geometry_msgs/Vector3 delta_vel
	float64 x
	float64 y
	float64 z

# Coning output [rad/s]
#
# NED convention:
#   x: X axis of the device frame
#   y: Y axis of the device frame
#   z: Z axis of the device frame
#
# ENU convention:
#   x: Y axis of the device frame
#   y: X axis of the device frame
#   z: -Z axis of the device frame
geometry_msgs/Vector3 delta_angle
	float64 x
	float64 y
	float64 z

  <h4>ros2 topics with message type</h4>
/parameter_events [rcl_interfaces/msg/ParameterEvent]
/rosout [rcl_interfaces/msg/Log]
/sbg/ekf_euler [sbg_driver/msg/SbgEkfEuler]
/sbg/ekf_nav [sbg_driver/msg/SbgEkfNav]
/sbg/ekf_quat [sbg_driver/msg/SbgEkfQuat]
/sbg/gps_hdt [sbg_driver/msg/SbgGpsHdt]
/sbg/gps_pos [sbg_driver/msg/SbgGpsPos]
/sbg/gps_raw [sbg_driver/msg/SbgGpsRaw]
/sbg/gps_vel [sbg_driver/msg/SbgGpsVel]
/sbg/imu_data [sbg_driver/msg/SbgImuData]
/sbg/imu_short [sbg_driver/msg/SbgImuShort]
/sbg/status [sbg_driver/msg/SbgStatus]
/sbg/utc_time [sbg_driver/msg/SbgUtcTime]
/tf [tf2_msgs/msg/TFMessage]
/tf_static [tf2_msgs/msg/TFMessage]

**------------------------------------------------------------------------------------**
