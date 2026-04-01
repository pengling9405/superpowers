# 更新日志

## [5.0.5] - 2026-03-17

### 修复

- **Brainstorm server ESM 修复**：将 `server.js` 重命名为 `server.cjs`，确保 brainstorming server 在 Node.js 22+ 下正常启动，因为根目录 `package.json` 的 `"type": "module"` 会导致 `require()` 失败。([PR #784](https://github.com/obra/superpowers/pull/784)，作者 @sarbojitrana，修复 [#774](https://github.com/obra/superpowers/issues/774)、[#780](https://github.com/obra/superpowers/issues/780)、[#783](https://github.com/obra/superpowers/issues/783))
- **Windows 上的 Brainstorm owner-PID**：在 Windows/MSYS2 下跳过 `BRAINSTORM_OWNER_PID` 生命周期监控，因为 Node.js 看不到对应的 PID namespace。这样可以避免服务在 60 秒后自杀式退出。30 分钟空闲超时仍作为安全兜底。([#770](https://github.com/obra/superpowers/issues/770)，文档见 [PR #768](https://github.com/obra/superpowers/pull/768)，作者 @lucasyhzhu-debug)
- **stop-server.sh 可靠性**：在报告成功前先确认服务进程确实已退出。会等待最多 2 秒进行优雅关闭，必要时升级到 `SIGKILL`，如果进程仍存活则报告失败。([#723](https://github.com/obra/superpowers/issues/723))

### 变更

- **执行交接**：在写完计划后，恢复用户在 `subagent-driven-development` 与 `executing-plans` 之间的选择权。仍然推荐 subagent-driven，但不再强制要求。（回滚 `5e51c3e`）
