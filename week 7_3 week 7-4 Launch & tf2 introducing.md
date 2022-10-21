# Launch & tf2 introducing


## Creating a launch file


First, we will create a new directory to store my launch filesL
```python
mkdir launch
```
Inside this directory, we will create a new file ```turtlesim_mimic_launch.py``` and paste the code below:
```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='turtlesim',
            namespace='turtlesim1',
            executable='turtlesim_node',
            name='sim'
        ),
        Node(
            package='turtlesim',
            namespace='turtlesim2',
            executable='turtlesim_node',
            name='sim'
        ),
        Node(
            package='turtlesim',
            executable='mimic',
            name='mimic',
            remappings=[
                ('/input/pose', '/turtlesim1/turtle1/pose'),
                ('/output/cmd_vel', '/turtlesim2/turtle1/cmd_vel'),
            ]
        )
    ])
```
To run the launch file created above, we will enter into the directory we created earlier and run the following command:
```python
ros2 launch turtlesim_mimic_launch.py
```
It will open two Turtlesim windows and I can see INFO messages in the terminal about which nodes my launch file has started:
```python
[INFO] [launch]: Default logging verbosity is set to INFO
[INFO] [turtlesim_node-1]: process started with pid [11714]
[INFO] [turtlesim_node-2]: process started with pid [11715]
[INFO] [mimic-3]: process started with pid [11716]
```
for seing the system in action, we will open a new terminal and run the command below:
```python
ros2 topic pub -r 1 /turtlesim1/turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: -1.8}}"
```
Now, we can see that both turtles follow the same path.

<br/>

