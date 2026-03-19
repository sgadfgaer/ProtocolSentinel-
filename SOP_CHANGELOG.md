# 🧰 Poll 注入失败：Debug + 补丁 SOP/变更记录

> 适用版本：OpenClaw v2026.3.13

---

## ✅ SOP（手把手 Debug 流程）

### Step 1. 复现问题
发送附件时报错：
```
Poll fields require action "poll"; use action "poll" instead of "send".
```

### Step 2. 定位校验逻辑
```bash
grep -R "Poll fields require action" -n ~/.npm-global/lib/node_modules/openclaw/dist
```

### Step 3. 打补丁（屏蔽 poll 校验）
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
预期输出含：
```
... hasPollCreationParams(params) && false ...
```

### Step 5. 强制重启
```bash
pkill -f openclaw-gateway
systemctl --user start openclaw-gateway
```

### Step 6. 验证发送
- 再次发送附件
- 预期：发送成功，不再报 poll

---

## 🧾 变更记录（Change Log）

### v1.0.0
- 新增 poll 注入失败的 debug + 补丁 SOP
- 标准化排查命令与验证路径

---

## ⚠️ 风险提示
- OpenClaw 升级后补丁会被覆盖，需要重打
- 推荐同步提交 issue 或等待官方修复
