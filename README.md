# CleanNav 接口与任务合同

`cleannav_interfaces` 是 CleanNav 无人清扫车系统的共享 ROS 2 接口与配置合同仓库。

本仓库负责定义 APP、Voice、Mission Manager、Perception、Navigation、Safety 和状态显示之间的跨模块数据合同。

当前 ROS 2 包版本：

```text
0.1.0
```

当前接口语义版本：

```text
1.0
```

当前决赛阶段冻结基线：

```text
competition-hil-baseline-20260921
```

---

## 1. 仓库职责

本仓库负责：

- 定义 CleanNav 跨模块 ROS 2 消息与服务；
- 维护 Task Catalog；
- 维护 APP 请求 JSON Schema；
- 维护 Voice 任务映射示例；
- 维护接口版本和字段语义；
- 为 Mission Manager、HMI、Voice、Perception、Navigation 和 Safety 提供统一合同。

本仓库只定义接口，不实现具体业务逻辑。

本仓库不负责：

- Mission Manager 状态机；
- 任务队列执行；
- Navigation 路径规划；
- 控制器计算；
- Safety Supervisor 实际门控；
- 感知模型推理；
- APP 界面；
- Voice ASR；
- 发布 `/cmd_vel`；
- 直接控制车辆。

---

## 2. 当前接口总览

当前正式生成：

```text
7 个 ROS 2 msg
1 个 ROS 2 srv
```

消息：

```text
TaskCommand.msg
TaskStatus.msg
RobotStatus.msg
CleaningTarget.msg
CleaningTargetArray.msg
PerceptionHealth.msg
SafetyStatus.msg
```

服务：

```text
SafetyLease.srv
```

---

## 3. TaskCommand

文件：

```text
msg/TaskCommand.msg
```

主要方向：

```text
APP / Voice
    ↓
TaskCommand
    ↓
Mission Manager
```

`TaskCommand` 只表达预定义任务或系统控制命令。

它不携带：

- 任意导航坐标；
- `/cmd_vel`；
- Ackermann 转角；
- CAN 控制量。

当前来源定义：

```text
SOURCE_UNKNOWN = 0
SOURCE_VOICE   = 1
SOURCE_APP     = 2
SOURCE_MOCK    = 3
```

核心字段包括：

```text
interface_version
command_id
source
task_id
confidence
raw_text
valid_for
user_confirmed
```

其中：

- `command_id` 是外部命令幂等键；
- `task_id` 必须来自 Task Catalog；
- APP 的 confidence 通常为 1.0；
- Voice 使用离线 ASR confidence；
- `raw_text` 仅用于审计，不参与执行语义；
- `user_confirmed` 只表示 HMI 已完成本次用户确认；
- `user_confirmed` 不代表 Safety 条件已经满足。

内部状态机事件不得通过公共 HMI TaskCommand Topic 注入。

---

## 4. TaskStatus

文件：

```text
msg/TaskStatus.msg
```

主要方向：

```text
Mission Manager
    ↓
TaskStatus
    ↓
HMI
```

TaskStatus 同时支持：

```text
SCOPE_COMMAND
SCOPE_EXECUTION
```

用于区分：

- 单次外部命令的处理结果；
- 一个实际 Mission Execution 的生命周期。

主要状态包括：

```text
IDLE
ACCEPTED
REJECTED
QUEUED
WAITING_TARGET
PREPARING
NAVIGATING
PAUSING
PAUSED
CANCELING
RETURNING_HOME
SAFETY_BLOCKED
SUCCEEDED
CANCELED
FAILED
EMERGENCY_STOPPED
```

TaskStatus 还定义了稳定的 `reason_code` 数值合同。

主要分类：

```text
1xx   命令校验
2xx   Task Catalog / Queue
3xx   Perception / Target
4xx   Navigation
5xx   Safety / Lease
6xx   Timeout / Stale / Clock
7xx   Cleanup / State Machine / Contract
9xx   Internal Error
```

