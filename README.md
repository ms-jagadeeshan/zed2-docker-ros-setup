# Setup of orin with zed2 docker(ROS)

- Why use docker?   
    For using ZED2, need to have zed2 sdk and cuda, you have to setup in each machine and resolving those issues are headache, so using this zed2 docker with ROS, reduces all those headache.

Install nvidia-container-toolkit- https://www.stereolabs.com/docs/docker/install-guide-linux

## Orin Communication options for setup

### **Option 1)** Wired   
Connect usb to usb c cable to orin
```
sudo screen /dev/ttyACM0 115200
```

### **Option 2)** Wireless

`ssh -o PubkeyAuthentication=no orin@mars.local`



## Setup

Connect to orin terminal either using ssh or usb cable.
### Using docker run
```bash
docker run -it --gpus all \
    -e ROS_IP=<device-ip> \
    -e ROS_MASTER_URI=<master-uri> \
    -e NVIDIA_DRIVER_CAPABILITIES=all \
    --runtime nvidia \
    --privileged \
    --network=host \
    -v /dev:/dev \
    --name zed2-docker  \
    jagamatrix/zed2-docker:orin-minimal /ros_entrypoint.sh bash
```

### Using docker create

#### Orin
- Nvidia + Network + orin image
```bash
# xhost +si:localuser:root
docker create -it \
    --gpus all \
    -e NVIDIA_DRIVER_CAPABILITIES=all \
    --runtime nvidia \
    --privileged \
    --network=host \
    -v /dev:/dev \
    --name zed2-docker \
    jagamatrix/zed2-docker:orin-minimal
```

### Starting and running

- Start the docker and run
```bash
# Starting docker
docker start zed2-docker

docker exec -it zed2-docker /ros_entrypoint.sh roslaunch zed_wrapper zed2i.launch

# For setting up ROS_IP and ROS_MASTER_URI
docker exec -it -e ROS_IP=172.16.18.212 -e ROS_MASTER_URI=http://172.16.18.212:11311 zed2-docker /ros_entrypoint.sh roslaunch zed_wrapper zed2i.launch

```

- Host machine
```
rviz -d zed2i.rviz
```

# Orin to base station communication
- For. eg 
  In this setup ros master is running in orin, and zed2 is running in docker.

  |Device|IP|
  |-|-|
  |Orin(master)|172.16.18.212|
  |Base station|172.16.21.186|

  |Device|IP| ROS_IP | ROS_MASTER_URI|
  |-|-|-|-|
  |Orin(master)|172.16.18.212|`ROS_IP=172.16.18.212`|`ROS_MASTER_URI=http://172.16.18.212:11311`|
  |Orin(docker)|*same as orin(master)*|`ROS_IP=172.16.18.212`|`ROS_MASTER_URI=http://172.16.18.212:11311`|
  |Base Station|172.16.21.186|`ROS_IP=172.16.21.186`|`ROS_MASTER_URI=http://172.16.18.212:11311`|
  
Note:
1. Docker needs to be created with `--network=host`, to communicate from local machine and local network. 
2. To receive topics from docker container to base station, make sure `ROS_IP` and `ROS_MASTER_URI` is set on Docker, either while creating or executing or manually setting.
    `-e ROS_IP=<device-ip> -e ROS_MASTER_URI=<master-uri>`



# Others

### Using docker create
- Nvidia + network
```bash
# xhost +si:localuser:root
docker create -it \
    --gpus all \
    -e NVIDIA_DRIVER_CAPABILITIES=all \
    --runtime nvidia \
    --privileged \
    --network=host \
    -v /dev:/dev \
    --name zed2-docker-minimal  \
    jagamatrix/zed2-docker:desktop-minimal
```

- Nvidia + network + display 
```bash
xhost +si:localuser:root
docker create -it \
    --gpus all \
    -e NVIDIA_DRIVER_CAPABILITIES=all \
    --runtime nvidia \
    --privileged \
    --network=host \
    -v /dev:/dev \
    -e DISPLAY \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    --name zed2-docker \
    jagamatrix/zed2-docker:desktop
```