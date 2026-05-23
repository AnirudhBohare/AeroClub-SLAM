# AeroClub IITD Summer Project - SLAM

Installation instructions of ROS2, Gazebo and Ardupilot in `INSTALL.md`.

## Installing MAVROS

```bash
sudo apt update
sudo apt install ros-humble-mavros ros-humble-mavros-extras
wget -O - https://raw.githubusercontent.com/mavlink/mavros/master/mavros/scripts/install_geographiclib_datasets.sh | sudo bash
```


## Running MAVROS

```bash
ros2 launch mavros apm.launch fcu_url:=udp://127.0.0.1:14550@14555
```

## Running the teleop node

```bash
# Build and source the worksapce
cd ~/AeroClub-SLAM/workspace
colcon build
source install/setup.bash
ros2 run slam drone_teleop
```