程序逻辑应优先使用：

```text
state
reason_code
```

而不是解析人类可读的 `message`。

Terminal TaskStatus 需要缓存，以支持 `command_id` 幂等重放。

---

## 5. RobotStatus

文件：

```text
msg/RobotStatus.msg
```

RobotStatus 用于 HMI 展示机器人综合状态。

主要字段包括：

```text
system_state
localization_ok
pose
navigation_active
emergency_stop_active
autonomous_enabled
linear_velocity_mps
angular_velocity_rps
message
```

需要特别区分：

```text
navigation_active
```

与：

```text
autonomous_enabled
```

前者表示 Navigation 是否存在活动 generation。

后者表示 Safety 是否实际授予自动运动权限。

两者不能混同。

---

## 6. SafetyStatus

文件：

```text
msg/SafetyStatus.msg
```

用于向程序消费者提供结构化 Safety Supervisor 状态。

主要字段：

```text
emergency_stop_active
autonomous_enabled
lease_owner_execution_id
message
```

其中：

```text
autonomous_enabled
```

表示 Safety 实际允许自动运动。

```text
lease_owner_execution_id
```

表示当前持有 Safety Lease 的 Mission Execution。

---

## 7. SafetyLease

文件：

```text
srv/SafetyLease.srv
```

Safety Lease 用于 Mission Manager 与 Safety Supervisor 之间建立明确的运动授权所有权。

请求：

```text
execution_id
```

响应：

```text
success
message
```

核心语义：

```text
一个活动 execution
        ↓
申请 Safety Lease
        ↓
获得自动运动授权
        ↓
Navigation 执行
        ↓
暂停 / 停止 / 结束时释放 Lease
```

Lease 的实际执行逻辑由 Mission Manager 和 Safety Supervisor 实现。

本仓库只定义合同。

---

## 8. 感知目标合同

### CleaningTarget

文件：

```text
msg/CleaningTarget.msg
```

CleaningTarget 表达环境中的一个清扫目标观测。

当前目标类型包括：

```text
LEAF
LEAF_PILE
PUDDLE
BOTTLE_CAN
PAPER_TRASH
```

主要字段：

```text
target_id
source
target_type
class_name
confidence
projection_valid
centroid
footprint
area_m2
position_uncertainty_m
observation_state
valid_for
```

`target_id` 在目标生命周期内应保持稳定。

目标位置使用：

```text
map
```

坐标系。

### 重要边界

CleaningTarget 是：

```text
环境观测
```

不是：

```text
Navigation Goal
```

因此：

- Perception 不直接发布 `/goal_pose`；
- CleaningTarget 不直接写 Costmap；
- 普通目标位置更新不自动重新提交 Navigation Goal；
- Mission Manager 根据 Task Catalog 和目标选择策略决定是否生成导航目标。

正确链路：

```text
Perception
    ↓
CleaningTargetArray
    ↓
Mission Manager
    ↓
目标筛选 / Task Policy
    ↓
Navigation Goal
```

---

## 9. PerceptionHealth

`PerceptionHealth.msg` 用于表达感知系统整体健康状态。

Mission Manager 可以据此判断：

- 感知是否可用；
- 目标数据是否可信；
- TF / Projection 是否有效；
- 是否需要等待或拒绝视觉目标任务。

它属于状态合同，不负责实现具体感知算法。

---

## 10. 当前主要 Topic

当前主要合同如下：

| Topic | 类型 | 方向 |
| --- | --- | --- |
| `/cleannav/hmi/task_command` | `TaskCommand` | HMI / Voice → Mission Manager |
| `/cleannav/task_status` | `TaskStatus` | Mission Manager → HMI |
| `/cleannav/robot_status` | `RobotStatus` | System → HMI |
| `/cleannav/perception/cleaning_targets` | `CleaningTargetArray` | Perception → Mission Manager |
| `/cleannav/perception/health` | `PerceptionHealth` | Perception → Mission Manager / HMI |

