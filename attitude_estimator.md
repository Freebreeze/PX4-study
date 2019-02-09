[TOC]

[![](https://img.shields.io/badge/PX4-FreeBreeze-brightgreen.svg)](https://github.com/Freebreeze/PX4-study)



**对于每一个变量，我们都需要知道它是在哪一个坐标系下的值**

![](https://raw.githubusercontent.com/Freebreeze/PX4-study/master/attitude_estimator.jpg)



# 函数解释
## attitude_estimator_q_main
主函数，再Cmake中会启动这个函数，用脚本启动，脚本上会有 attitude_estimator_q start
如果启动成功，就会进入到start函数
哦对，在启动过程中会有产生一个新的对象
> attitude_estimator_q::instance = new AttitudeEstimatorQ;
**注意申请的内存是以指针形式申请的**

## AttitudeEstimatorQ::start
这是跳入 进程函数
> /* start the task开始一个进程 */
	_control_task = px4_task_spawn_cmd("attitude_estimator_q",
					   SCHED_DEFAULT,
					   SCHED_PRIORITY_ESTIMATOR,
					   2000,
					   (px4_main_t)&AttitudeEstimatorQ::task_main_trampoline,
					   nullptr);

分别是进程初始函数，进程默认调度，进程优先级，进程私有栈的大小，进程的入口函数，进程的参数列表。

## AttitudeEstimatorQ::task_main
这里是处理的主函数

** 首先是订阅一系列传感器消息 **

> _sensors_sub = orb_subscribe(ORB_ID(sensor_combined));
	_vision_odom_sub = orb_subscribe(ORB_ID(vehicle_visual_odometry));
	_mocap_odom_sub = orb_subscribe(ORB_ID(vehicle_mocap_odometry));
	_params_sub = orb_subscribe(ORB_ID(parameter_update));
	_global_pos_sub = orb_subscribe(ORB_ID(vehicle_global_position));
	_magnetometer_sub = orb_subscribe(ORB_ID(vehicle_magnetometer));

** 更新数据 **
> update_parameters(true);

这里的数据是更新默认参数表中的数据，默认参数表存储在attitude_estimator_q_params.c里面

** 更新传感器参数 **
采用阻塞等待的方法订阅sensor_conbine
其他传感取数据采取更新订阅的方式

*更新GPS数据以进行校准*
有磁力计、速度、加速度更新，作为参考


** 更新姿态数据**
这里是收到所有传感器的数据之后，进行姿态计算并且更新

## AttitudeEstimatorQ::update
这是姿态更新函数
首先进行的是获得初值四元数
这里采用init函数


## AttitudeEstimatorQ::init()
获取初始姿态四元数，这里我们详细查看，以便对坐标有深入的认识
```c++
// Rotation matrix can be easily constructed from acceleration and mag field vectors
// 'k' is Earth Z axis (Down) unit vector in body frame
Vector3f k = -_accel;
```
将初始加速度计获得的数据作为Z轴，为了得到Z轴，我们需要知道Z轴在坐标系下的映射并把其填进去。正如下面算式所示：
$$\begin{bmatrix}
x_1\\
y_1\\
z_1\\
\end{bmatrix}
=\begin{bmatrix}
a_{11}&a_{12}&a_{13}\\
a_{21}&a_{22}&a_{23}\\
a_{31}&a_{32}&a_{33}\\
\end{bmatrix}\begin{bmatrix}
x_2\\
y_2\\
z_2\\
\end{bmatrix}​$$

这里的加速度就相当 $\begin{bmatrix}a_{11}& a_{12}& a_{13}\end{bmatrix}$
相当于加速度轴在坐标系中的映射

初始化的时候有对磁偏角的补偿，默认是没有磁偏角的，只有GPS有足够的精度下，利用GPS信号对磁偏角进行补偿。在下面代码可以看出，自动设置磁偏角。
```c++
if (orb_copy(ORB_ID(vehicle_global_position), _global_pos_sub, &gpos) == PX4_OK) {
				if (_mag_decl_auto && gpos.eph < 20.0f && hrt_elapsed_time(&gpos.timestamp) < 1000000) {
					/* set magnetic declination automatically */
					update_mag_declination(math::radians(get_mag_declination(gpos.lat, gpos.lon)));
				}
```
具体函数见update_mag_declination

# 传感器
1. 加速度计：加速度计测量的是在机体坐标系下三个轴的加速度
2. 陀螺仪：计算飞机的角加速度
3. 磁力计：计算磁偏角在机体坐标系下向量

# 坐标系
1.机体坐标系
	
2.地理坐标系（NED）

# 误差的计算
1.主要的数据来源是来自于陀螺仪。
	可以这样理解：在飞机起飞之前或得最初的姿态$q_0$,后面根据陀螺仪测得的角速度$w$可以积分得到角度再加上之前的角度就能得到当前姿态。这个步骤是通过四元数的微分方程获得的。四元数微分方程就是物体旋转过程中四元数微分值与旋转角速度之间的关系。
2.加速度计和地磁计都是为了修正误差
	加速度计通过计算出物体当前加速度，再减去物体运动加速度，就可以获得重力加速度在集体坐标系下的分解，这个值是准确的。我们只需要吧重力在地理坐标系下的向量$(0,0,1)$通过四元数旋转矩阵转化到机体坐标系下，就能得到另一种重力加速度在机体坐标系下的分解，这个分解是有误差的，原因是姿态测量存在误差导致旋转矩阵产生误差，我们只需要把误差计算出来然后再更新旋转矩阵就好。这里用到一次旋转矩阵，有一次误差。
	地磁计原理是一样的，我们通过地磁计测量出磁场在机体坐标系下的分解，这个是准确的，然后我们再把磁偏角（地理坐标系下）通过旋转矩阵转换到机体坐标系下，这个是有误差的，接着更新旋转矩阵就好。这里用到一次旋转矩阵，有一次误差。
	如果没有磁偏角数据，我们只需要先将地磁计测到的数据通过旋转矩阵转化到地理坐标系下，得到向量$(a_x,a_y,a_z)$，这时候$a_y$一般是不为零的，但是磁偏角在地理坐标系下$a_y$是为零的，所以我们就可以在做一次变化，把向量改写成$(\sqrt{a_x^2+a_y^2},0,a_z)$ ，这样转化之后再通过旋转矩阵变化成机体坐标系下的向量计算误差就可以了。这里用到了两次旋转矩阵，有两次误差。