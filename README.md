## 介绍
这是一个基于激光雷达实现避障与导航的的自主小型无人机项目
## 硬件型号
机架：自制5寸
电机：
飞控：Kakute H7 Mini，孔距20mm  
电调：Tekko32 F4 4in1 Mini 45A ESC，孔距20mm  
电脑：NUC 13代i7
雷达：Livox Mid360  
桨叶：
接收机：自制ELRS无线电协议和CRSF有线协议接收机  
## 软件环境
系统：Ubuntu 24.04.01 LTS，ROS2 Jazzy
建图算法：FAST-LIO2（暂定）
导航算法：
雷达驱动：Livox-SDK2，livox_ros_driver2
控制器：
飞控：修改后的BetaFlight 4.4.0固件
## 为什么不用ROS1？
- 随着硬件的更新，旧版的Linux内核不再支持新的硬件比如显卡和CPU，我目前使用的NUC 13代i7 cpu的核显就无法工作，进入设置-关于，里面的显卡显示为llvmpipe (LLVM 12.0.0, 256 bits)，也即纯软件渲染无显卡，说明核显不在工作，这也导致运行Rviz实时查看点云等渲染任务时极其卡顿。
- 而如果要升级内核，ubuntu20.04又不再支持了，但Ubuntu 20.04是最后一个支持ROS1的系统，更新的版本必须用ROS2了
- 故以后又全部转为ubuntu24.04和ROS2 jazzy，且不再改变了，尽管可预料的会出现很多问题。人不能总是留在过去，必须向前看。
## 改动记录（全部基于以上软硬件环境）
### 2024.11.08 by Nyf --环境的配置
- 雷达相关驱动的配置整体上参考[这里](https://www.bilibili.com/read/cv39372701/?jump_opus=1)，整体的过程非常完整，以下对其不适配ROS2的部分进行说明
    - 在安装Livox-SDK2时，编译报错类似于`not a type name`这种找不到声明/不在作用域等这一类报错，在这里需要在源码文件里`sdk_core/comm/define.h`和`sdk_core/logger_handler/file_manager.h`两个文件里添加`#include <cstdint>`，然后编译即可通过，参考[这里](https://qiita.com/porizou1/items/f2123c16af3f86200a06)
    - 
