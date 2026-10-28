# Robot Operating System Lab Manual 
@sathwikshetty0

## Course Overview

This manual covers all 10 experiments of Robot Operating System Lab (B24RA583), Semester VI, Department of Robotics and Automation, Rajarajeswari College of Engineering (Autonomous, VTU Belagavi). Every experiment uses ROS 2 Humble Hawksbill on Ubuntu 22.04 and follows the same layout: aim, theory, requirements, procedure, full code, expected output, result and viva questions.

**Simulation only:** no robot, sensor or microcontroller hardware is needed. Experiments 1 to 4 run as plain ROS 2 nodes, Experiments 5, 6 and 8 use Turtlesim, Experiment 7 uses RViz2, and Experiments 9 and 10 use a simulated robot with simulated LiDAR, camera and IMU in Gazebo. The only requirement is a PC or VM with Ubuntu 22.04 (4 GB RAM minimum; 8 GB and 3D acceleration recommended for Gazebo).

| Item | Detail |
| --- | --- |
| Course code | B24RA583 |
| Category | AEC-V |
| Teaching hours (L:T:P:S) | 0:0:2:0 |
| Total hours | 30 |
| Credits | 1 |
| CIE marks | 50 |
| SEE marks | 50 |
| Total marks | 100 |
| SEE duration | 3 hours |

### Course Learning Objectives

1. To introduce the architecture and core concepts of ROS 2.
2. To develop modular robotic applications using ROS 2 communication mechanisms.
3. To enable simulation-based robot control and visualization.
4. To provide hands-on experience in integrating ROS 2 components for autonomous robotic applications.

### Course Outcomes

| CO | Outcome | Experiments |
| --- | --- | --- |
| CO1 | Understand the architecture and environment setup of ROS 2 | 1, 2 |
| CO2 | Implement inter-node communication using ROS 2 topics, services and actions | 3, 4 |
| CO3 | Control and configure simulated robots using ROS 2 tools | 5, 6 |
| CO4 | Model and visualize robots using ROS 2 | 7, 8 |
| CO5 | Integrate ROS 2 components to implement basic autonomous robotic systems | 9, 10 |

The experiment-to-CO mapping above is a suggested one based on the topic of each experiment; confirm it with your faculty.

### Assessment Details

- CIE is 50% and SEE is 50% of the total.
- Minimum passing mark for CIE is 40% of 50 (20 marks); for SEE it is 35% of 50 (18 marks).
- A student must score at least 40% (40 out of 100) in the combined CIE and SEE total to earn the credits.

### Lab Rules

- Save every program inside your own workspace folder (`~/ros2_ws`); never edit system folders.
- Source ROS 2 in every new terminal before running any command.
- Close all nodes with `Ctrl+C` before shutting the system down.
- Write the aim, procedure, code, output and result in your record before the next lab.

## Pre-Lab Setup and Conventions

All experiments assume Ubuntu 22.04 LTS (native, dual boot, or a VM with at least 4 GB RAM and 40 GB disk) and ROS 2 Humble. Humble is the LTS release that matches Ubuntu 22.04, so use no other combination.

### Workspace layout used throughout

```bash
~/ros2_ws/
  src/        # your packages (source code)
  build/      # created by colcon
  install/    # created by colcon; contains setup.bash
  log/        # build and run logs
```

### Commands you will repeat in every experiment

```bash
source /opt/ros/humble/setup.bash     # underlay: ROS 2 itself
cd ~/ros2_ws
colcon build --symlink-install        # build all packages
source install/setup.bash             # overlay: your packages
```

To avoid typing the first line each time, add it to `~/.bashrc`:

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

### Packages needed across the lab

```bash
sudo apt update
sudo apt install -y ros-humble-desktop \
  ros-humble-turtlesim ros-humble-xacro \
  ros-humble-joint-state-publisher-gui ros-humble-robot-state-publisher \
  ros-humble-tf2-tools ros-humble-tf2-ros ros-humble-tf-transformations \
  ros-humble-gazebo-ros-pkgs ros-humble-gazebo-ros2-control \
  ros-humble-demo-nodes-cpp ros-humble-demo-nodes-py \
  python3-colcon-common-extensions python3-rosdep python3-argcomplete
```

### Common mistakes

| Symptom | Cause | Fix |
| --- | --- | --- |
| `ros2: command not found` | Underlay not sourced | `source /opt/ros/humble/setup.bash` |
| `Package 'x' not found` | Overlay not sourced after build | `source ~/ros2_ws/install/setup.bash` |
| Nodes in two terminals cannot see each other | Different `ROS_DOMAIN_ID` | Use the same value (`export ROS_DOMAIN_ID=0`) |
| `colcon build` fails with missing dependency | Dependency not in `package.xml` or not installed | Add `<depend>` and run `rosdep install` |
| Python edit has no effect | Built without `--symlink-install` | Rebuild with the flag |

Code blocks in this manual use `Ctrl+Shift+T` as the shortcut for opening a new terminal tab; each terminal is labelled Terminal 1, Terminal 2 and so on.

## Experiment 1: Installation and Verification of ROS 2 Humble and Basic ROS Commands

**Aim:** To install ROS 2 Humble on Ubuntu 22.04, verify the installation, and execute the basic ROS 2 command-line tools.

**CO mapped:** CO1

### Theory

ROS 2 is a middleware framework for robot software. A robot application is split into **nodes**, small processes that each do one job. Nodes exchange data through:

- **Topics:** one-way, many-to-many streams (publish/subscribe), such as sensor data.
- **Services:** two-way request/response calls, such as "reset the map".
- **Actions:** long-running goals with feedback and cancel, such as "navigate to a point".
- **Parameters:** named settings that belong to a node.

Unlike ROS 1, ROS 2 has no central `roscore`. Nodes find each other automatically using DDS (Data Distribution Service), and the `ROS_DOMAIN_ID` separates independent groups of nodes on one network.

### Requirements

Ubuntu 22.04 with internet access, 4 GB RAM, 40 GB disk, and sudo rights.

### Procedure

**Step 1: Set the locale (UTF-8 required)**

```bash
sudo apt update && sudo apt install -y locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

**Step 2: Enable the Ubuntu Universe repository**

```bash
sudo apt install -y software-properties-common
sudo add-apt-repository universe
```

**Step 3: Add the ROS 2 apt repository**

```bash
sudo apt update && sudo apt install -y curl
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

**Step 4: Install ROS 2 Humble and development tools**

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y ros-humble-desktop ros-dev-tools
```

`ros-humble-desktop` includes ROS, RViz2, demos and tutorials. `ros-dev-tools` adds colcon and rosdep.

**Step 5: Source the environment**

```bash
source /opt/ros/humble/setup.bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

**Step 6: Verify with the talker and listener demo**

Terminal 1:

```bash
ros2 run demo_nodes_cpp talker
```

Terminal 2:

```bash
ros2 run demo_nodes_py listener
```

**Step 7: Execute basic ROS 2 commands**

Keep the talker running and use a third terminal.

```bash
printenv | grep -i ROS               # check ROS_DISTRO=humble
ros2 --help                          # list all subcommands
ros2 node list                       # running nodes
ros2 node info /talker               # publishers, subscribers, services of a node
ros2 topic list -t                   # topics with message types
ros2 topic echo /chatter             # print messages
ros2 topic info /chatter             # type, publisher count, subscriber count
ros2 topic hz /chatter               # publishing rate
ros2 interface show std_msgs/msg/String   # message definition
ros2 service list                    # services
ros2 param list                      # parameters
ros2 pkg list | head                 # installed packages
ros2 pkg executables turtlesim       # executables in a package
rqt_graph                            # graph of nodes and topics
```

**Step 8: Try turtlesim**

```bash
ros2 run turtlesim turtlesim_node
ros2 run turtlesim turtle_teleop_key
```

Press the arrow keys in the teleop terminal and watch the turtle move.

### Where, When and Why (Experiment 1)

| Step | Where | When | Why |
| --- | --- | --- | --- |
| 1. Locale | Any terminal | First, before any ROS install | ROS 2 needs a UTF-8 locale; otherwise some tools fail with encoding errors |
| 2. Universe repo | Any terminal | Before adding the ROS repo | ROS packages depend on libraries stored in Ubuntu's Universe repository |
| 3. ROS apt repo | Any terminal (sudo) | After Step 2 | Ubuntu does not ship ROS 2; this tells `apt` where to download it and verifies it with a key |
| 4. Install | Any terminal (sudo) | After `apt update` | `ros-humble-desktop` gives the nodes, RViz2 and demos; `ros-dev-tools` gives colcon and rosdep used in all later experiments |
| 5. Source | Every new terminal (or once in `~/.bashrc`) | Before every ROS command | Sourcing adds ROS commands and libraries to that terminal's paths; without it you get `ros2: command not found` |
| 6. Talker/listener | Two separate terminals | After sourcing | Proves that nodes install, start and discover each other over DDS |
| 7. Basic commands | A third terminal, while the talker runs | While nodes are running | Introspection commands only show something while nodes are alive |
| 8. Turtlesim | Two terminals | Last | A visual check that a node can be driven by keyboard commands (topics at work) |

### Expected Output

- `printenv | grep -i ROS` shows `ROS_DISTRO=humble` and `ROS_VERSION=2`.
- The talker prints `Publishing: 'Hello World: 1'`, counting upward.
- The listener prints `I heard: [Hello World: 1]` with matching numbers.
- `ros2 topic list` shows `/chatter`, `/parameter_events` and `/rosout`.
- `ros2 topic hz /chatter` reports about 1 Hz.

### Result

ROS 2 Humble was installed and verified. Nodes were run, and topics, nodes, services, parameters and interfaces were inspected from the command line.

### Viva Questions

1. Why does ROS 2 not need `roscore`?
2. What is the difference between a topic, a service and an action?
3. What is DDS and what is `ROS_DOMAIN_ID` for?
4. What do the underlay and overlay mean in ROS 2?
5. Which command shows the message type of a topic?

## Experiment 2: Creation and Building of ROS 2 Packages Using Colcon

**Aim:** To create a ROS 2 workspace, build C++ (ament\_cmake) and Python (ament\_python) packages with colcon, and run a node from each.

**CO mapped:** CO1

### Theory

A **package** is the smallest unit of ROS 2 software: code, a `package.xml` (name, version, dependencies) and a build file. **colcon** is the build tool that builds every package in `src/` in dependency order. There are two build types:

| Build type | Language | Build file | Create with |
| --- | --- | --- | --- |
| `ament_cmake` | C++ | `CMakeLists.txt` | `--build-type ament_cmake` |
| `ament_python` | Python | `setup.py`, `setup.cfg` | `--build-type ament_python` |

After building, `install/setup.bash` adds the new packages to your environment (the overlay).

### Procedure

**Step 1: Create the workspace**

```bash
source /opt/ros/humble/setup.bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
```

**Step 2: Create a C++ package**

```bash
ros2 pkg create --build-type ament_cmake --node-name hello_cpp_node cpp_pkg --dependencies rclcpp
```

This generates `cpp_pkg/` with `src/hello_cpp_node.cpp`, `CMakeLists.txt` and `package.xml`.

**Step 3: Edit the C++ node** (`~/ros2_ws/src/cpp_pkg/src/hello_cpp_node.cpp`)

```cpp
#include "rclcpp/rclcpp.hpp"

int main(int argc, char **argv)
{
  rclcpp::init(argc, argv);
  auto node = rclcpp::Node::make_shared("hello_cpp_node");
  RCLCPP_INFO(node->get_logger(), "Hello from ROS 2 C++ node!");
  rclcpp::spin(node);
  rclcpp::shutdown();
  return 0;
}
```

**Step 4: Create a Python package**

```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python --node-name hello_py_node py_pkg --dependencies rclpy
```

**Step 5: Edit the Python node** (`~/ros2_ws/src/py_pkg/py_pkg/hello_py_node.py`)

