# 🤖 Gazebo & ROS 2 — From Zero to Hero
### A beginner-friendly guide (written like you're 10, explained till you're an engineer)

---

## 📚 Table of Contents
1. [What is a Robot?](#1-what-is-a-robot)
2. [What is a Simulator?](#2-what-is-a-simulator)
3. [What is Gazebo?](#3-what-is-gazebo)
4. [Gazebo — The Interface](#4-gazebo--the-interface)
5. [SDF Files — How Worlds are Built](#5-sdf-files--how-worlds-are-built)
6. [What is ROS 2?](#6-what-is-ros-2)
7. [ROS 2 — Core Concepts](#7-ros-2--core-concepts)
8. [Topics — How Robots Talk](#8-topics--how-robots-talk)
9. [Nodes — The Workers](#9-nodes--the-workers)
10. [Services & Actions](#10-services--actions)
11. [ROS 2 + Gazebo Together](#11-ros-2--gazebo-together)
12. [Hands-on Exercises](#12-hands-on-exercises)
13. [Cheat Sheet](#13-cheat-sheet)

---

## 1. What is a Robot?

Think of a robot as a machine that has three things:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   SENSORS   │ --> │   BRAIN     │ --> │   MOTORS    │
│ (it feels)  │     │ (it thinks) │     │ (it moves)  │
└─────────────┘     └─────────────┘     └─────────────┘
```

Just like you:
- **Sensors** = your eyes, ears, skin (camera, LiDAR, GPS on a robot)
- **Brain** = your mind (the computer / software)
- **Motors** = your arms and legs (wheels, propellers, joints)

A drone is a robot that flies. A Roomba is a robot that cleans. They all have these three parts.

---

## 2. What is a Simulator?

> Imagine you want to learn to drive. You wouldn't start on a real highway, right? You'd use a driving simulator first — safe, free to crash, no real damage.

That's exactly what robot simulators do.

| Real World | Simulator |
|---|---|
| Costs thousands of dollars | Free |
| If it crashes, it breaks | Just press reset |
| Can only test in one place | Test in any environment |
| Takes weeks to set up | Ready in minutes |
| Weather dependent | Perfect conditions always |

**ROS 2 + Gazebo is the "driving simulator" for robots.**

---

## 3. What is Gazebo?

Gazebo is a **3D robot simulator**. It creates a fake physical world on your computer where:

- Gravity exists ✅
- Objects collide and bounce ✅
- Sensors produce real data ✅
- Physics is accurate ✅

### What Gazebo can simulate

```
🚁 Drones          → quadcopters, fixed-wing planes
🤖 Ground robots   → wheeled robots, legged robots  
🦾 Robot arms      → factory manipulators, grippers
🌊 Underwater      → submarines, underwater drones
🚗 Self-driving    → autonomous cars
```

### The Gazebo family

Gazebo has had many versions. Here's what you need to know:

| Name | Also called | Status |
|---|---|---|
| Gazebo Classic | Gazebo 11 | Old, being retired |
| **Gazebo Harmonic** | **gz-harmonic** | ✅ What you have installed |
| Gazebo Ionic | gz-ionic | Newer, less stable |

> You installed **Gazebo Harmonic** — the current stable version. Good choice.

### On Mac specifically

Because of how macOS works, Gazebo splits into two parts:

```
Terminal 1:  gz sim -s world.sdf   →  SERVER (physics engine, runs in VM)
Terminal 2:  gz sim -g             →  GUI    (3D display, runs on Mac)
```

Linux users can run both in one command. Mac users always need two. This is normal.

---

## 4. Gazebo — The Interface

When you open Gazebo (`gz sim -g`), here's what you're looking at:

```
┌─────────────────────────────────────────────────────┐
│  ▶ ⏸ ⏹   [toolbar]              Time: 0.00s        │
├──────────┬──────────────────────────────┬────────────┤
│          │                              │            │
│  Entity  │                              │ Component  │
│   Tree   │       3D VIEWPORT            │ Inspector  │
│          │                              │            │
│ - world  │    (your robot lives here)   │ position:  │
│   - drone│                              │ rotation:  │
│   - wall │                              │ mass:      │
│          │                              │            │
└──────────┴──────────────────────────────┴────────────┘
```

### Navigation in the 3D viewport

| Action | How |
|---|---|
| Rotate view | Left click + drag |
| Pan | Middle click + drag |
| Zoom | Scroll wheel |
| Select object | Left click on it |
| Move object | Select → use arrows |

### The Play button

The simulation is **paused by default**. Hit ▶ to start physics. Objects will fall, robots will move, sensors will start publishing data.

> **Key insight:** Even when paused, you can inspect everything. Play only when you want physics to run.

---

## 5. SDF Files — How Worlds are Built

SDF = **Simulation Description Format**. It's just an XML file that describes your world.

Think of it like a recipe — Gazebo reads the recipe and cooks up the world.

### A simple SDF example

```xml
<?xml version="1.0" ?>
<sdf version="1.8">
  <world name="my_first_world">

    <!-- Add sunlight -->
    <light name="sun" type="directional">
      <cast_shadows>true</cast_shadows>
    </light>

    <!-- Add a ground plane -->
    <model name="ground_plane">
      <static>true</static>
      <link name="link">
        <collision name="collision">
          <geometry><plane><normal>0 0 1</normal></plane></geometry>
        </collision>
      </link>
    </model>

    <!-- Add a simple box -->
    <model name="my_box">
      <pose>0 0 0.5 0 0 0</pose>  <!-- x y z roll pitch yaw -->
      <link name="link">
        <inertial><mass>1.0</mass></inertial>
        <visual name="visual">
          <geometry><box><size>1 1 1</size></box></geometry>
        </visual>
        <collision name="collision">
          <geometry><box><size>1 1 1</size></box></geometry>
        </collision>
      </link>
    </model>

  </world>
</sdf>
```

### SDF structure breakdown

```
world                    ← the entire environment
├── light                ← sun, lamps, lighting
├── model                ← any physical object
│   ├── pose             ← where it is (x y z roll pitch yaw)
│   ├── static           ← true = it won't move (walls, ground)
│   └── link             ← a rigid body part
│       ├── visual       ← how it looks (shape, colour)
│       ├── collision    ← how it physically interacts
│       ├── inertial     ← mass and weight
│       └── sensor       ← camera, LiDAR, IMU attached here
└── plugin               ← extra behaviour (physics, ROS bridge)
```

### Pose explained

`<pose>x y z roll pitch yaw</pose>`

```
x   → forward/backward (metres)
y   → left/right (metres)  
z   → up/down (metres)
roll  → tilt left/right (radians)
pitch → tilt forward/back (radians)
yaw   → rotate left/right (radians)
```

> Your `world.sdf` in AeroClub-SLAM uses this exact format to place the drone and house model in the simulation.

---

## 6. What is ROS 2?

> Imagine you're building a robot with 10 different parts — a camera, a GPS, a motor controller, a map builder, a path planner, etc. Each part is written by a different person in a different programming language. How do they all talk to each other?

**That's exactly what ROS 2 solves.**

ROS 2 = **Robot Operating System 2**

Despite the name, it's not really an operating system. It's a **communication framework** — a set of tools and rules that let different pieces of robot software talk to each other.

### ROS 2 vs ROS 1

| | ROS 1 | ROS 2 |
|---|---|---|
| Year | 2007 | 2017 |
| Real-time support | ❌ | ✅ |
| Security | ❌ | ✅ |
| Multi-robot | Hard | Easy |
| Status | End of life 2025 | ✅ Active |

> Always use ROS 2. You installed **ROS 2 Humble** — the current LTS version.

### The ROS 2 ecosystem

```
                    ┌─────────────────────────────┐
                    │          ROS 2              │
                    │                             │
    ┌───────────────┼───────────────┐             │
    │               │               │             │
┌───▼───┐      ┌────▼────┐    ┌────▼────┐        │
│ Tools │      │Packages │    │  Comm   │        │
│       │      │         │    │         │        │
│ RViz2 │      │slam_tool│    │ Topics  │        │
│ ros2  │      │   box   │    │Services │        │
│  bag  │      │  nav2   │    │ Actions │        │
└───────┘      └─────────┘    └─────────┘        │
                    └─────────────────────────────┘
```

---

## 7. ROS 2 — Core Concepts

### The Graph

Every ROS 2 system is a **graph** of nodes connected by topics:

```
[Camera Node] ──/image──► [Object Detector Node] ──/objects──► [Path Planner Node]
                                                                        │
                                                                    /cmd_vel
                                                                        │
                                                                        ▼
                                                               [Motor Controller]
```

Each box is a **node**. Each arrow is a **topic**. Data flows like water through pipes.

---

## 8. Topics — How Robots Talk

A **topic** is like a radio channel. Anyone can broadcast on it, anyone can listen to it.

```
Publisher                    Topic                   Subscriber
(broadcasts)               (the channel)             (listens)

[LiDAR sensor] ──────────► /scan ──────────────► [SLAM node]
[Camera]       ──────────► /image ─────────────► [Object detector]
[GPS]          ──────────► /fix ───────────────► [Path planner]
[Path planner] ──────────► /cmd_vel ───────────► [Motor driver]
```

### Useful topic commands

```bash
# See all active topics
ros2 topic list

# See what's being published on a topic
ros2 topic echo /scan

# See how fast a topic publishes (Hz)
ros2 topic hz /scan

# See the data type of a topic
ros2 topic info /scan
```

### Common topic names you'll see

| Topic | What it carries |
|---|---|
| `/scan` | LiDAR laser scan data |
| `/image_raw` | Raw camera image |
| `/cmd_vel` | Velocity commands (move robot) |
| `/odom` | Odometry (where robot thinks it is) |
| `/map` | The occupancy grid map |
| `/tf` | Coordinate transforms |
| `/imu` | Accelerometer & gyroscope data |

### Messages

Every topic carries a specific **message type**. Like how a radio channel broadcasts only audio, not video.

```bash
# Common message types
sensor_msgs/LaserScan      → LiDAR data
sensor_msgs/Image          → camera image  
geometry_msgs/Twist        → velocity (linear + angular)
nav_msgs/OccupancyGrid     → a 2D map
nav_msgs/Odometry          → position + velocity
```

---

## 9. Nodes — The Workers

A **node** is a single program that does one job.

> Think of a restaurant kitchen. One chef chops vegetables (camera node), another cooks (processor node), another plates (output node). Each has one job, they pass food between them.

```bash
# See all running nodes
ros2 node list

# See details about a node
ros2 node info /slam_toolbox

# Run a simple test node
ros2 run demo_nodes_py talker
```

### Writing your own node (Python)

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MyNode(Node):
    def __init__(self):
        super().__init__('my_node')          # node name
        
        # Create a publisher on topic /hello
        self.publisher = self.create_publisher(String, '/hello', 10)
        
        # Create a timer that runs every 1 second
        self.timer = self.create_timer(1.0, self.timer_callback)
        
        self.get_logger().info('Node started!')

    def timer_callback(self):
        msg = String()
        msg.data = 'Hello from my robot!'
        self.publisher.publish(msg)

def main():
    rclpy.init()
    node = MyNode()
    rclpy.spin(node)      # keeps node alive
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### Writing a subscriber node

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class ListenerNode(Node):
    def __init__(self):
        super().__init__('listener_node')
        
        # Subscribe to /hello topic
        self.subscription = self.create_subscription(
            String,
            '/hello',
            self.listener_callback,
            10)

    def listener_callback(self, msg):
        self.get_logger().info(f'I heard: {msg.data}')

def main():
    rclpy.init()
    node = ListenerNode()
    rclpy.spin(node)
    rclpy.shutdown()
```

---

## 10. Services & Actions

Topics are for continuous data streams. Sometimes you need a **request → response** pattern instead.

### Services (quick, one-time requests)

```
Client                              Server
  │                                   │
  │ ── request: "take a photo" ──────► │
  │                                   │ (does the work)
  │ ◄── response: [image data] ─────── │
```

```bash
# List all services
ros2 service list

# Call a service manually
ros2 service call /take_photo std_srvs/srv/Trigger
```

### Actions (long tasks with feedback)

Used for things that take time — like "fly to this GPS coordinate".

```
Client                              Server
  │ ── goal: "go to x=5, y=3" ──────► │
  │ ◄── feedback: "50% done" ───────── │  (updates while working)
  │ ◄── feedback: "80% done" ───────── │
  │ ◄── result: "arrived!" ──────────── │
```

```bash
# List all actions
ros2 action list
```

### Summary of communication types

| Type | Pattern | Use when |
|---|---|---|
| Topic | publish/subscribe | Continuous sensor data |
| Service | request/response | Quick one-time tasks |
| Action | goal/feedback/result | Long tasks needing progress updates |

---

## 11. ROS 2 + Gazebo Together

This is where the magic happens. Gazebo simulates the physical world, ROS 2 provides the software brain. They connect through a **bridge**.

```
┌─────────────────────┐         ┌──────────────────────┐
│      GAZEBO         │         │       ROS 2          │
│                     │         │                      │
│  Virtual drone  ────┼─bridge──┼──► /scan topic       │
│  Virtual LiDAR  ────┼─bridge──┼──► /image topic      │
│  Virtual motors ◄───┼─bridge──┼─── /cmd_vel topic    │
│  Virtual world      │         │                      │
└─────────────────────┘         └──────────────────────┘
                    ros_gz_bridge
                  (what you compiled!)
```

### The bridge you built

Remember the 10-minute compile step? That was `ros_gz_bridge` — the translator between Gazebo's internal messages and ROS 2 topics.

Without it:
- Gazebo runs but ROS 2 can't see the sensor data
- ROS 2 nodes can't send commands to the Gazebo robot

With it:
- LiDAR data from Gazebo appears on `/scan` in ROS 2
- SLAM toolbox reads `/scan` and builds a map
- Nav2 reads the map and plans paths
- Commands go back through the bridge to move the robot

### The full SLAM pipeline

```
Gazebo (virtual drone flies)
    │
    │ LiDAR data
    ▼
ros_gz_bridge
    │
    │ /scan topic
    ▼
slam_toolbox
    │
    │ /map topic + /tf
    ▼
RViz2 (you see the map building live)
    │
    │ (optional) /goal_pose
    ▼
Nav2 (autonomous navigation)
    │
    │ /cmd_vel
    ▼
ros_gz_bridge
    │
    ▼
Gazebo (drone moves!)
```

---

## 12. Hands-on Exercises

### Exercise 1 — Explore topics in your SLAM simulation
> Start your 3-terminal SLAM setup, then in a 4th terminal:

```bash
# Enter VM
multipass shell aero-sim

# See all topics the drone simulation publishes
ros2 topic list

# Watch live LiDAR data
ros2 topic echo /scan

# Check how fast LiDAR publishes
ros2 topic hz /scan
```

### Exercise 2 — See the node graph
```bash
# Inside VM, with simulation running
rqt_graph
```
This shows a visual diagram of all nodes and topics connected together.

### Exercise 3 — Record and replay sensor data
```bash
# Record all topics for 30 seconds
ros2 bag record -a -o my_first_recording

# Play it back later (no simulation needed!)
ros2 bag play my_first_recording
```
This is hugely useful — record one good run, replay it 100 times for testing.

### Exercise 4 — Write your first publisher
Save this as `hello_drone.py` and run it in the VM:

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class DroneGreeter(Node):
    def __init__(self):
        super().__init__('drone_greeter')
        self.pub = self.create_publisher(String, '/drone_status', 10)
        self.timer = self.create_timer(2.0, self.greet)
        
    def greet(self):
        msg = String()
        msg.data = 'Drone is alive and flying!'
        self.pub.publish(msg)
        self.get_logger().info(msg.data)

def main():
    rclpy.init()
    rclpy.spin(DroneGreeter())
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

```bash
python3 hello_drone.py
```

---

## 13. Cheat Sheet

### Gazebo commands
```bash
gz sim -s world.sdf       # start physics server
gz sim -g                 # start 3D display
gz sim shapes.sdf         # open built-in demo (Linux only)
gz topic -l               # list all Gazebo topics
gz model -l               # list all models in simulation
```

### ROS 2 commands
```bash
ros2 topic list            # see all topics
ros2 topic echo /scan      # read a topic live
ros2 topic hz /scan        # check publish rate
ros2 topic info /scan      # see message type
ros2 node list             # see all nodes
ros2 node info /slam       # details about a node
ros2 service list          # see all services
ros2 action list           # see all actions
ros2 bag record -a -o name # record everything
ros2 bag play name         # replay recording
rqt_graph                  # visualise node graph
```

### Key terms quick reference
| Term | One line explanation |
|---|---|
| Node | A program that does one job |
| Topic | A named data stream (like a radio channel) |
| Publisher | A node that broadcasts on a topic |
| Subscriber | A node that listens to a topic |
| Message | The data type carried by a topic |
| Service | Request → Response (quick) |
| Action | Goal → Feedback → Result (slow) |
| SDF | XML file describing a Gazebo world |
| Bridge | Translator between Gazebo and ROS 2 |
| SLAM | Build a map + know your location simultaneously |
| LiDAR | Laser sensor that measures distances |
| Occupancy grid | Map where each cell = free/occupied/unknown |

---

*Made for AeroClub SLAM Project — Season 2025*
