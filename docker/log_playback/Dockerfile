# Ubuntu 18.04 with nvidia-docker2 beta opengl support
FROM nvidia/opengl:1.0-glvnd-devel-ubuntu18.04

# setup timezone
RUN echo 'Etc/UTC' > /etc/timezone && \
  ln -s /usr/share/zoneinfo/Etc/UTC /etc/localtime \
  && apt-get -qq update && apt-get -q -y install tzdata \
  && rm -rf /var/lib/apt/lists/* \
  && apt-get -qq clean

RUN apt-get -qq update && apt-get -y install \
  git \
  gnupg2 \
  lsb-release \
  sudo \
  vim \
  wget \
&& rm -rf /var/lib/apt/lists/*

# setup sources.list and keys for ROS and ignition
RUN echo "deb http://packages.ros.org/ros/ubuntu bionic main" > /etc/apt/sources.list.d/ros1-latest.list \
  && apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654 \
  && echo "deb [trusted=yes] http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list \
  && wget http://packages.osrfoundation.org/gazebo.key -O - | apt-key add -

# install ROS and required packages
RUN apt-get -qq update && apt-get -y install \
  python-catkin-tools \
  python-rosdep \
  python-rosinstall \
  ros-melodic-desktop \
  ros-melodic-joystick-drivers \
  ros-melodic-pointcloud-to-laserscan \
  ros-melodic-robot-localization \
  ros-melodic-spacenav-node \
  ros-melodic-tf2-sensor-msgs \
  ros-melodic-twist-mux \
  ros-melodic-rviz-imu-plugin \
  ros-melodic-rotors-control \
&& rosdep init \
&& rm -rf /var/lib/apt/lists/*

# install dependencies for gazebo (and other ignition packages)
run apt-get -qq update && apt-get -y install \
  cmake build-essential curl cppcheck g++-8 libbenchmark-dev libgflags-dev \
  doxygen ruby-ronn libtinyxml2-dev libtinyxml-dev software-properties-common \
  libeigen3-dev qtdeclarative5-models-plugin \
  libignition-cmake2-dev libignition-common3-dev libignition-math6-eigen3-dev \
  libignition-plugin-dev libignition-physics-dev libignition-rendering2-dev \
  libignition-tools-dev libignition-transport7-dev libignition-gui2-dev \
  libignition-msgs4-dev libsdformat8-dev \
# other ignition packages
  libignition-fuel-tools3-dev \
  libignition-math6-dev \
  libignition-sensors2-dev \
&& sudo rm -rf /var/lib/apt/lists/*

# Add a user with the same user_id as the user outside the container
# Requires a docker build argument `user_id`
ARG user_id
ENV USERNAME developer
RUN useradd -U --uid ${user_id} -ms /bin/bash $USERNAME \
 && echo "$USERNAME:$USERNAME" | chpasswd \
 && adduser $USERNAME sudo \
 && echo "$USERNAME ALL=NOPASSWD: ALL" >> /etc/sudoers.d/$USERNAME

# configure gcc8 and install colcon so that remaining ign libraries can be built from source
RUN sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 800 --slave /usr/bin/g++ g++ /usr/bin/g++-8 --slave /usr/bin/gcov gcov /usr/bin/gcov-8 \
  && sudo apt-get -qq update && sudo apt-get -y install \
    python3-colcon-common-extensions \
  && sudo rm -rf /var/lib/apt/lists/*

# Commands below run as the developer user
USER $USERNAME

RUN rosdep update

WORKDIR /home/$USERNAME

# Download the public models
RUN ign fuel download -v 4 -j 16 -u "https://fuel.ignitionrobotics.org/OpenRobotics/collections/SubT Tech Repo"

# make a workspace where ign-gazebo and ign-launch can be built from source
# (ign-gazebo will be loaded as a volume)
# this ws also has the ros-ign bridge
RUN mkdir -p /home/$USERNAME/ign_ws/src \
  && cd /home/$USERNAME/ign_ws/src \
  && git clone -b ign-launch1 https://github.com/ignitionrobotics/ign-launch.git \
  && git clone https://github.com/ignitionrobotics/ros_ign.git && cd ros_ign && git checkout 0.9.3 \
  && cd /home/$USERNAME/ign_ws \
  && sudo apt-get -qq update \
  && rosdep install --from-paths src -i -y --rosdistro melodic \
    --skip-keys=ignition-gazebo2 \
    --skip-keys=ignition-gazebo3 \
    --skip-keys=ignition-msgs4 \
    --skip-keys=ignition-msgs5 \
    --skip-keys=ignition-rendering2 \
    --skip-keys=ignition-rendering3 \
    --skip-keys=ignition-sensors2 \
    --skip-keys=ignition-sensors3 \
    --skip-keys=ignition-transport7 \
    --skip-keys=ignition-transport8 \
&& sudo rm -rf /var/lib/apt/lists/*

# make a subt_ws
RUN mkdir -p /home/$USERNAME/subt_ws/src \
  && cd /home/$USERNAME/subt_ws/src \
  && git clone https://github.com/osrf/subt.git
