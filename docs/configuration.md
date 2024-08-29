# Dynamic Configuration
## Contents
- [Dynamic Configuration](#dynamic-configuration)
  - [Contents](#contents)
  - [0. Introduction](#0-introduction)
  - [1. Dynamic Obstacles](#1-dynamic-obstacles)
  - [2. Static Obstacles](#2-static-obstacles)
  - [3. Costmap Layers](#3-costmap-layers)

## <span id="0">0. Introduction

In this reposity, you can simply change configs through modifing the `src/user_config/user_config.yaml`. When you run `main.sh`, our python script will re-generated `*.launch`, `*.world` and so on, according to your configs in `user_config.yaml`.

Below is an example of `user_config.yaml`

```yaml
map: "warehouse"
world: "warehouse"
rviz_file: "sim_env.rviz"

robots_config:
  - robot1_type: "turtlebot3_waffle"
    robot1_global_planner: "a_star"
    robot1_local_planner: "dwa"
    robot1_x_pos: "0.0"
    robot1_y_pos: "0.0"
    robot1_z_pos: "0.0"
    robot1_yaw: "-1.57"
  - robot2_type: "turtlebot3_burger"
    robot2_global_planner: "jps"
    robot2_local_planner: "pid"
    robot2_x_pos: "1.0"
    robot2_y_pos: "0.0"
    robot2_z_pos: "0.0"
    robot2_yaw: "0.0"

plugins:
  pedestrians: "pedestrian_config.yaml"
  obstacles: "obstacles_config.yaml"
  map_layers: "map_layers_config.yaml"
```

Explanation:

- `map`: static map，located in `src/sim_env/map/`, if `map: ""`, map_server will not publish map message which often used in SLAM.

- `world`: gazebo world，located in `src/sim_env/worlds/`, if `world: ""`, Gazebo will be disabled which often used in real world.

- `rviz_file`: RViz configure, automatically generated if `rviz_file` is not set.

- `robots_config`: robotic configuration.

  - `type`: robotic type，such as `turtlebot3_burger`, `turtlebot3_waffle` and `turtlebot3_waffle_pi`.

  - `global_planner`: global algorithm, details in [`Version`](#3).

  - `local_planner`: local algorithm, details in Section `Version`.

  - `xyz_pos and yaw`: initial pose.

- `plugins`: other applications using in simulation

  - `pedestrians`: configure file to add dynamic obstacles (e.g. pedestrians).

  - `obstacles`: configure file to add static obstacles.
  - `map_layers`: configure file to add costmap layers.

## <span id="1">1. Dynamic Obstacles


For *pedestrians* configuration file, the examples are shown below

```yaml
## pedestrians_config.yaml

# 这是为SFM(Social Force Model)算法配置的，模拟行人或其他自主体在环境中的运动。SFM 主要用于社交导航和避障任务，尤其在拥挤环境下模拟多主体的运动行为。
# sfm algorithm configure
social_force:
  # 动画因子可能用于控制模拟的速度或视觉化的步调。较大的值可能会使运动更快，较小的值则可能会减慢动画速度
  animation_factor: 5.1

  # only handle pedestrians within `people_distance`,这个参数定义了算法只考虑距主体（如机器人、其他行人等）6.0米以内的行人。超出这个距离的行人将不被纳入考虑
  people_distance: 6.0

  # weights of social force model
  # 目标权重控制了主体朝向目标位置移动的意愿。较高的权重意味着主体更倾向于朝目标方向前进。直接影响运动规划中的目标导向性，确保主体不会被过多的社交或障碍物因素干扰。
  goal_weight: 2.0

  #  障碍物权重定义了主体对避开障碍物的重视程度。数值越大，主体越会优先考虑避开障碍物
  obstacle_weight: 80.0

  # 社交权重决定了主体与其他行人之间的社交距离和互动影响。较大的值表示主体更会避免接近其他行人
  social_weight: 15
  group_gaze_weight: 3.0
  group_coh_weight: 2.0
  group_rep_weight: 1.0

# pedestrians setting
pedestrians:
  update_rate: 5
  ped_property:
    - name: human_1
      pose: 5 -2 1 0 0 1.57
      velocity: 0.9
      radius: 0.4
      cycle: true
      time_delay: 5
      ignore:
        model_1: ground_plane
        model_2: turtlebot3_waffle
      trajectory:
        goal_point_1: 5 -2 1 0 0 0
        goal_point_2: 5 2 1 0 0 0
    - name: human_2
      pose: 6 -3 1 0 0 0
      velocity: 1.2
      radius: 0.4
      cycle: true
      time_delay: 3
      ignore:
        model_1: ground_plane
        model_2: turtlebot3_waffle
      trajectory:
        goal_point_1: 6 -3 1 0 0 0
        goal_point_2: 6 4 1 0 0 0
```
Explanation:

- `social_force`: the weight factors that modify the navigation behavior. See the [Social Force Model](https://github.com/robotics-upo/lightsfm) for further information.

- `pedestrians/update_rate`: update rate of pedestrains presentation. The higher `update_rate`, the more sluggish the environment becomes.

- `pedestrians/ped_property`: pedestrians property configuration.
  
  - `name`: the id for each human.
  
  - `pose`: the initial pose for each human.

  - `velocity`: maximum velocity (*m/s*) for each human.

  - `radius`: approximate radius of the human's body (m).

  - `cycle`: if *true*, the actor will start the goal point sequence when the last goal point is reached.

  - `time_delay`: this is time in seconds to wait before starting the human motion.

  - `ignore_obstacles`: all the models that must be ignored as obstacles, must be indicated here. The other actors in the world are included automatically.

  - `trajectory`: the list of goal points that the actor must reach must be indicated here. The goals will be post into social force model.

## <span id="2">2. Static Obstacles

For *obstacles* configuration file, the examples are shown below

```yaml
## obstacles_config.yaml 

# static obstacles
obstacles:
  - type: BOX
    pose: 5 2 0 0 0 0
    color: Grey
    props:
      m: 1.00
      w: 0.25
      d: 0.50
      h: 0.80
```
Explanation:
- `type`: model type of specific obstacle, *optional: `BOX`, `CYLINDER` or `SPHERE`*.

- `pose`: fixed pose of the obstacle.

- `color`: color of the obstacle.

- `props`: property of the obstacle.

  - `m`: mass.

  - `w`: width.

  - `d`: depth.

  - `h`: height.

  - `r`: radius.

## <span id="3">3. Costmap Layers

For *map_layers* configuration file, the examples are shown below

```yaml
# map layers configure
global_costmap:
  - static_map
  - obstacle_layer
  - voronoi_layer
  - inflation_layer

local_costmap:
  - static_map
  - obstacle_layer
  - inflation_layer
```

Explanation:
- `global_costmap`: global costmap option
- `local_costmap`: local costmap option
