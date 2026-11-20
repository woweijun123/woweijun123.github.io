---
{"dg-publish":true,"permalink":"/Work/Tools/Openwrt/Clash X/Clash 故障检测/","title":"Clash 故障检测","tags":["踩坑","openwrt","openclash"],"noteIcon":"","created":"2026-07-16T11:31:44.081+08:00","updated":"2026-07-16T11:46:03.528+08:00","dg-note-properties":{"title":"Clash 故障检测","tags":["踩坑","openwrt","openclash"],"reference linking":null,"created":"2026-07-14"}}
---

# 结论
节点延迟检测失败不是单一节点故障，而是**生效配置、订阅缓存和自动订阅任务互相冲突**；回退到完整 `config.yaml` 后，108 个节点与多个远程规则 Provider 又多次触发 **OOM Kill**，导致 Clash 进程被内核杀死。
关联运行配置与日常操作见 [[Work/Tools/Openwrt/Clash X/Clash NX30Pro\|Clash NX30Pro]]。
# 事故影响
- **节点延迟全部失败**
- **面板不可达**
- Clash 进程反复被 **OOM Kill**
- **代理服务间歇性中断**
# 触发条件

| 检查项 | 故障时表现 | 修复后结果 |
| :-- | :-- | :-- |
| **OpenClash 服务** | LuCI 显示运行，但未加载精简配置 | 加载 `home-minimal.yaml` |
| **精简配置订阅** | Provider 仍使用旧 URL 对应的缓存 | 删除缓存后重新下载 |
| **自动订阅** | 重启时将配置切回 `config.yaml` | 任务已关闭 |
| **完整配置** | 大量节点与规则 Provider 导致 OOM | 不再作为运行配置 |
| **节点延迟** | 不稳定或失败 | `自动选择` 测得 510 ms |
# 根因链路
1. `/etc/openclash/config/home-minimal.yaml` 与 `/etc/openclash/home-minimal.yaml` 是不同文件，**订阅 URL 不一致**。
2. `config_path` 指向 `/etc/openclash/config/config.yaml` 时，实际运行的是**完整配置**，手工更新另一份精简配置不会生效。
3. 开启的 **OpenClash 自动订阅任务**会在重启后覆盖 `config_path` 选择，重新切换到 `config.yaml`。
4. 完整配置加载 108 个节点和多个远程规则 Provider；低内存下发生 **OOM Kill**，健康检查和延迟测试因此中断。
# 修复步骤
1. 备份旧的**精简配置**和 `airport` Provider 缓存。
2. 将较新的 `/etc/openclash/home-minimal.yaml` 同步到 `/etc/openclash/config/home-minimal.yaml`，此后只维护后者。
3. 禁用旧的 **OpenClash 自动订阅任务**，保持 `config_subscribe.enabled=0`。
4. 将 OpenClash 的 `config_path` 设置为 `/etc/openclash/config/home-minimal.yaml`。
5. 删除旧 `/etc/openclash/proxy_provider/airport.yaml`，重启 OpenClash，使 Mihomo 按更新的 URL 重新拉取节点。
# 验证结果
| 检查项 | 结果 |
| :-- | :-- |
| **当前配置路径** | `/etc/openclash/config/home-minimal.yaml` |
| **自动订阅覆盖** | 已禁用 |
| **Provider 节点条目** | 99 |
| **OpenClash 控制器** | 可访问 |
| **经 VPN 访问 `generate_204`** | 成功 |
| **`自动选择` 延迟测试** | 510 ms |
# 防复发建议
- 只维护 `/etc/openclash/config/home-minimal.yaml` 一个**运行配置**。
- 修改配置后立即检查 `uci get openclash.config.config_path`。
- 改订阅 URL 后删除 `airport.yaml` 缓存再重启。
- 保持 `config_subscribe.enabled=0`。
- 节点延迟异常先核查 `config_path`、Provider 缓存和 **OOM 日志**。