```python
import rclpy
from rclpy.node import Node


class HelloNode(Node):
    def __init__(self):
        super().__init__('hello_py_node')
        self.get_logger().info('Hello from ROS 2 Python node!')


def main(args=None):
    rclpy.init(args=args)
    node = HelloNode()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

The entry point is already registered in `setup.py` by `--node-name`; confirm it reads `'hello_py_node = py_pkg.hello_py_node:main'` under `console_scripts`.

**Step 6: Install dependencies and build**

```bash
cd ~/ros2_ws
sudo rosdep init   # only if never run before
rosdep update
rosdep install -i --from-path src --rosdistro humble -y
colcon build --symlink-install
```

Build one package only:

```bash
colcon build --packages-select cpp_pkg
```

**Step 7: Source the overlay and run**

```bash
source install/setup.bash
ros2 run cpp_pkg hello_cpp_node
ros2 run py_pkg hello_py_node
ros2 pkg list | grep -E "cpp_pkg|py_pkg"
```

### Where, When and Why (Experiment 2)

| Step | Where | When | Why |
| --- | --- | --- | --- |
| 1. Workspace | Home folder: `~/ros2_ws/src` | Once per semester | colcon expects a fixed layout with all your packages under `src/` |
| 2. C++ package | `~/ros2_ws/src` | Before writing C++ code | `ros2 pkg create` makes the folder, `package.xml` and `CMakeLists.txt` so you do not write them by hand |
| 3. Edit C++ node | `cpp_pkg/src/hello_cpp_node.cpp` | After creating the package | Replaces the template message with your own so you can see your code ran |
| 4. Python package | `~/ros2_ws/src` | Same as above | Python packages use `setup.py` instead of CMake, and register nodes as `console_scripts` |
| 5. Edit Python node | `py_pkg/py_pkg/hello_py_node.py` | After creating the package | `rclpy.init`, a node class and `spin` are the skeleton of every Python node in this lab |
| 6. rosdep and build | `~/ros2_ws` (workspace root, never inside `src`) | After any code or dependency change | `rosdep` installs missing dependencies; `colcon build` compiles C++ and installs both packages into `install/` |
| 7. Source overlay and run | `~/ros2_ws`, in every new terminal | After every successful build | `install/setup.bash` makes `ros2 run` find your new packages |

### Expected Output

```text
Summary: 2 packages finished [xx.xs]
[INFO] [hello_cpp_node]: Hello from ROS 2 C++ node!
[INFO] [hello_py_node]: Hello from ROS 2 Python node!
```

The C++ template may print its own default message if you did not replace the generated file; after Step 3 it prints the line above.

### Result

A workspace with a C++ and a Python package was created, built with colcon, and a node from each package ran successfully.

### Viva Questions

1. What is the difference between `build/`, `install/` and `log/`?
2. What does `--symlink-install` do?
3. What is the role of `package.xml`?
4. Where do Python nodes get registered so `ros2 run` can find them?
5. Why must `install/setup.bash` be sourced after building?

## Experiment 3: Publisher-Subscriber Communication Using ROS 2 Topics

**Aim:** To implement a publisher node and a subscriber node that exchange messages over a ROS 2 topic, in Python and in C++.

**CO mapped:** CO2

### Theory

A **topic** is a named channel with a fixed message type. A **publisher** sends messages to the topic; any number of **subscribers** receive them. The two sides never talk directly and need not know about each other. Delivery behaviour is set by a **QoS** (Quality of Service) profile; the `10` used below is the queue depth (keep the last 10 messages) with default reliable delivery.

Only the topic name and message type must match between the two nodes.

### Part A: Python implementation

Create the package:

```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python py_pubsub --dependencies rclpy std_msgs
```

**Publisher:** `~/ros2_ws/src/py_pubsub/py_pubsub/publisher_node.py`

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String


class MinimalPublisher(Node):
    def __init__(self):
        super().__init__('minimal_publisher')
        self.publisher_ = self.create_publisher(String, 'topic', 10)
        self.timer = self.create_timer(0.5, self.timer_callback)  # 2 Hz
        self.i = 0

    def timer_callback(self):
        msg = String()
        msg.data = f'Hello ROS 2: {self.i}'
        self.publisher_.publish(msg)
        self.get_logger().info(f'Publishing: "{msg.data}"')
        self.i += 1


def main(args=None):
    rclpy.init(args=args)
    node = MinimalPublisher()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

**Subscriber:** `~/ros2_ws/src/py_pubsub/py_pubsub/subscriber_node.py`

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String


class MinimalSubscriber(Node):
    def __init__(self):
        super().__init__('minimal_subscriber')
        self.subscription = self.create_subscription(
            String, 'topic', self.listener_callback, 10)

    def listener_callback(self, msg):
        self.get_logger().info(f'I heard: "{msg.data}"')


def main(args=None):
    rclpy.init(args=args)
    node = MinimalSubscriber()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

Register both in `setup.py`:

```python
entry_points={
    'console_scripts': [
        'talker = py_pubsub.publisher_node:main',
        'listener = py_pubsub.subscriber_node:main',
    ],
},
```

Check that `package.xml` contains `<exec_depend>rclpy</exec_depend>` and `<exec_depend>std_msgs</exec_depend>`.

Build and run:

```bash
cd ~/ros2_ws
colcon build --packages-select py_pubsub --symlink-install
source install/setup.bash
ros2 run py_pubsub talker        # Terminal 1
ros2 run py_pubsub listener      # Terminal 2 (source setup.bash first)
```

### Part B: C++ implementation

```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_cmake cpp_pubsub --dependencies rclcpp std_msgs
```

**Publisher:** `cpp_pubsub/src/publisher_node.cpp`

```cpp
#include <chrono>
#include <memory>
#include <string>
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"

using namespace std::chrono_literals;

class MinimalPublisher : public rclcpp::Node
{
public:
  MinimalPublisher() : Node("minimal_publisher"), count_(0)
  {
    publisher_ = this->create_publisher<std_msgs::msg::String>("topic", 10);
    timer_ = this->create_wall_timer(
      500ms, std::bind(&MinimalPublisher::timer_callback, this));
  }

private:
  void timer_callback()
  {
    auto message = std_msgs::msg::String();
    message.data = "Hello ROS 2: " + std::to_string(count_++);
    RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());
    publisher_->publish(message);
  }
  rclcpp::TimerBase::SharedPtr timer_;
  rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
  size_t count_;
};

int main(int argc, char *argv[])
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<MinimalPublisher>());
  rclcpp::shutdown();
  return 0;
}
```

**Subscriber:** `cpp_pubsub/src/subscriber_node.cpp`

```cpp
#include <memory>
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"

using std::placeholders::_1;

class MinimalSubscriber : public rclcpp::Node
{
public:
  MinimalSubscriber() : Node("minimal_subscriber")
  {
    subscription_ = this->create_subscription<std_msgs::msg::String>(
      "topic", 10, std::bind(&MinimalSubscriber::topic_callback, this, _1));
  }

private:
  void topic_callback(const std_msgs::msg::String &msg) const
  {
    RCLCPP_INFO(this->get_logger(), "I heard: '%s'", msg.data.c_str());
  }
  rclcpp::Subscription<std_msgs::msg::String>::SharedPtr subscription_;
};

int main(int argc, char *argv[])
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<MinimalSubscriber>());
  rclcpp::shutdown();
  return 0;
}
```

Add to `CMakeLists.txt` (after `find_package` lines, before `ament_package()`):

```cmake
add_executable(talker src/publisher_node.cpp)
ament_target_dependencies(talker rclcpp std_msgs)

add_executable(listener src/subscriber_node.cpp)
ament_target_dependencies(listener rclcpp std_msgs)

install(TARGETS talker listener DESTINATION lib/${PROJECT_NAME})
```

Build and run:

```bash
cd ~/ros2_ws
colcon build --packages-select cpp_pubsub
source install/setup.bash
ros2 run cpp_pubsub talker
ros2 run cpp_pubsub listener
```

### Inspecting the communication

```bash
ros2 topic list -t
ros2 topic echo /topic
ros2 topic hz /topic
ros2 topic pub --once /topic std_msgs/msg/String "{data: 'manual message'}"
rqt_graph
```

### Where, When and Why (Experiment 3)

| Step | Where | When | Why |
| --- | --- | --- | --- |
| Create packages | `~/ros2_ws/src` | First | One package for Python and one for C++ so you can compare the two |
| Publisher file | `py_pubsub/py_pubsub/publisher_node.py` | After creating the package | A timer calls a function every 0.5 s; each call publishes one message, which is how sensors publish at a fixed rate |
| Subscriber file | `py_pubsub/py_pubsub/subscriber_node.py` | After the publisher | A callback runs automatically for every message that arrives |
| `setup.py` entry points | `py_pubsub/setup.py` | Before building | Without the `console_scripts` lines `ros2 run py_pubsub talker` cannot find the program |
| `CMakeLists.txt` lines | `cpp_pubsub/CMakeLists.txt`, before `ament_package()` | Before building | Tells CMake to compile each file and install the result where `ros2 run` looks |
| Build | `~/ros2_ws` | After every edit of C++, `setup.py` or `package.xml` | Python edits show without rebuilding only when `--symlink-install` was used |
| Run | Terminal 1 talker, Terminal 2 listener (both sourced) | After the build | Two terminals show two independent processes communicating |
| Inspect | Third terminal | While both run | `echo`, `hz` and `rqt_graph` confirm the data, rate and connections without touching the code |

### Expected Output

```text
# Terminal 1 (talker)
[INFO] [minimal_publisher]: Publishing: "Hello ROS 2: 0"
[INFO] [minimal_publisher]: Publishing: "Hello ROS 2: 1"

# Terminal 2 (listener)
[INFO] [minimal_subscriber]: I heard: "Hello ROS 2: 0"
[INFO] [minimal_subscriber]: I heard: "Hello ROS 2: 1"
```

The Python and C++ packages both define executables named `talker` and `listener`, but under different packages, so `ros2 run <package> <executable>` keeps them separate. `rqt_graph` shows `/minimal_publisher` connected to `/minimal_subscriber` through `/topic`.

### Result

A publisher and a subscriber were implemented in Python and C++, and messages were exchanged over a topic and verified with command-line tools.

### Viva Questions

1. Can one topic have several publishers and several subscribers?
2. What does the queue depth `10` mean?
3. What happens if publisher and subscriber use different message types on the same topic name?
4. How is a timer different from a subscription callback?
5. Why is `rclpy.spin()` needed?

## Experiment 4: Development of Service-Client and Action Server-Client Nodes

**Aim:** To develop a service server and client (request/response) and an action server and client (long task with feedback) in ROS 2.

**CO mapped:** CO2

### Theory

| Feature | Topic | Service | Action |
| --- | --- | --- | --- |
| Pattern | Publish/subscribe | Request/response | Goal, feedback, result |
| Direction | One-way | Two-way | Two-way with progress |
| Duration | Continuous | Short | Long-running |
| Cancel | No | No | Yes |
| Example | Camera images | Reset odometry | Navigate to a pose |

A service uses a `.srv` file with a request and a response separated by `---`. An action uses a `.action` file with goal, result and feedback separated by `---`. This experiment uses the ready-made `example_interfaces` package (`AddTwoInts` service and `Fibonacci` action) so no custom interface build is needed.

### Setup

```bash
sudo apt install -y ros-humble-example-interfaces
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python py_srvcli --dependencies rclpy example_interfaces
ros2 pkg create --build-type ament_cmake cpp_srvcli --dependencies rclcpp example_interfaces
ros2 pkg create --build-type ament_python py_action --dependencies rclpy example_interfaces
```

### Part A: Service in Python

**Server:** `py_srvcli/py_srvcli/service_member_function.py`

```python
from example_interfaces.srv import AddTwoInts
import rclpy
from rclpy.node import Node


class MinimalService(Node):
    def __init__(self):
        super().__init__('minimal_service')
        self.srv = self.create_service(AddTwoInts, 'add_two_ints', self.add_callback)

    def add_callback(self, request, response):
        response.sum = request.a + request.b
        self.get_logger().info(f'Incoming request: a={request.a} b={request.b}')
        return response


def main(args=None):
    rclpy.init(args=args)
    node = MinimalService()
    rclpy.spin(node)
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

**Client:** `py_srvcli/py_srvcli/client_member_function.py`

```python
import sys
from example_interfaces.srv import AddTwoInts
import rclpy
from rclpy.node import Node


class MinimalClient(Node):
    def __init__(self):
        super().__init__('minimal_client')
        self.cli = self.create_client(AddTwoInts, 'add_two_ints')
        while not self.cli.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('Service not available, waiting...')
        self.req = AddTwoInts.Request()

    def send_request(self, a, b):
        self.req.a = a
        self.req.b = b
        future = self.cli.call_async(self.req)
        rclpy.spin_until_future_complete(self, future)
        return future.result()


def main(args=None):
    rclpy.init(args=args)
    client = MinimalClient()
    response = client.send_request(int(sys.argv[1]), int(sys.argv[2]))
    client.get_logger().info(
        f'Result of add_two_ints: {sys.argv[1]} + {sys.argv[2]} = {response.sum}')
    client.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

In `setup.py` of `py_srvcli`:

```python
'console_scripts': [
    'service = py_srvcli.service_member_function:main',
    'client = py_srvcli.client_member_function:main',
],
```

Run:

```bash
cd ~/ros2_ws && colcon build --packages-select py_srvcli && source install/setup.bash
ros2 run py_srvcli service          # Terminal 1
ros2 run py_srvcli client 2 3       # Terminal 2
```

### Part B: Service in C++

**Server:** `cpp_srvcli/src/add_two_ints_server.cpp`

```cpp
#include <memory>
#include "rclcpp/rclcpp.hpp"
#include "example_interfaces/srv/add_two_ints.hpp"

void add(const std::shared_ptr<example_interfaces::srv::AddTwoInts::Request> request,
         std::shared_ptr<example_interfaces::srv::AddTwoInts::Response> response)
{
  response->sum = request->a + request->b;
  RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Incoming request: a=%ld b=%ld",
              request->a, request->b);
}

int main(int argc, char **argv)
{
  rclcpp::init(argc, argv);
  auto node = rclcpp::Node::make_shared("add_two_ints_server");
  auto service = node->create_service<example_interfaces::srv::AddTwoInts>(
    "add_two_ints", &add);
  RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Ready to add two ints.");
  rclcpp::spin(node);
  rclcpp::shutdown();
}
```

**Client:** `cpp_srvcli/src/add_two_ints_client.cpp`

```cpp
#include <chrono>
#include <cstdlib>
#include <memory>
#include "rclcpp/rclcpp.hpp"
#include "example_interfaces/srv/add_two_ints.hpp"

using namespace std::chrono_literals;

int main(int argc, char **argv)
{
  rclcpp::init(argc, argv);
  if (argc != 3) {
    RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "usage: add_two_ints_client X Y");
    return 1;
  }
  auto node = rclcpp::Node::make_shared("add_two_ints_client");
  auto client = node->create_client<example_interfaces::srv::AddTwoInts>("add_two_ints");

  auto request = std::make_shared<example_interfaces::srv::AddTwoInts::Request>();
  request->a = atoll(argv[1]);
  request->b = atoll(argv[2]);

  while (!client->wait_for_service(1s)) {
    if (!rclcpp::ok()) {
      RCLCPP_ERROR(rclcpp::get_logger("rclcpp"), "Interrupted while waiting.");
      return 0;
    }
    RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Service not available, waiting...");
  }

  auto result = client->async_send_request(request);
  if (rclcpp::spin_until_future_complete(node, result) ==
      rclcpp::FutureReturnCode::SUCCESS)
  {
    RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Sum: %ld", result.get()->sum);
  } else {
    RCLCPP_ERROR(rclcpp::get_logger("rclcpp"), "Failed to call service");
  }
  rclcpp::shutdown();
  return 0;
}
```

Add to `CMakeLists.txt` before `ament_package()`:

```cmake
add_executable(server src/add_two_ints_server.cpp)
ament_target_dependencies(server rclcpp example_interfaces)

add_executable(client src/add_two_ints_client.cpp)
ament_target_dependencies(client rclcpp example_interfaces)

