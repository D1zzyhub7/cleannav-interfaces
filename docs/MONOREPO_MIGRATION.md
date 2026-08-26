# CleanNav Interfaces Monorepo Migration Record

## 1. 文档目的

本文记录 `cleannav_interfaces` 从原 CleanNav monorepo 独立提取为组件仓库的完整迁移事实。

该文档用于：

- 保存原始仓库与独立仓库之间的 Git 历史映射；
- 记录过滤范围与过滤依据；
- 记录源码逐字节一致性结果；
- 记录独立构建与接口验证结果；
- 区分真实历史与后续仓库治理 commit；
- 为未来 `cleannav-system` 固定组件 revision 提供审计依据。

本次迁移不修改 Interfaces 的业务合同，不人为补造历史提交。

## 2. 原始仓库

原 CleanNav monorepo：

    ~/code/cleannav

拆仓时原仓库分支：

    feature/mission-manager-m1

拆仓安全基线 HEAD：

    0830c301084df41fd2e39501c6d52e10c5465892

原仓库在 Interfaces 拆分过程中保持不变。

## 3. 拆仓前安全备份

拆仓前已经创建完整 Git bundle：

    ~/code/cleannav_backups/cleannav-pre-modularization-20260825.bundle

bundle SHA256：

    ed19c0268b4ed995bd86d1123a3e56913b28cfe7230593e2963fcfbbd4444427

该 bundle 已通过 `git bundle verify`。

同时已经从该 bundle 完成独立恢复测试，确认可从零恢复原仓库历史。

恢复验证时确认：

- HEAD 与原仓库一致；
- tree 与原仓库一致；
- commit 数量一致；
- 安全 tag 可解析；
- 工作区干净。

因此本次 Interfaces 提取不是唯一副本上的不可逆历史操作。

## 4. Interfaces 原始路径

原 monorepo 中 Interfaces package 位于：

    src/cleannav_interfaces/

拆分前该目录共有 12 个 tracked files：

    CMakeLists.txt
    README.md
    package.xml
    config/app_task_schema.json
    config/task_catalog.yaml
    config/voice_task_map_example.yaml
    msg/CleaningTarget.msg
    msg/CleaningTargetArray.msg
    msg/PerceptionHealth.msg
    msg/RobotStatus.msg
    msg/TaskCommand.msg
    msg/TaskStatus.msg

## 5. 历史审计

对原 monorepo 中：

    src/cleannav_interfaces/

进行历史审计后得到：

    Interfaces relevant commits = 1
    PURE commits = 1
    MIXED commits = 0

唯一 Interfaces 历史提交：

    f60a9d57713211f060f1622d5c3bc8c9c06617f8

提交主题：

    落地并验证 CleanNav v1.0 接口包

该提交只包含 Interfaces package 的 12 个文件，没有同时修改其他 monorepo 路径。

因此无需人工拆分 mixed commit，也无需伪造新的业务开发历史。

## 6. 分析副本

拆仓分析副本：

    ~/code/cleannav_modularization/cleannav-interfaces-extract

该副本从安全 bundle 创建。

在分析副本中执行：

    git filter-repo --analyze

分析结果：

    Processed blobs = 253
    Processed commits = 22

分析再次确认 Interfaces package 当前只涉及一个真实历史 commit。

分析副本仅用于审计，没有作为最终过滤仓库继续使用。

## 7. 最终过滤副本

最终独立过滤仓库：

    ~/code/cleannav_modularization/cleannav-interfaces-filtered

该仓库重新从安全 bundle 创建。

使用的历史过滤范围：

    src/cleannav_interfaces/

过滤方式等价于：

    git filter-repo --subdirectory-filter src/cleannav_interfaces

过滤后原 package 目录被提升为独立仓库根目录。

## 8. Git 历史映射

原 Interfaces commit：

    f60a9d57713211f060f1622d5c3bc8c9c06617f8

过滤后的独立仓库 commit：

    f686d93e1e799aca4e023d0ef4e2bc9f75b8d513

映射关系：

| Old monorepo commit | New Interfaces commit | Subject |
| --- | --- | --- |
| `f60a9d57713211f060f1622d5c3bc8c9c06617f8` | `f686d93e1e799aca4e023d0ef4e2bc9f75b8d513` | 落地并验证 CleanNav v1.0 接口包 |

