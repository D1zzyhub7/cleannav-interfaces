# CleanNav Interface Package v1.0 (候选)

v1.0 候选接口包，不是最终 frozen contract。已依据 M0 架构、接口、状态机和 Reason Code 文档生成。

## v1.0 Topic

| Topic | 消息类型 |
|-------|---------|
| `/cleannav/hmi/task_command` | `TaskCommand` |
| `/cleannav/task_status` | `TaskStatus` |
| `/cleannav/robot_status` | `RobotStatus` |
| `/cleannav/perception/cleaning_targets` | `CleaningTargetArray` |
| `/cleannav/perception/health` | `PerceptionHealth` |

## 版本边界

- v1.0 仅生成 6 个消息（TaskCommand、TaskStatus、RobotStatus、CleaningTarget、CleaningTargetArray、PerceptionHealth）
- `SpatialGoalRequest` 为 v1.1 文档预留，当前不生成
- `ManualDriveRequest` 不属于 v1.0 或当前 v1.1，不生成。APP 手动遥控当前不做

## 安全边界

- HMI 和感知不得发布 `/goal_pose` 或 `/cmd_vel`
- Mission Manager 不直接调用 FollowPath
- Safety Supervisor 是 `/cmd_vel` 唯一发布者

## 构建

```bash
cd ~/code/cleannav
colcon build --packages-select cleannav_interfaces
source install/setup.bash
```

## 配置文件

- `config/task_catalog.yaml` — Task Catalog：任务 ID、类型、启用状态、来源和参数
- `config/app_task_schema.json` — APP JSON Schema：从 APP 到 APP Bridge 的外部合同
- `config/voice_task_map_example.yaml` — 离线语音任务映射示例

## 权威文档

- `docs/mission_manager_architecture.md` — 总体架构
- `docs/mission_manager_interface.md` — 接口设计
- `docs/mission_manager_state_machine.md` — 状态机设计
- `docs/mission_manager_reason_codes.md` — Reason Code 设计
- `docs/mission_manager_test_plan.md` — 测试计划