install(TARGETS server client DESTINATION lib/${PROJECT_NAME})
```

Run: `ros2 run cpp_srvcli server` and `ros2 run cpp_srvcli client 5 7`.

Test from the command line without any client node:

```bash
ros2 service list -t
ros2 service call /add_two_ints example_interfaces/srv/AddTwoInts "{a: 4, b: 6}"
```

### Part C: Action Server and Client in Python

**Server:** `py_action/py_action/fibonacci_action_server.py`

```python
import time
import rclpy
from rclpy.action import ActionServer
from rclpy.node import Node
from example_interfaces.action import Fibonacci


class FibonacciActionServer(Node):
    def __init__(self):
        super().__init__('fibonacci_action_server')
        self._action_server = ActionServer(
            self, Fibonacci, 'fibonacci', self.execute_callback)

    def execute_callback(self, goal_handle):
        self.get_logger().info('Executing goal...')
        feedback_msg = Fibonacci.Feedback()
        feedback_msg.sequence = [0, 1]

        for i in range(1, goal_handle.request.order):
            feedback_msg.sequence.append(
                feedback_msg.sequence[i] + feedback_msg.sequence[i - 1])
            self.get_logger().info(f'Feedback: {feedback_msg.sequence}')
            goal_handle.publish_feedback(feedback_msg)
            time.sleep(1)

        goal_handle.succeed()
        result = Fibonacci.Result()
        result.sequence = feedback_msg.sequence
        return result


def main(args=None):
    rclpy.init(args=args)
    node = FibonacciActionServer()
    rclpy.spin(node)


if __name__ == '__main__':
    main()
```

**Client:** `py_action/py_action/fibonacci_action_client.py`

```python
import rclpy
from rclpy.action import ActionClient
from rclpy.node import Node
from example_interfaces.action import Fibonacci


class FibonacciActionClient(Node):
    def __init__(self):
        super().__init__('fibonacci_action_client')
        self._action_client = ActionClient(self, Fibonacci, 'fibonacci')

    def send_goal(self, order):
        goal_msg = Fibonacci.Goal()
        goal_msg.order = order
        self._action_client.wait_for_server()
        self._send_goal_future = self._action_client.send_goal_async(
            goal_msg, feedback_callback=self.feedback_callback)
        self._send_goal_future.add_done_callback(self.goal_response_callback)

    def goal_response_callback(self, future):
        goal_handle = future.result()
        if not goal_handle.accepted:
            self.get_logger().info('Goal rejected')
            return
        self.get_logger().info('Goal accepted')
        self._get_result_future = goal_handle.get_result_async()
        self._get_result_future.add_done_callback(self.get_result_callback)

    def get_result_callback(self, future):
        result = future.result().result
        self.get_logger().info(f'Result: {list(result.sequence)}')
        rclpy.shutdown()

    def feedback_callback(self, feedback_msg):
        self.get_logger().info(f'Received feedback: {list(feedback_msg.feedback.sequence)}')


def main(args=None):
    rclpy.init(args=args)
    node = FibonacciActionClient()
    node.send_goal(8)
    rclpy.spin(node)


if __name__ == '__main__':
    main()
```

In `setup.py` of `py_action`:

```python
'console_scripts': [
    'action_server = py_action.fibonacci_action_server:main',
    'action_client = py_action.fibonacci_action_client:main',
],
```

Run:

```bash
cd ~/ros2_ws && colcon build --packages-select py_action && source install/setup.bash
ros2 run py_action action_server    # Terminal 1
ros2 run py_action action_client    # Terminal 2
```

Inspect and send a goal from the command line:

```bash
ros2 action list -t
ros2 action info /fibonacci
ros2 action send_goal /fibonacci example_interfaces/action/Fibonacci "{order: 5}" --feedback
```

### Optional: define your own interface

Create `srv/AddThree.srv` in an `ament_cmake` package:

```text
int64 a
int64 b
int64 c
---
int64 sum
```

In `CMakeLists.txt` add `find_package(rosidl_default_generators REQUIRED)` and `rosidl_generate_interfaces(${PROJECT_NAME} "srv/AddThree.srv")`; in `package.xml` add `<buildtool_depend>rosidl_default_generators</buildtool_depend>`, `<exec_depend>rosidl_default_runtime</exec_depend>` and `<member_of_group>rosidl_interface_packages</member_of_group>`.

### Where, When and Why (Experiment 4)

| Step | Where | When | Why |
| --- | --- | --- | --- |
| Install `example_interfaces` | Any terminal (sudo) | Before creating packages | Provides ready-made `AddTwoInts` and `Fibonacci` definitions, so no custom interface build is needed |
| Service server | `py_srvcli/py_srvcli/service_member_function.py` | Start the server first | A server must be running before a client can call it |
| Service client | `py_srvcli/py_srvcli/client_member_function.py` | After the server | Sends one request and waits for the reply; `wait_for_service` prevents calling a server that is not up |
| C++ server/client | `cpp_srvcli/src/`, plus `CMakeLists.txt` | Same pattern | Shows that the Python and C++ nodes can talk to each other because both use the same interface |
| `ros2 service call` | Any sourced terminal | While the server runs | Tests a server without writing a client, which is the first step in debugging |
| Action server | `py_action/py_action/fibonacci_action_server.py` | Start before the client | Runs a long task, sends feedback each second, and returns the final result |
| Action client | `py_action/py_action/fibonacci_action_client.py` | After the server | Sends a goal, shows feedback, and receives the result using callbacks so it never blocks |
| `ros2 action send_goal --feedback` | Any sourced terminal | While the server runs | Lets you test an action and see feedback from the command line |
| Use service or action? | Design decision | Before writing a new node | Use a service for short requests; use an action when the task takes seconds or minutes and may need progress or cancel |

### Expected Output

```text
# Service
[INFO] [minimal_service]: Incoming request: a=2 b=3
[INFO] [minimal_client]: Result of add_two_ints: 2 + 3 = 5

# Action (order 8)
[INFO] [fibonacci_action_client]: Goal accepted
[INFO] [fibonacci_action_client]: Received feedback: [0, 1, 1]
...
[INFO] [fibonacci_action_client]: Result: [0, 1, 1, 2, 3, 5, 8, 13, 21]
```

### Result

Service and action nodes were developed and tested. The service returned a reply for each request; the action server reported progress through feedback and returned the final sequence.

### Viva Questions

1. When would you choose an action instead of a service?
2. What three parts does an action definition have?
3. Why is `call_async` preferred over a blocking call inside a node?
4. How can a running action goal be cancelled?
5. What does `ros2 service call` do and when is it useful?

## Experiment 5: Robot Motion Control Using Turtlesim and ROS 2 Topics

**Aim:** To control the motion of a simulated robot (turtle) by publishing velocity commands on a topic, and to build a closed-loop go-to-goal controller using pose feedback.

**CO mapped:** CO3

### Theory

Turtlesim is a 2D simulator whose robot is driven by velocity commands and reports its position. The interfaces are:

| Interface | Name | Type | Direction |
| --- | --- | --- | --- |
| Topic | `/turtle1/cmd_vel` | `geometry_msgs/msg/Twist` | Input to turtle |
| Topic | `/turtle1/pose` | `turtlesim/msg/Pose` | Output from turtle |
| Service | `/spawn`, `/kill`, `/clear`, `/reset` | turtlesim srv | Request/response |
| Service | `/turtle1/set_pen`, `/turtle1/teleport_absolute` | turtlesim srv | Request/response |

A `Twist` message has `linear.x` (forward speed, m/s) and `angular.z` (turning rate, rad/s). The turtle is a differential-drive style robot, so only these two fields matter.

The **go-to-goal** controller uses a proportional (P) law. With distance error and heading error:

```latex
d = \sqrt{(x_g - x)^2 + (y_g - y)^2}
\theta_g = \operatorname{atan2}(y_g - y,\; x_g - x)
v = K_v \, d \qquad \omega = K_\omega \, (\theta_g - \theta)
```

### Part A: Command-line control

```bash
ros2 run turtlesim turtlesim_node                       # Terminal 1
ros2 run turtlesim turtle_teleop_key                    # Terminal 2: arrow keys
```

Publish velocity manually from Terminal 3:

```bash
ros2 topic pub --rate 1 /turtle1/cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"

ros2 topic echo /turtle1/pose
ros2 service call /spawn turtlesim/srv/Spawn "{x: 2.0, y: 2.0, theta: 0.0, name: 'turtle2'}"
ros2 service call /clear std_srvs/srv/Empty
```

### Part B: Circle motion node (Python)

Create a package:

```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python turtle_control --dependencies rclpy geometry_msgs turtlesim
```

`turtle_control/turtle_control/draw_circle.py`

```python
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist


class DrawCircle(Node):
    def __init__(self):
        super().__init__('draw_circle')
        self.pub = self.create_publisher(Twist, '/turtle1/cmd_vel', 10)
        self.timer = self.create_timer(0.1, self.send_velocity)

    def send_velocity(self):
        msg = Twist()
        msg.linear.x = 2.0     # m/s
        msg.angular.z = 1.0    # rad/s -> radius = v / w = 2 units
        self.pub.publish(msg)


def main(args=None):
    rclpy.init(args=args)
    node = DrawCircle()
    rclpy.spin(node)
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### Part C: Go-to-goal controller with pose feedback

`turtle_control/turtle_control/go_to_goal.py`

```python
import math
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist
from turtlesim.msg import Pose


class GoToGoal(Node):
    def __init__(self):
        super().__init__('go_to_goal')
        self.declare_parameter('goal_x', 8.0)
        self.declare_parameter('goal_y', 8.0)
        self.declare_parameter('kv', 1.0)
        self.declare_parameter('kw', 4.0)
        self.declare_parameter('tolerance', 0.1)

        self.pose = None
        self.pub = self.create_publisher(Twist, '/turtle1/cmd_vel', 10)
        self.sub = self.create_subscription(Pose, '/turtle1/pose', self.pose_cb, 10)
        self.timer = self.create_timer(0.05, self.control_loop)

    def pose_cb(self, msg):
        self.pose = msg

    def control_loop(self):
        if self.pose is None:
            return
        gx = self.get_parameter('goal_x').value
        gy = self.get_parameter('goal_y').value
        kv = self.get_parameter('kv').value
        kw = self.get_parameter('kw').value
        tol = self.get_parameter('tolerance').value

        dx, dy = gx - self.pose.x, gy - self.pose.y
        distance = math.hypot(dx, dy)
        cmd = Twist()

        if distance > tol:
            heading_error = math.atan2(dy, dx) - self.pose.theta
            # wrap to [-pi, pi]
            heading_error = math.atan2(math.sin(heading_error), math.cos(heading_error))
            cmd.linear.x = kv * distance
            cmd.angular.z = kw * heading_error
        else:
            self.get_logger().info('Goal reached', once=True)

        self.pub.publish(cmd)


def main(args=None):
    rclpy.init(args=args)
    node = GoToGoal()
    rclpy.spin(node)
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

Register entry points in `setup.py`:

```python
'console_scripts': [
    'draw_circle = turtle_control.draw_circle:main',
    'go_to_goal = turtle_control.go_to_goal:main',
],
```

Build and run:

```bash
cd ~/ros2_ws && colcon build --packages-select turtle_control --symlink-install
source install/setup.bash
ros2 run turtlesim turtlesim_node            # Terminal 1
ros2 run turtle_control draw_circle          # Terminal 2 (Ctrl+C to stop)
ros2 service call /reset std_srvs/srv/Empty
ros2 run turtle_control go_to_goal --ros-args -p goal_x:=3.0 -p goal_y:=9.0
```

### Where, When and Why (Experiment 5)

| Step | Where | When | Why |
| --- | --- | --- | --- |
| Start `turtlesim_node` | Terminal 1, kept open all experiment | Always first | It is the simulated robot; nothing else can be tested without it |
| Teleop keys | Terminal 2 (click inside this terminal to type) | After the simulator | Shows manual control: key presses become `Twist` messages on `/turtle1/cmd_vel` |
| `ros2 topic pub` | Terminal 3 | After the simulator | Sends the same `Twist` by hand, so you see that any program can drive the robot by publishing |
| `ros2 topic echo /turtle1/pose` | Terminal 3 | While the turtle moves | The pose is the feedback signal used later by the controller |
| `draw_circle.py` | `turtle_control/turtle_control/` | After manual tests | Open-loop control: constant speeds give a circle of radius v/ω |
| `go_to_goal.py` | Same folder | After the circle | Closed-loop control: it reads the pose, computes the error and corrects it every 50 ms, so it reaches a goal even from a different start |
| `/reset` service | Any terminal | Between runs | Puts the turtle back at its start so each test begins in the same state |
| Parameters `-p goal_x:=3.0` | At `ros2 run` | Per test | Changes the goal without editing code |

### Expected Output

- Part A: the turtle moves in a circle under the manual `topic pub`; `/turtle1/pose` prints `x`, `y`, `theta`, `linear_velocity` and `angular_velocity`.
- Part B: the turtle draws a circle of radius 2 units.
- Part C: the turtle turns toward the goal, moves along a smooth curve and stops within 0.1 units of it; changing `goal_x` and `goal_y` sends it to a new point.

### Result

Turtlesim was controlled from the command line and from nodes using `Twist` messages, and a closed-loop proportional controller drove the turtle to a chosen goal using pose feedback.

### Viva Questions

1. Which message type carries velocity commands and what are its two useful fields here?
2. What is the radius of the circle for given `v` and `ω`?
3. Why do we wrap the heading error to \[-π, π\]?
4. What happens if `kw` is set too high?
5. What is the difference between open-loop and closed-loop control?

## Experiment 6: Parameter Tuning and Execution of Multi-Node Systems Using Launch Files

**Aim:** To declare, read, change and save node parameters, and to start several nodes together with a Python launch file.

**CO mapped:** CO3

### Theory

**Parameters** are named, typed settings that belong to one node (integer, float, string, bool, lists). They let you tune behaviour such as gains and limits without recompiling. A node must `declare_parameter` before it can use one, and parameters can be set at start-up (command line, YAML file, launch file) or changed while the node runs (`ros2 param set`).

A **launch file** describes a whole system: which nodes to start, with which names, namespaces, parameters and topic remaps. `ros2 launch` starts everything with one command and stops it all with one `Ctrl+C`. Python launch files are the most flexible and are used here.

### Part A: Command-line parameter handling

With `turtlesim_node` running:

```bash
ros2 param list
ros2 param get /turtlesim background_r
ros2 param set /turtlesim background_r 150
ros2 param describe /turtlesim background_g
ros2 param dump /turtlesim > turtlesim.yaml
ros2 param load /turtlesim turtlesim.yaml
ros2 run turtlesim turtlesim_node --ros-args --params-file turtlesim.yaml
```

The background colour of the window changes as soon as `background_r` is set.

### Part B: Node with dynamic parameters

We reuse the `go_to_goal` node from Experiment 5 (it already declares `goal_x`, `goal_y`, `kv`, `kw`, `tolerance`) and add a callback so a change is checked and logged while running. Add this to the `__init__` of `GoToGoal` and import `SetParametersResult`:

```python
from rcl_interfaces.msg import SetParametersResult

