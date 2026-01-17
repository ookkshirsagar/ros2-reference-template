# ROS 2 Project Template

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![ROS 2 Humble](https://img.shields.io/badge/ROS%202-Humble-green.svg)](https://docs.ros.org/en/humble/)
[![ROS 2 Iron](https://img.shields.io/badge/ROS%202-Iron-orange.svg)](https://docs.ros.org/en/iron/)
[![ROS 2 Jazzy](https://img.shields.io/badge/ROS%202-Jazzy-purple.svg)](https://docs.ros.org/en/jazzy/)
## Overview

This repository provides a production-ready ROS 2 workspace template for professional development. It demonstrates standard package structure, build configuration, development workflows, testing, and CI/CD integration for both Python and C++ nodes.

**Intended for:**
- Engineers starting new ROS 2 projects
- Teams establishing workspace conventions
- Developers requiring a clean, minimal baseline
- Organizations implementing ROS 2 best practices

**Not a tutorial.** This template assumes familiarity with ROS 2 concepts. It prioritizes practical structure over pedagogical explanation.

## Supported Platforms and Versions

- **ROS 2 Distributions:** Humble, Iron, Jazzy
- **Operating Systems:** Ubuntu 22.04 (Humble/Iron), Ubuntu 24.04 (Jazzy)
- **Build System:** colcon
- **Languages:** Python 3.10+, C++17
- **DDS Implementations:** FastDDS (default), CycloneDDS, Connext

## Workspace Structure
```text
ros2_ws/
├── src/
│   ├── py_package/
│   │   ├── py_package/
│   │   │   ├── __init__.py
│   │   │   └── talker_listener_node.py
│   │   ├── launch/
│   │   │   └── talker_listener_launch.py
│   │   ├── test/
│   │   │   └── test_talker_listener.py
│   │   ├── resource/
│   │   │   └── py_package
│   │   ├── package.xml
│   │   ├── setup.py
│   │   ├── setup.cfg
│   │   └── README.md
│   └── cpp_package/
│       ├── include/
│       │   └── cpp_package/
│       ├── src/
│       │   └── talker_listener_node.cpp
│       ├── launch/
│       │   └── talker_listener_launch.py
│       ├── test/
│       │   └── test_talker_listener.cpp
│       ├── CMakeLists.txt
│       ├── package.xml
│       └── README.md
├── build/
├── install/
├── log/
├── .github/
│   └── workflows/
│       └── ci.yml
├── .gitignore
├── CHANGELOG.md
├── CONTRIBUTING.md
├── Dockerfile
├── LICENSE
└── README.md
```

The `build/`, `install/`, and `log/` directories are generated during the build process and must not be committed to version control.

## Creating the ROS 2 Workspace
```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
```

Source your ROS 2 installation:
```bash
source /opt/ros/humble/setup.bash  # or iron, jazzy
```

Initialize the workspace by building with no packages:
```bash
colcon build
```

## Dependency Management

### System Dependencies

Install all dependencies using rosdep:
```bash
cd ~/ros2_ws
rosdep update
rosdep install --from-paths src --ignore-src -r -y
```

### Package Dependencies

Declare dependencies in `package.xml`:

- `<depend>` - Build and runtime dependency
- `<build_depend>` - Build-time only
- `<exec_depend>` - Runtime only
- `<test_depend>` - Testing only

Example:
```xml
<depend>rclpy</depend>
<depend>std_msgs</depend>
<test_depend>pytest</test_depend>
```

## Creating Packages

### Python Package
```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python py_package --dependencies rclpy std_msgs
```

Create additional directories:
```bash
mkdir -p py_package/launch
mkdir -p py_package/test
```

### C++ Package
```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_cmake cpp_package --dependencies rclcpp std_msgs
```

Create additional directories:
```bash
mkdir -p cpp_package/launch
mkdir -p cpp_package/test
```

## Python Node

**File:** `src/py_package/py_package/talker_listener_node.py`
```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String


class TalkerListenerNode(Node):
    """
    A ROS 2 node that publishes and subscribes to String messages.
    
    This node demonstrates basic pub/sub patterns in ROS 2 using Python.
    It publishes messages at 1 Hz and logs received messages.
    """
    
    def __init__(self):
        super().__init__('talker_listener_node')
        
        # Publisher
        self.publisher_ = self.create_publisher(String, 'chatter', 10)
        
        # Subscriber
        self.subscription_ = self.create_subscription(
            String,
            'chatter',
            self.listener_callback,
            10
        )
        
        # Timer for publishing
        self.timer_ = self.create_timer(1.0, self.timer_callback)
        self.counter_ = 0
        
        self.get_logger().info('Talker-Listener node initialized')

    def timer_callback(self):
        """Publish a message at regular intervals."""
        msg = String()
        msg.data = f'Hello from Python: {self.counter_}'
        self.publisher_.publish(msg)
        self.get_logger().info(f'Publishing: "{msg.data}"')
        self.counter_ += 1

    def listener_callback(self, msg):
        """Handle received messages."""
        self.get_logger().info(f'Received: "{msg.data}"')


def main(args=None):
    rclpy.init(args=args)
    node = TalkerListenerNode()
    
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

**File:** `src/py_package/setup.py`
```python
from setuptools import find_packages, setup
from glob import glob
import os

package_name = 'py_package'

setup(
    name=package_name,
    version='1.0.0',
    packages=find_packages(exclude=['test']),
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
        (os.path.join('share', package_name, 'launch'), glob('launch/*.py')),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='Your Name',
    maintainer_email='you@example.com',
    description='Python package with publisher and subscriber',
    license='MIT',
    tests_require=['pytest'],
    entry_points={
        'console_scripts': [
            'talker_listener_node = py_package.talker_listener_node:main',
        ],
    },
)
```

**File:** `src/py_package/setup.cfg`
```text
[develop]
script_dir=$base/lib/py_package
[install]
install_scripts=$base/lib/py_package
```

**File:** `src/py_package/package.xml`
```xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>py_package</name>
  <version>1.0.0</version>
  <description>Python package with publisher and subscriber</description>
  <maintainer email="you@example.com">Your Name</maintainer>
  <license>MIT</license>

  <depend>rclpy</depend>
  <depend>std_msgs</depend>

  <test_depend>ament_copyright</test_depend>
  <test_depend>ament_flake8</test_depend>
  <test_depend>ament_pep257</test_depend>
  <test_depend>python3-pytest</test_depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>
```

## C++ Node

**File:** `src/cpp_package/src/talker_listener_node.cpp`
```cpp
#include <chrono>
#include <memory>
#include <string>

#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"

using namespace std::chrono_literals;

/**
 * @brief A ROS 2 node that publishes and subscribes to String messages.
 * 
 * This node demonstrates basic pub/sub patterns in ROS 2 using C++.
 * It publishes messages at 1 Hz and logs received messages.
 */
class TalkerListenerNode : public rclcpp::Node
{
public:
  TalkerListenerNode()
  : Node("talker_listener_node"), counter_(0)
  {
    // Publisher
    publisher_ = this->create_publisher<std_msgs::msg::String>("chatter", 10);
    
    // Subscriber
    subscription_ = this->create_subscription<std_msgs::msg::String>(
      "chatter", 10,
      std::bind(&TalkerListenerNode::listener_callback, this, std::placeholders::_1));
    
    // Timer for publishing
    timer_ = this->create_wall_timer(
      1s, std::bind(&TalkerListenerNode::timer_callback, this));
    
    RCLCPP_INFO(this->get_logger(), "Talker-Listener node initialized");
  }

private:
  /**
   * @brief Publish a message at regular intervals.
   */
  void timer_callback()
  {
    auto message = std_msgs::msg::String();
    message.data = "Hello from C++: " + std::to_string(counter_);
    RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());
    publisher_->publish(message);
    counter_++;
  }

  /**
   * @brief Handle received messages.
   * @param msg The received message
   */
  void listener_callback(const std_msgs::msg::String::SharedPtr msg)
  {
    RCLCPP_INFO(this->get_logger(), "Received: '%s'", msg->data.c_str());
  }

  rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
  rclcpp::Subscription<std_msgs::msg::String>::SharedPtr subscription_;
  rclcpp::TimerBase::SharedPtr timer_;
  size_t counter_;
};

int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<TalkerListenerNode>());
  rclcpp::shutdown();
  return 0;
}
```

**File:** `src/cpp_package/CMakeLists.txt`
```cmake
cmake_minimum_required(VERSION 3.8)
project(cpp_package)

# Compiler options
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)

# Build executable
add_executable(talker_listener_node src/talker_listener_node.cpp)
ament_target_dependencies(talker_listener_node rclcpp std_msgs)

# Install targets
install(TARGETS
  talker_listener_node
  DESTINATION lib/${PROJECT_NAME}
)

# Install launch files
install(DIRECTORY
  launch
  DESTINATION share/${PROJECT_NAME}
)

# Testing
if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
  
  # Add gtest
  find_package(ament_cmake_gtest REQUIRED)
  ament_add_gtest(${PROJECT_NAME}_test test/test_talker_listener.cpp)
  target_link_libraries(${PROJECT_NAME}_test ${PROJECT_NAME})
  ament_target_dependencies(${PROJECT_NAME}_test rclcpp std_msgs)
endif()

ament_package()
```

**File:** `src/cpp_package/package.xml`
```xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>cpp_package</name>
  <version>1.0.0</version>
  <description>C++ package with publisher and subscriber</description>
  <maintainer email="you@example.com">Your Name</maintainer>
  <license>MIT</license>

  <buildtool_depend>ament_cmake</buildtool_depend>

  <depend>rclcpp</depend>
  <depend>std_msgs</depend>

  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>
  <test_depend>ament_cmake_gtest</test_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

## Launch Files

### Python Package Launch File

**File:** `src/py_package/launch/talker_listener_launch.py`
```python
from launch import LaunchDescription
from launch_ros.actions import Node


def generate_launch_description():
    """Generate launch description for Python talker-listener node."""
    return LaunchDescription([
        Node(
            package='py_package',
            executable='talker_listener_node',
            name='talker_listener_node',
            output='screen',
            parameters=[],
            remappings=[],
            arguments=['--ros-args', '--log-level', 'info']
        )
    ])
```

### C++ Package Launch File

**File:** `src/cpp_package/launch/talker_listener_launch.py`
```python
from launch import LaunchDescription
from launch_ros.actions import Node


def generate_launch_description():
    """Generate launch description for C++ talker-listener node."""
    return LaunchDescription([
        Node(
            package='cpp_package',
            executable='talker_listener_node',
            name='talker_listener_node',
            output='screen',
            parameters=[],
            remappings=[],
            arguments=['--ros-args', '--log-level', 'info']
        )
    ])
```

## Build and Run Instructions

### Build All Packages
```bash
cd ~/ros2_ws
colcon build
```

Source the workspace overlay:
```bash
source install/setup.bash
```

### Build with Optimization

For production deployment:
```bash
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
```

For debugging:
```bash
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Debug
```

### Run Python Node
```bash
ros2 launch py_package talker_listener_launch.py
```

### Run C++ Node
```bash
ros2 launch cpp_package talker_listener_launch.py
```

### Run Nodes Directly
```bash
ros2 run py_package talker_listener_node
ros2 run cpp_package talker_listener_node
```

## Development Workflow

### Symlink Install (Python Only)

For rapid Python development without rebuilding:
```bash
colcon build --symlink-install
```

Changes to Python source files take effect immediately. C++ packages still require rebuild.

### Build Single Package
```bash
colcon build --packages-select py_package
```

### Build with Dependencies
```bash
colcon build --packages-up-to cpp_package
```

### Build with Verbose Output
```bash
colcon build --event-handlers console_direct+
```

### Parallel Build
```bash
colcon build --parallel-workers 4
```

### Clean Build
```bash
colcon build --cmake-clean-cache
```

## Testing

### Run All Tests
```bash
cd ~/ros2_ws
colcon test
colcon test-result --verbose
```

### Run Tests for Specific Package
```bash
colcon test --packages-select py_package
colcon test-result --verbose
```

### Python Unit Tests

**File:** `src/py_package/test/test_talker_listener.py`
```python
import unittest
import rclpy
from py_package.talker_listener_node import TalkerListenerNode


class TestTalkerListenerNode(unittest.TestCase):
    
    @classmethod
    def setUpClass(cls):
        rclpy.init()
    
    @classmethod
    def tearDownClass(cls):
        rclpy.shutdown()
    
    def test_node_creation(self):
        """Test that the node can be created."""
        node = TalkerListenerNode()
        self.assertIsNotNone(node)
        node.destroy_node()
    
    def test_publisher_exists(self):
        """Test that the publisher is created."""
        node = TalkerListenerNode()
        self.assertEqual(len(node.publishers), 1)
        node.destroy_node()
    
    def test_subscription_exists(self):
        """Test that the subscription is created."""
        node = TalkerListenerNode()
        self.assertEqual(len(node.subscriptions), 1)
        node.destroy_node()


if __name__ == '__main__':
    unittest.main()
```

### C++ Unit Tests

**File:** `src/cpp_package/test/test_talker_listener.cpp`
```cpp
#include <gtest/gtest.h>
#include <rclcpp/rclcpp.hpp>
#include <std_msgs/msg/string.hpp>

TEST(TalkerListenerTest, NodeCreation) {
  rclcpp::init(0, nullptr);
  auto node = std::make_shared<rclcpp::Node>("test_node");
  ASSERT_NE(node, nullptr);
  rclcpp::shutdown();
}

TEST(TalkerListenerTest, PublisherCreation) {
  rclcpp::init(0, nullptr);
  auto node = std::make_shared<rclcpp::Node>("test_node");
  auto publisher = node->create_publisher<std_msgs::msg::String>("test_topic", 10);
  ASSERT_NE(publisher, nullptr);
  rclcpp::shutdown();
}

int main(int argc, char** argv) {
  testing::InitGoogleTest(&argc, argv);
  return RUN_ALL_TESTS();
}
```

### Code Coverage

Build with coverage flags:
```bash
colcon build --cmake-args -DCMAKE_CXX_FLAGS="--coverage"
colcon test
```

Generate coverage report:
```bash
lcov --capture --directory build --output-file coverage.info
lcov --remove coverage.info '/usr/*' --output-file coverage.info
lcov --list coverage.info
```

## Code Quality

### Static Analysis

#### C++ Linting
```bash
ament_cpplint src/cpp_package/src/
```

#### Python Linting
```bash
ament_flake8 src/py_package/
ament_pep257 src/py_package/
```

#### C++ Static Analysis
```bash
clang-tidy src/cpp_package/src/*.cpp -- -I/opt/ros/humble/include
```

### Code Formatting

#### C++ (clang-format)
```bash
find src/cpp_package -name '*.cpp' -o -name '*.hpp' | xargs clang-format -i
```

#### Python (black)
```bash
black src/py_package/
```

## Performance Considerations

### QoS Settings

Configure Quality of Service for different communication patterns:
```python
from rclpy.qos import QoSProfile, ReliabilityPolicy, HistoryPolicy, DurabilityPolicy

# Reliable communication
reliable_qos = QoSProfile(
    depth=10,
    reliability=ReliabilityPolicy.RELIABLE,
    history=HistoryPolicy.KEEP_LAST,
    durability=DurabilityPolicy.VOLATILE
)

self.publisher_ = self.create_publisher(String, 'topic', reliable_qos)
```

C++ equivalent:
```cpp
auto qos = rclcpp::QoS(rclcpp::KeepLast(10))
  .reliable()
  .durability_volatile();

publisher_ = this->create_publisher<std_msgs::msg::String>("topic", qos);
```

### Build Optimization

Release build with optimizations:
```bash
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_FLAGS="-O3"
```

### DDS Configuration

Switch to CycloneDDS for better performance:
```bash
sudo apt install ros-humble-rmw-cyclonedds-cpp
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
```

## Cleaning the Workspace

### Remove Build Artifacts
```bash
cd ~/ros2_ws
rm -rf build/ install/ log/
```

### Clean and Rebuild
```bash
rm -rf build/ install/ log/
colcon build
```

### Clean Specific Package
```bash
rm -rf build/py_package install/py_package
colcon build --packages-select py_package
```

## Continuous Integration

### GitHub Actions Workflow

**File:** `.github/workflows/ci.yml`
```yaml
name: ROS 2 CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build-and-test:
    runs-on: ubuntu-22.04
    strategy:
      matrix:
        ros_distribution: [humble, iron]
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
      
      - name: Setup ROS 2
        uses: ros-tooling/setup-ros@v0.7
        with:
          required-ros-distributions: ${{ matrix.ros_distribution }}
      
      - name: Install dependencies
        run: |
          source /opt/ros/${{ matrix.ros_distribution }}/setup.bash
          rosdep update
          rosdep install --from-paths src --ignore-src -r -y
      
      - name: Build workspace
        run: |
          source /opt/ros/${{ matrix.ros_distribution }}/setup.bash
          colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
      
      - name: Run tests
        run: |
          source /opt/ros/${{ matrix.ros_distribution }}/setup.bash
          source install/setup.bash
          colcon test --return-code-on-test-failure
      
      - name: Show test results
        if: always()
        run: colcon test-result --verbose
      
      - name: Run linters
        run: |
          source /opt/ros/${{ matrix.ros_distribution }}/setup.bash
          ament_flake8 src/py_package/
          ament_cpplint src/cpp_package/
```

## Docker Deployment

### Dockerfile

**File:** `Dockerfile`
```dockerfile
FROM ros:humble

# Install dependencies
RUN apt-get update && apt-get install -y \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

# Create workspace
WORKDIR /ros2_ws

# Copy source files
COPY src ./src

# Install rosdep dependencies
RUN apt-get update && \
    rosdep update && \
    rosdep install --from-paths src --ignore-src -r -y && \
    rm -rf /var/lib/apt/lists/*

# Build workspace
RUN . /opt/ros/humble/setup.sh && \
    colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release

# Source workspace
RUN echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc && \
    echo "source /ros2_ws/install/setup.bash" >> ~/.bashrc

CMD ["bash"]
```

### Build and Run Docker Container
```bash
docker build -t ros2_project .
docker run -it --rm --net=host ros2_project
```

Inside container:
```bash
ros2 launch py_package talker_listener_launch.py
```

### Docker Compose

**File:** `docker-compose.yml`
```yaml
version: '3.8'

services:
  ros2_node:
    build: .
    network_mode: host
    command: ros2 launch py_package talker_listener_launch.py
    volumes:
      - ./src:/ros2_ws/src
    environment:
      - ROS_DOMAIN_ID=0
```

## Advanced Configuration

### Environment Variables
```bash
# Network isolation
export ROS_DOMAIN_ID=42

# Change DDS implementation
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp

# Disable shared memory
export RMW_FASTRTPS_USE_QOS_FROM_XML=1

# Set log level
export RCUTILS_CONSOLE_OUTPUT_FORMAT="[{severity}] [{name}]: {message}"
```

### Parameter Files

**File:** `config/params.yaml`
```yaml
talker_listener_node:
  ros__parameters:
    publish_rate: 1.0
    queue_size: 10
    topic_name: "chatter"
```

Load parameters in launch file:
```python
from launch import LaunchDescription
from launch_ros.actions import Node
from ament_index_python.packages import get_package_share_directory
import os

def generate_launch_description():
    config = os.path.join(
        get_package_share_directory('py_package'),
        'config',
        'params.yaml'
    )
    
    return LaunchDescription([
        Node(
            package='py_package',
            executable='talker_listener_node',
            parameters=[config]
        )
    ])
```

## Troubleshooting

### Common Issues

**Problem:** `package 'X' not found`
```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
rosdep install --from-paths src --ignore-src -r -y
```

**Problem:** Changes not reflected after build

- Python: Rebuild with `--symlink-install`
- C++: Clean and rebuild package
```bash
rm -rf build/cpp_package install/cpp_package
colcon build --packages-select cpp_package
```

**Problem:** Launch file not found

Verify launch directory installation in `setup.py`:
```python
data_files=[
    # ...
    (os.path.join('share', package_name, 'launch'), glob('launch/*.py')),
]
```

Or in `CMakeLists.txt`:
```cmake
install(DIRECTORY
  launch
  DESTINATION share/${PROJECT_NAME}
)
```

**Problem:** Multiple nodes conflict

Use unique node names or namespaces:
```python
Node(
    package='py_package',
    executable='talker_listener_node',
    name='node_1',
    namespace='robot1'
)
```

### Debug Mode

#### C++ Debugging with GDB
```bash
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Debug
ros2 run --prefix 'gdb -ex run --args' cpp_package talker_listener_node
```

#### Python Debugging with pdb

Add to your Python code:
```python
import pdb; pdb.set_trace()
```

#### Enable Verbose Logging
```bash
ros2 run py_package talker_listener_node --ros-args --log-level debug
```

### Network Debugging

Check active nodes:
```bash
ros2 node list
```

Check active topics:
```bash
ros2 topic list
ros2 topic echo /chatter
```

Check message flow:
```bash
ros2 topic hz /chatter
ros2 topic bw /chatter
```

## Git Ignore

**File:** `.gitignore`
```text
# ROS 2 build artifacts
build/
install/
log/

# Python
*.pyc
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
pip-log.txt
pip-delete-this-directory.txt
.pytest_cache/

# C++
*.o
*.a
*.so
*.out
*.app
*.i*86
*.x86_64
*.hex

# IDE
.vscode/
.idea/
*.swp
*.swo
*~
.DS_Store

# CMake
CMakeCache.txt
CMakeFiles/
cmake_install.cmake
Makefile

# Coverage
*.gcda
*.gcno
*.gcov
coverage.info
coverage/

# Documentation
doc/html/
doc/latex/
```

## Rules and Best Practices

### Package Naming

- Use lowercase with underscores
- Be descriptive and concise
- Avoid generic names like `utils` or `common`
- Use consistent prefixes for related packages (e.g., `robot_driver`, `robot_msgs`)

### Node Naming

- Nodes should have clear, singular purposes
- Name reflects functionality (e.g., `camera_driver_node`, not `node1`)
- Use consistent naming: `<function>_node` pattern
- Avoid abbreviations unless widely recognized

### Topics and Services

- Use lowercase with underscores
- Namespace related topics (e.g., `/camera/image`, `/camera/info`)
- Use descriptive names that indicate data type
- Follow ROS naming conventions (REP-144)

### Code Organization

- One node per executable in simple cases
- Use libraries for shared code across nodes
- Keep nodes focused and composable
- Separate concerns: drivers, logic, visualization
- Use composition for complex multi-node systems

### Build Practices

- Always build from workspace root
- Source workspace overlay after every build
- Use `--symlink-install` during Python development
- Run tests before committing changes
- Use release builds for deployment

### Dependencies

- Declare all dependencies in `package.xml`
- Keep dependency lists minimal and explicit
- Prefer standard ROS packages over custom solutions
- Document non-ROS dependencies in README
- Use rosdep for system dependency management

### Documentation

- Document public APIs with docstrings/Doxygen
- Maintain package-level README files
- Keep CHANGELOG.md updated
- Document launch file parameters
- Include usage examples

### Version Control

- Never commit build artifacts
- Use meaningful commit messages
- Tag releases with semantic versioning
- Keep branches focused and short-lived
- Review code before merging

## Contributing

### Code Standards

#### Python
- Follow PEP 8 style guide
- Use type hints where appropriate
- Write docstrings for all public functions
- Maximum line length: 100 characters

#### C++
- Follow ROS 2 C++ Style Guide
- Use modern C++ features (C++17)
- Document public APIs with Doxygen
- Use RAII and smart pointers

### Pre-commit Checks

Run before committing:
```bash
# Python linting
ament_flake8 src/py_package/
ament_pep257 src/py_package/

# C++ linting
ament_cpplint src/cpp_package/

# Run tests
colcon test
colcon test-result --verbose
```

### Pull Request Process

1. Create a feature branch from `main`
```bash
   git checkout -b feature/your-feature-name
```

2. Make changes and commit with descriptive messages
```bash
   git commit -m "Add feature: brief description"
```
3. Ensure all tests pass
```bash
   colcon test
```

4. Push branch and create pull request
```bash
   git push origin feature/your-feature-name
```

5. Request review from maintainers
6. Address review comments
7. Merge after approval

### Commit Message Format
```text
<type>: <subject>

<body>

<footer>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

Example:
```text
feat: add parameter configuration for publish rate

Allows users to configure the publishing rate through
a ROS parameter instead of hardcoded value.

Closes #123
```

## Extending This Template

### Adding New Packages
```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type [ament_python|ament_cmake] package_name \
  --dependencies dep1 dep2 dep3
```

### Adding Custom Messages

Create a separate package for message definitions:
```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_cmake my_msgs
```

**File:** `src/my_msgs/msg/CustomMessage.msg`
```text
std_msgs/Header header
string data
float64 value
```

Update `CMakeLists.txt`:
```cmake
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/CustomMessage.msg"
  DEPENDENCIES std_msgs
)
```

Update `package.xml`:
```xml
<build_depend>rosidl_default_generators</build_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```

### Adding Custom Services

**File:** `src/my_msgs/srv/CustomService.srv`
```text
string request_data
---
bool success
string response_data
```

### Adding Actions

**File:** `src/my_msgs/action/CustomAction.action`
```text
# Goal
string goal_data
---
# Result
bool success
---
# Feedback
float32 progress
```

### Multi-Package Projects

Organize related packages under a common directory:
```text
src/
├── my_project/
│   ├── my_project_core/        # Main logic
│   ├── my_project_drivers/     # Hardware interfaces
│   ├── my_project_msgs/        # Custom messages
│   ├── my_project_bringup/     # Launch files
│   └── my_project_description/ # URDF/meshes
```

### Integration with Simulation

Add Gazebo integration:
```bash
sudo apt install ros-humble-gazebo-ros-pkgs
```

Create simulation launch file:
```python
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        IncludeLaunchDescription(
            PythonLaunchDescriptionSource([
                '/opt/ros/humble/share/gazebo_ros/launch/gazebo.launch.py'
            ])
        ),
        Node(
            package='py_package',
            executable='talker_listener_node',
            output='screen'
        )
    ])