To see the system in a graph, we will open new terminal and run the following code:
```
rqt_graph
```
The final result will be as below: 
![ssdd](https://user-images.githubusercontent.com/90182787/197158266-ce9f46bf-f181-4235-9575-eda0c2bb7dea.jpg)



<br/>
<br/>

## 2. Integrating launch files into ROS 2 packages


First, we will create a workspace:
```python
mkdir -p launch_ws/src
cd launch_ws/src
```
Then, we will create the new package:
```python
ros2 pkg create py_launch_example --build-type ament_python
```
Inside of this package, I will create a new directory called ```launch```. Then, I will open the ```setup.py``` file and make changes to it according to the code lines below:
```python
import os
from glob import glob
from setuptools import setup

package_name = 'py_launch_example'

setup(
    # Other parameters ...
    data_files=[
        # ... Other data files
        # Include all launch files.
        (os.path.join('share', package_name), glob('launch/*launch.[pxy][yma]*'))
    ]
)
```
Then, inside of my newly-created ```launch``` directory, we will create a new file called ```my_script_launch.py``` and paste the following code in:
```python
import launch
import launch_ros.actions

def generate_launch_description():
    return launch.LaunchDescription([
        launch_ros.actions.Node(
            package='demo_nodes_cpp',
            executable='talker',
            name='talker'),
  ])
```
Then, we will navigate to the root of my workspace and build the package as well as source it:
```python
colcon build
```
```python
. install/setup.bash
```
After being successful in building, we can run the launch file as below:
```python
ros2 launch py_launch_example my_script_launch.py
```





## 3. Using substitutions


At first, we will create a new package called __launch_tutorial__:
```python
ros2 pkg create launch_tutorial --build-type ament_python
```
Inside of this package, we will create a new folder __launch__:
```python
mkdir launch_tutorial/launch
```
Now, I will make some changes to the ```setup.py``` file in the package as below:
```python
import os
from glob import glob
from setuptools import setup

package_name = 'launch_tutorial'

setup(
    # Other parameters ...
    data_files=[
        # ... Other data files
        # Include all launch files.
        (os.path.join('share', package_name), glob('launch/*launch.[pxy][yma]*'))
    ]
)
```
Inside of the ```launch``` folder, I will create a launch file called ```example_main.launch.py``` and paste the code below:
```python
from launch_ros.substitutions import FindPackageShare

from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.substitutions import PathJoinSubstitution, TextSubstitution


def generate_launch_description():
    colors = {
        'background_r': '200'
    }

    return LaunchDescription([
        IncludeLaunchDescription(
            PythonLaunchDescriptionSource([
                PathJoinSubstitution([
                    FindPackageShare('launch_tutorial'),
                    'example_substitutions.launch.py'
                ])
            ]),
            launch_arguments={
                'turtlesim_ns': 'turtlesim2',
                'use_provided_red': 'True',
                'new_background_r': TextSubstitution(text=str(colors['background_r']))
            }.items()
        )
    ])
```
Then in the same folder, I will create a new file called __example_substitutions.launch.py__ and paste the folowing code:
```python
from launch_ros.actions import Node

from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, ExecuteProcess, TimerAction
from launch.conditions import IfCondition
from launch.substitutions import LaunchConfiguration, PythonExpression


def generate_launch_description():
    turtlesim_ns = LaunchConfiguration('turtlesim_ns')
    use_provided_red = LaunchConfiguration('use_provided_red')
    new_background_r = LaunchConfiguration('new_background_r')

    turtlesim_ns_launch_arg = DeclareLaunchArgument(
        'turtlesim_ns',
        default_value='turtlesim1'
    )
    use_provided_red_launch_arg = DeclareLaunchArgument(
        'use_provided_red',
        default_value='False'
    )
    new_background_r_launch_arg = DeclareLaunchArgument(
        'new_background_r',
        default_value='200'
    )

    turtlesim_node = Node(
        package='turtlesim',
        namespace=turtlesim_ns,
        executable='turtlesim_node',
        name='sim'
    )
    spawn_turtle = ExecuteProcess(
        cmd=[[
            'ros2 service call ',
            turtlesim_ns,
            '/spawn ',
            'turtlesim/srv/Spawn ',
            '"{x: 2, y: 2, theta: 0.2}"'
        ]],
        shell=True
    )
    change_background_r = ExecuteProcess(
        cmd=[[
            'ros2 param set ',
            turtlesim_ns,
            '/sim background_r ',
            '120'
        ]],
        shell=True
    )
    change_background_r_conditioned = ExecuteProcess(
        condition=IfCondition(
            PythonExpression([
                new_background_r,
                ' == 200',
                ' and ',
                use_provided_red
            ])
        ),
        cmd=[[
            'ros2 param set ',
            turtlesim_ns,
            '/sim background_r ',
            new_background_r
        ]],
        shell=True
    )

    return LaunchDescription([
        turtlesim_ns_launch_arg,
        use_provided_red_launch_arg,
        new_background_r_launch_arg,
        turtlesim_node,
        spawn_turtle,
        change_background_r,
        TimerAction(
            period=2.0,
            actions=[change_background_r_conditioned],
        )
    ])
```
Now, it is time to build my package and source it:
```
colcon build
```
```
. install/setup.bash
```
After the building is successful, i can launch ```example_main.launch.py``` file:
```
ros2 launch launch_tutorial example_main.launch.py
```python
This will do the following: <br/> <br/>

1. Start a turtlesim node with a blue background<br/>
2. Spawn the second turtle<br/>
3. Change the color to purple<br/>
4. Change the color to pink after two seconds<br/>

![photo_2022-10-21_18-25-35](https://user-images.githubusercontent.com/90182787/197162559-ed1bd69a-5c01-49d4-8d62-f1781b5e312f.jpg)



In order to see the arguments, we can run the command below:
```python
ros2 launch launch_tutorial example_substitutions.launch.py --show-args
```
This will show us the arguments which may be given to the launch file and their default values:
```python
Arguments (pass arguments as '<name>:=<value>'):

    'turtlesim_ns':
        no description given
        (default: 'turtlesim1')

    'use_provided_red':
        no description given
        (default: 'False')

    'new_background_r':
        no description given
        (default: '200')
```
we can also pass the desired arguments to the launch file as follows:
```python
ros2 launch launch_tutorial example_substitutions.launch.py turtlesim_ns:='turtlesim3' use_provided_red:='True' new_background_r:=200
```

<br/>
<br/>



#  Introducing tf2


we will start by installing the demo package and its dependencies:
```python
sudo apt-get install ros-humble-turtle-tf2-py ros-humble-tf2-tools ros-humble-tf-transformations
```
Now that we installed the ```turtle_tf2_py``` tutorial package let’s run the demo. First, I will open a new terminal and source my ROS 2 installation. Then I will run the following command:
```python
ros2 launch turtle_tf2_py turtle_tf2_demo.launch.py
```
As an output, we can see two turtles in the turtlesim:<br/>
we will open another terminal and run the following code to control the firs tturtle:
```python
ros2 run turtlesim turtle_teleop_key
```
If I move the first turtle, the second one follows continuously. Now I will use view_frames to create a diagram of the frames being broadcasted by tf2 over ROS:
```
ros2 run tf2_tools view_frames
```
The output will be:
```python
Listening to tf data during 5 seconds...
Generating graph in frames.pdf file...
```
Now, we will try to use tf2_echo which reports the transform between any two frames broadcasted over ROS.
```python
ros2 run tf2_ros tf2_echo turtle2 turtle1
```
we can see the transform displayed as the tf2_echo listener receives the frames broadcasted over ROS2:<br/>
![1](https://user-images.githubusercontent.com/90182787/197163796-b91a1e5d-8d0e-4e1d-a3d4-07a868be8c0e.jpg)
![2](https://user-images.githubusercontent.com/90182787/197163819-19b21f12-30df-4e66-b9bd-acbaf46acb8a.jpg)
![3](https://user-images.githubusercontent.com/90182787/197163834-7113e58c-3287-401a-96a9-6dcdd751858b.jpg)





Now, we will use ```rviz```. ```rviz``` is a visualization tool that is useful for examining tf2 frames:
```python
ros2 run rviz2 rviz2 -d $(ros2 pkg prefix --share turtle_tf2_py)/rviz/turtle_rviz.rviz
```
