---
title: ROS在Linux环境下的配置与部署指南
published: 2025-10-01
updated: 2025-11-29
description: ros的配置
tags: [嵌入式, 文档工程, 协作]
category: 工程实践
draft: false
---

# ROS在Linux环境下的配置与部署指南

## 环境要求

### 1. 系统要求
- **操作系统**: Ubuntu 20.04 LTS (Focal Fossa) 或 Ubuntu 22.04 LTS (Jammy Jellyfish)
- **内核版本**: Linux 5.4 或更高版本
- **存储空间**: 至少 20GB 可用空间
- **内存**: 至少 4GB RAM
- **处理器**: 64位 x86 或 ARM 架构

### 2. ROS版本选择
| ROS版本 | Ubuntu版本 | 支持状态 | 推荐用途 |
|---------|------------|----------|----------|
| ROS Noetic | 20.04 | 长期支持(LTS) | 生产环境 |
| ROS2 Humble | 22.04 | 长期支持(LTS) | 新项目开发 |
| ROS2 Foxy | 20.04 | 已结束支持 | 遗留项目 |

## 安装步骤

### 步骤1: 系统更新与准备
```bash
# 更新软件包列表
sudo apt update
sudo apt upgrade -y

# 安装必要工具
sudo apt install -y curl wget git build-essential
sudo apt install -y python3-pip python3-venv

# 设置软件源为国内镜像（可选，加速下载）
sudo sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
sudo sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
```

### 步骤2: 安装ROS Noetic（Ubuntu 20.04）

#### 2.1 配置软件源
```bash
# 添加ROS软件源
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

# 添加GPG密钥
sudo apt install -y curl
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
```

#### 2.2 安装ROS
```bash
# 更新软件包列表
sudo apt update

# 安装完整版ROS
sudo apt install -y ros-noetic-desktop-full

# 或者安装基础版
# sudo apt install -y ros-noetic-desktop

# 或者安装最小版
# sudo apt install -y ros-noetic-ros-base
```

#### 2.3 环境配置
```bash
# 初始化rosdep
sudo rosdep init
rosdep update

# 设置环境变量
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc

# 安装构建工具依赖
sudo apt install -y python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential
sudo apt install -y python3-catkin-tools
```

### 步骤3: 安装ROS2 Humble（Ubuntu 22.04）

#### 3.1 设置语言环境
```bash
# 确保语言环境支持UTF-8
sudo apt update && sudo apt install -y locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

#### 3.2 添加ROS2软件源
```bash
# 添加GPG密钥
sudo apt install -y software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install -y curl
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

# 添加软件源
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

#### 3.3 安装ROS2
```bash
# 更新软件包列表
sudo apt update

# 安装ROS2桌面版
sudo apt install -y ros-humble-desktop

# 或者安装基础版
# sudo apt install -y ros-humble-ros-base

# 安装开发工具
sudo apt install -y python3-colcon-common-extensions
sudo apt install -y python3-rosdep2

# 初始化rosdep
sudo rosdep init
rosdep update
```

#### 3.4 环境配置
```bash
# 设置环境变量
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

## 创建工作空间

### 创建Catkin工作空间（ROS1）
```bash
# 创建工作空间目录
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/

# 初始化工作空间
catkin_init_workspace src

# 构建工作空间
catkin_make

# 配置环境变量
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 创建Colcon工作空间（ROS2）
```bash
# 创建工作空间目录
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws

# 构建工作空间
colcon build

# 配置环境变量
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

## 基础功能测试

### ROS1功能测试
```bash
# 启动roscore
roscore &

# 在新的终端中运行小海龟示例
# 终端1：运行turtlesim节点
rosrun turtlesim turtlesim_node

# 终端2：运行控制节点
rosrun turtlesim turtle_teleop_key

# 查看节点信息
rosnode list
rostopic list
```

### ROS2功能测试
```bash
# 在新的终端中运行小海龟示例
# 终端1：运行turtlesim节点
ros2 run turtlesim turtlesim_node

# 终端2：运行控制节点
ros2 run turtlesim turtle_teleop_key

# 查看节点信息
ros2 node list
ros2 topic list
```

## 常用工具安装

### 通用工具
```bash
# 安装VS Code（可选）
sudo snap install code --classic

# 安装Terminator终端
sudo apt install -y terminator

# 安装网络工具
sudo apt install -y net-tools nmap

# 安装串口工具
sudo apt install -y minicom cutecom
```

### ROS开发工具
```bash
# ROS1工具
sudo apt install -y ros-noetic-rviz ros-noetic-rqt
sudo apt install -y ros-noetic-rosbridge-server
sudo apt install -y ros-noetic-tf2-tools

# ROS2工具
sudo apt install -y ros-humble-rviz2 ros-humble-rqt
sudo apt install -y ros-humble-rosbridge-server
sudo apt install -y ros-humble-nav2-bringup
```

## 机器人特定配置

### 安装机器人相关包
```bash
# ROS1机器人包
sudo apt install -y ros-noetic-turtlebot3
sudo apt install -y ros-noetic-moveit
sudo apt install -y ros-noetic-gazebo-ros-pkgs