# inside __init__, after declare_parameter calls
self.add_on_set_parameters_callback(self.param_callback)

def param_callback(self, params):
    for p in params:
        if p.name in ('kv', 'kw') and p.value < 0.0:
            return SetParametersResult(successful=False, reason='gain must be >= 0')
        self.get_logger().info(f'Parameter {p.name} set to {p.value}')
    return SetParametersResult(successful=True)
```

Test while the node runs:

```bash
ros2 param set /go_to_goal kw 2.0       # accepted
ros2 param set /go_to_goal kv -1.0      # rejected
ros2 param set /go_to_goal goal_x 2.0   # turtle heads to the new goal
```

### Part C: Parameter YAML file

Create `turtle_control/config/goal_params.yaml`:

```yaml
go_to_goal:
  ros__parameters:
    goal_x: 2.0
    goal_y: 3.0
    kv: 1.2
    kw: 3.0
    tolerance: 0.15
```

The top-level key must be the node name.

### Part D: Launch files

Create the folders `turtle_control/launch/` and `turtle_control/config/`.

**Launch file 1: two simulators and a mimic node** (`launch/mimic_launch.py`). The second turtle copies the movement of the first.

```python
from launch import LaunchDescription
from launch_ros.actions import Node


def generate_launch_description():
    return LaunchDescription([
        Node(package='turtlesim', namespace='turtlesim1',
             executable='turtlesim_node', name='sim'),
        Node(package='turtlesim', namespace='turtlesim2',
             executable='turtlesim_node', name='sim'),
        Node(package='turtlesim', executable='mimic', name='mimic',
             remappings=[
                 ('/input/pose', '/turtlesim1/turtle1/pose'),
                 ('/output/cmd_vel', '/turtlesim2/turtle1/cmd_vel'),
             ]),
    ])
```

**Launch file 2: simulator and controller with parameters** (`launch/goal_launch.py`):

```python
import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration
from launch_ros.actions import Node
from launch_ros.parameter_descriptions import ParameterValue


def generate_launch_description():
    config = os.path.join(
        get_package_share_directory('turtle_control'), 'config', 'goal_params.yaml')

    goal_x = DeclareLaunchArgument('goal_x', default_value='2.0')

    return LaunchDescription([
        goal_x,
        Node(package='turtlesim', executable='turtlesim_node', name='sim',
             output='screen'),
        Node(package='turtle_control', executable='go_to_goal', name='go_to_goal',
             output='screen',
             parameters=[config,
                         {'goal_x': ParameterValue(LaunchConfiguration('goal_x'),
                                                   value_type=float)}]),
    ])
```

Install the new folders by editing `data_files` in `setup.py`:

```python
import os
from glob import glob

data_files=[
    ('share/ament_index/resource_index/packages', ['resource/' + package_name]),
    ('share/' + package_name, ['package.xml']),
    (os.path.join('share', package_name, 'launch'), glob('launch/*.py')),
    (os.path.join('share', package_name, 'config'), glob('config/*.yaml')),
],
```

Add `<exec_depend>ros2launch</exec_depend>` to `package.xml`, then build and launch:

```bash
cd ~/ros2_ws && colcon build --packages-select turtle_control --symlink-install
source install/setup.bash

ros2 launch turtle_control mimic_launch.py
# in another terminal
ros2 topic pub -r 1 /turtlesim1/turtle1/cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 2.0}, angular: {z: 1.8}}"

ros2 launch turtle_control goal_launch.py
ros2 launch turtle_control goal_launch.py goal_x:=9.0
```

Inspect the running system:

```bash
ros2 node list
ros2 param get /go_to_goal goal_x
rqt_graph
ros2 launch turtle_control goal_launch.py --show-args
```

### Where, When and Why (Experiment 6)

| Step | Where | When | Why |
| --- | --- | --- | --- |
| `ros2 param` commands | Any sourced terminal, with `turtlesim_node` running | First | Parameters exist only while their node runs; this shows how any node can be tuned live |
| `dump` and `load` | Folder where you want the YAML saved | After tuning | Saves a good set of values so you can reuse it instead of retyping |
| Parameter callback | Inside `go_to_goal.py`, in `__init__` | When you want to check values | Rejects invalid values (for example a negative gain) before they reach the controller |
| YAML file | `turtle_control/config/goal_params.yaml` | Before the launch file | Keeps all settings in one file; the first key must match the node name or the values are ignored |
| `mimic_launch.py` | `turtle_control/launch/` | After the YAML | One command starts three nodes; namespaces stop the two simulators from clashing on topic names, and remaps connect turtle 1's pose to turtle 2's command |
| `goal_launch.py` | `turtle_control/launch/` | After the mimic launch | Shows launch arguments (`goal_x:=9.0`) overriding a value from the YAML |
| `data_files` in `setup.py` | `turtle_control/setup.py` | Before building | Python packages install only what is listed; without this `ros2 launch` cannot find your launch and config files |
| Rebuild and source | `~/ros2_ws` | After every change to launch, config or setup files | Launch and config files are copied into `install/` at build time |

### Expected Output

- Launch 1 opens two turtle windows. Moving turtle 1 makes turtle 2 follow the same path.
- Launch 2 opens one window; the turtle moves to (2.0, 3.0) with the YAML gains, or to x = 9.0 when `goal_x:=9.0` is given.
- `ros2 param set /go_to_goal kv -1.0` returns `Setting parameter failed: gain must be >= 0`.

### Result

Parameters were read, set, saved and loaded, and multi-node systems with namespaces, remaps, YAML parameters and launch arguments were started from launch files.

### Viva Questions

1. Why must a parameter be declared before use in Humble?
2. What is the difference between a launch argument and a node parameter?
3. What do namespaces and remappings achieve?
4. How do you override one value from a YAML file at launch time?
5. Why must launch and config folders be listed in `data_files`?

## Experiment 7: Robot Modeling Using URDF and Xacro and Visualization in RViz2

**Aim:** To model a differential-drive mobile robot using URDF with Xacro macros and to visualize and move its joints in RViz2.

**CO mapped:** CO4

### Theory

**URDF** (Unified Robot Description Format) is an XML file that describes a robot as a tree of **links** (rigid bodies) connected by **joints**. Each link can have a `visual` (what you see), `collision` (what the physics engine uses) and `inertial` (mass and inertia) element. Each joint has a parent, a child, an origin (position and orientation relative to the parent) and a type.

| Joint type | Motion | Use |
| --- | --- | --- |
| `fixed` | None | Sensors, mounts |
| `continuous` | Unlimited rotation about an axis | Wheels |
| `revolute` | Rotation within limits | Arm joints |
| `prismatic` | Sliding within limits | Linear actuators |

**Xacro** (XML Macros) extends URDF with `xacro:property` (named constants), `${ }` math expressions, `xacro:macro` (reusable blocks) and `xacro:include` (split files). It removes copy-paste, for example one wheel macro is used for both wheels. The `xacro` tool expands it into plain URDF.

The `robot_state_publisher` node reads the URDF from the `robot_description` parameter and, using joint values from `/joint_states`, publishes the transform of every link on `/tf` and `/tf_static`. RViz2 draws the model from these transforms.

### Robot Dimensions

| Part | Shape | Size (m) | Joint |
| --- | --- | --- | --- |
| `base_footprint` | Frame on the ground | none | root |
| `base_link` | Box | 0.40 × 0.25 × 0.10 | fixed to footprint, z = 0.08 |
| `left_wheel_link`, `right_wheel_link` | Cylinder | r = 0.05, width 0.04 | continuous, axis y |
| `caster_link` | Sphere | r = 0.025 | fixed, at the rear |
| `lidar_link` | Cylinder | r = 0.04, h = 0.05 | fixed, at the front top |

### Procedure

**Step 1: Install tools and create the package**

```bash
sudo apt install -y ros-humble-xacro ros-humble-joint-state-publisher-gui \
  ros-humble-robot-state-publisher ros-humble-rviz2 liburdfdom-tools
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_cmake diff_bot_description
cd diff_bot_description
mkdir -p urdf launch rviz
```

**Step 2: Write the Xacro model** `urdf/diff_bot.urdf.xacro`

```xml
<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro" name="diff_bot">

  <!-- Parameters -->
  <xacro:property name="base_length"  value="0.40"/>
  <xacro:property name="base_width"   value="0.25"/>
  <xacro:property name="base_height"  value="0.10"/>
  <xacro:property name="base_z"       value="0.08"/>
  <xacro:property name="wheel_radius" value="0.05"/>
  <xacro:property name="wheel_width"  value="0.04"/>
  <xacro:property name="caster_radius" value="0.025"/>

  <!-- Materials -->
  <material name="blue"><color rgba="0.2 0.3 0.8 1.0"/></material>
  <material name="black"><color rgba="0.1 0.1 0.1 1.0"/></material>
  <material name="red"><color rgba="0.8 0.1 0.1 1.0"/></material>
  <material name="grey"><color rgba="0.6 0.6 0.6 1.0"/></material>

  <!-- Inertia macros -->
  <xacro:macro name="box_inertia" params="m x y z">
    <inertial>
      <mass value="${m}"/>
      <inertia ixx="${m*(y*y+z*z)/12}" ixy="0" ixz="0"
               iyy="${m*(x*x+z*z)/12}" iyz="0" izz="${m*(x*x+y*y)/12}"/>
    </inertial>
  </xacro:macro>

  <xacro:macro name="wheel_inertia" params="m r h">
    <inertial>
      <origin xyz="0 0 0" rpy="${pi/2} 0 0"/>
      <mass value="${m}"/>
      <inertia ixx="${m*(3*r*r+h*h)/12}" ixy="0" ixz="0"
               iyy="${m*(3*r*r+h*h)/12}" iyz="0" izz="${m*r*r/2}"/>
    </inertial>
  </xacro:macro>

  <!-- Root frame on the ground -->
  <link name="base_footprint"/>

  <!-- Chassis -->
  <link name="base_link">
    <visual>
      <geometry><box size="${base_length} ${base_width} ${base_height}"/></geometry>
      <material name="blue"/>
    </visual>
    <collision>
      <geometry><box size="${base_length} ${base_width} ${base_height}"/></geometry>
    </collision>
    <xacro:box_inertia m="5.0" x="${base_length}" y="${base_width}" z="${base_height}"/>
  </link>

  <joint name="base_joint" type="fixed">
    <parent link="base_footprint"/>
    <child link="base_link"/>
    <origin xyz="0 0 ${base_z}" rpy="0 0 0"/>
  </joint>

  <!-- Wheel macro: used twice -->
  <xacro:macro name="wheel" params="prefix y_reflect">
    <link name="${prefix}_wheel_link">
      <visual>
        <origin xyz="0 0 0" rpy="${pi/2} 0 0"/>
        <geometry><cylinder radius="${wheel_radius}" length="${wheel_width}"/></geometry>
        <material name="black"/>
      </visual>
      <collision>
        <origin xyz="0 0 0" rpy="${pi/2} 0 0"/>
        <geometry><cylinder radius="${wheel_radius}" length="${wheel_width}"/></geometry>
      </collision>
      <xacro:wheel_inertia m="0.5" r="${wheel_radius}" h="${wheel_width}"/>
    </link>
    <joint name="${prefix}_wheel_joint" type="continuous">
      <parent link="base_link"/>
      <child link="${prefix}_wheel_link"/>
      <origin xyz="0 ${y_reflect*(base_width/2 + wheel_width/2)} ${wheel_radius - base_z}"
              rpy="0 0 0"/>
      <axis xyz="0 1 0"/>
    </joint>
  </xacro:macro>

  <xacro:wheel prefix="left"  y_reflect="1"/>
  <xacro:wheel prefix="right" y_reflect="-1"/>

  <!-- Rear caster -->
  <link name="caster_link">
    <visual>
      <geometry><sphere radius="${caster_radius}"/></geometry>
      <material name="grey"/>
    </visual>
    <collision>
      <geometry><sphere radius="${caster_radius}"/></geometry>
    </collision>
  </link>
  <joint name="caster_joint" type="fixed">
    <parent link="base_link"/>
    <child link="caster_link"/>
    <origin xyz="${-(base_length/2 - 0.05)} 0 ${caster_radius - base_z}" rpy="0 0 0"/>
  </joint>

  <!-- LiDAR mount -->
  <link name="lidar_link">
    <visual>
      <geometry><cylinder radius="0.04" length="0.05"/></geometry>
      <material name="red"/>
    </visual>
  </link>
  <joint name="lidar_joint" type="fixed">
    <parent link="base_link"/>
    <child link="lidar_link"/>
    <origin xyz="0.12 0 ${base_height/2 + 0.025}" rpy="0 0 0"/>
  </joint>

