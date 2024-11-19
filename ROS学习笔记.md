参考链接：[(11条消息) Ros1入门到入土_Addam Holmes的博客-CSDN博客](https://blog.csdn.net/m0_55534317/article/details/125544490)



# 创建工作空间

工作空间（workspace）是一个存放工程开发相关文件的文件夹

 src：代码空间（Source Space）【放置功能包的代码，功能包的配置文件等】

 build：编译空间（Build Space）【放置编译过程中所产生的的中间文件，这个文件夹可以不用管】

 devel：开发空间（Development Space）【放置编译生成的可执行文件，库以及一些脚本等】

 install：安装空间（Install Space）【用install安装的东西放在这个空间，也可以不用管】

**创建工作空间**

```c++
mkdir -p ~/my_ws/src/     // my_ws是文件夹名字，随便怎么取,创建文件夹以及文件夹的子文件夹src
cd /my_ws/src
catkin_init_wordspace     // 初始化生成一个.txt文件，把这个文件夹初始化变成一个工作空间
```

**编译工作空间**

```c++
cd ~/my_ws/		    //进入项目根目录			
catkin_make			// 产生build和devel
catkin_make install //产生install安装空间
```

**设置环境变量**

```c++
source devel/setup.bash
```

每一次运行都需要使用`source devel/setup.bash`重新编译，非常容易忘而且非常麻烦，我们在系统根目录按Ctrl+H找到 `.bashrc `，点开之后在最后加上这样一条命令

```C++
source 系统名字/项目名/devel/setup.bash
// 比如我的就是source ~/my_Ws/devel/setup.bash
```

![image-20230528135556987](C:\Users\qq156\AppData\Roaming\Typora\typora-user-images\image-20230528135556987.png)

**检查环境变量**

```C++
echo $ROS_PACKAGE_PATH
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/1be7cfcf835e4fd099e6631bbd289816.png#pic_center)





# 创建功能包

功能包是放置Ros源码的最小单元，所以所有源码必须放在功能包里面。

**我们需要先创建一个功能包**
功能包的放置一定要放置到src中。test_pkg是我们的包名，后面std_msgs rospy roscpp使我们这个功能包需要依赖的一些库，std_msgs是一些基本的数据类型，rospy是ros的Python依赖，roscpp是c++依赖。【PS：同一个工作空间下，不允许存在相同的功能包名，不同的工作空间下允许存在相同的功能包名】



```C++
cd ~/my_ws/src
catkin_create_pkg learning_topic std_msgs rospy roscpp
```



**编译功能包**

```C++
cd ~/my_ws
catkin_make
source ~/my_ws/devel/setup.bash
```



**功能包结构如下：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/9dfbf240a69b4da4b76fab36e88d2880.png#pic_center)

package.xml描述了这个功能包的一些基本信息，比如名字，版本号等。

![在这里插入图片描述](https://img-blog.csdnimg.cn/844c1d45aa454db5979a1396bab32e21.png#pic_center)

下面还有这个功能包的依赖信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/b0eea78e5c214db296031ce0cd0bb3e6.png#pic_center)

CMakeList.txt文件设置了功能包代码的编译规则

![在这里插入图片描述](https://img-blog.csdnimg.cn/d9b0d28ff9884d009c0a4a6557a1f6f4.png#pic_center)









# 编写代码

以编写一个发布者为例



* 在功能包src文件下创建velocity_publisher.cpp
* 编写代码

```C++
#include <ros/ros.h>
#include <geometry_msgs/Twist.h>
int main(int argc, char **argv)
{
	// ROS节点初始化 [velocity_publisher为节点名字]
	ros::init(argc, argv, "velocity_publisher");
	// 创建节点句柄 [管理结点的资源]
	ros::NodeHandle n;
	// 创建一个Publisher，发布名为/turtle1/cmd_vel的topic，消息类型为geometry_msgs::Twist，队列长度10『表示如果发布者发布特别块，先把数据存到对列里去，如果队列存满了，就根据时间戳抛弃最久的』
	ros::Publisher turtle_vel_pub = n.advertise<geometry_msgs::Twist>("/turtle1/cmd_vel", 10);
	// 设置循环的频率
	ros::Rate loop_rate(10);
	int count = 0;
	while (ros::ok())
	{
	    // 初始化geometry_msgs::Twist类型的消息
		geometry_msgs::Twist vel_msg;
		vel_msg.linear.x = 0.5;
		vel_msg.angular.z = 0.2;
	    // 发布消息
		turtle_vel_pub.publish(vel_msg);
		ROS_INFO("Publsh turtle velocity command[%0.2f m/s, %0.2f rad/s]", 
				vel_msg.linear.x, vel_msg.angular.z);
	    // 按照循环频率延时
	    loop_rate.sleep();
	}
	return 0;
}
```

* 配置CMakeLists.txt中的编译规则

* * 设置需要编译的代码和生成的可执行文件
  * 配置链接库

```python
# 描述把src/velocity_publisher.cpp和可执行文件velocity_publisher建立连接
add_executable(velocity_publisher src/velocity_publisher.cpp)
# 把可执行文件和ROS的库去做连接
target_link_libraries(velocity_publisher ${catkin_LIBRARIES})
```

![Screenshot 2023-05-28 142204](C:\Users\qq156\Pictures\Screenshots\Screenshot 2023-05-28 142204.png)

* 编译并运行发布者

```C++
cd ~/my_ws
catkin_make
source devel/setup.bash
//新开一个终端
roscore
//新开一个终端
rosrun turtlesim turtlesim_node
//新开一个终端
cd my_Ws/src
rosrun learing_topic velocity_publisher
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/54ed1791c9e641a5965ea8a07b845915.png#pic_center)















# 话题消息的定义与使用

之前我们都是使用ROS给我们定义好的Twist（发布）和Post（订阅）方法。但是有的时候开发过程中ROS定义好的消息无法满足我们的需求，这个时候我们就需要自己去定义消息类型。

我们可以自己定义消息类别，然后传输一个Person的个人信息。

– 通过语言无关的文件定义一个Topic（话题），话题名叫“ person_info ”

– Publisher（发布者）会通过这个话题发布这个人的信息

– Subscriber（订阅者）会通过这个话题订阅这个人的信息。
![在这里插入图片描述](https://img-blog.csdnimg.cn/193ecb0255244f43b44b3c6ba9e5f3b5.png#pic_center)





## 自定义话题消息

**定义一个msg文件**

我们具体实施如下：

1） 在组件目录下创建一个文件夹名为“msg”，把跟消息相关的定义都放在该文件夹里面。

2）在msg文件夹下右键，新建终端，打开该文件夹下的终端。然后输入以下代码创建一个“ .msg ”文件

```C++
cd my_ws/src/learning_topic
mkdir msg
cd msg/
touch Person.msg
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2c3db9c262d245e7a960e99c855ac973.png#pic_center)

3）双击打开这个Person.msg文件，输入以下内容然后保存

```C++
string name
uint8 sex
uint8 age

uint8 unknown = 0
uint8 male    = 1
uint8 female  = 2
```

4） 添加功能包依赖及编译选项

在package.xml中添加功能包依赖

```python
<!-- 编译依赖 -->
<build_depend>message_generation</build_depend>
<!-- 执行依赖 -->
<exec_depend>message_runtime</exec_depend>
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/3b89fbfd68834c6ab9a242089f3a7f90.png#pic_center)

然后在CMakeList.txt添加编译选项

```C++
find_package(
	......
	message_generation
)
    
add_message_files(FILES Person.msg)
generate_messages(DEPENDENCIES std_msgs)

catkin_package(
	......
	message_runtime
)
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/d190cab9cf6149ebb30db9d521ef2811.png#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2ec3f1bf827b47b5b70198c9cea193fc.png#pic_center)

现在我们就完成了msg文件的生成，可以回到工作空间的根目录下试一下编译catkin_make，只要底下的输出没出现红色，出现什么颜色都是好颜色。编译完成之后就会生成一个C++的头文件。

```C++
cd ~/my_ws
catkin_make
```

![image-20230528143831007](C:\Users\qq156\AppData\Roaming\Typora\typora-user-images\image-20230528143831007.png)







## C++实现发布者和订阅者创建

**发布者的创建**

在`my_ws/src/learning_topic/src`创建person_publisher.cpp，然后编写以下代码

```C++
/**
 * 该例程将发布/person_info话题，自定义消息类型learning_topic::Person
 */
#include <ros/ros.h>
#include "learning_topic/Person.h"
int main(int argc, char **argv)
{
    // ROS节点初始化
    ros::init(argc, argv, "person_publisher");
    // 创建节点句柄
    ros::NodeHandle n;
    // 创建一个Publisher，发布名为/person_info的topic，消息类型为learning_topic::Person，队列长度10
    ros::Publisher person_info_pub = n.advertise<learning_topic::Person>("/person_info", 10);
    // 设置循环的频率
    ros::Rate loop_rate(1);
    int count = 0;
    while (ros::ok())
    {
        // 初始化learning_topic::Person类型的消息
    	learning_topic::Person person_msg;
		person_msg.name = "Addam";
		person_msg.age  = 21;
		person_msg.sex  = learning_topic::Person::male;
        // 发布消息
		person_info_pub.publish(person_msg);
       	ROS_INFO("Publish Person Info: name:%s  age:%d  sex:%d", 
				  person_msg.name.c_str(), person_msg.age, person_msg.sex);
        // 按照循环频率延时
        loop_rate.sleep();
    }
    return 0;
}

```

还是一样的流程，先初始化，创建节点句柄，创建一个Publisher，设置循环频率

初始化一个learning_topic::Person类型的消息，消息主要就是名字，年龄，和调用刚才宏定义生成的性别。

然后发布消息并添加延时，这个发布者会不停发布这个个人信息。







**订阅者的创建**

在`my_ws/src/learning_topic/src`创建person_subscriber.cpp，然后编写以下代码

```C++
/**
 * 该例程将订阅/person_info话题，自定义消息类型learning_topic::Person
 */
#include <ros/ros.h>
#include "learning_topic/Person.h"
// 接收到订阅的消息后，会进入消息回调函数
void personInfoCallback(const learning_topic::Person::ConstPtr& msg)
{
    // 将接收到的消息打印出来
    ROS_INFO("Subcribe Person Info: name:%s  age:%d  sex:%d", 
			 msg->name.c_str(), msg->age, msg->sex);
}
int main(int argc, char **argv)
{
    // 初始化ROS节点
    ros::init(argc, argv, "person_subscriber");
    // 创建节点句柄
    ros::NodeHandle n;
    // 创建一个Subscriber，订阅名为/person_info的topic，注册回调函数personInfoCallback
    ros::Subscriber person_info_sub = n.subscribe("/person_info", 10, personInfoCallback);
    // 循环等待回调函数
    ros::spin();
    return 0;
}
```

先初始化节点，创建节点句柄，创建一个Subscriber，订阅名为/person_info的主题，并且注册回调函数

回到函数就会把这个人的信息打印出来。

订阅者发布者都需要先引入这个头文件，而且这个发布和订阅的话题也是我们自己定义的

```C++
#include "learning_topic/Person.h"
```



**接下来编译我们的发布者订阅者**

配置CMakeList.txt中的编译规则

1）设置需要编译的代码和生成的可执行文件

2）设置链接库

3）添加依赖项

```python
add_executable(person_publisher src/person_publisher.cpp)
target_link_libraries(person_publisher ${catkin_LIBRARIES})
add_dependencies(person_publisher ${PROJECT_NAME}_generate_messages_cpp)

add_executable(person_subscriber src/person_subscriber.cpp)
target_link_libraries(person_subscriber ${catkin_LIBRARIES})
add_dependencies(person_subscriber ${PROJECT_NAME}_generate_messages_cpp)
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/ee54493d0f8e4920ad7c89120239bfd7.png#pic_center)































# launch启动文件



## launch文件概述

launch启动文件旨在减少用户不断打开终端，不断去输入的重复工作。

launch文件：通过XML文件实现多节点的配置和启动（可自动启动ROS Master）

一个launch文件使用XML描述的内容和属性，launch文件可以通过如下这种XML的形式实现ROS中多个节点的配置与启动，如果使用launch文件就不需要不断打开终端使用rosrun的命令了，在一个复杂的机器人系统中节点是很多的，一个个去rosrun非常麻烦，我们可以通过roslaunch去启动一个launch文件，实现launch文件包含节点的所有功能。

每个node搜是一个节点，就是一个rosrun的命令。

launch文件在启动时会自动去查找当前系统中有没有ROS Master（roscore）在运行，如果有的话就不会去运行ROS Master，如果没有就自己启动一个ROS Master
![在这里插入图片描述](https://img-blog.csdnimg.cn/4566ea3bf9fc450fbb85611201d72bb0.png#pic_center)









## launch文件常用语法

**（1）node节点标签**
launch中的所有内容都是通过XML这种标签来做描述的。

所有launch文件的内容都必须要包含到一个根标签（根元素），launch文件中的根元素采用标签定义。

为启动节点

其中 

**pkg：节点所在的功能包的名称**

**type： 节点的可执行文件**

**name：节点运行时的名称**

node节点中除了刚才的三个属性外还有其他的**可选属性**

**output：用来控制某个节点是不是要把日志信息打印到当前的终端里面的**

**respawn：控制节点如果启动运行挂掉了是否重启**

**required：表示launch文件里某个节点是不是必须要启动起来**

**ns： ns代表命名空间，可以给每个节点创建命名空间，避免命名冲突**

**args： 节点的输入参数。**

```xml
<launch>
    <node pkg="turtlesim" name="sim1" type="turtlesim_node"/>
    <node pkg="turtlesim" name="sim2" type="turtlesim_node"/>
</launch>
```



**（2）参数设置标签**

launch文件中还包含了一些其他标签，比如参数设置的标签

用来设置ROS系统运行中的参数，存储在ROS参数服务器中，其中，

**name：参数名**

**value：参数值**

可以加载参数文件中多个参数，把一个参数文件中的所有参数都加载到服务器中来

```xml
<param name="output_frame" value="odom"/>
<rosparam file="params.yaml" command="load" ns="params"/>
```

表示某个launch文件所使用的局部变量。仅限于该launch文件使用，使用语法见下方。其中

**name：参数名**

**default：参数值**

和param不同，param表示存在参数服务器中的参数，而arg表示只存在我们这个launch文件中的参数只允许内部使用。

```xml
<arg name="arg-name" default="arg-value"/>
<!-- 调用的方式=》$(arg 参数名) -->
<param name="foo" value="$(arg arg-name)"/>
<node name="node" pkg="package" type="type" args="$(arg arg-name)"/>
```



**<rosparam>加载文件中的多个参数**

```
<rosparam file="params.yaml" command="load" ns="params" />
```





**（3）重映射 标签**

可以把ROS中某些计算图的资源进行重新命名，重映射ROS计算图资源的命名，语法如下，其中

**from为原命名**

**to为映射之后的命名**

```xml
<remap from="/turtlebot/cmd_vel" to="/cmd_vel"/>
```



**（4）嵌套 标签**

可以包含其他launch文件，在开发过程中有可能会出现launch文件比较多，可以用include去给launch文件模块化，然后一个main函数就包含多个launch文件

```xml
<include file="$(dirname)/other.launch"/>
```

file ：包含的其他launch文件路径













## launch使用演示

**（1）创建一个功能包，这个功能包不需要任何的依赖**

```C++
cd ~/my_ws
catkin_create_pkg learning_launch
```

然后在新的功能包里创建一个文件夹，命名为launch，创建一个launch文件

```
mkdir launch
cd launch
touch simple.launch
```



**（2） 第一个launch文件（simple.launch）**

用一个launch的根标签包含两个node节点。

第一个节点是位于learning_topic的listener，节点名叫做person_subscriber

第二个节点是位于learning_topic的talker，节点名叫做person_publisher

后面跟这个output=“screen”，默认两个节点运行成功之后日志信息不会打印到终端里来，为了看到他们的输入效果，我们让他们在终端进行显示，就是让 output=“screen”。

```xml
<launch>
    <node pkg="learning_topic" type="person_subscriber" name="listener" output="screen" />
    <node pkg="learning_topic" type="person_publisher" name="talker" output="screen" /> 
</launch>
```

可以启动一下这个launch文件

还是回到项目根目录使用catkin_make进行编译

然后使用roslaunch 包名 文件名进行运行

```
cd ~/my_ws
catkin_make
roslaunch learning_launch simple.launch
```





**（3）用launch文件配置参数（turtlesim_parameter_config.launch）**

这里用到了param和rosparam的标签都是设置变量的

首先一开始设置了一个海龟的数量为2，我们会把这个变量及参数传输到参数服务器。
接下来我们运行功能包turtlesim下的节点turtlesim_node，就是海龟仿真器节点。

接下来利用param创建两个参数及值，海龟1的名字叫Tom，海龟2的名字叫Jerry

然后用rosparam去加载一个参数文件。

最后启动我们的键盘控制节点

```xml
<launch>

	<param name="/turtle_number"   value="2"/>

    <node pkg="turtlesim" type="turtlesim_node" name="turtlesim_node">
		<param name="turtle_name1"   value="Tom"/>
		<param name="turtle_name2"   value="Jerry"/>

		<rosparam file="$(find learning_launch)/config/param.yaml" command="load"/>
	</node>

    <node pkg="turtlesim" type="turtle_teleop_key" name="turtle_teleop_key" output="screen"/>

</launch>
```

建立一个config文件夹，在该文件夹里面再创建param.yaml文件

```
mkdir config
cd config
touch param.yaml
```

打开param.yaml，编写

```yaml
A: 123
B: "hello"

group:
  C: 456
  D: "hello"
```

可以新建一个终端，rosparam list可以看到设置的参数都在了，设置在node里面的前面就多一个命名空间，就是node的名字

![在这里插入图片描述](https://img-blog.csdnimg.cn/4496c8c69c8a4309bcf1e43a0221c0a5.png#pic_center)

还有yaml文件夹中的，我们在文件中设置了group的命名空间，但是由于这个rosparam在节点中，所以优先会加载节点中的命名空间。

![在这里插入图片描述](https://img-blog.csdnimg.cn/4c8b259138b7469591243aadb2334855.png#pic_center)







**（4）启动两个海龟的跟随文件的launch案例**

创建turtle_two.launch

```
touch turtle_two.launch
```

编写

```xml
<launch>

    <!-- Turtlesim Node-->
    <node pkg="turtlesim" type="turtlesim_node" name="sim"/>
    <node pkg="turtlesim" type="turtle_teleop_key" name="teleop" output="screen"/>

    <node pkg="learning_tf" type="turtle_tf_broadcaster" args="/turtle1" name="turtle1_tf_broadcaster" />
    <node pkg="learning_tf" type="turtle_tf_broadcaster" args="/turtle2" name="turtle2_tf_broadcaster" />

    <node pkg="learning_tf" type="turtle_tf_listener" name="listener" />
</launch>
```





**（5）remap标签和include标签的使用案例**

通过include使得这个launch文件可以包含并启动learning_launch包下的launch/simple.launch文件。

通过node标签启动一个仿真器节点。在启动海龟后我们不希望海龟输入指令的话题名叫/turtle1/cmd_vel，就可以通过remap直接改成/cmd_vel，然后/turtle1/cmd_vel就不复存在了

```xml
<launch>

    <include file="$(find learning_launch)/launch/simple.launch" />

    <node pkg="turtlesim" type="turtlesim_node" name="turtlesim_node">
        <remap from="/turtle1/cmd_vel" to="/cmd_vel"/>
    </node>

</launch>
```























# 常见可视化工具使用





## rqt可视化工具

ros有提供一个qt工具箱，都是基于qt开发的可视化工具，可以使用一系列rqt开头的命令行来启动这些qt工具。

之前使用过rqt_graph，这是一个计算图可视化工具，箭头代表话题的方向，椭圆代表节点。





### rqt_graph

rqt_graph 可以将我们当前运行话题topic与节点node的关系图形化显示出来，十分直观！

```
rqt_graph
```

![img](https://img-blog.csdnimg.cn/3411b5f9f1ce40ff87138b737e640757.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAUeWwj-mRqw==,size_20,color_FFFFFF,t_70,g_se,x_16)







### rqt_console（日志输出工具）



还是以小海龟为例

roscore + rosrun turtlesim turtlesim_node启动海龟节点

然后新建终端，rqt_console。

![在这里插入图片描述](https://img-blog.csdnimg.cn/9a9ec5764ffd4e6ebf3c493d32df00ac.png#pic_center)

上面的窗口使用来显示ros系统中的日志信息的，包括warn信息，infer，error等

下面就可以对上面的信息进行筛选

`rosrun turtlesim turtle_teleop_key`开启一个海龟的控制节点并且让海龟去撞墙。这个时候就会输出warn信息告诉我们海龟撞墙了

![在这里插入图片描述](https://img-blog.csdnimg.cn/52dc972787124308b79ce7648b45adce.png#pic_center)





### rqt_plot（数据绘图工具）

```
rqt_plot
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/3dd18aed2adb43af8c644e5e7bab871b.png#pic_center)

我们可以选择要绘制的数据，比如把/turtle1/pose数据绘制出来，那就在Topic上选择/turtle1/pose

![在这里插入图片描述](https://img-blog.csdnimg.cn/ab8a3c91188a4f5d80d9da45972f79b7.png#pic_center)













### rqt_image_view（图像渲染工具）

该工具可以显示图像信息，但是由于现在没有摄像头，当ros调用摄像头就会开始显示图像信息了

![在这里插入图片描述](https://img-blog.csdnimg.cn/c7f734eb2e5b4f90bb051b37fceb42dd.png#pic_center)















## Rviz可视化工具

机器人开发过程中的数据可视化界面，可以通过ros中的rviz工具显示点云、地图、模型、障碍物、图像等



**Rviz概述:**

Rviz是一款三维可视化工具，可以很好的兼容基于ROS软件框架的机器人平台。
在rviz中，可以使用可扩展标记语言XML对机器人、周围物体等任何实物进行尺寸、质量、位置、材质、关节等属性的描述，并且在界面中呈现出来。

同时，rviz还可以通过图形化的方式，实时显示机器人传感器的信息、机器人的运动状态、周围环境的变化等信息。

总而言之，rviz通过机器人模型参数、机器人发布的传感信息等数据，为用户进行所有可监测信息的图形化显示。用户和开发者也可以在rviz的控制界面下，通过按钮、滑动条、数值等方式，控制机器人的行为。




**Rviz运行区域描述:**
**0：**显示区，比如点云、模型、障碍物、图像等，都会显示在这个区域。

**1：**工具栏。可以选择Rviz的工具。

**2：**显示项列表。比如我们要显示一个图像，就需要add添加一个图像，设置话题等参数。

**3：**用来显示视角的，可以通过上面的选项调整观看模型的角度

**4：**时间县市区：主要显示ROS启动的时间以及系统本身默认的时间
![在这里插入图片描述](https://img-blog.csdnimg.cn/60e4238155b8499a8329410801c2bb09.png#pic_center)





**启动Rviz**

```
roscore
rosrun rviz rviz
```

















## Gazebo三维物理仿真平台

### Gazebo概述

Rviz是数据显示平台，显示的前提是有数据。Gazebo是数据仿真平台，就是本来没有数据去创建数据。

Gazebo是一款功能强大的三维物理仿真平台

 具备强大的物理引擎

 高质量的图形渲染

 方便的编程与图形接口

 开源免费

Gazebo典型的应用场景包括

 测试机器人算法

 机器人的设计

 显示情景下的回溯测试






### Gazebo工作区介绍

0：显示仿真出来的模型和传感器

1：基本的配置区，可以调整一些光线及模型的位姿

2和3：是中间显示的模型的列表以及属性，会有哪些模型或几个机器人。insert可以去插入我们模型库里的模型

4：和Rviz一样也是时间

![在这里插入图片描述](https://img-blog.csdnimg.cn/d81c5a3fb9d945fc818342f43348d118.png#pic_center)





### 运行Gazebo

运行资源会需求比较多，用虚拟机可能会运行不起来

```
roslaunch gazebo_ros willowgarage_world.launch
```













# ros添加配置文件yaml读取参数

[(13条消息) ros添加配置文件yaml读取参数_ros读取yaml参数_渡之的博客-CSDN博客](https://blog.csdn.net/qq_42237381/article/details/118933993?ops_request_misc=%7B%22request%5Fid%22%3A%22168614888516800197014924%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=168614888516800197014924&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-118933993-null-null.142^v88^control_2,239^v2^insert_chatgpt&utm_term=ros yaml文件&spm=1018.2226.3001.4187)



## 下载yaml-cpp并编译

[GitHub](https://so.csdn.net/so/search?q=GitHub&spm=1001.2101.3001.7020)上下载yaml-cpp的源码https://github.com/jbeder/yaml-cpp

```
git clone https://github.com/jbeder/yaml-cpp.git
```



然后进入目录创建build文件编译安装

```
mkdir build
cd build
cmake ..
make -j4
sudo make install
```



这时候在ros工程包中就可以包含yaml-cpp了

有了yaml库，在CMakeLists.txt中加入，

```
link_directories(/usr/local/lib)
include_directories(/usr/local/include/yaml-cpp)
```



然后在代码中添加头文件

```c++
#include <yaml-cpp/yaml.h>
```

这时候发现可以跳转到yaml了







## ros中添加yaml文件读取配置参数

在路径下新建一个.yaml文件

例如：

config.yaml

```yaml
#摄像头参数
cam:
 FRAME_WIDTH : 1920
 FRAME_HIGH : 1080
#16mm 1920*1080参数
 fc_x : 5586
 fc_y : 5586
 cc_x : 960
 cc_y : 540
```

新建launch文件，将参数文件配置到launch中

```xml
<?xml version="1.0"?>
<launch>
  <node pkg="test_yaml" name="test_yaml_node" type="test_yaml" output="screen"/>
  		　<rosparam file="$(find test_yaml)/launch/config.yaml"/>
</launch>
```

接着在代码中添加：

```c++
#include <ros/ros.h>
 
using namespace std;
 
//读取参数模板
template<typename T>
T getParam(const std::string& name,const T& defaultValue)//This name must be namespace+parameter_name
{
    T v;
    if(ros::param::get(name,v))//get parameter by name depend on ROS.
    {
        ROS_INFO_STREAM("Found parameter: "<<name<<",\tvalue: "<<v);
        return v;
    }
    else 
        ROS_WARN_STREAM("Cannot find value for parameter: "<<name<<",\tassigning default: "<<defaultValue);
    return defaultValue;//if the parameter haven't been set,it's value will return defaultValue.
}

int main(int argc, char **argv)
{
    ros::init(argc,argv,"test_tf_node");
    ros::NodeHandle nh;
     int FRAME_WIDTH=0;
     double fc_x = 0.0f;
     getParam<int>("cam/FRAME_WIDTH",0);
      getParam<double>("cam/fc_x",0.0f);
         
 
return 1；
}
```

在编译运行就能获取到yaml中的参数了

![img](https://img-blog.csdnimg.cn/20210720145225764.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMjM3Mzgx,size_16,color_FFFFFF,t_70)





































