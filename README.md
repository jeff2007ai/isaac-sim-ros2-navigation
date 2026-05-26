# 🤖 Isaac Sim + ROS 2 Navigation — Complete Tutorial

> Run NVIDIA Isaac Sim 5.1 with ROS 2 Jazzy Navigation Stack on Ubuntu 24.04

![Isaac Sim](https://img.shields.io/badge/Isaac%20Sim-5.1.0-76b900?style=flat-square&logo=nvidia)
![ROS 2](https://img.shields.io/badge/ROS%202-Jazzy-22314E?style=flat-square&logo=ros)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04-E95420?style=flat-square&logo=ubuntu)
![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python)

---

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [System Requirements](#system-requirements)
- [Installation](#installation)
  - [1. NVIDIA Driver Setup](#1-nvidia-driver-setup)
  - [2. Isaac Sim via Isaac Lab](#2-isaac-sim-via-isaac-lab)
  - [3. ROS 2 Jazzy Setup](#3-ros-2-jazzy-setup)
  - [4. ROS 2 Isaac Sim Bridge](#4-ros-2-isaac-sim-bridge)
- [Running Isaac Sim with ROS 2](#running-isaac-sim-with-ros-2)
  - [Environment Variables](#environment-variables)
  - [Launch Isaac Sim](#launch-isaac-sim)
  - [Launch Navigation Stack](#launch-navigation-stack)
- [Navigation Demo](#navigation-demo)
- [Troubleshooting](#troubleshooting)
- [References](#references)

---

## Prerequisites

Before starting, ensure you have:

- Ubuntu 24.04 LTS (fresh install recommended)
- NVIDIA GPU with CUDA support (RTX series recommended)
- At least 16GB RAM (32GB recommended)
- 50GB free disk space
- Active internet connection

---

## System Requirements

| Component | Minimum      | Recommended        |
|-----------|--------------|--------------------|
| GPU       | RTX 3060     | RTX 4060 or higher |
| VRAM      | 6GB          | 8GB+               |
| RAM       | 16GB         | 32GB               |
| Storage   | 50GB SSD     | 100GB NVMe SSD     |
| OS        | Ubuntu 22.04 | Ubuntu 24.04       |
| CUDA      | 11.8         | 12.0+              |

---

## Installation

### 1. NVIDIA Driver Setup

Install DKMS and kernel headers first — **this is the most commonly missed step:**

```bash
# Install DKMS (required for kernel module compilation)
sudo apt install dkms linux-headers-$(uname -r)

# Install NVIDIA driver
sudo apt install nvidia-driver-580

# Rebuild kernel module
sudo apt install --reinstall nvidia-driver-580

# Reboot
sudo reboot
```

Verify the driver is working:

```bash
nvidia-smi
```

You should see your GPU listed with driver version and CUDA version. If not, see [Troubleshooting](#troubleshooting).

---

### 2. Isaac Sim via Isaac Lab

**Install Miniconda:**

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
source ~/.bashrc
```

**Create the Isaac Lab environment:**

```bash
git clone https://github.com/isaac-sim/IsaacLab.git ~/IsaacLab
cd ~/IsaacLab

# Create conda environment
./isaaclab.sh --conda env_isaaclab
conda activate env_isaaclab

# Install Isaac Lab and all dependencies
./isaaclab.sh --install
```

**Verify Isaac Sim launches:**

```bash
conda activate env_isaaclab
cd ~/IsaacLab
./isaaclab.sh -s
```

---

### 3. ROS 2 Jazzy Setup

**Add ROS 2 repository:**

```bash
sudo apt install software-properties-common curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
  http://packages.ros.org/ros2/ubuntu \
  $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | \
  sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

sudo apt update
```

**Install ROS 2 Jazzy + Navigation:**

```bash
# Full desktop install
sudo apt install ros-jazzy-desktop -y

# Navigation stack
sudo apt install ros-jazzy-navigation2 ros-jazzy-nav2-bringup -y

# Additional tools
sudo apt install ros-jazzy-slam-toolbox ros-jazzy-robot-localization -y

# ROS dev tools
sudo apt install python3-colcon-common-extensions python3-rosdep -y
```

**Initialize rosdep:**

```bash
sudo rosdep init
rosdep update
```

**Source ROS 2 (add to ~/.bashrc):**

```bash
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

---

### 4. ROS 2 Isaac Sim Bridge

**Install the Isaac Sim ROS 2 bridge:**

```bash
conda activate env_isaaclab

# Set ROS 2 environment variables for Isaac Sim
export ROS_DISTRO=jazzy
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/ros/jazzy/lib

# Install Isaac ROS bridge
pip install isaacsim-ros2-bridge --extra-index-url https://pypi.nvidia.com
```

**Create a ROS 2 workspace:**

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws

# Clone Isaac Sim ROS 2 examples
git clone https://github.com/isaac-sim/IsaacSim-ros_workspaces.git src/isaac_ros

# Build
source /opt/ros/jazzy/setup.bash
colcon build --symlink-install
source install/setup.bash
```

---

## Running Isaac Sim with ROS 2

### Environment Variables

Always set these before launching Isaac Sim with ROS 2:

```bash
export ROS_DISTRO=jazzy
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/ros/jazzy/lib
export ROS_DOMAIN_ID=0
```

Add to `~/.bashrc` to make permanent:

```bash
cat >> ~/.bashrc << 'EOF'

# Isaac Sim + ROS 2 settings
export ROS_DISTRO=jazzy
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/ros/jazzy/lib
export ROS_DOMAIN_ID=0
EOF
source ~/.bashrc
```

---

### Launch Isaac Sim

Open **Terminal 1** — Launch Isaac Sim with ROS 2 bridge enabled:

```bash
conda activate env_isaaclab
cd ~/IsaacLab
./isaaclab.sh -s
```

Once Isaac Sim is open:

1. Go to **Window → Extensions**
2. Search for `ROS2`
3. Enable **isaacsim.ros2.bridge**
4. Load your robot scene or use a sample: **File → Open → sample_navigation.usd**

---

### Launch Navigation Stack

Open **Terminal 2** — Source and launch Nav2:

```bash
source /opt/ros/jazzy/setup.bash
source ~/ros2_ws/install/setup.bash

# Launch Nav2 with your robot's config
ros2 launch nav2_bringup navigation_launch.py \
  use_sim_time:=True \
  params_file:=~/ros2_ws/src/your_robot/config/nav2_params.yaml
```

Open **Terminal 3** — Launch RViz2 for visualization:

```bash
source /opt/ros/jazzy/setup.bash

ros2 launch nav2_bringup rviz_launch.py
```

---

## Navigation Demo

### Running the Carter Robot Navigation Demo

Isaac Sim includes a Carter robot demo with full navigation support:

**Terminal 1 — Isaac Sim:**
```bash
conda activate env_isaaclab
cd ~/IsaacLab

# Launch with the Carter navigation scene
./isaaclab.sh -p scripts/ros2/carter_navigation.py
```

**Terminal 2 — Nav2:**
```bash
source /opt/ros/jazzy/setup.bash

ros2 launch nav2_bringup tb3_simulation_launch.py \
  use_sim_time:=True \
  slam:=True
```

**Terminal 3 — Send a navigation goal:**
```bash
source /opt/ros/jazzy/setup.bash

# Send a goal pose
ros2 topic pub /goal_pose geometry_msgs/PoseStamped \
  "{header: {frame_id: 'map'}, pose: {position: {x: 2.0, y: 1.0, z: 0.0}, orientation: {w: 1.0}}}" \
  --once
```

### Verify ROS 2 Topics from Isaac Sim

```bash
source /opt/ros/jazzy/setup.bash

# List all topics published by Isaac Sim
ros2 topic list

# Check laser scan data
ros2 topic echo /scan

# Check odometry
ros2 topic echo /odom

# Check camera feed
ros2 topic echo /rgb
```

### Expected Topics

| Topic      | Type                    | Description       |
|------------|-------------------------|-------------------|
| `/scan`    | `sensor_msgs/LaserScan` | LiDAR data        |
| `/odom`    | `nav_msgs/Odometry`     | Robot odometry    |
| `/rgb`     | `sensor_msgs/Image`     | Camera feed       |
| `/depth`   | `sensor_msgs/Image`     | Depth image       |
| `/cmd_vel` | `geometry_msgs/Twist`   | Velocity commands |
| `/tf`      | `tf2_msgs/TFMessage`    | Transform tree    |

---

## Troubleshooting

### NVIDIA driver not loading
```bash
# Check if DKMS is installed
dkms status

# If missing or build failed
sudo apt install dkms linux-headers-$(uname -r)
sudo apt install --reinstall nvidia-driver-580
sudo reboot
```

### `ModuleNotFoundError: No module named 'isaaclab'`
```bash
conda activate env_isaaclab
cd ~/IsaacLab
./isaaclab.sh --install
```

### ROS 2 topics not appearing from Isaac Sim
```bash
# Make sure these are set before launching Isaac Sim
export ROS_DISTRO=jazzy
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/ros/jazzy/lib

# Verify the ROS 2 bridge extension is enabled in Isaac Sim
# Window → Extensions → search "ROS2" → enable isaacsim.ros2.bridge
```

### `isaaclab.sh: No such file or directory`
```bash
# Always cd into IsaacLab directory first
cd ~/IsaacLab
./isaaclab.sh -s
```

### Assets not loading / HUB NOT DETECTED
Isaac Sim streams assets from NVIDIA's cloud. Ensure you have a stable internet connection. To increase timeout:

In Isaac Sim: **Edit → Preferences → Isaac Sim → Streaming Timeout** → set to `60`

### Process Memory red warning
Isaac Sim uses ~10GB RAM. Close other applications and optionally add swap:
```bash
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### Nav2 not receiving sensor data
```bash
# Check if Isaac Sim is publishing
ros2 topic hz /scan

# Check transform tree
ros2 run tf2_tools view_frames

# Verify sim time is being used
ros2 param get /bt_navigator use_sim_time
```

---

## Contributions

[robocademy.com](https://www.robocademy.com)

## References

- [Isaac Sim Documentation](https://docs.isaacsim.omniverse.nvidia.com/)
- [Isaac Lab GitHub](https://github.com/isaac-sim/IsaacLab)
- [ROS 2 Jazzy Documentation](https://docs.ros.org/en/jazzy/)
- [Nav2 Documentation](https://navigation.ros.org/)
- [Isaac Sim ROS 2 Bridge](https://docs.isaacsim.omniverse.nvidia.com/latest/ros2_tutorials/index.html)

---

## License

MIT License — feel free to use and modify for your projects.

---

> Made with ❤️ for the robotics community