</robot>
```

**Step 3: Check the model**

```bash
cd ~/ros2_ws/src/diff_bot_description/urdf
xacro diff_bot.urdf.xacro > /tmp/diff_bot.urdf     # expand Xacro to URDF
check_urdf /tmp/diff_bot.urdf                        # validate and print the link tree
urdf_to_graphviz /tmp/diff_bot.urdf /tmp/diff_bot    # creates /tmp/diff_bot.pdf of the tree
```

**Step 4: Write the launch file** `launch/display.launch.py`

```python
import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch.substitutions import Command
from launch_ros.actions import Node
from launch_ros.parameter_descriptions import ParameterValue


def generate_launch_description():
    pkg = get_package_share_directory('diff_bot_description')
    xacro_file = os.path.join(pkg, 'urdf', 'diff_bot.urdf.xacro')
    rviz_config = os.path.join(pkg, 'rviz', 'display.rviz')

    robot_description = ParameterValue(Command(['xacro ', xacro_file]), value_type=str)
    rviz_args = ['-d', rviz_config] if os.path.exists(rviz_config) else []

    return LaunchDescription([
        Node(package='robot_state_publisher', executable='robot_state_publisher',
             parameters=[{'robot_description': robot_description}]),
        Node(package='joint_state_publisher_gui', executable='joint_state_publisher_gui'),
        Node(package='rviz2', executable='rviz2', arguments=rviz_args, output='screen'),
    ])
```

**Step 5: Package files**

`CMakeLists.txt` (replace the body):

```cmake
cmake_minimum_required(VERSION 3.8)
project(diff_bot_description)
find_package(ament_cmake REQUIRED)
install(DIRECTORY urdf launch rviz DESTINATION share/${PROJECT_NAME})
ament_package()
```

In `package.xml` add:

```xml
<exec_depend>robot_state_publisher</exec_depend>
<exec_depend>joint_state_publisher_gui</exec_depend>
<exec_depend>rviz2</exec_depend>
<exec_depend>xacro</exec_depend>
```

**Step 6: Build and launch**

```bash
cd ~/ros2_ws
colcon build --packages-select diff_bot_description --symlink-install
source install/setup.bash
ros2 launch diff_bot_description display.launch.py
```

**Step 7: Configure RViz2**

1. Set **Global Options > Fixed Frame** to `base_footprint`.
2. Click **Add > RobotModel**; set **Description Topic** to `/robot_description`.
3. Click **Add > TF** to show the coordinate axes of each link.
4. Move the sliders in the Joint State Publisher window; the wheels rotate.
5. Save with **File > Save Config As** to `~/ros2_ws/src/diff_bot_description/rviz/display.rviz`, rebuild once, and the next launch opens with this view.

**Step 8: Inspect**

```bash
ros2 topic echo /joint_states --once
ros2 run tf2_tools view_frames
ros2 param get /robot_state_publisher robot_description | head
```

### Where, When and Why (Experiment 7)

| Step | Where | When | Why |
| --- | --- | --- | --- |
| Install tools | Any terminal (sudo) | First | `xacro` expands macros, `check_urdf` validates, and the GUI joint publisher lets you move joints with sliders |
| Create `urdf`, `launch`, `rviz` folders | `~/ros2_ws/src/diff_bot_description/` | Before writing files | `rviz` must exist before building because `CMakeLists.txt` installs it |
| Xacro file | `urdf/diff_bot.urdf.xacro` | After the folders | Properties give one place to change a size; the wheel macro avoids writing the same code twice |
| `xacro`, `check_urdf` | Terminal in the `urdf` folder | Before every launch | Catches typos and broken links early, when the error message is easy to read |
| Launch file | `launch/display.launch.py` | After the model is valid | Starts `robot_state_publisher` (builds TF from the URDF), the joint slider GUI, and RViz2 together |
| `CMakeLists.txt` and `package.xml` | Package root | Before building | A model package has no code, so it only needs to install its folders and list its run dependencies |
| Build and launch | `~/ros2_ws` | After editing any file | The launch file reads the installed copy of the Xacro |
| RViz2 Fixed Frame | RViz2 left panel | After RViz2 opens | The Fixed Frame is the reference everything is drawn against; it must be a frame that exists, here `base_footprint` |
| RobotModel and TF displays | RViz2 Add button | After setting the Fixed Frame | RobotModel draws the shape; TF draws the axes of each link so you can check positions |

### Expected Output

`check_urdf` prints:

```text
robot name is: diff_bot
---------- Successfully Parsed XML ---------------
root Link: base_footprint has 1 child(ren)
    child(1):  base_link
        child(1):  caster_link
        child(2):  left_wheel_link
        child(3):  lidar_link
        child(4):  right_wheel_link
```

RViz2 shows a blue box with two black wheels, a grey rear caster and a red lidar cylinder. The wheel joints can be rotated with the sliders. The order of children in `check_urdf` can differ.

### Result

A differential-drive robot was modeled with Xacro properties and macros, validated with `check_urdf`, and visualized and animated in RViz2.

### Viva Questions

1. What is the difference between `visual` and `collision` elements?
2. Why is Xacro preferred over writing plain URDF?
3. Which node turns the URDF into TF transforms?
4. Why must a URDF be a tree and not a loop?
5. Why do we add a `base_footprint` link?

## Experiment 8: Visualization and Debugging of Robot Coordinate Frames Using TF2

**Aim:** To broadcast static and dynamic coordinate frames, listen to transforms between frames, and visualize and debug the frame tree using TF2 tools and RViz2.

**CO mapped:** CO4

### Theory

A robot has many coordinate frames: the world, the base, each wheel, each sensor. **TF2** keeps track of the position and orientation of every frame relative to its parent, as a tree, and over time. Given any two frames, it can compute the transform between them.

- **Broadcaster:** publishes a transform (parent frame, child frame, translation, rotation as a quaternion, timestamp) on `/tf`.
- **Static broadcaster:** publishes a fixed transform once on `/tf_static` (for example a sensor mounted on the body).
- **Listener and buffer:** stores recent transforms and answers `lookup_transform(target_frame, source_frame, time)`.

Rules of the tree: each frame has exactly one parent, there are no loops, and frames must be connected to be looked up. A point seen in frame A can be converted to frame B using the transform from A to B.

For a rotation by yaw angle θ about z, the quaternion is:

```latex
q = (x, y, z, w) = (0,\; 0,\; \sin(\theta/2),\; \cos(\theta/2))
```

### Part A: Inspect the robot frames from Experiment 7

Launch `diff_bot_description display.launch.py`, then in new terminals:

```bash
ros2 run tf2_tools view_frames           # creates frames_<time>.pdf and .gv in current folder
ros2 run tf2_ros tf2_echo base_footprint lidar_link
ros2 run tf2_ros tf2_monitor
ros2 topic echo /tf_static --once
```

Open the PDF to see the tree: `base_footprint -> base_link -> {left_wheel_link, right_wheel_link, caster_link, lidar_link}`. Move the wheel sliders and watch `tf2_echo base_link left_wheel_link` change its rotation, while the fixed `lidar_link` transform stays the same.

### Part B: Static transform

From the command line (adds a camera 0.15 m ahead and 0.12 m above `base_link`):

```bash
ros2 run tf2_ros static_transform_publisher --x 0.15 --y 0 --z 0.12 \
  --yaw 0 --pitch 0 --roll 0 --frame-id base_link --child-frame-id camera_link
```

Create the lab package for the code parts:

```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python tf_lab --dependencies rclpy tf2_ros geometry_msgs turtlesim
mkdir -p tf_lab/launch
```

**Static broadcaster node:** `tf_lab/tf_lab/static_broadcaster.py`

```python
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import TransformStamped
from tf2_ros.static_transform_broadcaster import StaticTransformBroadcaster


class StaticFramePublisher(Node):
    def __init__(self):
        super().__init__('static_frame_publisher')
        self.tf_static_broadcaster = StaticTransformBroadcaster(self)

        t = TransformStamped()
        t.header.stamp = self.get_clock().now().to_msg()
        t.header.frame_id = 'base_link'
        t.child_frame_id = 'camera_link'
        t.transform.translation.x = 0.15
        t.transform.translation.y = 0.0
        t.transform.translation.z = 0.12
        t.transform.rotation.x = 0.0
        t.transform.rotation.y = 0.0
        t.transform.rotation.z = 0.0
        t.transform.rotation.w = 1.0
        self.tf_static_broadcaster.sendTransform(t)
        self.get_logger().info('Published static transform base_link -> camera_link')


def main():
    rclpy.init()
    node = StaticFramePublisher()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    rclpy.shutdown()
```

### Part C: Dynamic broadcaster and listener (turtle follows turtle)

Each turtle's pose is turned into a frame `world -> turtleN`. A listener finds where `turtle1` is relative to `turtle2` and drives `turtle2` toward it.

**Broadcaster:** `tf_lab/tf_lab/turtle_tf2_broadcaster.py`

```python
import math
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import TransformStamped
from tf2_ros import TransformBroadcaster
from turtlesim.msg import Pose


class FramePublisher(Node):
    def __init__(self):
        super().__init__('turtle_tf2_frame_publisher')
        self.declare_parameter('turtlename', 'turtle')
        self.turtlename = self.get_parameter('turtlename').get_parameter_value().string_value
        self.tf_broadcaster = TransformBroadcaster(self)
        self.subscription = self.create_subscription(
            Pose, f'/{self.turtlename}/pose', self.handle_turtle_pose, 1)

    def handle_turtle_pose(self, msg):
        t = TransformStamped()
        t.header.stamp = self.get_clock().now().to_msg()
        t.header.frame_id = 'world'
        t.child_frame_id = self.turtlename
        t.transform.translation.x = msg.x
        t.transform.translation.y = msg.y
        t.transform.translation.z = 0.0
        t.transform.rotation.x = 0.0
        t.transform.rotation.y = 0.0
        t.transform.rotation.z = math.sin(msg.theta / 2.0)
        t.transform.rotation.w = math.cos(msg.theta / 2.0)
        self.tf_broadcaster.sendTransform(t)


def main():
    rclpy.init()
    node = FramePublisher()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    rclpy.shutdown()
```

**Listener:** `tf_lab/tf_lab/turtle_tf2_listener.py`

```python
import math
import rclpy
from rclpy.node import Node
from rclpy.time import Time
from geometry_msgs.msg import Twist
from tf2_ros import TransformException
from tf2_ros.buffer import Buffer
from tf2_ros.transform_listener import TransformListener


class FrameListener(Node):
    def __init__(self):
        super().__init__('turtle_tf2_frame_listener')
        self.declare_parameter('target_frame', 'turtle1')
        self.target_frame = self.get_parameter('target_frame').get_parameter_value().string_value

        self.tf_buffer = Buffer()
        self.tf_listener = TransformListener(self.tf_buffer, self)
        self.publisher = self.create_publisher(Twist, 'turtle2/cmd_vel', 1)
        self.timer = self.create_timer(0.1, self.on_timer)

    def on_timer(self):
        try:
            # pose of target_frame (turtle1) expressed in turtle2's frame
            t = self.tf_buffer.lookup_transform('turtle2', self.target_frame, Time())
        except TransformException as ex:
            self.get_logger().info(f'Could not transform turtle2 to {self.target_frame}: {ex}')
            return

        x = t.transform.translation.x
        y = t.transform.translation.y
        msg = Twist()
        msg.angular.z = 1.0 * math.atan2(y, x)
        msg.linear.x = 0.5 * math.sqrt(x ** 2 + y ** 2)
        self.publisher.publish(msg)


def main():
    rclpy.init()
    node = FrameListener()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    rclpy.shutdown()
```

**Launch file:** `tf_lab/launch/turtle_follow_launch.py`

```python
from launch import LaunchDescription
from launch.actions import ExecuteProcess, TimerAction
from launch_ros.actions import Node


def generate_launch_description():
    spawn = ExecuteProcess(
        cmd=['ros2', 'service', 'call', '/spawn', 'turtlesim/srv/Spawn',
             "{x: 4.0, y: 2.0, theta: 0.0, name: 'turtle2'}"],
        output='screen')

    return LaunchDescription([
        Node(package='turtlesim', executable='turtlesim_node', name='sim'),
        Node(package='tf_lab', executable='turtle_tf2_broadcaster',
             name='broadcaster1', parameters=[{'turtlename': 'turtle1'}]),
        TimerAction(period=2.0, actions=[spawn]),
        TimerAction(period=4.0, actions=[
            Node(package='tf_lab', executable='turtle_tf2_broadcaster',
                 name='broadcaster2', parameters=[{'turtlename': 'turtle2'}]),
            Node(package='tf_lab', executable='turtle_tf2_listener', name='listener'),
        ]),
    ])
