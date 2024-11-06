# ROS

## ROS概述

作为一个ROS（Robot Operating System）初学者，了解ROS的基本概念和常用组件是很重要的。以下是你在实际使用过程中需要掌握的一些关键知识和常用命令：

### 1. **ROS的基本概念**
   - **节点（Node）**：ROS中的一个可执行文件或进程。一个节点通常是执行某个具体功能的最小单元，多个节点共同完成复杂的机器人功能。
   - **主题（Topic）**：ROS节点之间用来传输数据的通道。一个节点可以发布（publish）消息到某个主题，另一个节点可以订阅（subscribe）这个主题以接收数据。
   - **消息（Message）**：节点之间通过主题传递的信息。消息可以是标准类型（例如整数、浮点数、字符串）或自定义类型。
   - **服务（Service）**：节点之间的请求/应答通信方式。与主题的异步通信不同，服务用于同步的请求和响应。
   - **参数服务器（Parameter Server）**：用于存储配置参数，节点可以从参数服务器读取或写入参数。

### 2. **重要的ROS组件**
   - **roscore**：ROS系统的核心组件，提供了节点通信的基础设施。每次启动ROS系统时，你首先需要启动`roscore`。
   - **roslaunch**：用于启动一组ROS节点或配置文件的工具。你可以通过定义`.launch`文件来批量启动多个节点，简化系统的启动流程。
   - **rostopic**：用于检查、发布和订阅ROS主题的命令行工具。你可以通过它查看当前有哪些主题存在，或者发布测试消息。
   - **rosnode**：管理ROS节点的命令行工具，可以查看节点信息、杀死节点等。
   - **rosservice**：管理ROS服务的命令行工具，可以列出、调用或检查服务。
   - **rosparam**：用于查看和设置参数服务器上参数的工具。
   - **rviz**：ROS中的可视化工具，可以用来显示机器人传感器数据、地图、轨迹等。
   - **rqt**：一组基于Qt的可视化工具，用于图形化管理ROS系统，方便调试。

### 3. **常用ROS命令**
   - **`roscore`**：启动ROS主节点和基础服务。
     ```bash
     roscore
     ```
   - **`rosrun`**：运行单个ROS节点。
     ```bash
     rosrun <package_name> <node_name>
     ```
   - **`roslaunch`**：启动多个ROS节点，通常通过`.launch`文件。
     ```bash
     roslaunch <package_name> <file.launch>
     ```
   - **`rostopic list`**：列出当前活动的所有主题。
     ```bash
     rostopic list
     ```
   - **`rostopic echo /topic_name`**：打印某个主题发布的消息。
     ```bash
     rostopic echo /topic_name
     ```
   - **`rostopic pub`**：发布消息到某个主题。
     ```bash
     rostopic pub /topic_name <message_type> <message_data>
     ```
   - **`rosnode list`**：列出当前所有节点。
     ```bash
     rosnode list
     ```
   - **`rosnode info <node_name>`**：查看某个节点的信息。
     ```bash
     rosnode info /node_name
     ```
   - **`rosparam list`**：列出当前所有参数。
     ```bash
     rosparam list
     ```
   - **`rosparam get <param_name>`**：获取某个参数的值。
     ```bash
     rosparam get /param_name
     ```
   - **`rosparam set <param_name> <value>`**：设置某个参数的值。
     ```bash
     rosparam set /param_name <value>
     ```
   - **`rosservice list`**：列出当前所有可用服务。
     ```bash
     rosservice list
     ```
   - **`rosservice call /service_name`**：调用一个服务。
     ```bash
     rosservice call /service_name
     ```

### 4. **典型的工作流程**
   1. **启动 `roscore`**：每次启动ROS应用时，首先运行 `roscore`。
   2. **启动节点**：使用 `rosrun` 或 `roslaunch` 启动一个或多个ROS节点。
   3. **检查节点、主题和服务**：使用 `rosnode list`、`rostopic list` 和 `rosservice list` 检查系统中有哪些节点、主题和服务在运行。
   4. **调试数据流**：使用 `rostopic echo` 来查看节点之间通过主题传输的数据。
   5. **可视化调试**：使用 `rviz` 可视化传感器数据、路径和地图等信息。