```

## Documentation

### Generating API Documentation

#### Python (Sphinx)
```bash
pip install sphinx sphinx-rtd-theme
cd ~/ros2_ws/src/py_package
sphinx-quickstart docs
```

#### C++ (Doxygen)
```bash
sudo apt install doxygen graphviz
cd ~/ros2_ws/src/cpp_package
doxygen -g Doxyfile
doxygen Doxyfile
```

### Package Documentation

Each package should include:

- `README.md` with package-specific details
- API documentation (Sphinx/Doxygen)
- Usage examples
- Configuration parameters
- Known limitations

## Versioning

This project follows [Semantic Versioning](https://semver.org/):

- **MAJOR:** Breaking API changes
- **MINOR:** New features (backward compatible)
- **PATCH:** Bug fixes (backward compatible)

### Changelog

See [CHANGELOG.md](CHANGELOG.md) for release history.

**Example CHANGELOG.md:**
```markdown
# Changelog

## [1.0.0] - 2025-01-17

### Added
- Initial release
- Python and C++ example nodes
- Launch files
- Unit tests
- CI/CD pipeline

### Changed
- N/A

### Deprecated
- N/A

### Removed
- N/A

### Fixed
- N/A

### Security
- N/A
```

## Security

### Reporting Vulnerabilities

Report security issues to: security@yourproject.com

**Do not create public issues for security vulnerabilities.**

### Security Best Practices

- Keep dependencies updated
- Use rosdep for system packages
- Run security audits regularly
- Validate all external inputs
- Use secure communication when needed

### Code Scanning

Static analysis tools:
```bash
# C++ security checks
cppcheck --enable=all src/cpp_package/src/

