# Comprehensive-Design
&nbsp; &nbsp; ESP32 micro-ROS 小车控制系统

基于 ESP32 + FreeRTOS + micro-ROS 的小车控制系统，通过订阅 ROS 2 的四元数消息，实现对小车运动的实时控制，并使用队列机制保证指令处理的稳定性。

&nbsp; &nbsp; 项目功能

✅ 使用 micro-ROS 连接 ROS 2 Agent

✅ 订阅` /cmd_quaternion` 话题`（geometry_msgs/msg/Quaternion）`

✅ 使用 FreeRTOS 队列 缓冲运动指令

✅ 多任务并行：

运动指令处理任务

micro-ROS 通信任务

系统监控任务（可选）

✅ 支持运动延迟控制（通过 w 分量）

&nbsp; &nbsp; 系统架构
```text
ROS 2 Agent
     │
     │  /cmd_quaternion (Quaternion)
     ▼
micro-ROS Subscriber
     │
     ▼
FreeRTOS Queue (MotionCommand)
     │
     ▼
Motion Processing Task
     │
     ▼
Motor Driver (Motion_Ctrl)
```

&nbsp; &nbsp; 依赖环境
硬件

ESP32 开发板

电机驱动模块

两个直流电机 + 被动轮

电池供电

软件

ESP-IDF

FreeRTOS

micro-ROS

ROS 2（PC 端运行 Agent）

&nbsp; &nbsp; 主要文件说明
文件	说明
`app_main.c`	主程序入口
`car_motion.h / .c`	小车运动控制接口
`uros_network_interfaces`	micro-ROS 网络接口
&nbsp; &nbsp; ROS 话题说明
订阅话题

Topic

`/cmd_quaternion`


消息类型

`geometry_msgs/msg/Quaternion`


字段含义

字段	含义
`x`	运动控制参数 X
`y`	运动控制参数 Y
`z`	运动控制参数 Z
`w`	延迟时间（ms，可选）
  工作流程

ESP32 启动并初始化电机、网络和任务

micro-ROS 连接 ROS 2 Agent

订阅 `/cmd_quaternion` 话题

接收到的消息被封装为` MotionCommand`

指令进入 FreeRTOS 队列

运动处理任务按顺序执行指令

调用 `Motion_Ctrl(x, y, z) `控制小车运动

根据` w `值进行延时（可选）

&nbsp; &nbsp; FreeRTOS 任务说明
1️⃣ micro_ros_task

负责 ROS 2 通信

订阅四元数话题

将数据发送到队列

2️⃣ motion_processing_task

从队列中读取运动指令

执行电机控制

支持指令延迟

3️⃣ monitoring_task（可选）

监控队列状态

监控任务运行状态

打印剩余堆内存

&nbsp; &nbsp; 关键参数配置
```text
#define MOTION_QUEUE_SIZE 20
#define ROS_NAMESPACE    CONFIG_MICRO_ROS_NAMESPACE
#define ROS_DOMAIN_ID    CONFIG_MICRO_ROS_DOMAIN_ID
#define ROS_AGENT_IP     CONFIG_MICRO_ROS_AGENT_IP
#define ROS_AGENT_PORT   CONFIG_MICRO_ROS_AGENT_PORT
```

&nbsp; &nbsp; 启动方式
1️⃣ 启动 ROS 2 Agent（PC 端）
`ros2 run micro_ros_agent micro_ros_agent udp4 --port 8888`

2️⃣ 烧录 ESP32 程序
```text
idf.py build
idf.py flash monitor
```

&nbsp; &nbsp; 示例 ROS 2 发布命令
```text
ros2 topic pub /cmd_quaternion geometry_msgs/msg/Quaternion \
"{x: 1.0, y: 0.0, z: 0.0, w: 500}"
```


表示小车运动，并延迟 500 ms

&nbsp; &nbsp; 日志与调试

使用` ESP_LOGI / ESP_LOGW / ESP_LOGE`

队列满载会自动丢弃指令并报警

每 10 条指令输出一次运行状态

&nbsp; &nbsp; License

MIT License
