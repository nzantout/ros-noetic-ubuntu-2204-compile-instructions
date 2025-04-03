# ROS Noetic Compile Instructions on Ubuntu 22.04
Instructions to compile ROS Noetic on Ubuntu 22.04, excerpted from [this Reddit post](https://www.reddit.com/r/ROS/comments/158icpy/compiling_ros1_noetic_from_source_on_ubuntu_2204/) by user [u/M2-TE](https://www.reddit.com/user/M2-TE/).

The commands are based on the official guide, see http://wiki.ros.org/noetic/Installation/Source

1. Get apt packages:
```
sudo apt-get install python3-rosdep python3-rosinstall-generator python3-vcstools python3-vcstool build-essential
```
2. Create directory and download source:
```
sudo rosdep init && rosdep update

mkdir ~/ros_catkin_ws && cd ~/ros_catkin_ws

rosinstall_generator desktop --rosdistro noetic --deps --tar > noetic-desktop.rosinstall

mkdir ./src && vcs import --input noetic-desktop.rosinstall ./src
```
3. The fun part:
Exclude hddtemp from the ros dependencies as it does not exist for jammy
```
rosdep install --from-paths ./src --ignore-packages-from-source --rosdistro noetic -y --skip-keys hddtemp
```
Now we need to fix some of the source files (inside the ./src folder) to be able to compile. First, we need to explicitly set the C++ version of some CMakeLists.txt files to c++17. They each specify either 11 or 14, so just look for those near the beginning of each of these CMake files:

```
resource_retriever

rqt_image_view

geometry/tf

laser_geometry

urdf/urdf

kdl_parser

robot_state_publisher
```

Having done that, there's one .cpp file we need to adjust, namely the one located at `./src/rosconsole/src/rosconsole/impl/rosconsole_log4cxx.cpp`

Some pointers and weak pointers need to be converted to shared pointers. Here is a list of all lines of code I changed:
```
Line 169: logger->addAppender(std::make_shared<ROSConsoleStdioAppender>());
Line 203: return log4cxx::Logger::getLogger(name).get();
Line 219: log4cxx::spi::LoggerRepositoryPtr repo(log4cxx::Logger::getLogger(ROSCONSOLE_ROOT_LOGGER_NAME)->getLoggerRepository());
Line 355: std::shared_ptr<Log4cxxAppender> g_log4cxx_appender;
Line 359: g_log4cxx_appender = std::make_shared<Log4cxxAppender>(appender);
Line 369: g_log4cxx_appender.reset();
Line 386: std::shared_ptr<log4cxx::spi::LoggerRepository>(log4cxx::Logger::getRootLogger()->getLoggerRepository())->shutdown();
```
4. Compile and pray
Now that all files should be corrected, compile and install into the ./install_isolated folder:
```
./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release
```
If you get python errors, simply add `-DPYTHON_EXECUTABLE=/usr/bin/python3` and adjust the path if you installed python3 somewhere else.

Lastly, it is often desired to have ros installs in the usual /opt/ros folder:
```
sudo cp -r ./install_isolated /opt/ros/noetic
```
Now you can use ros1 noetic in all its glory (though you should probably upgrade to a ros2 distribution regardless). Enjoy!
