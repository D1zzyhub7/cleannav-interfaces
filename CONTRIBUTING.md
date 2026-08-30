# 贡献 CleanNav Interfaces

- `main` 只保留稳定、通过 Gate 的接口；不要直接在 `main` 开发。
- 普通功能使用 `feature/*`，bugfix 使用 `fix/*`；接口变更建议使用 `feature/interface-*`。
- 一个 PR 只处理一个明确 scope；提交 PR 前必须完成适用的 build/test。
- 禁止对 `main` force push；禁止改写、reset 或 rebase 已共享的公共历史。
- 跨仓库接口变更必须同步协调 Mission Manager、Navigation、Perception、HMI 等消费者。
- 修改 msg/srv/action/config schema 时，必须明确记录兼容性影响。
- 不要提交 `build/`、`install/`、`log/`、token、密码、私钥、数据库或 rosbag。
- 合并前必须完成 review。
