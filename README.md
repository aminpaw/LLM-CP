# LLM-CP (Large Language Model for Command Planning)

This repository contains a proof of concept implementation of a large language model for command planning. By command planning, we mean the task of generating a sequence of commands that can be executed by a system to achieve a desired goal. The model is used to generate high level commands that we preprogrammed, allowing us to use the model to convert natural language instructions into a sequence of commands that can be executed by a system.

## Installation

### Requirements

- ROS2 Humble
- CoppeliaSim 4.5.1
- Python 3.8 or higher

### Steps

1. Clone the repository

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
git clone
```

2. Install the required Python packages

```bash
pip3 install openai numpy
```

3. Build the ROS2 workspace

```bash
cd ~/ros2_ws
colcon build --symlink-install
```

4. Source the ROS2 workspace

```bash
source ~/ros2_ws/install/setup.bash
```

5. Run the CoppeliaSim simulation

```bash
./CoppeliaSim.sh
```

Then open the scene `controledViaROS2.ttt` located in the `simulation_env` folder.

6. Run the Commander node

```bash
ros2 run commander commander
```

7. Run the API node (in a new terminal)

```bash
ros2 run api_caller api_caller
```

## Usage

The code is divided into two main components: the Commander node and the API Caller node.
The Commander node is responsible for communicating with the CoppeliaSim simulation and executing the commands generated by the model. The API Caller node is responsible for generating the prompts and calling the model to generate the commands.

### Commander Node

This node subscribes to sensor topics from the simulation and publishes commands to the simulation. It also prepares the commands to be executed by the simulation.
Currently the node is able to execute the following commands:

- Move Forward
- Move Backward
- Turn Left
- Turn Right

### API Caller Node

This node is responsible for generating the prompts and calling the model to generate the commands. The node subscribes to the sensor topics from the simulation and publishes the generated commands to the Commander node. <br>
Currently we use the GPT-3.5-Turbo model from OpenAI to generate the commands. The model is called using the OpenAI API so you need to have an API key to use it.

### Prompt

```markdown
You are the best and {CHARACTERISTIC} NASA Robot Controller in the world.
Your are currently controlling a mobile robot. Your goal is to accomplish the given task. You respond with a bullet point list of moves.
Your Task is:
{GOAL}
......
Your observations are:
{OBSERVATIONS}
.....
Your last commands were:
{LAST COMMANDS}
.....
The moves you can use are: - Move Forward - Move Backward - Rotate Right - Rotate Left
.....
Reply with a bullet point list of moves. The format should be: `- <name of the move>` separated by a new line.
Example if your task is to reach an object 1m away from you that is 20 degree to your right: - Rotate Right - Rotate Right - Move Forward
```

## Additional Features

### Multi Agents

The paper ["More Agents Is All You Need"](https://arxiv.org/pdf/2402.05120.pdf) shows that using multiple agents to generate the commands can improve the performance of the model. We implemented this feature in the API Caller node. You can specify the number of agents to use in the `api_caller.py` file. <br>
It calls the api multiple times and then selects the best command generated by the agents by sampling and voting.

## Restrictions

- Currently there are only 4 commands that are preprogrammed in the Commander node.
- The observation is hardcoded to distance and angle to the goal.
- The api's rate limit is 3 requests per minute and 200 per day.
