# CleanNav Interfaces 项目状态

## 1. 模块定位

`cleannav_interfaces` 是 CleanNav 模块化系统的共享 ROS 2 接口与配置合同层。

它负责维护 Mission Manager、Perception、HMI 及其他系统组件之间的跨模块数据合同，不承载具体业务状态机、导航控制、感知算法或安全执行逻辑。

当前 ROS package：

`cleannav_interfaces`

当前 ROS 2 发行版：

`Humble`

当前 package version：

`0.1.0`

当前接口语义版本：

`1.0` 候选合同

当前独立仓库分支：

`main`

## 2. 当前 Git 基线

原 CleanNav monorepo 中 Interfaces 的唯一历史 commit：

`f60a9d57713211f060f1622d5c3bc8c9c06617f8`

提交主题：

`落地并验证 CleanNav v1.0 接口包`

经过 `git-filter-repo` 提取后的独立仓库 commit：

`f686d93e1e799aca4e023d0ef4e2bc9f75b8d513`

历史映射：

`f60a9d57713211f060f1622d5c3bc8c9c06617f8`

→

`f686d93e1e799aca4e023d0ef4e2bc9f75b8d513`

正式历史提取基线 tag：

`interfaces-v0.1.0-baseline`

## 3. 历史提取状态

原 monorepo 中 `src/cleannav_interfaces/` 的历史审计结果：

- Interfaces 相关历史 commit：1
- PURE commit：1
- MIXED commit：0
- 原 package 跟踪文件：12
- 过滤后跟踪文件：12
- 过滤后独立历史 commit：1
- 12 个文件逐字节一致性失败数：0

因此 Interfaces 的拆仓没有需要人工拆分的混合提交，也没有人为补造业务历史。

## 4. 当前仓库内容

当前技术内容包括 6 个 ROS 2 消息：

- `TaskCommand.msg`
- `TaskStatus.msg`
- `RobotStatus.msg`
- `CleaningTarget.msg`
- `CleaningTargetArray.msg`
- `PerceptionHealth.msg`

当前配置合同包括：

- `config/task_catalog.yaml`
- `config/app_task_schema.json`
- `config/voice_task_map_example.yaml`

构建与 package 元数据包括：

- `CMakeLists.txt`
- `package.xml`

## 5. 接口职责边界

### HMI → Mission Manager

`TaskCommand.msg`

用于传递预定义 mission 或 CONTROL 命令。

当前合同不允许通过该消息携带任意坐标或速度命令。

`command_id` 用于外部命令幂等。

### Mission Manager → HMI

`TaskStatus.msg`

用于表达 Command Scope 与 Execution Scope 生命周期状态。

State 与 Reason Code 的已发布数值必须保持兼容，不能在后续版本中随意改变原有含义。

### System → HMI

`RobotStatus.msg`

用于表达机器人综合状态。

`navigation_active`、`autonomous_enabled` 与 `emergency_stop_active` 是不同语义，不应混用。

### Perception → Mission Manager

`CleaningTarget.msg`

`CleaningTargetArray.msg`

用于表达 map frame 下的环境目标观测。

感知目标属于 Observation，不直接成为 Navigation Goal，也不直接写入 Costmap。

### Perception → Mission Manager / HMI

`PerceptionHealth.msg`

用于表达感知子系统健康状态，包括 camera、inference、TF 与 projection 等信息。

## 6. Task Catalog

权威任务配置：

`config/task_catalog.yaml`

当前 SHA256：

`5f75f8b9d68fb426cb2b5db4b6bb11abedc13ed57cc0e6b76399156e9dc7be4c`

Task Catalog 是任务定义与任务分流的权威配置来源。

消费者不应依据 `task_id` 数值区间自行推断实际任务行为。

Mission Manager 独立仓库已经改为通过 ROS package share 获取 Task Catalog，而不是依赖 monorepo 中的相对目录结构。

## 7. 当前冻结值验证

独立构建后已验证以下 `TaskStatus` 数值：

`STATE_EMERGENCY_STOPPED = 16`

`REASON_EXECUTION_ACTIVATED = 7`

`REASON_EXECUTION_RECORD_CORRUPT = 713`

