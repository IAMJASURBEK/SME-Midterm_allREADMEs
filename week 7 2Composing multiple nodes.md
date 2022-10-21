# Composing multiple nodes in a single process


First, we will check what kind of components available in my workspace:
```python
ros2 component types
```
After confirming the list of available components in my workspace, we will start a component container:
```python
ros2 run rclcpp_components component_container
```
Then we open another terminal and check the container is running:
```python
ros2 component list
```
This command gives me an output of __/ComponentManager__ which is the name of the current component. Then, I will load the talker component in the second terminal that I have opened:
```python
ros2 component load /ComponentManager composition composition::Talker
```
we can see the Talker component has been loaded with a unique ID:
```python
Loaded component 1 into '/ComponentManager' container node as '/talker'
```
In the second terminal, we will load the listener component too:
```python
ros2 component load /ComponentManager composition composition::Listener
```
... and, as an output, we can see that Listener component was loaded as a component 2:
```python
Loaded component 2 into '/ComponentManager' container node as '/listener'
```
To see the state of my container, we can run the below code below:
```python
ros2 component list
```
... which gives me an output of:
```python
/ComponentManager
   1  /talker
   2  /listener
```
we will run the below command in the first shell:
```python
ros2 run rclcpp_components component_container
```
In the second terminal, we will run the code below to execute the loaded components:
```python
ros2 component load /ComponentManager composition composition::Server
ros2 component load /ComponentManager composition composition::Client
```
The output of the command will be as below:
![cn1](https://user-images.githubusercontent.com/90182787/197153535-e129e975-c6a4-497c-a0d6-cf9916d578e2.jpg)









<br/>

we can also combine the couple components together by running:
```python
ros2 run composition manual_composition
```
To open each component library and create one instance of each node, I can run the below command:
```
ros2 run composition dlopen_composition `ros2 pkg prefix composition`/lib/libtalker_component.so `ros2 pkg prefix composition`/lib/liblistener_component.so
```
This will return a repeated output for each sent and received message.
<br/>

I can also use:
```
ros2 launch composition composition_demo.launch.py
```
to compose and start the components at the same time.

<br/>

Now, I will try to unload the components from the container. To do that, I will run the command below:
```
ros2 run rclcpp_components component_container
```
Then, I will verify the container is running:
```
ros2 component list
```
After verifying, I will run load listener and talker components in the second terminal:
```
ros2 component load /ComponentManager composition composition::Talker
ros2 component load /ComponentManager composition composition::Listener
```
After that, I will unload the node from the component container using a unique ID:
```
ros2 component unload /ComponentManager 1 2
```
My terminal will return:
```
Unloaded component 1 from '/ComponentManager' container
Unloaded component 2 from '/ComponentManager' container
```
Now, I can see that, the messages from talker and listener have stopped in the forst terminal.

<br/>

It is possible to remap container name and namespace using standard command line arguments:
```python
ros2 run rclcpp_components component_container --ros-args -r __node:=MyContainer -r __ns:=/ns
```
Now, in the second shell, I can load the components with the updated container name:
```python
ros2 component load /ns/MyContainer composition composition::Listener
```
I can also adjust the names and namespaces of the components through arguments. To do that, I will run the component container in the first terminal:
```python
ros2 run rclcpp_components component_container
```
In the second terminal I can remap: <br/>
Node name:
```python
ros2 component load /ComponentManager composition composition::Talker --node-name talker2
```
Namespace:
```python
ros2 component load /ComponentManager composition composition::Talker --node-namespace /ns
```
Both:
```python
ros2 component load /ComponentManager composition composition::Talker --node-name talker3 --node-namespace /ns2
```
To see the entries, I can use ```ros2``` command line:
```python
ros2 component list
```
 which returns me the entries as below:
```python
/ComponentManager
   1  /talker2
   2  /ns/talker
   3  /ns2/talker3
```
we can also pass additional arguments into components as foolows:
```python
ros2 component load /ComponentManager composition composition::Talker -e use_intra_process_comms:=true
```

<br/>
![cn2](https://user-images.githubusercontent.com/90182787/197153979-dbdbd8ae-5431-46f3-97eb-1aebd90a4dd0.jpg)



