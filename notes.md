# RL_Quadcopter_2 笔记

## 项目理解：
1. 项目结构：  

```py
project
|
|---agents
|---|   actor.py                行动者
|---|   agent.py                智能体
|---|   critic.py               评论者
|---|   ou_noise.py             OU 噪点
|---|   policy_search.py        notebook 中测试的智能体
|---|   replay_buffer.py        回放缓冲区
|   Quadcopter_Project.ipynb
|   data.txt                    四轴飞行器的信息
|   notes.md                    本项目的个人笔记
|   physics_sim.py              模拟器信息
|   README.md
|   requirements.txt
|   task.py                     为智能体定义的任务
```
2. 项目理解
本项目采用了 DDPG 方法，引入 **行动者（策略）模型**、**评论者（值）模型** 来构造 agent，使用 ReplayBuffer 来存放经验元组，OU_Noise 流程来添加噪点，以促进 agent 的探索行为，最后设置 task。
需要实现 `actor.py`，`critic.py`，`agent.py`，`replay_buffer.py`，`task.py` 和 `ou_noise.py` 这几大模块，具体需要调整的是 `actor.py`，`critic.py` 和 `task.py`，其他模块都已经给出。
在做这个项目之前，我观看了项目直播，推荐大家看看这个直播，帮助理解项目本身，这里是[直播配套的 Notebook](https://github.com/chinaq/bloq/blob/master/posts/2018-05-17-Quadcopter%E7%AC%94%E8%AE%B0_Udacity/quadcopter.md)。

3. 分析
整个项目的框架其实都已经搭好了，DDPG、Actor、Critic、Replay 和 Noise 的大体代码都已经给出，需要做的几点是：
- 设置目标 Target：本项目因为时间原因，只将目标设置为起飞，在**定义任务**模块中要记着写上 `init_pose=[0., 0., 0., 0., 0., 0.]`，或者将 `target_pos` 改为和 `init_pose` 不同
- 设置奖励 Reward：
    - 为了保证无人机直线上升而不偏离 z 轴，将 reward 设置为 x 和 y 轴方向偏离时惩罚较多，而沿着 z 轴上升时奖励较多
    - 第二遍尝试的时候加入速度的考虑
- 设置超参数 Hyperparameters：设置调整 Actor 和 Critic 中的网络
- 实现可视化 Visualisation：添加了 3D 的飞行轨迹
4. 实现
- Reward 设置（`task.py`）：
```py
reward_xy = np.tanh(1.-.009*(abs(self.sim.pose[:2] - self.target_pos[:2]))).sum()
reward_z = np.tanh(1.-.003*(abs(self.sim.pose[2] - self.target_pos[2]))).sum()
reward = reward_xy + reward_z
```
- Actor 设置（`actor.py`）：
```py
layer1: state -> Dense(512, regularizers.l2(1e-6)) -> BatchNormalization -> relu
layer2: layer1 -> Dense(256, regularizers.l2(1e-6)) -> BatchNormalization -> relu
layer3: layer2 -> Dense(action_size, initializers.RandomUniform(-0.003, 0.003))
optimizers.Adam(lr=0.0001)
```
- Critic 设置（`critic.py`）：
    - For state pathway:
    ```py
    state_layer1: states -> Dense(512, regularizers.l2(1e-6)) -> BatchNormalization -> relu
    ```
    - For action pathway:
    ```py
    state_layer2: state_layer1 -> Dense(256, regularizers.l2(1e-6)) -> relu
    action_layer1: actions -> Dense(256, regularizers.l2(1e-6))-> relu
    ```
    - Combine state and action pathways:
    ```py
    net: [net_states, net_actions] -> Add -> relu    
    ```

    - Final output Q values:
    ```py
    Q_values: net -> Dense(1, initializers.RandomUniform(-0.003, 0.003))
    ```
    - Optimizer:
    ```py
    optimizers.Adam(lr=0.001)
    ```
