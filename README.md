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
- 雷达相关驱动的配置整体上参考[这里](https://www.bilibili.com/read/cv39372701/?jump_opus=1)，整体的过程非常完整，以下对其不适配ROS2的部分进行说明，也即按照他的操作报错或者没有可选项时，就看后面的说明进行操作
    - 在安装Livox-SDK2时，编译报错类似于`not a type name`这种找不到声明/不在作用域等这一类报错，在这里需要在源码文件里`sdk_core/comm/define.h`和`sdk_core/logger_handler/file_manager.h`两个文件里添加`#include <cstdint>`，然后编译即可通过，参考[这里](https://qiita.com/porizou1/items/f2123c16af3f86200a06)，编译完成后记得在同路径（即build文件夹）下`sudo make install`一下
    - 在安装livox_ros_driver2时，首先需要注意路径，源码必须下载到ws_livox/src下。虽然我是jazzy，但是编译仍然可以按照源码README里使用`./build.sh humble`进行编译，因为都是ROS2
        - 与ROS1不同，这里需要在两个地方都修改json文件来配置网口通信，即`...../ws_livox/src/livox_ros_driver2/config`和`ws_livox/install/livox_ros_driver2/share/livox_ros_driver2/config`两个地方的MID360_config.json文件都需要修改对应的IP
        - 然后应该会发现编译不通过，报错缺少路径`/usr/local/lib`下的文件`liblivox_lidar_sdk _shared.so`或者其他什么，这也是和ROS1不一样，colcom build构建完成后似乎不自动添加环境变量，所以需要手动在添加：`sudo gedit ~/.bashrc`，然后在打开的文件最后一行加入`export LD_LIBRARY_PATH=/usr/local/lib`,然后`source ~/.bashrc`更新一下环境变量
        - 在运行前，注意要先source一下install文件夹里的setup.sh，因为ROS1中编译后的setup文件会放在devel1文件夹下，但ROS2是放在install文件夹下
        - 对于ROS2的livox_ros_driver2，没有.launch文件了，全部变成.py来进行启动，先后在两个分别source后的终端输入`ros2 launch livox_ros_driver2 msg_MID360_launch.py`和`ros2 launch livox_ros_driver2 rviz_MID360_launch.py`，开始通过rviz查看点云
        - 如果有报错`Failed to init livox lidar sdk`，有可能就是像前面所说，网络没有给两个jeon文件进行配置
    - 部署FAST LIO2，按照[这里](https://github.com/hku-mars/FAST_LIO/tree/ROS2?tab=readme-ov-file#1.3)操作即可，需要注意的时需要先把install文件夹里的setup.sh给source一下，然后先后使用命令`ros2 launch livox_ros_driver2 msg_MID360_launch.py`将Mid360驱动起来，使用`ros2 launch fast_lio mapping.launch.py config_file:=avia.yaml`建图并可视化
- 番外：如果git clone时用的是https，则需要token来获取权限；如果是git，则通过SSH获取。

