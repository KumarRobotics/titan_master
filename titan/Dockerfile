FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive
RUN echo "APT::Get::Assume-Yes \"true\";\nAPT::Get::allow \"true\";" | tee -a  /etc/apt/apt.conf.d/90_no_prompt
RUN apt update
RUN apt install -y vim gnupg

## Install ROS 
RUN apt-get update && apt-get install -y lsb-release && apt-get clean all
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
RUN apt install curl -y
RUN curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -

RUN apt update
RUN apt install ros-noetic-desktop-full -y
RUN apt install python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential -y
RUN apt install -y python3-rosdep python3-pip

RUN apt install -y python3-catkin-tools
RUN apt-get install -qy g++ libeigen3-dev git

## Install dependencies of ROFL
RUN git clone --depth 1 --branch v0.6.0 https://github.com/google/glog.git && cd glog && cmake -S . -B build -G "Unix Makefiles" -DCMAKE_CXX_STANDARD=17 && cmake --build build && cmake --build build --target install
RUN git clone --depth 1 --branch 8.1.0 https://github.com/fmtlib/fmt.git && cd fmt && cmake -DCMAKE_POSITION_INDEPENDENT_CODE=TRUE -DCMAKE_CXX_STANDARD=17 -DFMT_TEST=False && make install
RUN git clone --depth 1 --branch 20220623.0 https://github.com/abseil/abseil-cpp.git && cd abseil-cpp && mkdir build && cd build && cmake -DABSL_BUILD_TESTING=OFF -DCMAKE_CXX_STANDARD=17 -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_POSITION_INDEPENDENT_CODE=ON .. && sudo cmake --build . --target install
RUN git clone https://github.com/strasdat/Sophus.git && cd Sophus && git checkout 785fef3 && mkdir build && cd build && cmake -DBUILD_SOPHUS_TESTS=OFF -DBUILD_SOPHUS_EXAMPLES=OFF -DCMAKE_CXX_STANDARD=17 .. && make install -j8
RUN git clone https://github.com/google/benchmark.git && cd benchmark && cmake -E make_directory "build" &&  cmake -E chdir "build" cmake -DBENCHMARK_DOWNLOAD_DEPENDENCIES=on -DCMAKE_BUILD_TYPE=Release ../ && cmake --build "build" --config Release --target install

## Install dependencies for OpenCV
ENV HOME /root
WORKDIR /root
RUN mkdir Repos
RUN cd Repos && git clone --recursive https://github.com/stevenlovegrove/Pangolin.git && cd Pangolin && ./scripts/install_prerequisites.sh recommended && cmake -B build && cd build && make -j$(nproc) install 
RUN cd Repos && git clone https://github.com/iandouglas96/ORB_SLAM3.git && cd ORB_SLAM3 && \
	rm -rf Thirdparty/Sophus && \
	sed -i '/${PROJECT_SOURCE_DIR}\/Thirdparty\/Sophus/c\#${PROJECT_SOURCE_DIR}\/Thirdparty\/Sophus' CMakeLists.txt && \
	sed -i '/#include "Thirdparty\/Sophus\/sophus\/geometry.hpp"/c\#include "sophus\/geometry.hpp"' /root/Repos/ORB_SLAM3/include/Frame.h && \
	sed -i '/#include "Thirdparty\/Sophus\/sophus\/geometry.hpp"/c\#include "sophus\/geometry.hpp"' /root/Repos/ORB_SLAM3/include/Converter.h && \
	sed -i '/#include "Thirdparty\/Sophus\/sophus\/sim3.hpp"/c\#include "sophus\/sim3.hpp"' /root/Repos/ORB_SLAM3/include/Converter.h && \
	sed -i '/find_package(OpenCV 3)/c\find_package(OpenCV 4)' CMakeLists.txt && \
	sed -i '/CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)/c\CHECK_CXX_COMPILER_FLAG("-std=c++14" COMPILER_SUPPORTS_CXX14)' CMakeLists.txt && \
	sed -i '/if(COMPILER_SUPPORTS_CXX11)/c\if(COMPILER_SUPPORTS_CXX14)' CMakeLists.txt && \
	sed -i '/set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")/c\set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")' CMakeLists.txt && \
	sed -i '23 i #include<iostream>' /root/Repos/ORB_SLAM3/include/ImuTypes.h && \
	sed -i '23 i #include<iostream>' /root/Repos/ORB_SLAM3/include/GeometricTools.h && \
	sed -i '126 i -lfmt' CMakeLists.txt && \ 
	chmod +x build.sh && ./build.sh #&& cd build && make install 
RUN cd Repos/ORB_SLAM3/Vocabulary/ && tar xf ORBvoc.txt.tar.gz

#WORKDIR /root
RUN mkdir -p /root/catkin_ws/src
RUN . /opt/ros/noetic/setup.sh && mkdir -p /root/catkin_ws/src && cd /root/catkin_ws && catkin init && catkin build -j2 --no-status -DCMAKE_BUILD_TYPE=Release

## Install ROS dependencies for OpenCV
RUN apt install ros-noetic-costmap-2d ros-noetic-gtsam ros-noetic-octomap-msgs
RUN apt install ros-noetic-vision-msgs ros-noetic-mavros-msgs

RUN apt install iputils-ping default-jre iproute2 tmux
RUN pip3 install torch zmq rospkg defusedxml utm

# Checkout latest tag with reference to ouster_ros
RUN cd /root/catkin_ws/src/ && git clone https://github.com/ouster-lidar/ouster_example.git && cd ouster_example && git checkout 20220107
RUN cd /root/catkin_ws/src/ && git clone --recursive https://github.com/KumarRobotics/spomp-system.git

# Latest 3 commits point to a non-existent tag in ouster decoder so revert?
RUN cd /root/catkin_ws/src/spomp-system/dependencies/ouster_decoder/ && git checkout 04eae71
RUN cd /root/catkin_ws && . /opt/ros/noetic/setup.sh && catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release && catkin b -j8

# Copy the required files for MOCHA rajant interface
run mkdir  /root/catkin_ws/src/spomp-system/MOCHA/interface_rajant/scripts/thirdParty/watchstate
COPY thirdParty/watchstate/* /root/catkin_ws/src/spomp-system/MOCHA/interface_rajant/scripts/thirdParty/watchstate
