# Release Notes / Changelog (Public)

## ProtocolSentinel v1.0.0 (2026-03-19)

**Highlights**
- OpenClaw v2026.3.13 Poll 注入 bug 彻底定位与补丁
- 手把手 Debug SOP + 补丁 SOP

**What’s New**
- 发布完整 README：复现 → 定位 → 补丁 → 验证 → 风险提示
- 新增 `SOP_CHANGELOG.md`（Debug + 补丁 SOP / 变更记录）

**Fix Summary**
- 屏蔽 `message.send` 被误判为 poll 的校验逻辑
- 强制重启 gateway 以加载补丁

**Verification**
- Feishu 附件发送恢复正常，`Poll fields require action "poll"` 不再出现