```

**Register and build.** In `setup.py` add:

```python
import os
from glob import glob
# inside setup(): data_files list gets this extra entry
(os.path.join('share', package_name, 'launch'), glob('launch/*.py')),
# entry points
'console_scripts': [
    'static_broadcaster = tf_lab.static_broadcaster:main',
    'turtle_tf2_broadcaster = tf_lab.turtle_tf2_broadcaster:main',
    'turtle_tf2_listener = tf_lab.turtle_tf2_listener:main',
],
```

```bash
cd ~/ros2_ws && colcon build --packages-select tf_lab --symlink-install
source install/setup.bash
ros2 launch tf_lab turtle_follow_launch.py          # Terminal 1
ros2 run turtlesim turtle_teleop_key                # Terminal 2: drive turtle1
```

In a third terminal visualize and debug:

```bash
ros2 run tf2_tools view_frames
ros2 run tf2_ros tf2_echo turtle2 turtle1
ros2 run tf2_ros tf2_monitor
rviz2        # Fixed Frame = world, Add > TF
```

### Debugging Guide

| Message or symptom | Likely cause | Fix |
| --- | --- | --- |
| `Frame "x" does not exist` | No broadcaster for that frame | Check `ros2 run tf2_tools view_frames` and the frame name spelling |
| `Could not find a connection between 'a' and 'b'` | Frames are in two separate trees | Add the missing parent-child transform |
| `Lookup would require extrapolation into the future/past` | Timestamp not yet available or too old | Use `Time()` (latest), or wait with a timeout |
| Model flies away or flickers in RViz | Two nodes publish the same child frame | Make sure each frame has one publisher |
| Wrong orientation | Quaternion not normalised, or wrong parent | Normalise; check axes with the TF display |
| Frame missing in RViz | Fixed Frame not set, or `use_sim_time` mismatch | Set Fixed Frame; check time source |

### Where, When and Why (Experiment 8)

| Step | Where | When | Why |
| --- | --- | --- | --- |
| Part A: inspect robot frames | Terminals next to the running Experiment 7 launch | While `display.launch.py` runs | TF data exists only while a publisher is running; this shows a real tree before you write any code |
| `view_frames` | A folder you can find again (it saves a PDF there) | After the system is running | Gives a picture of the tree; use it first whenever TF behaves strangely |
| `tf2_echo` | Any sourced terminal | While frames are published | Prints the exact transform between two frames, to check numbers against expectation |
| Static transform (CLI) | Any sourced terminal | When a part is fixed to the robot | A camera mounted on the body never moves, so it is published once on `/tf_static` |
| `tf_lab` package | `~/ros2_ws/src/tf_lab` | Before the code files | Keeps all TF code in one place |
| Broadcaster | `tf_lab/tf_lab/turtle_tf2_broadcaster.py` | One instance per turtle | Converts each turtle's pose into a frame, so TF knows where every turtle is |
| Listener | `tf_lab/tf_lab/turtle_tf2_listener.py` | After the broadcasters | Asks TF where turtle 1 is as seen from turtle 2, which is what a follower needs |
| Launch file with timers | `tf_lab/launch/` | When nodes depend on each other | `TimerAction` waits for the simulator before spawning turtle 2 and starting the follower |
| RViz2 with Fixed Frame `world` | RViz2 | During the run | You see the two frames move together as the turtles move |
| Debugging table | Use when stuck | When a lookup fails | Most TF errors are a missing frame, two parents for one frame, or a timing problem |

### Expected Output

- `view_frames` shows `world -> turtle1` and `world -> turtle2`.
- `tf2_echo turtle2 turtle1` prints the translation and rotation of turtle1 relative to turtle2, changing as turtle1 moves.
- Turtle 2 spawns at (4, 2) and chases turtle 1 as it is driven with the keyboard.
- The static broadcaster adds `camera_link` 0.15 m forward and 0.12 m up from `base_link` in the Part A tree.

### Result

Static and dynamic transforms were broadcast, looked up and used for control, and the frame tree was visualized and debugged with `view_frames`, `tf2_echo`, `tf2_monitor` and RViz2.

### Viva Questions

1. What is the difference between `/tf` and `/tf_static`?
2. Why must the TF structure be a tree?
3. Why are quaternions used for rotation and not roll-pitch-yaw?
4. What does `lookup_transform('turtle2', 'turtle1', Time())` return?
5. How does `robot_state_publisher` use TF2?

## Experiment 9: Sensor Integration and Data Handling

**Aim:** To add a LiDAR, a camera and an IMU to the simulated robot in Gazebo, read their data in ROS 2 nodes, filter and process it, and record it with rosbag.

**CO mapped:** CO5

### Theory

Gazebo (classic, used with Humble through `gazebo_ros_pkgs`) simulates physics and sensors. Each sensor is declared inside a `<gazebo reference="link">` block of the URDF, and a plugin publishes the data to ROS 2 topics.

| Sensor | Gazebo type and plugin | ROS 2 topic | Message type |
| --- | --- | --- | --- |
| 2D LiDAR | `ray`, `libgazebo_ros_ray_sensor.so` | `/scan` | `sensor_msgs/msg/LaserScan` |
| Camera | `camera`, `libgazebo_ros_camera.so` | `/camera/image_raw` | `sensor_msgs/msg/Image` |
| IMU | `imu`, `libgazebo_ros_imu_sensor.so` | `/imu` | `sensor_msgs/msg/Imu` |
| Wheel odometry | `libgazebo_ros_diff_drive.so` | `/odom`, `/cmd_vel` | `nav_msgs/msg/Odometry`, `geometry_msgs/msg/Twist` |

Important message fields:

- `LaserScan`: `angle_min`, `angle_max`, `angle_increment`, `range_min`, `range_max`, `ranges[]`. Invalid readings are `inf` or `nan` and must be removed before processing. The distance at angle `a` is `ranges[int((a - angle_min) / angle_increment)]`.
- `Image`: `height`, `width`, `encoding` (such as `rgb8`), `data[]`.
- `Imu`: `orientation` (quaternion), `angular_velocity`, `linear_acceleration`.

Sensor topics usually use the best-effort `qos_profile_sensor_data` QoS profile: fast, and a missed message is not resent.

### Step 1: Install and prepare

```bash
sudo apt install -y ros-humble-gazebo-ros-pkgs ros-humble-teleop-twist-keyboard \
  ros-humble-rqt-image-view ros-humble-rosbag2 python3-numpy
cd ~/ros2_ws/src/diff_bot_description
mkdir -p worlds
```

In `CMakeLists.txt` change the install line so the new folder is installed, and add `<exec_depend>gazebo_ros</exec_depend>` to `package.xml`:

```cmake
install(DIRECTORY urdf launch rviz worlds DESTINATION share/${PROJECT_NAME})
```

### Step 2: Simulation model with sensors

Create `urdf/diff_bot_sim.urdf.xacro`. It includes the Experiment 7 robot and adds a camera, an IMU, Gazebo materials and plugins.

```xml
<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro" name="diff_bot">

  <xacro:include filename="$(find diff_bot_description)/urdf/diff_bot.urdf.xacro"/>

  <!-- Camera link on the front face -->
  <link name="camera_link">
    <visual>
      <geometry><box size="0.02 0.06 0.03"/></geometry>
      <material name="black"/>
    </visual>
  </link>
  <joint name="camera_joint" type="fixed">
    <parent link="base_link"/>
    <child link="camera_link"/>
    <origin xyz="0.2 0 0.03" rpy="0 0 0"/>
  </joint>

  <!-- IMU link at the centre of the body -->
  <link name="imu_link"/>
  <joint name="imu_joint" type="fixed">
    <parent link="base_link"/>
    <child link="imu_link"/>
    <origin xyz="0 0 0" rpy="0 0 0"/>
  </joint>

  <!-- Colours and friction -->
  <gazebo reference="base_link"><material>Gazebo/Blue</material></gazebo>
  <gazebo reference="left_wheel_link">
    <material>Gazebo/Black</material><mu1>1.0</mu1><mu2>1.0</mu2>
  </gazebo>
  <gazebo reference="right_wheel_link">
    <material>Gazebo/Black</material><mu1>1.0</mu1><mu2>1.0</mu2>
  </gazebo>
  <gazebo reference="caster_link">
    <material>Gazebo/Grey</material><mu1>0.0</mu1><mu2>0.0</mu2>
  </gazebo>
  <gazebo reference="lidar_link"><material>Gazebo/Red</material></gazebo>

  <!-- Differential drive: listens to /cmd_vel, publishes /odom and odom -> base_footprint -->
  <gazebo>
    <plugin name="diff_drive" filename="libgazebo_ros_diff_drive.so">
      <update_rate>50</update_rate>
      <left_joint>left_wheel_joint</left_joint>
      <right_joint>right_wheel_joint</right_joint>
      <wheel_separation>0.29</wheel_separation>
      <wheel_diameter>0.10</wheel_diameter>
      <max_wheel_torque>20</max_wheel_torque>
      <max_wheel_acceleration>1.0</max_wheel_acceleration>
      <command_topic>cmd_vel</command_topic>
      <odometry_topic>odom</odometry_topic>
      <odometry_frame>odom</odometry_frame>
      <robot_base_frame>base_footprint</robot_base_frame>
      <publish_odom>true</publish_odom>
      <publish_odom_tf>true</publish_odom_tf>
      <publish_wheel_tf>false</publish_wheel_tf>
    </plugin>
  </gazebo>

  <!-- Wheel joint states so robot_state_publisher can move the wheels -->
  <gazebo>
    <plugin name="joint_state" filename="libgazebo_ros_joint_state_publisher.so">
      <update_rate>30</update_rate>
      <joint_name>left_wheel_joint</joint_name>
      <joint_name>right_wheel_joint</joint_name>
    </plugin>
  </gazebo>

  <!-- 2D LiDAR -->
  <gazebo reference="lidar_link">
    <sensor name="lidar" type="ray">
      <always_on>true</always_on>
      <visualize>true</visualize>
      <update_rate>10</update_rate>
      <ray>
        <scan>
          <horizontal>
            <samples>360</samples>
            <resolution>1.0</resolution>
            <min_angle>-3.14159</min_angle>
            <max_angle>3.14159</max_angle>
          </horizontal>
        </scan>
        <range><min>0.12</min><max>8.0</max><resolution>0.01</resolution></range>
        <noise><type>gaussian</type><mean>0.0</mean><stddev>0.01</stddev></noise>
      </ray>
      <plugin name="lidar_plugin" filename="libgazebo_ros_ray_sensor.so">
        <ros><remapping>~/out:=scan</remapping></ros>
        <output_type>sensor_msgs/LaserScan</output_type>
        <frame_name>lidar_link</frame_name>
      </plugin>
    </sensor>
  </gazebo>

  <!-- Camera -->
  <gazebo reference="camera_link">
    <sensor name="camera" type="camera">
      <always_on>true</always_on>
      <visualize>true</visualize>
      <update_rate>15</update_rate>
      <camera>
        <horizontal_fov>1.089</horizontal_fov>
        <image><width>640</width><height>480</height><format>R8G8B8</format></image>
        <clip><near>0.05</near><far>8.0</far></clip>
      </camera>
      <plugin name="camera_plugin" filename="libgazebo_ros_camera.so">
        <frame_name>camera_link</frame_name>
      </plugin>
    </sensor>
  </gazebo>

  <!-- IMU -->
  <gazebo reference="imu_link">
    <sensor name="imu_sensor" type="imu">
      <always_on>true</always_on>
      <update_rate>50</update_rate>
      <plugin name="imu_plugin" filename="libgazebo_ros_imu_sensor.so">
        <ros><remapping>~/out:=imu</remapping></ros>
        <initial_orientation_as_reference>false</initial_orientation_as_reference>
        <frame_name>imu_link</frame_name>
      </plugin>
    </sensor>
  </gazebo>

</robot>
```

### Step 3: Test world with obstacles

Create `worlds/lab_world.world`:

```xml
<?xml version="1.0"?>
<sdf version="1.6">
  <world name="lab_world">
    <include><uri>model://ground_plane</uri></include>
    <include><uri>model://sun</uri></include>

    <model name="box_1">
      <static>true</static>
      <pose>2.0 0.5 0.25 0 0 0</pose>
      <link name="link">
        <collision name="c"><geometry><box><size>0.5 0.5 0.5</size></box></geometry></collision>
        <visual name="v">
          <geometry><box><size>0.5 0.5 0.5</size></box></geometry>
          <material><ambient>0.8 0.3 0.1 1</ambient><diffuse>0.8 0.3 0.1 1</diffuse></material>
        </visual>
      </link>
    </model>

    <model name="cylinder_1">
      <static>true</static>
      <pose>3.5 -1.0 0.25 0 0 0</pose>
      <link name="link">
        <collision name="c"><geometry><cylinder><radius>0.3</radius><length>0.5</length></cylinder></geometry></collision>
        <visual name="v">
          <geometry><cylinder><radius>0.3</radius><length>0.5</length></cylinder></geometry>
          <material><ambient>0.1 0.6 0.2 1</ambient><diffuse>0.1 0.6 0.2 1</diffuse></material>
        </visual>
      </link>
    </model>

    <model name="box_2">
      <static>true</static>
      <pose>1.0 -2.0 0.25 0 0 0.5</pose>
      <link name="link">
        <collision name="c"><geometry><box><size>0.8 0.3 0.5</size></box></geometry></collision>
        <visual name="v">
          <geometry><box><size>0.8 0.3 0.5</size></box></geometry>
          <material><ambient>0.2 0.2 0.8 1</ambient><diffuse>0.2 0.2 0.8 1</diffuse></material>
        </visual>
      </link>
    </model>

    <model name="box_3">
      <static>true</static>
      <pose>4.5 1.5 0.25 0 0 0</pose>
      <link name="link">
        <collision name="c"><geometry><box><size>0.5 0.5 0.5</size></box></geometry></collision>
        <visual name="v">
          <geometry><box><size>0.5 0.5 0.5</size></box></geometry>
          <material><ambient>0.7 0.7 0.1 1</ambient><diffuse>0.7 0.7 0.1 1</diffuse></material>
        </visual>
      </link>
    </model>
  </world>
</sdf>
```

### Step 4: Gazebo launch file

Create `launch/gazebo.launch.py` in `diff_bot_description`:

```python
import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.substitutions import Command
from launch_ros.actions import Node
from launch_ros.parameter_descriptions import ParameterValue


def generate_launch_description():
    pkg = get_package_share_directory('diff_bot_description')
    xacro_file = os.path.join(pkg, 'urdf', 'diff_bot_sim.urdf.xacro')
    world = os.path.join(pkg, 'worlds', 'lab_world.world')

    gazebo = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(os.path.join(
            get_package_share_directory('gazebo_ros'), 'launch', 'gazebo.launch.py')),
        launch_arguments={'world': world}.items())

    robot_state_publisher = Node(
        package='robot_state_publisher', executable='robot_state_publisher',
        parameters=[{
            'robot_description': ParameterValue(Command(['xacro ', xacro_file]), value_type=str),
            'use_sim_time': True}])

    spawn = Node(
        package='gazebo_ros', executable='spawn_entity.py',
        arguments=['-topic', 'robot_description', '-entity', 'diff_bot',
                   '-x', '0.0', '-y', '0.0', '-z', '0.1'],
        output='screen')

    return LaunchDescription([gazebo, robot_state_publisher, spawn])