# ROS2机器人包
sudo apt install -y ros-humble-turtlebot3
sudo apt install -y ros-humble-moveit
sudo apt install -y ros-humble-gazebo-ros-pkgs
```

### 配置TurtleBot3（示例）
```bash
# 设置TurtleBot3型号
echo "export TURTLEBOT3_MODEL=burger" >> ~/.bashrc
source ~/.bashrc

# 下载TurtleBot3仿真模型
sudo apt install -y ros-noetic-turtlebot3-simulations
# 或者ROS2
sudo apt install -y ros-humble-turtlebot3-simulations
```

## 网络配置（多机通信）

### 单机配置
```bash
# 查看本机IP
hostname -I

# 设置环境变量
export ROS_IP=$(hostname -I | awk '{print $1}')
export ROS_HOSTNAME=$ROS_IP
export ROS_MASTER_URI=http://$ROS_IP:11311

# 持久化配置
echo "export ROS_IP=\$(hostname -I | awk '{print \$1}')" >> ~/.bashrc
echo "export ROS_HOSTNAME=\$ROS_IP" >> ~/.bashrc
echo "export ROS_MASTER_URI=http://\$ROS_IP:11311" >> ~/.bashrc
```

### 多机通信配置
```bash
# 主控机配置
export ROS_MASTER_URI=http://<master_ip>:11311
export ROS_IP=<local_ip>

# 从机配置
export ROS_MASTER_URI=http://<master_ip>:11311
export ROS_IP=<local_ip>
```

## Docker容器配置（可选）

### 创建ROS Docker容器
```bash
# 拉取ROS镜像
docker pull osrf/ros:noetic-desktop-full
# 或者ROS2
docker pull osrf/ros:humble-desktop

# 运行Docker容器
docker run -it \
  --env="DISPLAY" \
  --env="QT_X11_NO_MITSHM=1" \
  --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
  osrf/ros:noetic-desktop-full \
  bash
```

## 故障排除

### 常见问题及解决方案

#### 问题1: rosdep初始化失败
```bash
# 解决方案：手动下载配置文件
sudo mkdir -p /etc/ros/rosdep/sources.list.d/
sudo curl -o /etc/ros/rosdep/sources.list.d/20-default.list https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/sources.list.d/20-default.list
```

#### 问题2: 权限不足错误
```bash
# 解决方案：修复权限
sudo chmod -R 755 /opt/ros/
sudo chown -R $USER:$USER ~/catkin_ws/
```

#### 问题3: 编译错误
```bash
# 解决方案：清理并重新编译
cd ~/catkin_ws
rm -rf build devel
catkin_make clean
catkin_make
```

#### 问题4: 环境变量未生效
```bash
# 解决方案：手动刷新环境
source ~/.bashrc
# 或者
exec bash
```

## 性能优化

### 系统优化配置
```bash
# 禁用不必要的服务
sudo systemctl disable bluetooth.service
sudo systemctl disable cups.service

# 调整交换空间（如果内存不足）
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# 添加开机自动挂载
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### ROS编译优化
```bash
# 使用多核编译
catkin_make -j$(nproc)

# 或者使用更快的构建系统
catkin build --jobs $(nproc) --limit-status-rate 0.01
```

## 维护与更新

### 定期维护命令
```bash
# 更新ROS软件包
sudo apt update
sudo apt upgrade ros-noetic-*  # ROS1
# 或者
sudo apt upgrade ros-humble-*  # ROS2

# 清理无用包
sudo apt autoremove
sudo apt autoclean

# 清理ROS日志
rosclean purge -y
```

### 备份配置
```bash
# 备份工作空间
tar -czf ~/ros_backup_$(date +%Y%m%d).tar.gz ~/catkin_ws/

# 备份环境配置
cp ~/.bashrc ~/.bashrc.backup_$(date +%Y%m%d)
```

## 附录

### 有用的命令速查表

| 命令 | ROS1 | ROS2 | 功能 |
|------|------|------|------|
| 启动核心 | roscore | N/A | 启动ROS主节点 |
| 运行节点 | rosrun <pkg> <node> | ros2 run <pkg> <node> | 运行指定节点 |
| 列出节点 | rosnode list | ros2 node list | 显示运行中的节点 |
| 列出话题 | rostopic list | ros2 topic list | 显示活跃话题 |
| 话题信息 | rostopic info <topic> | ros2 topic info <topic> | 显示话题详情 |
| 消息发布 | rostopic pub <topic> <msg> | ros2 topic pub <topic> <msg> | 发布消息到话题 |
| 构建系统 | catkin_make | colcon build | 构建工作空间 |

### 学习资源
- **官方文档**: [ros.org](http://wiki.ros.org/)
- **ROS2文档**: [docs.ros.org](https://docs.ros.org/)
- **中文社区**: [roschina.org](http://www.roschina.org/)
- **GitHub**: [ros/rosdistro](https://github.com/ros/rosdistro)
- **问答社区**: [answers.ros.org](https://answers.ros.org/)
```
