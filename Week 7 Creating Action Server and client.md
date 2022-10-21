# Creating Action Server and Client
<br/>

## Step 1: Creating an action
<br/>

First, we navigate to our workspace __ros2_ws/src__ and create a package with the name of ```action_tutorials_interfaces```:
```pyhton
ros2 pkg create action_tutorials_interfaces
```
Then we navigate to the __action_tutorials_interfaces__ and create a new folder called __action__

```python
mkdir action
```

Inside of this folder, we wcreate a new file called __Fibonacci.action__:

```python
gedit Fibonacci.action
```

Then input the following code inside of it:
```python
int32 order
---
int32[] sequence
---
int32[] partial_sequence
```
Then, we go back to the __action_tutorials_interfaces__ and open the __CMakeLists.txt__ file to add the following lines of code before the ament_package() line:
```python
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "action/Fibonacci.action"
)
```

