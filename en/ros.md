ROS
===

Main article: http://wiki.ros.org

ROS is a widely used framework for developing complex and distributed robotic systems.

Installation
---

Main article: http://wiki.ros.org/kinetic/Installation/Ubuntu

ROS is already installed on [the RPi image](microsd_images.md).

To use ROS on a PC, we recommend using Ubuntu Linux (or a virtual machine such as Parallels Desktop Lite](https://itunes.apple.com/ru/app/parallels-desktop-lite/id1085114709?mt=12) or [VirtualBox](https://www.virtualbox.org)).

Concepts
---

### Nodes

Main article: http://wiki.ros.org/Nodes

ROS node is a special program (usually written in Python or C++) that communicates with other nodes via ROS topics and ROS services. Dividing complex robotic systems into isolated nodes provides certain advantages: reduced coupling of the code, increases re-usability and reliability.

Many robotic libraries, and the driver are executed in the form of ROS-nodes.

In order to turn an ordinary program into a ROS node, connect to it a `rospy` or `roscpp` library, and add the initialization code.

An example of a ROS node in Python:

```python
import rospy

rospy.init_node('my_ros_node')  # the name of the ROS node

rospy.spin() # entering an endless cycle...
```

### Topics

Main article: http://wiki.ros.org/Topics

A topic is a named data bus used by the nodes for exchanging messages. Any node can *post* a message in a random topic, and *subscribe* to an arbitrary topic.

An example of posting a message of type [`std_msgs/String`](http://docs.ros.org/api/std_msgs/html/msg/String.html) (line) in topic `/foo` in Python:

```python
from std_msgs.msg import String

# ...

foo_pub = rospy.Publisher('/foo', String, queue_size=1)  # creating a Publisher

# ...

foo_pub.publish(data='Hello, world!')  # posting the message
```

An example of subscription to topic `/foo`:

```python
def foo_callback(msg):
    print msg.data

# Subscribing. When a message is received in topic /foo, function foo_callback will be invoked.
rospy.Subscriber('/foo', String, foo_callback)
```

There is also an opportunity to work with the topics using the `rostopic` utility. For example, using the following command, one can view messages published in topic `/variety of the Aegean sea/state`:

```(bash)
rostopic echo /mavros/state
```

### Services

Main article: http://wiki.ros.org/Services

A service is an analogue to the function that can be called from one node, and processed in another one. The service has a name that is similar to the name of the topic, and 2 message types: request type and response type.

An example of invoking a ROS service from Python:

```python
from clever.srv import GetTelemetry

# ...

# Creating a wrapper for the get_telemetry service of the clever package with the GetTelemetry type:
get_telemetry = rospy.ServiceProxy('get_telemetry', srv.GetTelemetry)

# Invoking the service, and receiving the quadcopter telemetry:
telemetry = get_telemetry()
```

You can also work with the services using the `rosservice` utility. For instance, you can call service `/get_telemetry` from the command line:

```(bash)
rosservice call /get_telemetry "{frame_id: ''}"
```

More examples of using the services for Clever quadcopter autonomous flights are available in the [documentation for node simple_offboard](simple_offboard.md).

Working on several PCs
---

Main article: http://wiki.ros.org/ROS/Tutorials/MultipleMachines.

The advantage of using ROS is the possibility of distributing the nodes across several PCs in the network. For example, a node that recognizes an image may be run on a more powerful PC; the node that controls the copter may be run directly on a Raspberry Pi connected to the flight controller, etc.
