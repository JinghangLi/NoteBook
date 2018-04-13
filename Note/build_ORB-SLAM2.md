### ORB-SLAM2 安装运行笔记
1. [ORB-SLAM2开源网址](https://github.com/raulmur/ORB_SLAM2)
2. [主要参考的安装教程](https://www.cnblogs.com/zengcv/p/6021512.html)
****
**注意**
* 安装过程以GitHub为主，教程为辅  

**安装主要依赖的教程**
* [安装OpenCV教程](https://www.cnblogs.com/darkknightzh/p/5638117.html)
* [安装Pangolin教程](https://www.2cto.com/kf/201706/649952.html)
[编译错误解决办法1](https://github.com/stevenlovegrove/Pangolin/issues/287)
[编译错误解决办法2](https://github.com/stevenlovegrove/Pangolin/issues/118)
[编译错误解决办法3](https://github.com/stevenlovegrove/Pangolin/issues/189)

**ORB-SLAM2安装过程**
1. 构建工作空间
[参考教程](https://www.cnblogs.com/lizhongpingchn/p/5531551.html)
2. 克隆repo
```
$ git clone https://github.com/raulmur/ORB_SLAM2.git ORB_SLAM2
```
3. 开始编译ORB-SLAM2
```
$ cd ORB_SLAM2
```
**为防止编译过程中卡住，先修改`build.sh`文件，将所有的`make -j`改为`make`**
```
$ chmod +x build.sh
$ ./build.sh
```
4. 编译完成后，开始编译ROS节点
* 见[参考网址](https://blog.csdn.net/sinat_38343378/article/details/78883919)将两个库移动到`lib`文件夹中，但是不修改CMakeLists.txt文件
* 解决如下报错1
```
MakeFiles/Makefile2:67: recipe for target 'CMakeFiles/RGBD.dir/all' failed
make[1]: *** [CMakeFiles/RGBD.dir/all] Error 2
```
[参考网址](https://github.com/raulmur/ORB_SLAM2/issues/490)
修改`Example/ROS/CMakeLists.txt`路径下的`CMakeLists.txt` 文件
```
Example/ROS/CMakeLists.txt
set(LIBS
${OpenCV_LIBS}
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${PROJECT_SOURCE_DIR}/../../../Thirdparty/DBoW2/lib/libDBoW2.so
${PROJECT_SOURCE_DIR}/../../../Thirdparty/g2o/lib/libg2o.so
${PROJECT_SOURCE_DIR}/../../../lib/libORB_SLAM2.so
-lboost_system
)
```
增加了`-lboost_system`
* 解决如下报错2
```
undefined reference to symbol '_ZN2cv6String10deallocateEv'
```
[参考网址](https://blog.csdn.net/xiangxianghehe/article/details/78828352)
编辑`CMakeLists.txt`，把`find_package(OpenCV 2.4.9 REQUIRED)`中的版本号改为自己装的opencv版本号即可。
* 接下来按照教程添加路径，并编译即可，最好手动编译，不使用`build_ros.sh`文件

5. 安装Kinetic V1 驱动
* [参考链接](https://blog.csdn.net/huapiaoxiang21/article/details/73830913)
运行如下命令
```
$ sudo apt-get install ros-kinetic-freenect-camera ros-kinetic-freenect-stack ros-kinetic-freenect-launch
```
* 还参考了以下网址，进行了配置安装，安装了 `openNI2` `libfreenect` `Nite`
[参考网址1](https://blog.csdn.net/x_r_su/article/details/52901876)  
[参考网址2](https://blog.csdn.net/strugglepeach/article/details/53992919)

6. 配置launch文件
在解压后的ORB-SLAM2的根目录下新建文件kinect_orbslam2.launch , 其内容为(根据需要，红色字体改为对应的路径，其他的无需修改)：
```
<launch>
  <param name="orb_use_viewer" value="false"/>
  <node pkg="ORB_SLAM2" type="RGBD" name="ORB_SLAM2"
        args="/home/ljh/Documents/ORB_SLAM2/Vocabulary/ORBvoc.txt
         /home/ljh/Documents/ORB_SLAM2/Examples/RGB-D/TUM1.yaml" cwd="node" output="screen"/>

  <include file="$(find freenect_launch)/launch/freenect.launch">
    <!-- use device registration -->
    <arg name="depth_registration" value="true" />
    <arg name="rgb_processing" value="true" />
    <arg name="ir_processing" value="false" />
    <arg name="depth_processing" value="false" />
    <arg name="depth_registered_processing" value="true" />
    <arg name="disparity_processing" value="false" />
    <arg name="disparity_registered_processing" value="false" />
    <arg name="sw_registered_processing" value="false" />
    <arg name="hw_registered_processing" value="true" />
  </include>
</launch>
```
7. 运行ORB-SLAM2
```
$ cd ORB-SLAM2
$ ./build.sh
$ roslaunch kinect_orbslam2.launch
 ```