这些结果来自独立生成的 `cleannav_interfaces` Python 消息类型。

## 8. 独立构建验证

拆仓后已经在干净 shell 中完成验证。

验证环境仅加载：

`/opt/ros/humble/setup.bash`

没有加载：

`~/code/cleannav/install/setup.bash`

验证结果：

`ENV_LEAK=0`

`BUILD=0`

`PREFIX=0`

`OVERLAY_LEAK=0`

`INTERFACE_FAILURES=0`

`PYTHON_IMPORT=0`

`CONFIG_IDENTITY_FAILURES=0`

`COLCON_TEST=0`

`TEST_RESULT=0`

## 9. ROS 接口发现验证

以下 6 个消息均通过 `ros2 interface show`：

`CleaningTarget`

`CleaningTargetArray`

`PerceptionHealth`

`RobotStatus`

`TaskCommand`

`TaskStatus`

接口发现失败数：

`INTERFACE_FAILURES=0`

## 10. Python 导入验证

以下消息均可从独立 install space 导入：

`CleaningTarget`

`CleaningTargetArray`

`PerceptionHealth`

`RobotStatus`

`TaskCommand`

`TaskStatus`

Python import 返回：

`PYTHON_IMPORT=0`

## 11. 配置安装验证

以下三个配置文件已经从独立仓库正确安装：

`app_task_schema.json`

`task_catalog.yaml`

`voice_task_map_example.yaml`

安装结果与源码逐字节一致。

配置一致性失败数：

`CONFIG_IDENTITY_FAILURES=0`

Task Catalog source SHA256 与 installed SHA256 均为：

`5f75f8b9d68fb426cb2b5db4b6bb11abedc13ed57cc0e6b76399156e9dc7be4c`

## 12. Test 状态

当前 package 未定义独立测试目标。

因此当前：

`colcon test-result`

结果为：

`0 tests, 0 errors, 0 failures, 0 skipped`

这不是构建失败。

当前主要验证证据由以下内容构成：

- clean-shell 独立 build
- ROS package prefix
- 6 个接口发现
- 6 个 Python message import
- 3 个 config 安装
- 3 个 config byte identity
- 冻结 Reason Code / State 数值检查

## 13. 当前安全边界

本接口合同继续保持以下系统边界：

- HMI 不直接发布 `/goal_pose`
- HMI 不直接发布 `/cmd_vel`
- Perception 不直接向 Navigation 提交 Goal
- Perception 不直接写 Costmap
- Mission Manager 不发布 `/cmd_vel`
- Safety Supervisor 负责最终运动安全门控

Interfaces 仓库只定义合同，不替代这些组件自身的实现。

## 14. 当前不包含的接口

当前 v1.0 候选实现不生成：

`SpatialGoalRequest`

`ManualDriveRequest`

独立 `ReasonCodes` 消息

`SpatialGoalRequest` 仅作为未来接口演进方向。

APP 手动遥控也不属于当前合同范围。

## 15. 当前模块化治理状态

Interfaces 独立仓库的主体模块化工作已经完成：

- 原 monorepo 安全备份与恢复验证
- Interfaces 历史只读审计
- 独立分析副本
- 独立过滤副本
- `git-filter-repo` 历史提取
- 原始文件 byte identity 验证
- clean-shell 独立构建
- ROS interface discovery
- Python import
- config install / byte identity
- 当前正式开发分支调整为 `main`
- 仓库 Git identity 配置
- `interfaces-v0.1.0-baseline` annotated tag
- `.gitignore`
- 独立仓库 README
- `CHANGELOG.md`
- `docs/PROJECT_STATUS.md`
- `docs/MONOREPO_MIGRATION.md`
- 五份治理文件总审计
- 治理 commit `ddaa54f1e17863c06f33fd4984d8dc352f96593e`
- 治理 commit clean-shell 独立回归验证
- `interfaces-modularization-baseline-20260826` annotated tag
- 过滤过程 inherited refs 独立审计
- inherited refs 清理
- 业务接口文件零修改验证