# Python security checks
bandit -r src/py_package/
```

## Resources

### Official Documentation

- [ROS 2 Documentation](https://docs.ros.org/)
- [ROS 2 Design](https://design.ros2.org/)
- [ROS 2 Humble API](https://docs.ros.org/en/humble/API.html)
- [ROS 2 Tutorials](https://docs.ros.org/en/humble/Tutorials.html)

### Community

- [ROS Discourse](https://discourse.ros.org/)
- [ROS Answers](https://answers.ros.org/)
- [GitHub Discussions](https://github.com/ros2/ros2/discussions)
- [ROS Discord](https://discord.gg/ros)

### Related Projects

- [ros2_control](https://github.com/ros-controls/ros2_control)
- [navigation2](https://github.com/ros-planning/navigation2)
- [moveit2](https://github.com/ros-planning/moveit2)

## License

This project is licensed under the MIT License.
```text
MIT License

Copyright (c) 2025

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## Acknowledgments

This template is built on best practices from the ROS 2 community and incorporates patterns from successful production deployments.

## Support

For questions and support:
- Open an issue on GitHub
- Join ROS Discourse discussions
- Consult official ROS 2 documentation

---

**Maintained by:** Your Name / Organization  
**Last Updated:** 2026-01-17  
**ROS 2 Version:** Humble / Iron / Jazzy
