# Changelog

本文件记录 `cleannav_interfaces` 独立仓库的重要接口、配置、构建和仓库治理变化。

当前 ROS package 版本为 `0.1.0`。接口语义版本与 ROS package 版本分别维护；当前接口语义为 `1.0` 候选合同。

## [Unreleased]

### Repository modularization

- 从原 CleanNav monorepo 中独立提取 `src/cleannav_interfaces/`。
- 使用 `git-filter-repo` 保留真实 Interfaces 历史，不人为补造或重写业务开发记录。
- 原 monorepo Interfaces commit：
  `f60a9d57713211f060f1622d5c3bc8c9c06617f8`
- 过滤后的独立仓库 commit：
  `f686d93e1e799aca4e023d0ef4e2bc9f75b8d513`
- 两个 commit 中的 12 个 Interfaces 文件已经完成逐字节一致性验证。
- 建立独立历史基线 tag：
  `interfaces-v0.1.0-baseline`
- 独立仓库默认开发分支调整为 `main`。
- 增加独立仓库 `.gitignore`。
- 重写 README，使构建、职责、依赖和历史说明不再依赖原 monorepo 路径。
- 已补齐独立仓库治理文档，包括 `CHANGELOG.md`、`docs/PROJECT_STATUS.md` 和 `docs/MONOREPO_MIGRATION.md`。
- 建立独立仓库治理 commit：`ddaa54f1e17863c06f33fd4984d8dc352f96593e`。
- 对治理 commit 完成独立 clean-shell 回归；构建、接口发现、Python import、配置一致性、Task Catalog 和 `colcon test` 均通过。
- 回归验证过程中发现验证脚本的 `colcon test-result` 遗漏 `--log-base`，修正验证器后确认仓库根目录不存在 `build/`、`install/` 或 `log/` 构建产物。
- 建立 annotated tag `interfaces-modularization-baseline-20260826`，固定已验证治理基线 `ddaa54f1e17863c06f33fd4984d8dc352f96593e`。
- 已审计并删除过滤过程继承的两个旧 Mission Manager branch 和旧 `pre-modularization-20260825` tag；这些 refs 不包含独有 commit。

### Independent validation

已在不加载原 CleanNav monorepo overlay 的干净 ROS 2 Humble 环境中验证：

- `ENV_LEAK=0`
- `BUILD=0`
- `PREFIX=0`
- `OVERLAY_LEAK=0`
- `INTERFACE_FAILURES=0`
- `PYTHON_IMPORT=0`
- `CONFIG_IDENTITY_FAILURES=0`
- `COLCON_TEST=0`
- `TEST_RESULT=0`

独立构建验证包括：

- `cleannav_interfaces` 可单独完成 `colcon build`；
- ROS package prefix 指向独立验证 install space；
- 原 monorepo install 不在 `AMENT_PREFIX_PATH` 中；
- 6 个消息均可通过 `ros2 interface show` 发现；
- 6 个消息均可完成 Python import；
- 3 个配置文件均成功安装；
- 安装后的 3 个配置文件与源码逐字节一致；
- 当前 package 未定义独立测试目标，因此 `colcon test-result` 为 0 tests。

### Contract validation

已验证以下冻结数值：

- `TaskStatus.STATE_EMERGENCY_STOPPED = 16`
- `TaskStatus.REASON_EXECUTION_ACTIVATED = 7`
- `TaskStatus.REASON_EXECUTION_RECORD_CORRUPT = 713`

当前 Task Catalog SHA256：

`5f75f8b9d68fb426cb2b5db4b6bb11abedc13ed57cc0e6b76399156e9dc7be4c`

### Current boundaries

本轮仓库模块化不修改：

- 6 个 ROS 2 `.msg` 文件；
- `task_catalog.yaml`；
- `app_task_schema.json`；
- `voice_task_map_example.yaml`；
- `package.xml` 中的 package version `0.1.0`；
- 原有接口语义。

本轮只处理仓库拆分、独立构建验证和治理。

## [0.1.0] - Historical baseline

原 CleanNav monorepo 中首次落地并验证 Interfaces v1.0 候选包的历史版本。

原始 commit：

`f60a9d57713211f060f1622d5c3bc8c9c06617f8`

独立仓库对应 commit：

`f686d93e1e799aca4e023d0ef4e2bc9f75b8d513`

该版本包含 6 个 ROS 2 消息：

- `TaskCommand`
- `TaskStatus`
- `RobotStatus`
- `CleaningTarget`
- `CleaningTargetArray`
- `PerceptionHealth`

以及 3 个配置合同：

- `config/task_catalog.yaml`
- `config/app_task_schema.json`
- `config/voice_task_map_example.yaml`

这一历史节点通过 tag `interfaces-v0.1.0-baseline` 保留。

后续仓库治理 commit 不改变这一历史 commit，也不重写其原始作者信息。