clean-shell 回归中曾发现验证脚本的 `colcon test-result` 未显式指定 `--log-base`，导致验证器在仓库根目录生成 ignored `log/`。该问题已定位为验证脚本缺陷，生成物已精确清理；修正后的命令验证通过，仓库根目录 `build/`、`install/`、`log/` 均不存在。

本状态更新提交完成后，只剩最终 refs、HEAD、工作区、文档与业务文件边界闭环审计。该审计通过后即可正式关闭 Interfaces G4。

当前尚未进入：

- GitHub remote 配置
- GitHub push
- `cleannav-system` 版本固定
- `cleannav-navigation` 拆仓

## 16. 遗留 Git refs 清理结果

`git-filter-repo` 完成后曾继承以下原 monorepo refs：

`backup/pre-modularization-20260825`

`feature/mission-manager-m0`

`pre-modularization-20260825`

它们在 Interfaces 过滤仓库中均最终指向：

`f686d93e1e799aca4e023d0ef4e2bc9f75b8d513`

清理前已经逐项验证：

- 两个 branch 相对 `main` 的独有 commit 数均为 0；
- inherited tag 的目标相对 `main` 的独有 commit 数为 0；
- `f686d93e1e799aca4e023d0ef4e2bc9f75b8d513` 仍是 `main` 的祖先；
- `interfaces-v0.1.0-baseline` 独立固定该历史提取节点；
- 原 monorepo 的安全 branch、annotated tag 和完整 bundle 均保持有效。

因此上述 3 个 inherited refs 已从 Interfaces 独立仓库删除。

当前本地正式 refs 收敛为：

- branch：`main`
- tag：`interfaces-v0.1.0-baseline`
- tag：`interfaces-modularization-baseline-20260826`

原 monorepo 语境中的旧 Mission Manager branch/tag 名称不再作为 Interfaces 仓库的活动 refs。

## 17. 分支策略

当前正式开发分支：

`main`

历史提取节点通过：

`interfaces-v0.1.0-baseline`

固定。

后续普通开发从 `main` 分支推进。

是否建立功能分支，应根据具体接口变更范围决定，而不继续沿用原 Mission Manager 的 branch 名称。

## 18. 版本策略

ROS package version 与接口语义版本分开理解。

当前：

package version：

`0.1.0`

接口语义：

`1.0` 候选合同

本轮拆仓不修改 package version，也不宣称接口已升级到新的正式版本。

模块化治理本身属于仓库组织变化，不应伪装成新的接口功能版本。

## 19. 与 Mission Manager 的关系

`cleannav_mission_manager` 是当前已知直接依赖 `cleannav_interfaces` 的 ROS package。

Mission Manager 独立仓库已经完成 Task Catalog 独立路径适配。

消费者应通过 ROS package 安装结果访问配置资源。

例如：

    from ament_index_python.packages import get_package_share_directory

而不是通过两个 Git 仓库之间的固定相对路径读取配置。

## 20. 与未来 CleanNav System 的关系

计划中的组件仓库包括：

`cleannav-interfaces`

`cleannav-mission-manager`

`cleannav-navigation`

`cleannav-perception`

`cleannav-hmi`

未来 `cleannav-system` 作为系统级仓库固定各组件的精确 Git revision。

Interfaces 仓库只负责共享合同自身的版本历史。

系统级组合版本不应写入 Interfaces 的业务 commit 历史。

## 21. 下一步

本状态更新提交完成后，执行 Interfaces G4 最终闭环审计：

- 当前 branch 必须仅为 `main`；
- 当前正式 tags 必须仅为 `interfaces-v0.1.0-baseline` 和 `interfaces-modularization-baseline-20260826`；
- HEAD、历史基线 tag 和模块化治理 baseline tag 必须保持精确目标；
- 工作区必须 clean；
- 6 个 ROS 2 消息、3 个配置合同、`CMakeLists.txt` 和 `package.xml` 不得因收尾文档更新发生变化；
- 原 monorepo 安全 branch、tag 与 bundle 必须保持有效。

上述审计通过后，正式关闭 Interfaces G4。

之后进入 `cleannav-navigation` 的拆仓工作。

GitHub remote 与 push 暂不提前进行，待本地组件仓库治理继续完成后统一处理。
