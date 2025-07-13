# ROS2 笔记

## 安装篇

### 安装docker

```bash
sudo pacman -S docker
sudo systemctl enable docker
sudo systemctl start docker
```

tips:
如果在启动docker daemon前运行了sudo pacman -Syu更新系统，会导致启动失败。需要重启电脑

### 配置docker VPN

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo nvim /etc/systemd/system/docker.service.d/http-proxy.conf
```

添加如下内容

```bash
[Service]
Environment="HTTP_PROXY=socks5://127,0,0,1:1080/"
Environment="HTTPS_PROXY=socks5://127,0,0,1:1080/"
```

```bash
sudo docker pull osrf/ros:humble-desktop
```

enjoy ros2

### 图形化显示

需要在宿主机上运行xhost +local:docker

## Ros2 学习篇

### Ros2节点

#### 节点之间交互

- 话题 - topics

- 服务 - services

- 动作 - actions

- 参数 - parameters

#### 启动节点

```c
ros2 run <package_name> <executable_name>
```

#### 通过命令行查看节点信息

- 查看节点列表

```c
ros2 node list
```

- 查看节点信息

```c
ros2 node info <node_name>
```

- 重映射节点名称

```c
ros2 run turtlesim turtlesim_node --ros-args --remap __node:=my_turtle
```

- 运行节点时设置参数

```c
ros2 run example_parameters_rclcpp parameters_basic --ros-args -p rcl_log_level:=10
```

### 功能包与工作空间

#### 工作空间

工作空间是包含若干个功能包的目录，一开始大家把工作空间理解成一个文件夹就行了。这个文件夹包含下有`src`。所以一般新建一个工作空间的操作就像下面一样

```c
cd d2lros2/chapt2/
mkdir -p chapt2_ws/src
```

#### 功能包

功能包可以理解为存放节点的地方，ROS2中功能包根据编译方式的不同分为三种类型。

- ament_python，适用于python程序
- cmake，适用于C++
- ament_cmake，适用于C++程序,是cmake的增强版

##### 功能包获取的两种方式

- 安装获取

```c
sudo apt install ros-<version>-package_name
```

- 手动编译获取

编译安装后需要source install/setup.bash

##### 功能包相关的指令

```c
create       Create a new ROS2 package
executables  Output a list of package specific executables
list         Output a list of available packages
prefix       Output the prefix path of a package
xml          Output the XML of the package manifest or a specific tag
```

- 创建功能包

```c
ros2 pkg create <package-name>  --build-type  {cmake,ament_cmake,ament_python}  --dependencies <依赖名字>
```

- 列出可执行文件
  
  - 列出所有
  
  ```c
  ros2 pkg executables
  ```
  
  - 列出某个功能包下的所有可执行文件
  
  ```c
  ros2 pkg executables turtlesim
  ```

- 列出所有的包

```c
ros2 pkg list
```

- 输出某个包所在路径的前缀

```c
ros2 pkg prefix  <package-name>
```

- 列出包的清单描述文件

每一个功能包都有一个标配的manifest.xml文件，用于记录这个包的名字，构建工具，编译信息，拥有者，干啥用的等信息。

通过这个信息，就可以自动为该功能包安装依赖，构建时确定编译顺序等

```c
ros2 pkg xml turtlesim 
```

### ROS2构建工具 - Colcon

#### 编译示例

- 创建工作去

```bash
cd d2lros2/chapt2/
mkdir colcon_test_ws && cd colcon_test_ws
```

- 下载源码

```bash
git clone https://github.com/ros2/examples src/examples -b humble
```

- 编译工程

```bash
colcon build
```

- 编译完成后的目录结构

```bash
.
├── build
├── install
├── log
└── src

4 directories, 0 files
```

- `build` 目录存储的是中间文件。对于每个包，将创建一个子文件夹，在其中调用例如CMake
- `install` 目录是每个软件包将安装到的位置。默认情况下，每个包都将安装到单独的子目录中。
- `log` 目录包含有关每个colcon调用的各种日志信息。

#### 运行编译好的节点

1. 打开一个终端使用 cd colcon_test_ws进入我们刚刚创建的工作空间，先source 一下资源
   
   ```bash
   source install/setup.bash
   ```

2. 运行一个订者节点，你将看不到任何打印，因为没有发布者
   
   ```bash
   ros2 run examples_rclcpp_minimal_subscriber subscriber_member_function
   ```

3. 打开一个新的终端，先source，再运行一个发行者节点
   
   ```bash
   source install/setup.bash
   ros2 run examples_rclcpp_minimal_publisher publisher_member_function
   ```

#### 其他指令

#### 只编译一个包

```
colcon build --packages-select YOUR_PKG_NAME 
```

#### 不编译测试单元

```
colcon build --packages-select YOUR_PKG_NAME  --cmake-args -DBUILD_TESTING=0
```

#### 运行编译的包的测试

```
colcon test
```

#### 安装时只建立软连接，不拷贝可执行文件

每次调整 python 脚本时都不必重新build了

```
colcon build --symlink-install
```

### ROS2 Topic

#### ROS2 话题工具

- rqt_graph

```bash
ros2 run demo_nodes_py listener
ros2 run demo_nodes_cpp talker
rqt_graph
```

- 返回系统中当前活动的所有主题的列表

```bash
ros2 topic list
```

- topic list 增加消息类型

```bash
ros2 topic list -t
```

- 打印实时话题内容

```bash
ros2 topic echo /chatter
```

- 查看主题信息

```bash
ros2 topic info /chatter
```

- 查看消息类型

```bash
ros2 topic show std_msgs/msg/String
```

- 手动发布消息

```bash
ros2 topic pub /chatter std_msgs/msg/String 'data: "123"'
```

### ROS2 服务常用命令

- 查看服务列表

```bash
ros2 service list
```

- 手动调用服务

```bash
ros2 service call /add_two_ints example_interfaces/srv/AddTwoInts "{a: 5, b: 10}"
```

- 查看服务接口类型

```bash
ros2 service type /add_two_ints
```

- 查找使用某一接口的服务

```bash
ros2 service find example_interfaces/srv/AddTwoInts
```
