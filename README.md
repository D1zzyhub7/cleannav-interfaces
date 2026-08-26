# CleanNav Interfaces

`cleannav_interfaces` 是 CleanNav 无人清扫车系统的共享 ROS 2 接口与配置合同仓库。

该仓库从原 CleanNav monorepo 中独立提取，用于在 Mission Manager、感知、HMI 以及其他系统组件之间维护稳定、可版本化的跨模块数据合同。

当前 ROS package 版本为 `0.1.0`，当前接口语义版本为 `1.0` 候选合同。

## 1. 仓库职责

本仓库负责：

- 定义 CleanNav 跨模块 ROS 2 消息；
- 维护 Task Catalog；
- 维护 APP 请求 JSON Schema；
- 提供离线语音任务映射示例；
- 安装上述配置文件到 ROS package share；
- 为 Mission Manager、感知和 HMI 等组件提供统一接口合同。

本仓库不负责：

- Mission Manager 状态机或任务调度实现；
- 导航规划、轨迹跟踪或动态避障；
- Safety Supervisor 的执行逻辑；
- 感知算法和目标检测实现；
- APP 或语音 Bridge 的业务实现；
- 发布 `/cmd_vel`；
- 直接向 Navigation 提交任意目标。

## 2. 当前接口

当前生成 6 个 ROS 2 消息。

### HMI 与任务管理

- `TaskCommand.msg`
  - HMI → Mission Manager
  - 表达预定义任务或控制命令；
  - 不携带任意坐标或速度命令；
  - `command_id` 作为外部幂等请求标识。

- `TaskStatus.msg`
  - Mission Manager → HMI
  - 表达 Command Scope 和 Execution Scope 的任务生命周期状态；
  - 包含冻结的 state 与 reason code 数值合同。

- `RobotStatus.msg`
  - Mission Manager / Status Aggregator → HMI
  - 表达机器人综合运行状态；
  - 区分 navigation active、autonomous enabled 与 emergency stop。

### 感知与任务管理

- `CleaningTarget.msg`
  - 表达单个清扫目标环境观测；
  - 目标位置位于 `map` frame；
  - 不直接成为 Navigation Goal 或 Costmap 数据。

- `CleaningTargetArray.msg`
  - 表达一批清扫目标观测；
  - `sequence` 仅用于批次排序，不用于 command 幂等。

- `PerceptionHealth.msg`
  - 表达感知子系统健康状态；
  - 包含 camera、inference、TF 和 projection 等健康信息。

## 3. 当前 Topic 合同

当前候选合同中的主要 Topic 为：

| Topic | 消息类型 | 主要方向 |
| --- | --- | --- |
| `/cleannav/hmi/task_command` | `TaskCommand` | HMI → Mission Manager |
| `/cleannav/task_status` | `TaskStatus` | Mission Manager → HMI |
| `/cleannav/robot_status` | `RobotStatus` | System → HMI |
| `/cleannav/perception/cleaning_targets` | `CleaningTargetArray` | Perception → Mission Manager |
| `/cleannav/perception/health` | `PerceptionHealth` | Perception → Mission Manager / HMI |

Topic 的具体 QoS、状态机语义和消费者行为由系统级设计及对应组件实现共同约束。

## 4. 配置合同

### `config/task_catalog.yaml`

Task Catalog 是任务定义与命令分流的权威配置来源。

它定义：

- `task_id`；
- `task_kind`；
- `enabled`；
- `allowed_sources`；
- confirmation 要求；
- 固定目标、路线或视觉目标相关参数。

Mission Manager 不应依据数字 ID 区间自行推断任务行为，而应通过 Task Catalog Loader 读取实际任务定义。

安装后应通过 ROS package share 获取该文件，而不是依赖仓库之间的相对路径。

Python 消费者可使用：

    from ament_index_python.packages import get_package_share_directory
    from pathlib import Path

    share = Path(
        get_package_share_directory("cleannav_interfaces")
    )

    catalog = share / "config" / "task_catalog.yaml"

### `config/app_task_schema.json`

定义 APP 到 APP Bridge 的外部任务请求 JSON Schema。

当前核心字段包括：

- `command_id`；
- `task_id`；
- `timestamp_ms`；
- `valid_for_ms`。

APP JSON 请求本身不是 ROS 2 `TaskCommand`，应由 APP Bridge 完成验证和转换。

### `config/voice_task_map_example.yaml`

提供离线语音到预定义任务的映射示例。

该文件是 HMI / Voice Bridge 的示例配置，不改变 Task Catalog 的权威地位。

语音 Bridge 在产生命令前仍应检查：

- Task 是否 enabled；
- VOICE 是否在 `allowed_sources` 中；
- confidence 和 confirmation 条件。

## 5. 版本边界

当前接口语义版本为 `1.0` 候选合同。