`.git/filter-repo/commit-map` 中只有这一条 non-zero retained mapping。

其他原 monorepo commit 因未包含 `src/cleannav_interfaces/` 内容，在过滤结果中映射为 zero SHA。

## 9. 过滤后历史

过滤完成后：

    tracked files = 12
    unique commits = 1

过滤后的唯一历史 commit：

    f686d93e1e799aca4e023d0ef4e2bc9f75b8d513

原提交作者仍保持历史记录中的：

    d1zzy <d1zzy@localhost>

该历史作者信息不进行 rewrite。

后续独立仓库的新治理 commit 使用当前仓库配置：

    d1zzy <1339980053@qq.com>

历史作者与未来作者信息明确区分。

## 10. 文件逐字节一致性

过滤后 12 个 package 文件逐一与原 monorepo 中对应文件进行 `cmp`。

结果：

    BYTE_IDENTITY_FAILURES=0

以下文件全部一致：

    CMakeLists.txt
    README.md
    package.xml
    config/app_task_schema.json
    config/task_catalog.yaml
    config/voice_task_map_example.yaml
    msg/CleaningTarget.msg
    msg/CleaningTargetArray.msg
    msg/PerceptionHealth.msg
    msg/RobotStatus.msg
    msg/TaskCommand.msg
    msg/TaskStatus.msg

因此 `git-filter-repo` 只改变了仓库历史和路径结构，没有改变 Interfaces 原始业务内容。

## 11. Task Catalog 基线

原 monorepo Task Catalog：

    src/cleannav_interfaces/config/task_catalog.yaml

过滤后：

    config/task_catalog.yaml

SHA256：

    5f75f8b9d68fb426cb2b5db4b6bb11abedc13ed57cc0e6b76399156e9dc7be4c

拆仓前 source 与原 monorepo installed catalog 已验证一致。

拆仓后 source 与独立 install catalog 也再次验证一致。

## 12. 独立验证环境

独立验证目录：

    ~/code/cleannav_modularization/interfaces-validation

构建、安装和日志均放置在仓库外：

    interfaces-validation/build
    interfaces-validation/install
    interfaces-validation/log

验证 shell 使用：

    env -i

并且在加载 ROS 前确认：

    AMENT_PREFIX_PATH=UNSET
    COLCON_PREFIX_PATH=UNSET
    CMAKE_PREFIX_PATH=UNSET
    PYTHONPATH=UNSET
    LD_LIBRARY_PATH=UNSET

随后只加载：

    /opt/ros/humble/setup.bash

没有加载原 monorepo：

    ~/code/cleannav/install/setup.bash

## 13. 独立构建结果

Clean-shell 独立验证结果：

    ENV_LEAK=0
    BUILD=0
    PREFIX=0
    OVERLAY_LEAK=0
    INTERFACE_FAILURES=0
    PYTHON_IMPORT=0
    CONFIG_IDENTITY_FAILURES=0
    COLCON_TEST=0
    TEST_RESULT=0

独立 package prefix：

    ~/code/cleannav_modularization/interfaces-validation/install/cleannav_interfaces

激活的 AMENT prefix 仅包括：

    ~/code/cleannav_modularization/interfaces-validation/install/cleannav_interfaces
    /opt/ros/humble

原 monorepo install 未进入验证环境。

## 14. ROS Interface 验证

以下 6 个消息全部通过：

    ros2 interface show

验证消息：

    CleaningTarget
    CleaningTargetArray
    PerceptionHealth
    RobotStatus
    TaskCommand
    TaskStatus

结果：

    INTERFACE_FAILURES=0

## 15. Python Import 验证

以下 6 个生成消息类型全部能够从独立 install space 导入：

    CleaningTarget
    CleaningTargetArray
    PerceptionHealth
    RobotStatus
    TaskCommand
    TaskStatus

结果：

    PYTHON_IMPORT=0

同时验证冻结值：

    TaskStatus.STATE_EMERGENCY_STOPPED = 16
    TaskStatus.REASON_EXECUTION_ACTIVATED = 7
    TaskStatus.REASON_EXECUTION_RECORD_CORRUPT = 713

## 16. 配置安装验证

独立构建后验证以下配置：

    app_task_schema.json
    task_catalog.yaml
    voice_task_map_example.yaml

三个 installed config 均与 source config 逐字节一致。

结果：

    CONFIG_IDENTITY_FAILURES=0