```

```bash
cd ~/ros2_ws && colcon build --packages-select diff_bot_description --symlink-install
source install/setup.bash
ros2 launch diff_bot_description gazebo.launch.py
```

Drive the robot from another terminal:

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

### Step 5: Sensor data handling node

```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python sensor_lab --dependencies rclpy sensor_msgs nav_msgs std_msgs geometry_msgs
```

Create `sensor_lab/sensor_lab/sensor_handler.py`. It cleans the LiDAR scan, finds the nearest obstacle in the front, left and right sectors, smooths the front distance with a moving average, reads yaw from the IMU, measures image brightness, and publishes the front distance.

```python
import math
from collections import deque

import numpy as np
import rclpy
from rclpy.node import Node
from rclpy.qos import qos_profile_sensor_data
from sensor_msgs.msg import LaserScan, Imu, Image
from nav_msgs.msg import Odometry
from std_msgs.msg import Float32


def sector_min(scan, a0, a1):
    """Smallest valid range between angles a0 and a1 (radians)."""
    i0 = int((a0 - scan.angle_min) / scan.angle_increment)
    i1 = int((a1 - scan.angle_min) / scan.angle_increment)
    i0, i1 = max(0, min(i0, i1)), min(len(scan.ranges) - 1, max(i0, i1))
    valid = [r for r in scan.ranges[i0:i1 + 1]
             if math.isfinite(r) and scan.range_min <= r <= scan.range_max]
    return min(valid) if valid else float('inf')


class SensorHandler(Node):
    def __init__(self):
        super().__init__('sensor_handler')
        self.create_subscription(LaserScan, '/scan', self.scan_cb, qos_profile_sensor_data)
        self.create_subscription(Imu, '/imu', self.imu_cb, qos_profile_sensor_data)
        self.create_subscription(Image, '/camera/image_raw', self.image_cb, qos_profile_sensor_data)
        self.create_subscription(Odometry, '/odom', self.odom_cb, 10)
        self.front_pub = self.create_publisher(Float32, '/front_distance', 10)
        self.window = deque(maxlen=5)          # moving-average filter
        self.timer = self.create_timer(1.0, self.report)
        self.front = self.left = self.right = float('inf')
        self.yaw_deg = 0.0
        self.brightness = 0.0
        self.x = self.y = 0.0

    def scan_cb(self, scan):
        front = sector_min(scan, -0.52, 0.52)      # about +-30 deg
        self.left = sector_min(scan, 0.52, 1.57)   # 30 to 90 deg
        self.right = sector_min(scan, -1.57, -0.52)
        if math.isfinite(front):
            self.window.append(front)
        if self.window:
            self.front = sum(self.window) / len(self.window)
            self.front_pub.publish(Float32(data=float(self.front)))

    def imu_cb(self, msg):
        q = msg.orientation
        self.yaw_deg = math.degrees(math.atan2(
            2.0 * (q.w * q.z + q.x * q.y), 1.0 - 2.0 * (q.y * q.y + q.z * q.z)))

    def image_cb(self, msg):
        if msg.encoding in ('rgb8', 'bgr8'):
            img = np.frombuffer(msg.data, dtype=np.uint8).reshape(msg.height, msg.width, 3)
            self.brightness = float(img.mean())

    def odom_cb(self, msg):
        self.x = msg.pose.pose.position.x
        self.y = msg.pose.pose.position.y

    def report(self):
        self.get_logger().info(
            f'front={self.front:.2f} m  left={self.left:.2f} m  right={self.right:.2f} m | '
            f'yaw={self.yaw_deg:.1f} deg | brightness={self.brightness:.1f} | '
            f'pos=({self.x:.2f}, {self.y:.2f})')


def main(args=None):
    rclpy.init(args=args)
    node = SensorHandler()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

In `setup.py` of `sensor_lab` add `'sensor_handler = sensor_lab.sensor_handler:main'` under `console_scripts`, then:

```bash
cd ~/ros2_ws && colcon build --packages-select sensor_lab --symlink-install
source install/setup.bash
ros2 run sensor_lab sensor_handler
```

### Step 6: Inspect, visualize and record the data

```bash
ros2 topic list -t
ros2 topic hz /scan
ros2 topic echo /imu --once
ros2 topic echo /odom --once
ros2 run rqt_image_view rqt_image_view          # select /camera/image_raw
rviz2                                           # Fixed Frame = odom; Add LaserScan (/scan), RobotModel, TF

ros2 bag record -o sensor_bag /scan /imu /odom /camera/image_raw
ros2 bag info sensor_bag
ros2 bag play sensor_bag
```

In RViz2 set the LaserScan topic reliability to **Best Effort**; if the scan does not show, this is the usual reason.

### Where, When and Why (Experiment 9)

| Step | Where | When | Why |
| --- | --- | --- | --- |
| Install Gazebo packages | Any terminal (sudo) | First | `gazebo_ros_pkgs` connects the simulator to ROS 2 topics |
| `worlds` folder and `CMakeLists.txt` line | `diff_bot_description/` | Before building | New folders are only installed if listed; otherwise the launch file cannot find the world |
| `diff_bot_sim.urdf.xacro` | `diff_bot_description/urdf/` | After Experiment 7 works | Reuses the Experiment 7 model and adds only what the simulator needs: sensors, friction, colours and plugins |
| Diff-drive plugin | Inside the sim Xacro | With the model | Turns `/cmd_vel` into wheel motion and publishes `/odom`; without it the robot cannot be driven |
| Sensor blocks (LiDAR, camera, IMU) | Inside the sim Xacro | With the model | Each block creates a simulated sensor and publishes its data to a ROS 2 topic, replacing real hardware |
| `lab_world.world` | `diff_bot_description/worlds/` | Before launch | Gives the LiDAR obstacles to detect |
| `gazebo.launch.py` | `diff_bot_description/launch/` | After the model and world exist | Starts Gazebo, publishes the robot description, and spawns the robot into the world |
| Build, source, launch | `~/ros2_ws` | After every file change | Gazebo reads the installed copy of the files |
| Teleop keyboard | Second terminal (click inside to type) | After the robot appears | Moves the robot so sensor values change |
| `sensor_lab` node | `~/ros2_ws/src/sensor_lab` | While Gazebo runs | Shows how a real program reads, cleans (`inf` and `nan`) and smooths raw sensor data |
| `rqt_image_view`, RViz2 LaserScan | Separate terminals | While Gazebo runs | A visual check that each sensor works before using it for control in Experiment 10 |
| `ros2 bag record` | A folder with free disk space | During a driving run | Saves the data so you can replay and analyze it later without running Gazebo |

### Expected Output

- Gazebo opens with the robot, four obstacles and the lidar rays visible.
- `ros2 topic hz /scan` reports about 10 Hz; `/imu` about 50 Hz; `/camera/image_raw` about 15 Hz.
- `sensor_handler` prints a line each second, for example `front=1.43 m left=2.10 m right=inf | yaw=12.4 deg | brightness=118.6 | pos=(0.55, 0.03)`. The front distance falls as the robot approaches an obstacle.
- `rqt_image_view` shows the camera view; the RViz2 scan outlines the obstacles.
- `ros2 bag info` lists the recorded topics with their message counts.

### Result

LiDAR, camera, IMU and odometry were integrated into the simulated robot, their data was read, filtered and processed in a node, and the data was visualized and recorded.

### Viva Questions

1. What does `inf` in a `LaserScan` mean and how do you handle it?
2. Why are sensor topics published with best-effort QoS?
3. What does the diff-drive plugin publish and subscribe?
4. How is yaw computed from the IMU quaternion?
5. What is the use of rosbag?

## Experiment 10: Autonomous Task Integration

**Aim:** To integrate the robot model, simulation, sensors, topics, parameters and launch files from the earlier experiments into one autonomous system that visits waypoints while avoiding obstacles.

**CO mapped:** CO5

### Theory

An autonomous mobile robot repeats a **sense, decide, act** loop:

- **Sense:** `/odom` gives the robot's position and heading; `/scan` gives the distance to obstacles.
- **Decide:** a state machine chooses what to do.
- **Act:** velocity commands are published on `/cmd_vel`, which the Gazebo diff-drive plugin turns into wheel motion.

This experiment uses a simple **reactive** method, not a map-based planner. The robot steers toward the current waypoint, and when the LiDAR sees an obstacle too close in front it stops moving forward and turns toward the more open side until the way is clear. Hysteresis (leaving the avoid state only when the distance has grown by 0.3 m beyond the trigger distance) prevents rapid switching between states.

| State | Entered when | Behaviour | Leaves when |
| --- | --- | --- | --- |
| `GO` | Start, or the path is clear | Turn toward the waypoint, drive at P-controlled speed, slow down near obstacles | Front distance < `obstacle_distance`, or all waypoints are done |
| `AVOID` | Front distance < `obstacle_distance` | Stop moving forward, rotate toward the side with more free space | Front distance > `obstacle_distance` + 0.3 m |
| `DONE` | Last waypoint reached | Publish zero velocity | Never (restart the node) |

The control law for `GO` is:

```latex
v = \min\big(v_{max},\; K_v \, d\big)\cdot s, \qquad \omega = \operatorname{clip}\big(K_\omega\, e_\theta,\; -\omega_{max},\; \omega_{max}\big)
```

Here `d` is the distance to the waypoint, `e_θ` is the heading error wrapped to \[-π, π\], and `s` is a slow-down factor between 0 and 1 that falls to 0 as the front distance drops from `slow_distance` to `obstacle_distance`. The robot does not drive forward while |e\_θ| > 0.6 rad; it rotates first.

### System Overview

| Node | Package | Subscribes | Publishes |
| --- | --- | --- | --- |
| Gazebo diff-drive plugin | `gazebo_ros` | `/cmd_vel` | `/odom`, `odom -> base_footprint` TF |
| Gazebo sensor plugins | `gazebo_ros` | none | `/scan`, `/imu`, `/camera/image_raw` |
| `robot_state_publisher` | `robot_state_publisher` | `/joint_states` | `/tf`, `/tf_static` |
| `autonomous_navigator` | `auto_nav` | `/odom`, `/scan` | `/cmd_vel`, `/nav_status` |
| `sensor_handler` (optional) | `sensor_lab` | `/scan`, `/imu`, `/odom`, camera | `/front_distance` |

### Step 1: Create the package

```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python auto_nav \
  --dependencies rclpy geometry_msgs nav_msgs sensor_msgs std_msgs
mkdir -p auto_nav/launch auto_nav/config
```

### Step 2: The navigator node

Create `auto_nav/auto_nav/autonomous_navigator.py`:

```python
import math

import rclpy
from rclpy.node import Node
from rclpy.qos import qos_profile_sensor_data
from geometry_msgs.msg import Twist
from nav_msgs.msg import Odometry
from sensor_msgs.msg import LaserScan
from std_msgs.msg import String


def clamp(v, lo, hi):
    return max(lo, min(hi, v))


def wrap(a):
    return math.atan2(math.sin(a), math.cos(a))


def sector_min(scan, a0, a1):
    """Smallest valid range between angles a0 and a1 (radians)."""
    i0 = int((a0 - scan.angle_min) / scan.angle_increment)
    i1 = int((a1 - scan.angle_min) / scan.angle_increment)
    i0, i1 = max(0, min(i0, i1)), min(len(scan.ranges) - 1, max(i0, i1))
    valid = [r for r in scan.ranges[i0:i1 + 1]
             if math.isfinite(r) and scan.range_min <= r <= scan.range_max]
    return min(valid) if valid else float('inf')


class AutonomousNavigator(Node):
    def __init__(self):
        super().__init__('autonomous_navigator')

        self.declare_parameter('waypoints', [3.0, 0.0, 3.0, 2.5, 0.0, 2.5, 0.0, 0.0])
        self.declare_parameter('max_linear', 0.3)
        self.declare_parameter('max_angular', 1.0)
        self.declare_parameter('kv', 0.8)
        self.declare_parameter('kw', 1.5)
        self.declare_parameter('goal_tolerance', 0.15)
        self.declare_parameter('obstacle_distance', 0.5)
        self.declare_parameter('slow_distance', 1.0)

        flat = list(self.get_parameter('waypoints').value)
        self.waypoints = [(flat[i], flat[i + 1]) for i in range(0, len(flat) - 1, 2)]
        self.max_linear = self.get_parameter('max_linear').value
        self.max_angular = self.get_parameter('max_angular').value
        self.kv = self.get_parameter('kv').value
        self.kw = self.get_parameter('kw').value
        self.goal_tol = self.get_parameter('goal_tolerance').value
        self.obstacle_dist = self.get_parameter('obstacle_distance').value
        self.slow_dist = self.get_parameter('slow_distance').value

        self.x = self.y = self.yaw = 0.0
        self.have_odom = False
        self.scan = None
        self.idx = 0
        self.state = 'GO'
        self.avoid_dir = 1.0

        self.cmd_pub = self.create_publisher(Twist, '/cmd_vel', 10)
        self.status_pub = self.create_publisher(String, '/nav_status', 10)
        self.create_subscription(Odometry, '/odom', self.odom_cb, 10)
        self.create_subscription(LaserScan, '/scan', self.scan_cb, qos_profile_sensor_data)
        self.timer = self.create_timer(0.1, self.control_loop)    # 10 Hz

        self.get_logger().info(f'Navigator started with {len(self.waypoints)} waypoints')

    # ---------- callbacks ----------
    def odom_cb(self, msg):
        self.x = msg.pose.pose.position.x
        self.y = msg.pose.pose.position.y
        q = msg.pose.pose.orientation
        self.yaw = math.atan2(2.0 * (q.w * q.z + q.x * q.y),
                              1.0 - 2.0 * (q.y * q.y + q.z * q.z))
        self.have_odom = True

    def scan_cb(self, msg):
        self.scan = msg

    # ---------- helpers ----------
    def set_state(self, new_state):
        if new_state != self.state:
            self.get_logger().info(f'State: {self.state} -> {new_state}')
            self.state = new_state
        self.status_pub.publish(String(data=f'{self.state} waypoint={self.idx + 1}/{len(self.waypoints)}'))

    def stop(self):
        self.cmd_pub.publish(Twist())

    # ---------- main loop ----------
    def control_loop(self):
        if not self.have_odom or self.scan is None:
            return

        if self.idx >= len(self.waypoints):
            self.stop()
            if self.state != 'DONE':
                self.get_logger().info('All waypoints reached. Mission complete.')
                self.set_state('DONE')
            return

        gx, gy = self.waypoints[self.idx]
        dx, dy = gx - self.x, gy - self.y
        dist = math.hypot(dx, dy)

        if dist < self.goal_tol:
            self.get_logger().info(f'Reached waypoint {self.idx + 1}: ({gx:.1f}, {gy:.1f})')
            self.idx += 1
            self.stop()
            return

        front = sector_min(self.scan, -0.52, 0.52)
        left = sector_min(self.scan, 0.52, 1.57)
        right = sector_min(self.scan, -1.57, -0.52)

        if self.state == 'GO' and front < self.obstacle_dist:
            self.avoid_dir = 1.0 if left > right else -1.0     # turn to the freer side
            self.set_state('AVOID')
        elif self.state == 'AVOID' and front > self.obstacle_dist + 0.3:
            self.set_state('GO')

        cmd = Twist()
        if self.state == 'AVOID':
            cmd.angular.z = self.avoid_dir * self.max_angular
            self.cmd_pub.publish(cmd)
            return

        heading_error = wrap(math.atan2(dy, dx) - self.yaw)
        cmd.angular.z = clamp(self.kw * heading_error, -self.max_angular, self.max_angular)

        if abs(heading_error) < 0.6:                          # rotate first if badly aligned
            speed = min(self.max_linear, self.kv * dist)
            if front < self.slow_dist:
                speed *= (front - self.obstacle_dist) / (self.slow_dist - self.obstacle_dist)
            cmd.linear.x = max(0.0, speed)

        self.cmd_pub.publish(cmd)


def main(args=None):
    rclpy.init(args=args)
    node = AutonomousNavigator()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.stop()
        node.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### Step 3: Parameter file

Create `auto_nav/config/nav_params.yaml`:

```yaml
autonomous_navigator:
  ros__parameters:
    waypoints: [3.0, 0.0, 3.0, 2.5, 0.0, 2.5, 0.0, 0.0]   # x1, y1, x2, y2, ...
    max_linear: 0.3
    max_angular: 1.0
    kv: 0.8
    kw: 1.5
    goal_tolerance: 0.15
    obstacle_distance: 0.5
    slow_distance: 1.0
