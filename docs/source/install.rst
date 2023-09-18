Installation
=====

To use these Softwares, first install it using:

.. code-block:: console

   colcon build --packages-select boreal_bikes_bringup

.. autoexception:: lumache.InvalidKindError


Initial Setup Instructions
==========================

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
=================
run.sh: This script is used to perform the initial setup of the system. It installs the required dependencies, sets up environment variables, configures Docker, sets up DNS, applies udev rules, generates services, and initiates a system reboot.

config: This configuration file contains various environment variables and their corresponding values used during the setup process. You can modify the values in this file to suit your requirements.

scripts/set-dns.sh: This script is responsible for modifying the DNS configuration in the /etc/hosts file. It associates the hostname boreal.control with a specific IP address defined in the config file.

scripts/set-env.sh: This script updates the /etc/environment file. It appends the content of the config file to the end of the /etc/environment file, allowing the environment variables to be accessed globally.

udev/99-boreal-devices.rules: This file is a udev rule used for device management. It sets the appropriate permissions and creates a symlink for the u-blox GNSS receiver.