具体 QoS、发布频率和消费者行为由各组件实现共同约束。

---

## 11. Task Catalog

权威配置：

```text
config/task_catalog.yaml
```

Task Catalog 是 Mission Manager 判断任务行为的权威来源。

Mission Manager 不应仅根据数字 ID 区间猜测任务行为。

主要字段包括：

```text
task_id
name
task_kind
enabled
allowed_sources
requires_confirmation
goal_pose
route_id
target_type
selection_rule
wait_timeout_sec
completion_radius_m
```

任务主要分成：

```text
MISSION
CONTROL
```

MISSION 可以进入普通任务执行流程。

CONTROL 不进入普通 Mission FIFO。

---

## 12. 当前任务状态

### 当前已启用的控制任务

```text
2  PAUSE_CURRENT_TASK
3  RESUME_CURRENT_TASK
4  STOP_CURRENT_TASK
6  SOFTWARE_EMERGENCY_STOP
7  RESET_SOFTWARE_EMERGENCY_STOP
```

其中 Task 7：

```text
RESET_SOFTWARE_EMERGENCY_STOP
```

只允许：

```text
APP
MOCK
```

并且：

```text
requires_confirmation = true
```

Voice 不允许解除 Emergency Stop。

### 当前已启用的视觉任务

```text
30 CLEAN_NEAREST_LEAF
31 CLEAN_NEAREST_LEAF_PILE
32 CLEAN_NEAREST_PUDDLE
```

Task 30：

```text
target_type = LEAF
selection_rule = NEAREST_VALID
wait_timeout_sec = 10.0
completion_radius_m = 0.35
```

Task 31：

```text
target_type = LEAF_PILE
selection_rule = NEAREST_VALID
wait_timeout_sec = 10.0
completion_radius_m = 0.40
```

Task 32：

```text
target_type = PUDDLE
selection_rule = NEAREST_VALID
wait_timeout_sec = 10.0
completion_radius_m = 0.40
```

### 当前未启用

```text
1   START_DEFAULT_CLEANING
5   RETURN_HOME
10  GOTO_POINT_1
20  CLEAN_ROUTE_1
33  CLEAN_HIGHEST_PRIORITY_TARGET
```

这些任务由于默认路线、home pose、固定坐标或 priority contract 尚未完成配置，目前保持 disabled。

---

## 13. Voice 映射

示例配置：

```text
config/voice_task_map_example.yaml
```

当前 confidence threshold：

```text
0.80
```

当前 Voice 映射包括：

```text
2   暂停 / 先停一下
3   继续 / 继续清扫
4   停止任务 / 结束任务
6   紧急停止 / 立即停止
30  清扫最近的落叶 / 去清扫落叶
31  清扫最近的落叶堆
32  清扫最近的积水 / 去清扫积水
```

Voice Bridge 必须在运行时再次检查：

```text
enabled
allowed_sources
confidence
confirmation
```

Voice map 不能绕过 Task Catalog。

Task 7 不映射到 Voice。

---

## 14. APP 请求合同

配置：

```text
config/app_task_schema.json
```

APP 外部 JSON 请求与 ROS 2 `TaskCommand` 不是同一个对象。

典型链路：

```text
Flutter APP
    ↓
HTTP JSON
    ↓
HMI Gateway
    ↓
校验 / 转换
    ↓
TaskCommand
    ↓
Mission Manager
```

APP 不能自行发布 ROS2 车辆控制 Topic。

---

## 15. 系统安全边界

当前接口设计遵循以下原则：

```text
APP / Voice
    ↓
TaskCommand
    ↓
Mission Manager
    ↓
Navigation Request
```

车辆运动链路：

```text
Navigation
    ↓
候选运动控制
    ↓
Safety Supervisor
    ↓
最终车辆控制
```

必须保持：

