Welcome to Boreal Bikes documentation!
===================================

**Boreal Bikes Softwares** is a collection of packages for the control of the Bikes of the Boreal project.
We are currently developing the packages

To install the packages, please follow the instructions in the :ref:`installation` section.

To run examples, please follow the instructions in the :ref:`tutorial` section.

.. note::

   This project is under active development.

Installation
------------

To use these Softwares, first install it using:

.. code-block:: console

   colcon build --packages-select boreal_bikes_bringup

.. autoexception:: lumache.InvalidKindError


Initial Setup Instructions
--------------------------

Follow the steps below to perform the initial setup using the provided files:

Open the config file in a text editor.

Modify the values of the environment variables according to your requirements. The variables and their purposes are described below:

REPO_ROOT: The root directory of your project repository.
LIDAR_HOST: The IP address of the LIDAR host.
ADHOC_HOST: The desired IP address for the boreal.control hostname.
RECORDING_ROOT: The directory where captured data will be stored.
GPS_FREQ: The frequency at which GPS updates are received.
Update these variables with the appropriate values and save the file.

Open a terminal and navigate to this directory.

Run the following command to start the initial setup process:

sudo ./run.sh

This script will install the required dependencies, configure the environment variables, set up Docker, update DNS settings, apply udev rules, generate services, and initiate a system reboot.

After the system restarts, the initial setup process will be complete, and the environment will be ready for use.

Please note that during the initial setup process, the script may prompt for your password or ask for confirmation for certain operations. Make sure to provide the necessary inputs when prompted.

If you encounter any issues or have questions, please refer to the documentation or contact the appropriate support channels.

Note: Ensure that you have the necessary permissions to execute the scripts and modify system files.



Files and Scripts
-----------------

The following files and scripts are used during the initial setup process:


run.sh: This script is used to perform the initial setup of the system. It installs the required dependencies, sets up environment variables, configures Docker, sets up DNS, applies udev rules, generates services, and initiates a system reboot.

config: This configuration file contains various environment variables and their corresponding values used during the setup process. You can modify the values in this file to suit your requirements.

scripts/set-dns.sh: This script is responsible for modifying the DNS configuration in the /etc/hosts file. It associates the hostname boreal.control with a specific IP address defined in the config file.

scripts/set-env.sh: This script updates the /etc/environment file. It appends the content of the config file to the end of the /etc/environment file, allowing the environment variables to be accessed globally.

udev/99-boreal-devices.rules: This file is a udev rule used for device management. It sets the appropriate permissions and creates a symlink for the u-blox GNSS receiver.


Tutorial
--------

.. autosummary::
   :toctree: generated



# Live code for Holoscene X bike

## Introduction

