# FASTNav: Fine-tuned Adaptive Small-language-models Trained for Multi-point Robot Navigation
This docker contains the code for [FASTNav: Fine-tuned Adaptive Small-language-models Trained for Multi-point Robot Navigation]() by Yuxuan Chen\*, Yixin Han\*, and Xiao Li (\* denotes equal contribution).You can run the experiment example through following codes.

### Environment Setup

#### OpenAI API Key
Before we run `simulation.py`,`simulation_slm.py` and `simulation_single.py`, we should set OpenAI API Key to use OpenAI API.
```
gpt4 = ChatOpenAI(model='gpt-4-turbo', base_url='https://api.openai.com/v1', openai_api_key=<your_openai_api_key>, temperature=0.1)
```

#### Simulation Environment
Our simulation environment employs [AWS RoboMaker Hospital World ROS package](https://github.com/aws-robotics/aws-robomaker-hospital-world) through Gazebo.

The model has been downloaded already in the docker.

## Running
### Running Simulator
Here’s how to set up a virtual screen inside Docker if no display is forwarded when starting Docker:
```
Xvfb :99 -screen 0 1024x768x16 &
export DISPLAY=:99
```
Then, in the same terminal, run:
```
ros2 launch nav2_bringup tb3_simulation_launch.py headless:=True map:=/workspace/hospital/hospital.yaml world:=/workspace/hospital/hospital_with_waffle.world
```
This launch file will launch Nav2 with the AMCL localizer in the simulation environment. It will also launch the robot state publisher to provide transforms, a Gazebo instance with the Turtlebot3 URDF, and RVIZ.

### Running FastChat
FastChat is used to provide LLM inference interfaces through API. We need 3 terminals to run these commands:
```
python3 -m fastchat.serve.controller  # launch the controller
python3 -m fastchat.serve.model_worker --model-path arctic126/hospital_h2o-danube2-1.8b-base  # launch the model worker(s)
python3 -m fastchat.serve.openai_api_server --host localhost --port 8000  # launch the RESTful API server
```
We are currently providing only the fine-tuned h2o-danube2-1.8b-base as a sample, with more models to be released later.
Now, the local LLM can execute queries through OpenAI API with variables set as below:
```
openai.api_key = "EMPTY"
openai.base_url = "http://localhost:8000/v1/"
```

### Running FASTNav
#### FASTNav with Teacher-student Iteration
```
cd simulation
python simulation.py --model <model_name>
```
The <model_name> here is different from that when running FastChat. For example, in FastChat, it is `google/gemma-2b`, while here, it is `gemma-2b`, just the model name without usenames.
#### FASTNav with Memory
```
python simulation_slm.py --model <model_name>
```
This runs SLMs with the memory during iterations, so we need to generate a feedback file first through `simulation.py`.
#### Running with Single Models
```
python simulation_single.py --model <model_name>
```
This runs a single model like GPT-4 to finish the navigation tasks.

