# Locobot Control Package

## Overview

This package contains the code to run the LoCoBot WX200 both in simulation and real world. It includes functionalities for controlling the Locobot arm, gripper, and base using ROS 2, MoveIt2, and Nav2.

## Directory Structure

The package is organized into the following directories:

- `config/`: Contains configuration files for various components.
- `include/`: Contains header files for the package.
- `launch/`: Contains launch files for bringing up required nodes.
- `src/`: Contains source code files.

## Files and Folders

### config/

- `rs_camera.yaml`: Configuration file for the Realsense camera.

### include/

- `state_machine/StateMachine.h`: Header file for the `StateMachine` class, which controls the different states of the robot behavior.
- `LocobotControl.h`: Header file for the `LocobotControl` class, defining its methods and member variables.

### launch/

- `arm_to_sleep.launch.py`: Launch file for sending the robot's arm to the sleep position.
- `robot.launch.py`: Launch file to bring up the required packages in the NUC of the Locobot.

> **Note:** Read the launch file sections for more information on the arguments required for each launch file.

### src/

- `state_machine/StateMachine.cpp`: Implements the `StateMachine` class, responsible for controlling the different states of the robot behavior.
- `ArmSleepPosition.cpp`: A demo file for the `LocobotControl` class that sends the robot's arm to the sleep position.
- `LocobotControl.cpp`: Implements the `LocobotControl` class, responsible for controlling the Locobot arm, gripper, and base.


## Classes

### LocobotControl Class

The `LocobotControl` class is a ROS 2 node responsible for controlling the Locobot arm, gripper, and base. It provides methods to move the Locobot arm and gripper to specified poses using MoveIt2 and to move the Locobot base to specified points using the Navigation2 stack.

#### Parameters

- `navigation_server`: Name of the navigation action server (default: `navigate_to_pose`).
- `arm_interface`: Name of the MoveIt arm interface (default: `interbotix_arm`).
- `gripper_interface`: Name of the MoveIt gripper interface (default: `interbotix_gripper`).
- `timeout`: Timeout for the navigation action server in seconds (default: `2.0`).

#### Key Components

- **Enums**
  - `ArmPose`: Represents predefined arm poses (HOME, SLEEP, UPRIGHT, UNKNOWN).
  - `GripperState`: Represents predefined gripper states (HOME, RELEASED, GRASPING, UNKNOWN).

- **Classes**
  - `ArmStatus`: Stores the status of the arm, including its pose, gripper state, motion status, and error status.
  - `NavigationStatus`: Stores the status of the navigation, including remaining distance, ETA, progress status, and error status.

#### Main Functions

- **Constructor**
  - Initializes the node, declares parameters, and sets up the action client for navigation.

- **MoveBaseTo**
  - Sends a goal to the navigation stack to move the robot to a specified pose.

- **SetArmPose**
  - Moves the robot arm to a specified pose.

- **SetGripper**
  - Moves the gripper to a specified state.

- **ExecutePlan**
  - Plans and executes the movement using MoveGroupInterface.

- **StopArm**
  - Stops the arm movement if it is in progress.

- **CancelNavigationGoal**
  - Cancels all goals sent to the navigation server.

- **General Functions**
  - `coordinates_to_pose`: Converts coordinates to a PoseStamped message.

#### Callbacks

- **goal_response_callback**: Handles the response from the action server when a goal is sent.
- **feedback_callback**: Handles feedback from the action server during goal execution.
- **result_callback**: Handles the result from the action server after goal execution.

### StateMachine Class

The `StateMachine` class is a ROS 2 node that controls the different states of the robot behavior. It uses a state machine to manage the robot's actions, such as navigating to a goal, interacting with objects, and handling errors. The state machine is designed to be used by another node to control the robot's behavior via the provided services.

#### Parameters

- `robot_tag_frame`: Frame of the robot tag (default: `locobot_tag`).
- `map_frame`: Frame of the map tag (default: `map`).
- `human_tag_frame`: Frame of the human tag (default: `human_tag`).
- `follow_human`: Indicates if the human should be followed after the first position (default: `true`).
- `goal_update_topic`: Topic to update the navigation goal (default: `goal_update`).
- `sleep_time`: Time [ms] to sleep between cycles of the state machine (default: `100`).
- `state_topic`: Topic where the state of the state machine is published (default: `machine_state`).
- `debug`: Publish the internal state of the machine (default: `true`).

#### Services

The `StateMachine` node provides several services to control and monitor its behavior:

- **Control States Service**: This service allows changing the state of the machine to IDLE, NAVIGATION, INTERACTION, or ABORT.
  - Service name: `/state_control`
  - Service type: `locobot_control_interfaces/srv/ControlStates`

- **Clear Error Service**: This service clears the error or aborted state and message error. The machine will enter the IDLE state if possible.
  - Service name: `/clear_error_state`
  - Service type: `locobot_control_interfaces/srv/ClearError`

- **Last Error Service**: This service returns the last error message and state of the state machine.
  - Service name: `/get_last_error`
  - Service type: `locobot_control_interfaces/srv/LastError`

#### Internal and External States

The state machine operates with a set of internal and external states to manage the robot's behavior. The internal states represent the current state of the machine, while the external states represent the result of the machine's operation.

**Internal States**

The internal states of the state machine are as follows:

- `IDLE`: The machine is waiting for a new command.
- `SECURE_ARM`: The machine secures the arm to avoid collision during navigation.
- `WAIT_ARM_SECURING`: The machine waits for the arm to be secured.
- `SEND_NAV_GOAL`: The machine sends the navigation goal to the navigation stack.
- `WAIT_NAVIGATION`: The machine waits for the navigation to reach the goal.
- `WAIT_ARM_EXTENDING`: The machine waits for the arm to be extended.
- `ARM_EXTENDED`: The machine opens the gripper to release the object upon command.
- `WAIT_GRIPPER`: The machine waits for the gripper to open.
- `WAIT_ARM_RETRACTING`: The machine waits for the arm to retract.
- `ERROR`: The machine is in an error state.
- `ABORT`: The machine is aborted.
- `STOPPING`: The machine is stopping.

![Internal States](internal_states.svg)

**External States**

The external states of the state machine are as follows:

- `SUCCESS`: The machine has completed the task successfully.
- `FAILURE`: The machine has failed to complete the task.
- `RUNNING`: The machine is still running.
- `INITIALIZED`: The machine has been initialized.

![External States](external_states.svg)

#### Safety Precautions
The state machine start the robot in the ERROR state. This is to prevent the robot from moving unexpectedly. To start the robot, the state must be switched to the IDLE state using the `\clear_error_state` service:

```bash
ros2 service call /clear_error_state locobot_control_interfaces/srv/ClearError
```


## Launch Files

### robot.launch.py

This launch file is used to bring up the required packages in the NUC of the Locobot.

**Input Arguments:**
- `rs_camera_param`: The file path to the Realsense camera configuration file. (default: *rs_camera.yaml*)


### Topic Remapping

To use the `LocobotControl` class, it is necessary to remap the topics as shown in the [robot.launch.py](launch/robot.launch.py) file. This is because Interbotix publishes the robot descriptions on specific topics.

### Stop Interface
The stop interface for MoveIt2 in the state machine has been commented out in the code because it exhibits unexpected behavior when used with the real robot. An issue should be opened in the Interbotix package repository to address this problem.

## License

This project is licensed under the Apache-2.0 License. See the `LICENSE` file for details.