This directory contains various [ROS2 workspaces](https://docs.ros.org/en/foxy/Tutorials/Workspace/Creating-A-Workspace.html), abbreviated as `ws`. 

+ Each ROS2 node runs in a separate Docker container.
+ ROS2 nodes (in containers) communicate with each other over the host network.
+ Each ROS2 node binds the respective workspaces (directories ending with `_ws` under `/project`) as under `/workspace` within the container's filesystem. 
+ There is a `Dockerfile` under each workspace which defines the Docker image for that workspace.
+ `docker-compose up` is used to launch the full application.
+  workspaces can be launched separately in different terminals, and as long as the network configuration is correct, they will be able to send/receive ROS2 messages to/from each other.

## Tutorial: How to Build Your Own Docker ROS2 Node

Please see the [README for tutorial_ws](tutorial_ws) for an example of how to create and launch one workspace.

## System Architecture

![](../docs/diagrams/merckx-ros2-overview.svg)

*Edit this diagram: Go to [diagrams.net](https://diagrams.net) > 'Start Now' > 'Save Diagrams To' > GitHub > 'Open Existing Diagram' > borealbikes-dev/codename-merckx > docs > merckx-ros2-overview.drawio. Edit the file then save, and 'Export as' > 'SVG' and save in the same location. The above diagram will automatically update.*

## Details about ROS2 (Where is the actual functional code?)

One of its small downsides of ROS2 is that main program gets buried under a lot of ROS2 boilerplate directory/file structure.

For example, the main Python code that reads videos and publishes them as ROS2 messages is [`file_stream_node.py`](https://github.com/borealbikes-dev/codename-merckx/blob/main/project/file_stream_ws/src/file_stream_package/file_stream_package/file_stream_node.py). However, it is a few levels deep inside its workspace directory:

```
file_stream_ws
├── base_dockerfile
│   └── Dockerfile.ros.galactic
├── Dockerfile.desktop
├── Dockerfile.jetson
├── docker_run.desktop.sh
├── docker_run.jetson.sh
├── entry.sh
└── src
    ├── batch_image_interface
    └── multi_stream_package
        ├── multi_stream_package
        │   ├── __init__.py
        │   └── multi_stream_node.py  	<--- HERE IT IS
        ├── package.xml
        ├── resource
        ├── setup.cfg
        ├── setup.py
        └── test
```
Don't worry though, because most of the files/directories are created by ROS2 automatically.

See [`docs/simplified_ros_tutorials.md`](../docs/simplified_ros_tutorials.md) for minimal demonstrations of how ROS2 packages are created, built, and run.

If you typically use interpreted languages like Python, keep in mind that ROS2 has an extra `colcon build` step that needs to be run manually for any code changes to be reflected in the execution.

## Brief Workspace Descriptions

| Dockerized ROS2 workspace | What does it do? |
| --- | --- |
| `arduino_ws` | Reads environmental sensors from an `Arduino Nano 33 BLE Sense` board, and allows writing values to its pins |
| `file_stram_ws` | Reads five .mp4 video files and publishes the frames as a batch of images. It was designed to be used as a develop-time emulation of camera inputs (see `multi_stream_ws`). | 
| `gps_ws` | Receives real-time GPS data from an attached GPS module and publishes it |
| `livox_ws` | Receives real-time LiDAR data from five Livox sensors and publishes them as point cloud |
| `multi_stream_ws` | Reads from five cameras and publishes the frames as a batch of images |
| `rosbag_ws` | A somewhat empty ROS2 node that simply records all of the ROS2 message traffic |
| `vesc_ws` | VESC is an open source motor controller. This node reads sensor values from the motor & controller. It is possible to activate the motor from this code, but as a safety measure we have excluded that ability |
| `docker-compose.*.yml` | These are not ROS2 workspaces, obviously. These files (one for x86 desktop and one for NVIDIA Jetson) describe how the above nodes will be launched together. [See Docker documentation](https://docs.docker.com/compose/compose-file/) |

## Docker How-Tos

**How to Individually Run Docker Containers:**

Within each workspace (directories under `/project`), there is a `docker_run.sh`. Pass in `-d` or `--debug` to open a terminal in the container without running the application. For example,
```
project/multi_stream_ws/docker_run.sh -d
```

**How to launch multiple terminals from one running container:**
```
scripts/docker/new_term_existing_container.sh -c CONTAINER_NAME
```

**How To: Build docker files from provided Dockerfile** - Each workspace (directories under `/project`) is designed to run in separate docker containers, as defined by its respective `Dockerfile`. To build the images yourself, navigate to a workspace and: `docker build -t <DESIRED_IMAGE_NAME> .` (don't miss that last dot!) You may also upload them to Docker Hub to allow others to use it without building it. First, `docker login` then `docker tag <IMAGE_NAME> <DOCKERHUB_USERNAME/IMAGE_NAME>`, then `docker push <DOCKERHUB_USERNAME/IMAGE_NAME>`

**How To: Access the terminal of a workspace docker container without running the ROS2 application**. You might want to do this to debug or test changes to the source. To run the docker container without starting the ROS2 application, launch `./docker_run.sh -d` (`-d` is short for `--debug`). The `entry.sh` app launch script will not run and instead give you a terminal.

**How to: Switch between Docker/local environments: `rm -r install/ build/` before `colcon build`.** CMake is unable to deal with changes to the path of the source files. Therefore, we need to 'reset' the build when moving between Docker and local running environments. To aoid this issue completely, I recommend editing / building the source code from within a docker containers (`./docker_run.sh -d`).

**How to: Delete all docker images and reset everything: `docker system prune -a --volumes`**

# ROS2 Message Specifications

[See list of custom ROS2 messages used in this repo.](../docs/ROS_MESSAGES_README.md)

[Read the ROS2 documentation about custom messages](https://docs.ros.org/en/galactic/Tutorials/Custom-ROS2-Interfaces.html)

Each workspace (ROS2 package) runs on a customized version of this base docker image. All of them are available from docker hub.