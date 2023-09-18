Tutorial
===

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