### 5. **扩展学习资源**
   - ROS官网： [http://www.ros.org/](http://www.ros.org/)
   - 官方教程：[http://wiki.ros.org/ROS/Tutorials](http://wiki.ros.org/ROS/Tutorials)
   - 常用ROS包：有很多常用的ROS包，如 `turtlesim`（教学用），`move_base`（用于机器人导航）等，你可以通过它们学习ROS的不同功能模块。

这些是ROS入门时需要掌握的基础内容和命令。通过实践不同的ROS例子和项目，你会逐渐熟悉这些工具。



## ROS工作空间

**ROS工作空间**（ROS Workspace）是ROS（Robot Operating System）中用于组织、构建和管理ROS项目的核心概念。你可以把它想象成一个容器，里面包含了你的所有ROS项目相关的文件，包括：

- **源码：** 你编写的C++、Python等代码。
- **功能包：** 将相关的源码、配置、启动文件等组织在一起的单元。
- **编译中间文件：** 在编译过程中产生的临时文件。

### 为什么需要ROS工作空间？

- **统一管理：** 将所有项目相关文件集中在一个地方，方便管理和查找。
- **编译系统：** ROS使用catkin工具来编译工作空间中的所有功能包，确保它们之间的依赖关系正确。
- **环境设置：** 工作空间提供了设置环境变量等功能，方便你使用ROS工具和功能包。

### ROS工作空间的结构

一个典型的ROS工作空间的目录结构如下：

```
src/  # 存放源代码
build/  # 编译过程中产生的中间文件
devel/  # 编译完成后生成的安装目录
```

- **src**：这个目录下包含多个功能包，每个功能包都是一个独立的文件夹。
- **build**：编译系统会在这个目录下生成一些临时文件。
- **devel**：编译完成后，生成的库文件、可执行文件等都会被安装到这个目录下。

### 创建和使用ROS工作空间

1. **创建工作空间：**

   Bash

   ```
   mkdir -p ~/catkin_ws/src
   cd ~/catkin_ws/src
   catkin_init_workspace
   ```

   这里创建了一个名为`catkin_ws`的工作空间。

2. **创建功能包：**

   Bash

   ```
   catkin_create_pkg my_package std_msgs rospy
   ```

   这将在`src`目录下创建一个名为`my_package`的功能包，并添加了`std_msgs`和`rospy`这两个依赖。

3. **编译工作空间：**

   Bash

   ```
   cd ~/catkin_ws
   catkin_make
   ```

   这会编译工作空间中的所有功能包。

4. **设置环境变量：**

   Bash

   ```
   source devel/setup.bash
   ```

   这会将工作空间的环境变量添加到你的shell中，这样你就可以使用工作空间中的功能包了。



**更多内容：**

- **功能包：** 功能包是ROS中代码组织的基本单元，包含节点、消息、服务、参数等。
- **catkin：** catkin是ROS的编译系统，用于构建ROS工作空间。
- **环境变量：** 环境变量用于告诉系统在哪里可以找到ROS工具和功能包。





## .msg文件 - 定义自定义消息

我们通过一个简单的例子来说明`.msg`文件在ROS系统中的实际使用。

假设我们要实现一个控制机器人速度的ROS系统，机器人需要根据收到的消息调整速度。

### 1. 定义一个简单的`.msg`文件
假设我们需要传递一个简单的速度指令，包含线速度和角速度，我们可以定义一个自定义的消息类型`Velocity.msg`：

```bash
# 文件路径：/your_ros_workspace/src/your_package/msg/Velocity.msg
float32 linear_velocity   # 线速度
float32 angular_velocity  # 角速度
```

这个`.msg`文件定义了一个消息，其中包含两个字段：`linear_velocity` 和 `angular_velocity`，它们的类型都是`float32`。

### 2. 配置CMakeLists.txt和package.xml

在`CMakeLists.txt`中，确保添加了生成消息的部分：

```cmake
find_package(catkin REQUIRED COMPONENTS
  roscpp
  std_msgs
  message_generation
)

add_message_files(
  FILES
  Velocity.msg
)

generate_messages(
  DEPENDENCIES
  std_msgs
)

catkin_package(
  CATKIN_DEPENDS message_runtime
)
```

在`package.xml`中，确保加入了`message_generation`和`message_runtime`依赖：

```xml
<build_depend>message_generation</build_depend>
<exec_depend>message_runtime</exec_depend>
```

### 3. 编写发布（Publisher）节点

发布节点负责发送包含线速度和角速度的消息给机器人。我们可以用C++或者Python编写一个简单的发布节点。

**Python发布节点例子**（`/your_ros_workspace/src/your_package/scripts/publisher_node.py`）：

```python
#!/usr/bin/env python
import rospy
from your_package.msg import Velocity  # 引入自定义的Velocity消息

def velocity_publisher():
    pub = rospy.Publisher('velocity_topic', Velocity, queue_size=10)
    rospy.init_node('velocity_publisher', anonymous=True)
    rate = rospy.Rate(10)  # 10Hz

    while not rospy.is_shutdown():
        vel_msg = Velocity()
        vel_msg.linear_velocity = 1.0  # 设置线速度为1.0
        vel_msg.angular_velocity = 0.5  # 设置角速度为0.5

        rospy.loginfo("Publishing velocity: linear=%.2f angular=%.2f" % (vel_msg.linear_velocity, vel_msg.angular_velocity))
        pub.publish(vel_msg)  # 发布消息
        rate.sleep()

if __name__ == '__main__':
    try:
        velocity_publisher()
    except rospy.ROSInterruptException:
        pass
```

### 4. 编写订阅（Subscriber）节点

订阅节点接收并处理从发布节点发来的`Velocity`消息。

**Python订阅节点例子**（`/your_ros_workspace/src/your_package/scripts/subscriber_node.py`）：

```python
#!/usr/bin/env python
import rospy
from your_package.msg import Velocity  # 引入自定义的Velocity消息

def velocity_callback(data):
    rospy.loginfo("Received velocity: linear=%.2f angular=%.2f" % (data.linear_velocity, data.angular_velocity))

def velocity_subscriber():
    rospy.init_node('velocity_subscriber', anonymous=True)
    rospy.Subscriber('velocity_topic', Velocity, velocity_callback)
    rospy.spin()  # 保持节点运行

if __name__ == '__main__':
    try:
        velocity_subscriber()
    except rospy.ROSInterruptException:
        pass
```

### 5. 运行整个系统

编译好工作空间之后，你可以在终端中运行以下命令：

```bash
# 启动ROS核心
roscore

# 运行发布节点
rosrun your_package publisher_node.py

# 运行订阅节点
rosrun your_package subscriber_node.py
```

这时，发布节点会不断发布线速度和角速度的消息，订阅节点则接收这些消息并在终端中打印出来。

### 总结
在这个例子中，`.msg`文件（`Velocity.msg`）定义了发布和订阅的消息结构，确保发布节点和订阅节点能够以固定格式进行数据交换。ROS系统通过这个消息格式来规范通信，使得不同节点可以准确地理解彼此发送的数据。