当前仅生成以下 6 个消息：

    TaskCommand
    TaskStatus
    RobotStatus
    CleaningTarget
    CleaningTargetArray
    PerceptionHealth

当前不生成：

- `SpatialGoalRequest`
- `ManualDriveRequest`
- 独立 `ReasonCodes` 消息

`SpatialGoalRequest` 属于未来接口版本的预留方向，不属于当前 v1.0 实现。

APP 手动遥控当前也不属于本接口版本范围。

## 6. 安全边界

接口合同遵循以下系统边界：

- HMI 不直接发布 `/goal_pose`；
- HMI 不直接发布 `/cmd_vel`；
- 感知输出属于 Observation，而不是 Navigation Command；
- 感知目标不直接写入 Costmap；
- Mission Manager 不发布 `/cmd_vel`；
- Safety Supervisor 保持最终运动安全门控职责。

接口包只定义跨模块合同，不替代各组件内部的安全逻辑。

## 7. ROS 2 依赖

当前 package 主要依赖：

- ROS 2 Humble；
- `ament_cmake`；
- `rosidl_default_generators`；
- `builtin_interfaces`；
- `std_msgs`；
- `geometry_msgs`；
- `rosidl_default_runtime`。

## 8. 独立构建

推荐在独立的 colcon workspace 或外部 build/install/log 目录中构建。

最简单的独立构建示例：

    source /opt/ros/humble/setup.bash

    cd <workspace>

    colcon build \
      --packages-select cleannav_interfaces

    source install/setup.bash

验证 package：

    ros2 pkg prefix cleannav_interfaces

验证消息：

    ros2 interface show \
      cleannav_interfaces/msg/TaskCommand

    ros2 interface show \
      cleannav_interfaces/msg/TaskStatus

如果直接从本仓库进行工程验证，建议将 build、install 和 log 放到仓库外部，以避免生成文件参与源码审计。

## 9. 已验证的独立性

拆仓后已经在只加载 `/opt/ros/humble` 的干净 shell 中完成独立验证。

验证内容包括：

- 原 CleanNav monorepo overlay 未进入环境；
- `colcon build` 成功；
- ROS 正确解析独立安装的 `cleannav_interfaces`；
- 6 个消息均可通过 `ros2 interface show` 发现；
- 6 个消息均可完成 Python import；
- 3 个配置文件均成功安装；
- 安装后的配置与源码逐字节一致；
- `colcon test` 和 `colcon test-result` 正常完成。

当前 package 未定义独立测试目标，因此 `colcon test-result` 为 0 tests；接口生成、消息发现、Python import 和配置一致性验证作为当前主要功能验证。

当前 Task Catalog SHA256：

    5f75f8b9d68fb426cb2b5db4b6bb11abedc13ed57cc0e6b76399156e9dc7be4c

## 10. 仓库结构

    .
    ├── .gitignore
    ├── CHANGELOG.md
    ├── CMakeLists.txt
    ├── README.md
    ├── package.xml
    ├── config/
    │   ├── app_task_schema.json
    │   ├── task_catalog.yaml
    │   └── voice_task_map_example.yaml
    ├── docs/
    │   ├── MONOREPO_MIGRATION.md
    │   └── PROJECT_STATUS.md
    └── msg/
        ├── CleaningTarget.msg
        ├── CleaningTargetArray.msg
        ├── PerceptionHealth.msg
        ├── RobotStatus.msg
        ├── TaskCommand.msg
        └── TaskStatus.msg

仓库治理文档：

- `CHANGELOG.md`：记录接口、配置、验证与仓库治理的重要变化；
- `docs/PROJECT_STATUS.md`：记录当前技术基线、验证状态、边界与下一步；
- `docs/MONOREPO_MIGRATION.md`：记录 monorepo 拆分、Git SHA 映射、独立验证和回滚依据。

## 11. 历史迁移

本仓库从原 CleanNav monorepo 中的：

    src/cleannav_interfaces/

独立提取。

原接口实现 commit：

    f60a9d57713211f060f1622d5c3bc8c9c06617f8

过滤后的独立仓库 commit：

    f686d93e1e799aca4e023d0ef4e2bc9f75b8d513

两者对应的接口源码和配置内容保持逐字节一致。

正式历史基线 tag：

    interfaces-v0.1.0-baseline

更完整的迁移映射、验证结果和治理状态将在 `docs/` 中持续维护。

## 12. 与其他 CleanNav 仓库的关系

CleanNav 模块化后计划由多个独立组件仓库组成，包括：

- `cleannav-interfaces`
- `cleannav-mission-manager`
- `cleannav-navigation`
- `cleannav-perception`
- `cleannav-hmi`

未来系统级仓库 `cleannav-system` 将负责固定各组件的精确版本，从而支持整套系统的可重复构建与回滚。

`cleannav_interfaces` 是这些跨模块组件之间的共享合同层，不属于任何单一业务模块的私有实现。
