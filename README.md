# Considering Usability in an Automatic Signal Measuring System
## 環境建置
### 遠端環境
參照文件: https://hackmd.io/@fIvDDI_pQbyc2cx_Nu-cgw/HyaeFkHL9

### ROS安裝
參照官方文件: http://wiki.ros.org/Installation/Ubuntu
```bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt install curl
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
sudo apt update
sudo apt install ros-noetic-desktop-full

echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### Source install
```bash
# create a catkin workspace
mkdir -p ~/{catkin_ws}/src
cd ~/{catkin_ws}/src/

# clone into the catkin workspace
git clone https://github.com/AILab121/OOSE.git

# use rosdep to install all dependencies (including ROS itself)
sudo apt-get update -qq
sudo apt-get install -qq -y python-rosdep
sudo rosdep init
rosdep update
rosdep install --from-paths ./ -i -y --rosdistro noetic

# build all packages in the catkin workspace
source /opt/ros/noetic/setup.bash
catkin_init_workspace
cd ~/catkin_ws
catkin_make -DCMAKE_BUILD_TYPE=RelWithDebugInfo

source ~/catkin_ws/devel/setup.bash
```

## Gazebo demo(existing map)
```bash
# gazebo:
roslaunch mir_gazebo mir_maze_world.launch
```
- click the "start" button in the Gazebo GUI
```bash
# localization:
roslaunch mir_navigation amcl.launch

# navigation:
roslaunch mir_navigation start_planner.launch \
    map_file:=$(rospack find mir_gazebo)/maps/map.yaml
rviz -d $(rospack find mir_navigation)/rviz/navigation.rviz
```

## Gazebo demo(mapping)
```bash
# gazebo:
roslaunch mir_gazebo mir_maze_world.launch
```
- click the "start" button in the Gazebo GUI
```bash
# mapping:
roslaunch mir_navigation hector_mapping.launch

# navigation:
roslaunch mir_navigation move_base.xml with_virtual_walls:=false
rviz -d $(rospack find mir_navigation)/rviz/navigation.rviz

# save map:
rosrun map_server map_saver -f /home/ailab/new_robot_ws/src/mir_robot/mir_gazebo/maps/test
```

## Adjustment parameters
- /move_base_node/DWBLocalPlanner
- **max_speed_xy** can control speed
```bash
rosrun rqt_reconfigure rqt_reconfigure
```

## Connect the joystick
```bash
roslaunch robot5g teleop_joy.launch
```

## Complete Coverage Path Planning
- You need to create the map first

```bash
roslaunch path_coverage path_coverage.launch
```
```diff
- 根據是在運行模擬或是實體改變script/path_coverage_node.py之啟用行數;67為模擬用;68為實體用
```
- Click Publish Point at the top of RViz
- Click a single corner of n corners of the region
- Repeat for n times. After that you'll see a polygon with n corners
- The position of the final point should be close to the first
- When the closing point is detected the robot starts to cover the area

## Signal heat map
- You need to pay attention to the robot_width in path_coverage.launch and the size of map2darray in heatmap.py
- Need to change the speed to 0.2
```bash
rosrun signal heatmap.py 
```

## Running the driver on the real robot
### Start up the robot
- switch on Mir
- connect to its wifi
- open mir.com

### Start the ROS driver
```bash
roslaunch mir_driver mir.launch
roslaunch mir_navigation costmap.xml
rviz -d $(rospack find mir_navigation)/rviz/navigation.rviz
```
