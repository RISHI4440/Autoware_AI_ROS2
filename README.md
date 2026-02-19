
CARLA server = 0.9.15 (external)
Python API = 0.9.15 (cp310)
Bridge patched to 0.9.15
ROS 2 Humble using Python 3.10


# the autoware ai ros2 docker image is build to be paired with carla docker image found at the following address 

https://hub.docker.com/layers/lehong/carla/0.9.15/images/sha256-1af47c314443f4a2d0869ac399c5b5eabde138a5bc39dd31dcbb1540e22f5588

Both docker containers should be hosted on the same machine and share the same network 



**From Autoware container, test CARLA by container name, not localhost 

nc -zv <carla_container_name> 2000

eg.
nc -zv carla_build_rushi 2000
It should say succeeded.


**from autoware container launch the bridge 
ros2 launch carla_ros_bridge carla_ros_bridge.launch.py host:=<carla_container_name>  port:=2000

eg
ros2 launch carla_ros_bridge carla_ros_bridge.launch.py host:=determined_heyrovsky  port:=2000



# OPTION B :-> connect via CARLA container IP

On the host, get CARLA container IP:
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <containers_name>

eg.
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' carla_build_rushi
172.17.0.5

inside Autoware container run:

ros2 launch carla_ros_bridge carla_ros_bridge.launch.py host:=<carla_ip> port:=2000 timeout:=10000

eg.
ros2 launch carla_ros_bridge carla_ros_bridge.launch.py host:=172.17.0.5 port:=2000 timeout:=10000


# ***** Note when running docker image for the first time if you ran into an issue ROS2 not found please follow these steps ***
# ROS 2 NOT FOUND

Whats happing -> when we starte fresh shell inside the running containter it does NOT automatically source ROS unless irs in ".bashrc" or we manually course it. to get around this

1) Inside autoware_sim run:
     echo 'source /opt/ros/humble/setup.bash' >> ~/.bashrc
      echo 'source ~/autoware/install/setup.bash' >> ~/.bashrc

3) Get into the continer
   docker exec -it autoware_sim bash
# ROS will already be sourced.

5) quick CARLA connectivity test + run bridge (inside autoware_sim)

  timeout 2 bash -c "</dev/tcp/localhost/2000" && echo "CARLA port OK" || echo "CARLA port FAIL"

If it says OK, launch

ros2 launch carla_ros_bridge carla_ros_bridge.launch.py host:=localhost port:=2000 timeout:=10000






# Issures Resolved 

1) CARLA Python version mismatch

fixed that by:

Installing carla==0.9.15 for Python 3.10

Patching CARLA_VERSION in the bridge to 0.9.15

Rebuilding carla_ros_bridge

2) Missing carla_msgs

The bridge depends on ROS message package: "carla_msgs"
You fixed it by:
git clone --recurse-submodules https://github.com/carla-simulator/ros-bridge.git

3) Python interpreter conflict (3.7 vs 3.10)
   
Earlier we tried forcing Python 3.7 because of the .egg file. That broke ROS 2 Humble because:

ROS 2 Humble = Python 3.10

rclpy C extensions are compiled for Python 3.10


# Version mismatch between CARLA server, CARLA Python API, and ROS bridge. (resolved)


