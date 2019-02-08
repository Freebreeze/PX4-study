[TOC]

**对于每一个变量，我们都需要知道它是在哪一个坐标系下的值**

# 流程图
```flow
st=>start: attitude_estimator_q_main
op=>operation: Your Operation
cond=>condition: Yes or No?
e=>end
st->op->cond
cond(yes)->e
cond(no)->op
```

```flow
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: get_hotel_ids|past
op2=>operation: get_proxy|current
sub1=>subroutine: get_proxy|current
op3=>operation: save_comment|current
op4=>operation: set_sentiment|current
op5=>operation: set_record|current

cond1=>condition: ids_remain空?
cond2=>condition: proxy_list空?
cond3=>condition: ids_got空?
cond4=>condition: 爬取成功??
cond5=>condition: ids_remain空?

io1=>inputoutput: ids-remain
io2=>inputoutput: proxy_list
io3=>inputoutput: ids-got

st->op1(right)->io1->cond1
cond1(yes)->sub1->io2->cond2
cond2(no)->op3
cond2(yes)->sub1
cond1(no)->op3->cond4
cond4(yes)->io3->cond3
cond4(no)->io1
cond3(no)->op4
cond3(yes, right)->cond5
cond5(yes)->op5
cond5(no)->cond3
op5->e
```

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

# 其他