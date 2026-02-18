the autoware ai ros2 docker image is build to be paired with carla docker image found at the following address 

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



OPTION B :-> connect via CARLA container IP

On the host, get CARLA container IP:
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <containers_name>

eg.
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' carla_build_rushi
172.17.0.5

inside Autoware container run:

ros2 launch carla_ros_bridge carla_ros_bridge.launch.py host:=<carla_ip> port:=2000 timeout:=10000

eg.
ros2 launch carla_ros_bridge carla_ros_bridge.launch.py host:=172.17.0.5 port:=2000 timeout:=10000