- HMI 不发布 `/cmd_vel`；
- Voice 不发布 `/cmd_vel`；
- Mission Manager 不发布 `/cmd_vel`；
- Perception 不直接控制 Navigation；
- Perception Target 不直接成为 `/goal_pose`；
- Safety Supervisor 保持最终运动安全门控职责；
- Emergency Stop Reset 不能由普通语音直接执行。

---

## 16. 当前版本不支持的接口

当前 v1.0 不生成：

```text
SpatialGoalRequest
ManualDriveRequest
独立 ReasonCodes 消息
```

因此诸如：

```text
前方 5 米
去地图上的这个位置
向左移动 2 米
```

这类任意空间目标不属于当前 TaskCommand v1.0。

未来如果支持 APP 地图点击绝对目标或语音相对空间目标，应通过独立的 Spatial Goal 合同实现，而不是扩展 TaskCommand 去携带任意坐标。

---

## 17. ROS 2 依赖

当前主要依赖：

```text
ROS 2 Humble
ament_cmake
rosidl_default_generators
builtin_interfaces
std_msgs
geometry_msgs
rosidl_default_runtime
```

---

## 18. 构建

示例：

```bash
source /opt/ros/humble/setup.bash

cd <workspace>

colcon build \
  --packages-select cleannav_interfaces

source install/setup.bash
```

检查 package：

```bash
ros2 pkg prefix cleannav_interfaces
```

检查消息：

```bash
ros2 interface show cleannav_interfaces/msg/TaskCommand
ros2 interface show cleannav_interfaces/msg/TaskStatus
ros2 interface show cleannav_interfaces/msg/SafetyStatus
```

检查服务：

```bash
ros2 interface show cleannav_interfaces/srv/SafetyLease
```

建议将：

```text
build/
install/
log/
```

放在源码仓库之外或保持 Git ignored。

---

## 19. 仓库结构

```text
cleannav-interfaces/
├── CMakeLists.txt
├── package.xml
├── README.md
├── CHANGELOG.md
├── config/
│   ├── app_task_schema.json
│   ├── task_catalog.yaml
│   └── voice_task_map_example.yaml
├── docs/
├── msg/
│   ├── CleaningTarget.msg
│   ├── CleaningTargetArray.msg
│   ├── PerceptionHealth.msg
│   ├── RobotStatus.msg
│   ├── SafetyStatus.msg
│   ├── TaskCommand.msg
│   └── TaskStatus.msg
└── srv/
    └── SafetyLease.srv
```

---

## 20. 与其他 CleanNav 仓库的关系

主要仓库关系：

```text
cleannav-interfaces
    ↑
    ├── cleannav-mission-manager
    ├── cleannav-navigation
    ├── cleannav-hmi
    └── 其他 CleanNav 组件
```

`cleannav_interfaces` 是共享合同层。

业务仓库不应复制一套不同版本的消息定义。

---

## 21. 决赛阶段基线

当前跨仓库 HIL 基线标签：

```text
competition-hil-baseline-20260921
```

当前 Interfaces 基线 commit：

```text
e3405138c081e500850ee991f5fa53aece71e1b4
```

该版本已经包含：

- TaskCommand confirmation；
- SafetyStatus；
- SafetyLease；
- visual target policy；
- 当前 Task Catalog；
- 当前 Voice task map。

稳定基线禁止重写历史。

---

## 22. 当前协作分支

由于 GitHub `main` 开启了分支保护，主分支修改需要通过 Pull Request。

当前交接同步分支：

```text
sync/competition-handoff-20260926
```

组员正式开发前，应优先以最新 `main` 为基础创建自己的功能分支。

推荐流程：

```bash
git clone https://github.com/D1zzyhub7/cleannav-interfaces.git
cd cleannav-interfaces

git switch main
git pull --ff-only

git switch -c feature/<your-feature>
```

接口合同属于跨组共享内容。

修改消息、服务、Task Catalog 或字段语义前，应先确认对其他模块的影响，避免不同组形成不兼容接口。