```

### Step 4: Launch file

Create `auto_nav/launch/auto_nav_launch.py`. It starts Gazebo with the robot and world from Experiment 9, then starts the sensor monitor and the navigator after the robot has spawned.

```python
import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription, TimerAction
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch_ros.actions import Node


def generate_launch_description():
    sim = IncludeLaunchDescription(PythonLaunchDescriptionSource(os.path.join(
        get_package_share_directory('diff_bot_description'), 'launch', 'gazebo.launch.py')))

    config = os.path.join(
        get_package_share_directory('auto_nav'), 'config', 'nav_params.yaml')

    navigator = Node(package='auto_nav', executable='autonomous_navigator',
                     name='autonomous_navigator', output='screen',
                     parameters=[config, {'use_sim_time': True}])
    monitor = Node(package='sensor_lab', executable='sensor_handler',
                   name='sensor_handler', output='screen')

    return LaunchDescription([
        sim,
        TimerAction(period=8.0, actions=[monitor, navigator]),   # wait for the robot to spawn
    ])
```

### Step 5: Register, build and run

In `auto_nav/setup.py`:

```python
import os
from glob import glob
# data_files gets two extra entries
(os.path.join('share', package_name, 'launch'), glob('launch/*.py')),
(os.path.join('share', package_name, 'config'), glob('config/*.yaml')),
# entry points
'console_scripts': [
    'autonomous_navigator = auto_nav.autonomous_navigator:main',
],
```

Add `<exec_depend>diff_bot_description</exec_depend>`, `<exec_depend>sensor_lab</exec_depend>` and `<exec_depend>ros2launch</exec_depend>` to `package.xml`, then:

```bash
cd ~/ros2_ws
colcon build --symlink-install
source install/setup.bash
ros2 launch auto_nav auto_nav_launch.py
```

Monitor and visualize in other terminals:

```bash
ros2 topic echo /nav_status
ros2 topic echo /cmd_vel
rqt_graph
rviz2        # Fixed Frame = odom; add RobotModel, TF, LaserScan (/scan, Best Effort), Odometry (/odom)
```

Change behaviour without editing code:

```bash
ros2 param set /autonomous_navigator max_linear 0.2
ros2 launch auto_nav auto_nav_launch.py        # after editing nav_params.yaml waypoints
```

### Test Cases and Observations

Fill the last column in your record after running each test.

| Test | Setup | What should happen | Observed |
| --- | --- | --- | --- |
| 1. Free path | Waypoints `[2.0, -1.0]` with `box_1` not on the line | Robot turns, drives straight and stops at the goal |  |
| 2. Obstacle in front | Default waypoints; `box_1` at (2.0, 0.5) near the first path | State changes to `AVOID`, robot turns away and then returns to `GO` |  |
| 3. Full mission | Default four waypoints | Log shows 4 "Reached waypoint" lines, then "Mission complete" |  |
| 4. Gain tuning | `kw` = 0.5, then 3.0 | Low value: slow, wide turns; high value: faster, may oscillate |  |
| 5. Speed limit | `max_linear` = 0.15, then 0.5 | Compare mission time and how close the robot comes to obstacles |  |

### Where, When and Why (Experiment 10)

| Step | Where | When | Why |
| --- | --- | --- | --- |
| Check Experiments 7 and 9 first | `~/ros2_ws` | Before starting | This experiment launches the earlier packages; if Gazebo with sensors does not run alone, this will not run either |
| Create `auto_nav` with `launch` and `config` | `~/ros2_ws/src` | First | Separate package for the final application, so the earlier ones stay unchanged |
| `autonomous_navigator.py` | `auto_nav/auto_nav/` | After the package exists | Contains the decision logic: read pose and scan, choose a state, send velocity |
| `nav_params.yaml` | `auto_nav/config/` | Before the launch file | Waypoints and gains live outside the code, so you can tune and retest without editing Python |
| `auto_nav_launch.py` | `auto_nav/launch/` | After the node and YAML | One command starts the simulator, then waits 8 s for the robot to spawn before starting the navigator, so it does not read sensors that are not there yet |
| `setup.py` and `package.xml` | `auto_nav/` | Before building | Registers the node, installs launch and config files, and lists the other packages it needs |
| `colcon build` and source | `~/ros2_ws` | After every change | Python and YAML changes still need a rebuild for launch to see them (unless you used `--symlink-install`) |
| `ros2 launch auto_nav auto_nav_launch.py` | Terminal 1 | When ready to run | Starts the whole system |
| `ros2 topic echo /nav_status` | Terminal 2 | During the run | Shows the current state and waypoint number without reading logs |
| RViz2 | Terminal 3 | During the run | See the scan, the robot and its path to judge avoidance behaviour |
| `ros2 param set` and test cases | Terminal 2 | After the first successful run | Changing one value at a time shows its effect, which is how tuning is done on real robots |

### Expected Output

```text
[autonomous_navigator]: Navigator started with 4 waypoints
[autonomous_navigator]: State: GO -> AVOID
[autonomous_navigator]: State: AVOID -> GO
[autonomous_navigator]: Reached waypoint 1: (3.0, 0.0)
[autonomous_navigator]: Reached waypoint 2: (3.0, 2.5)
[autonomous_navigator]: Reached waypoint 3: (0.0, 2.5)
[autonomous_navigator]: Reached waypoint 4: (0.0, 0.0)
[autonomous_navigator]: All waypoints reached. Mission complete.
```

The exact number of `AVOID` entries depends on gains and wheel slip. The robot returns close to its starting point. A reactive method can fail in narrow gaps or dead ends; mapping and planning (SLAM and Nav2) solve that and are the natural next step.

### Result

The robot model, Gazebo simulation, LiDAR and odometry, topics, parameters and launch files were combined into one system in which the robot visited several waypoints and avoided obstacles on its own.

### Viva Questions

1. Explain the sense-decide-act loop using the nodes in this experiment.
2. Why is hysteresis used when leaving the `AVOID` state?
3. What is the weakness of a purely reactive obstacle-avoidance method and how can Nav2 help?
4. Why do we rotate in place when the heading error is large?
5. How does `use_sim_time` change the behaviour of timers and TF?

## Appendix

### A. ROS 2 Command Cheat Sheet

| Task | Command |
| --- | --- |
| Source ROS 2 | `source /opt/ros/humble/setup.bash` |
| Create package | `ros2 pkg create --build-type ament_python <name> --dependencies rclpy` |
| Build all / one package | `colcon build --symlink-install` / `colcon build --packages-select <pkg>` |
| Clean build | `rm -rf build install log` |
| Run a node | `ros2 run <pkg> <executable>` |
| Run with parameter / remap | `ros2 run <pkg> <exe> --ros-args -p name:=value -r old:=new` |
| Launch | `ros2 launch <pkg> <file>.py arg:=value` |
| List nodes / topics / services / actions | `ros2 node list`, `ros2 topic list -t`, `ros2 service list -t`, `ros2 action list -t` |
| Node details | `ros2 node info /<node>` |
| Topic echo / rate / publish | `ros2 topic echo <t>`, `ros2 topic hz <t>`, `ros2 topic pub <t> <type> "{...}"` |
| Show interface | `ros2 interface show <pkg>/msg/<Type>` |
| Service call | `ros2 service call <srv> <type> "{...}"` |
| Action goal | `ros2 action send_goal <action> <type> "{...}" --feedback` |
| Parameters | `ros2 param list / get / set / dump / load` |
| Frames | `ros2 run tf2_tools view_frames`, `ros2 run tf2_ros tf2_echo <a> <b>` |
| Record / play data | `ros2 bag record -o <name> <topics>`, `ros2 bag play <name>` |
| Graph tools | `rqt_graph`, `rqt_console`, `rqt_image_view`, `rviz2` |
| Check dependencies | `rosdep install -i --from-path src --rosdistro humble -y` |
| Doctor | `ros2 doctor --report` |

### B. Troubleshooting

| Problem | What to check |
| --- | --- |
| Node does not appear in `ros2 node list` | Same `ROS_DOMAIN_ID` in all terminals; both terminals sourced; firewall or VM network mode (use bridged or NAT consistently) |
| `No executable found` | Entry point missing in `setup.py` or `CMakeLists.txt`; rebuild; source the overlay |
| Python package builds but import fails | Missing `__init__.py` in the module folder; wrong package name in `entry_points` |
| C++ build error `fatal error: ... .hpp` | Dependency missing from `find_package`, `ament_target_dependencies` or `package.xml` |
| `launch file not found` | `launch/` folder not listed in `data_files` or `install(DIRECTORY ...)`; rebuild |
| Gazebo window stays black or crashes | `killall gzserver gzclient`, then relaunch; check graphics drivers in a VM (enable 3D acceleration) |
| Robot does not move in Gazebo | Check `ros2 topic info /cmd_vel`; confirm diff-drive plugin loaded in the Gazebo log |
| RViz shows no robot or scan | Fixed Frame set correctly; LaserScan reliability set to Best Effort; `robot_description` topic selected |
| `use_sim_time` warnings or TF extrapolation | Run Gazebo-related nodes with `use_sim_time: True` consistently |
| Out-of-date build artifacts | `rm -rf build install log` and rebuild |

### C. Viva Question Bank (General)

1. List the major differences between ROS 1 and ROS 2.
2. What is the role of DDS and QoS in ROS 2?
3. Explain nodes, topics, services, actions and parameters with one robotics example each.
4. What is an overlay and an underlay workspace?
5. Why are launch files used in large robot systems?
6. Explain the main elements of a URDF file.
7. What does `robot_state_publisher` do and what does it need as input?
8. What is TF2 and what are `/tf` and `/tf_static`?
9. How does a differential-drive robot turn?
10. Name three sensors used on mobile robots and the ROS 2 message type for each.
11. What are the limits of reactive obstacle avoidance and what is Nav2?
12. How is simulation time different from wall time?

### D. Lab Record Format

Write each experiment in the record in this order:

1. Experiment number, title and date
2. Aim
3. Theory (short, in your own words, with a labelled diagram where useful)
4. Software and hardware requirements
5. Procedure or algorithm
6. Program or commands (include comments)
7. Output (terminal output printed or screenshots, RViz or Gazebo screenshots)
8. Result
9. Viva answers (written)

### E. Suggested Weekly Schedule

The course has 30 contact hours (2 hours per week). The suggested plan below fits 10 experiments plus revision and test; adjust it to your timetable.

| Week | Experiment | Focus |
| --- | --- | --- |
| 1 | 1 | Install ROS 2, basic commands |
| 2 | 2 | Workspace and packages |
| 3 | 3 | Topics: publisher and subscriber |
| 4 | 4 | Services and actions |
| 5 | 5 | Turtlesim control |
| 6 | 6 | Parameters and launch files |
| 7 | 7 | URDF, Xacro and RViz2 |
| 8 | 8 | TF2 |
| 9 | 9 | Sensors in Gazebo |
| 10 | 10 | Autonomous integration |
| 11-12 | Revision and internal test | Repeat weak experiments |

### F. Notes for the Instructor and Student

- Package names, topic names and file paths in this manual are suggestions; faculty may rename them, but keep them consistent across the launch files, `setup.py` entries and commands.
- Experiments 7 to 10 build on each other. Keep the `diff_bot_description`, `sensor_lab` and `auto_nav` packages in the same workspace.
- Gazebo (classic) is used here because it is bundled with Humble through `gazebo_ros_pkgs`. If your lab uses a different simulator or ROS distribution, plugin names and launch files will change.
- Programs were written to be consistent with the Humble APIs, but run each one once on your own machine before the lab, and fix any package-name or path mismatch you find.