Task Catalog source 与 install SHA256 均为：

    5f75f8b9d68fb426cb2b5db4b6bb11abedc13ed57cc0e6b76399156e9dc7be4c

## 17. Colcon Test

执行：

    colcon test --packages-select cleannav_interfaces

返回：

    COLCON_TEST=0

执行：

    colcon test-result --verbose

返回：

    0 tests, 0 errors, 0 failures, 0 skipped

以及：

    TEST_RESULT=0

当前 Interfaces package 没有定义独立 test target。

因此 0 tests 是当前 package 结构的正常结果，不代表构建失败。

## 18. 独立仓库历史基线

过滤后正式建立：

    main

作为独立仓库默认开发分支。

原始提取节点 tag：

    interfaces-v0.1.0-baseline

该 annotated tag 指向：

    f686d93e1e799aca4e023d0ef4e2bc9f75b8d513

tag 说明：

    CleanNav Interfaces v0.1.0 历史提取基线

该 tag 用于永久区分：

- 从 monorepo 原样提取出来的真实历史内容；
- 后续独立仓库治理 commit。

## 19. 过滤过程继承的旧 refs

`git-filter-repo` 后以下原 monorepo refs 被保留并塌缩到同一个 Interfaces commit：

    backup/pre-modularization-20260825
    feature/mission-manager-m0
    pre-modularization-20260825

它们当前均最终指向：

    f686d93e1e799aca4e023d0ef4e2bc9f75b8d513

这些名称属于原 monorepo 语境，不是新的 Interfaces 分支或 tag 策略。

在独立仓库治理 commit 和新的 modularization baseline tag 建立完成前暂时保留。

后续会单独审计并清理，不与业务修改混在同一个步骤中。

## 20. 当前 package version

`package.xml` 当前版本：

    0.1.0

本次拆仓不修改 package version。

仓库拆分与治理属于 repository organization change，不伪装成新的接口功能版本。

接口语义当前仍为：

    1.0 候选合同

## 21. 与 Mission Manager 的依赖关系

当前已知直接声明：

    cleannav_interfaces

依赖的 ROS package 为：

    cleannav_mission_manager

Mission Manager 独立仓库已经完成 Task Catalog 路径适配。

其真实 catalog 测试使用：

    get_package_share_directory("cleannav_interfaces")

读取安装后的 package share。

因此拆仓后不再依赖两个 Git repository 的固定兄弟目录结构。

## 22. 原 monorepo 安全状态

Interfaces 过滤、构建与治理过程中，原 monorepo 保持在：

    0830c301084df41fd2e39501c6d52e10c5465892

没有执行历史重写。

没有删除原仓库。

没有将独立过滤结果覆盖回原仓库。

## 23. Mission Manager 独立仓库安全状态

Interfaces 拆分期间，Mission Manager 独立仓库保持在治理基线：

    7cb740a475f7e9667b084c51fbccdf6c5a42905a

Interfaces 拆分没有修改 Mission Manager 历史或工作树。

## 24. 后续治理

本迁移记录所在的独立仓库治理提交已经包含：

- README 独立仓库化；
- `.gitignore`；
- `CHANGELOG.md`；
- `docs/PROJECT_STATUS.md`；
- `docs/MONOREPO_MIGRATION.md`；
- 治理文件总审计；
- 五份治理文件精确 staging。

上述治理内容不修改当前 6 个 ROS 2 消息、3 个配置合同或 `package.xml` 的 `0.1.0` 版本。

治理提交完成后剩余的 Interfaces 模块化收尾工作为：

- clean-shell 独立回归验证；
- 创建 Interfaces modularization baseline tag；
- 审查并清理过滤过程继承的旧 branch / tag；
- 完成最终仓库与 refs 审计。

这些收尾步骤完成后，Interfaces 模块化 G4 才正式关闭。

## 25. 系统级回滚原则

未来计划由：

    cleannav-system

作为系统级组合仓库固定各组件的精确 Git revision。

Interfaces 自身的 tag 用于表达 Interfaces 历史与里程碑。

系统级 release/tag 则负责固定：

- `cleannav-interfaces`
- `cleannav-mission-manager`
- `cleannav-navigation`
- `cleannav-perception`
- `cleannav-hmi`

的精确组合版本。

因此单个组件仓库不需要通过伪造跨仓库 commit 来表达整套系统状态。
