# 🛡️ ProtocolSentinel

**OpenClaw v2026.3.13** 专用：**poll 注入 bug + 修复补丁** 全流程指南（手把手）。

> 目标：解决 `message.send` 被误判为 poll，导致 **附件发送失败** 的问题。

---

## ✅ 现象复现（必看）

在 Feishu 通道发送文件（`message.send` + `filePath`）时报错：

```
Poll fields require action "poll"; use action "poll" instead of "send".
```

即使参数里没传 poll，也会被拒绝。

---

## 🔍 根因分析

OpenClaw 的消息发送层在 `action === "send"` 时会检测 **poll 创建字段**（如 `pollQuestion` / `pollOption` / `pollDurationHours` 等）。

只要请求参数里出现这些字段（哪怕是空值），就会抛错并阻断发送。

而在 v2026.3.13 中，消息发送流程**会注入 poll 相关参数**，导致 `send` 被误判成 `poll`。

---

## 🧰 修复方案（补丁法，已验证）

**方案**：在 OpenClaw dist 中屏蔽 poll 校验。

### Step 1. 找到 OpenClaw 实际运行路径

```bash
which openclaw
readlink -f $(which openclaw)
```

典型路径：

```
/home/zfy/.npm-global/lib/node_modules/openclaw/openclaw.mjs
```

### Step 2. 定位包含校验逻辑的 dist 文件

```bash
grep -R "Poll fields require action" -n ~/.npm-global/lib/node_modules/openclaw/dist
```

你会看到类似这些文件：

- `reply-*.js`
- `model-selection-*.js`
- `auth-profiles-*.js`
- `discord-*.js`

### Step 3. 打补丁（禁用校验）

```bash
for f in ~/.npm-global/lib/node_modules/openclaw/dist/*.js; do
  if grep -q "Poll fields require action \"poll\"" "$f"; then
    sed -i "s/action === \"send\" && hasPollCreationParams(params)/action === \"send\" && hasPollCreationParams(params) && false/" "$f";
  fi
 done
```

### Step 4. 验证补丁是否生效

```bash
strings ~/.npm-global/lib/node_modules/openclaw/dist/reply-*.js | grep "Poll fields require action" | head
```

如果看到：

```
... hasPollCreationParams(params) && false ...
```

说明补丁已命中。

### Step 5. 强制重启网关（必须）

```bash
pkill -f openclaw-gateway
systemctl --user start openclaw-gateway
```

> 注意：普通 restart 可能被 SIGTERM 中断，建议用 **pkill + start**。

---

## ✅ 验证结果

- 发送附件成功（Feishu）
- `message.send` 不再误判 poll
- 文档/附件恢复可用

---

## 📷 配图建议（请自行选择）

建议插入 3~4 张图（可从网络获取）：

1. **报错截图**（`Poll fields require action "poll"`）
2. **grep 定位 dist 文件截图**
3. **sed 替换补丁截图**
4. **附件发送成功截图**

---

## 🧩 适用范围

- OpenClaw **v2026.3.13**（已验证）
- Feishu 通道附件发送场景

---

## ⚠️ 风险提示

该补丁属于 **本地修改 dist**（非官方 release），升级 OpenClaw 后可能被覆盖，需重新应用。

---

## ✅ 维护建议

- 记录补丁步骤
- 关注后续官方修复
- 升级后重新验证

---

如需更高级的“发送层参数过滤补丁”，可继续扩展。🫡
