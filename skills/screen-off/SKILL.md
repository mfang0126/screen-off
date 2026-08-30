---
name: screen-off
description: "Use when toggling Mac built-in display on/off."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [macos, display, screen, toggle, cli, hardware]
    related_skills: [computer-use]
---

# Screen Off — Mac 内建显示器开关

零依赖 Python CLI，通过 CoreGraphics/SkyLight 私有 API 开关 Mac 内建显示器。不需要 root、不需要辅助功能权限。

**来源**: [github.com/zy0816/ScreenOff](https://github.com/zy0816/ScreenOff)

## 何时使用

- 接了外接显示器，想关掉某个屏幕省电/减少干扰
- 调试显示器配置问题
- 多屏环境快速切换

## 何时不用

- 需要镜像/排列设置 — 这是系统设置的事

## CLI 用法

脚本位置：`~/.hermes/skills/devops/screen-off/scripts/screen-off.py`

```bash
# 列出所有显示器（带 ID、状态）
screen-off --status

# 按 ID 开/关（ID 稳定，不会因开关变化）
screen-off --off 3
screen-off --on 3

# 按序号开/关（# 前缀，对应 --status 的 #列）
screen-off --off #2
screen-off --on #1

# 按名字匹配
screen-off --off main          # 关主屏
screen-off --off builtin       # 关内建屏
screen-off --off "S2700"       # 模糊匹配

# 强制关（即使它是唯一亮着的屏）
screen-off --off 3 --force

# 重启后保留
screen-off --off 3 --permanent
```

**目标解析优先级**: 纯数字=按 ID（稳定），`#N`=按序号（会变），`main`/`builtin`=按角色，其他=模糊匹配名字

## 安全机制

1. **黑屏保护**：除非 `--force`，拒绝在没有其他点亮屏幕时关闭内建屏
2. **返回码不可信**：`CGSCompleteDisplayConfiguration` 经常返回 1001 但屏幕实际已生效，以轮询状态为准
3. **虚拟屏识别**：vendor=0x756e6b6e("unkn")、model=0x76697274("virt") 的不是真屏

## 依赖

- macOS（SkyLight/CoreGraphics 框架）
- Python 3.8+（系统自带即可）
- 无需 pip install

## Pitfall

- **禁用后消失**：外接屏被禁用后变成 `1x1 disabled`，仍然在列表里，可用 `--on <ID>` 开回
- **Display ID 不透明**：某些 Mac 上是 1、2、3，另一些是 69733248 这类大数
- **返回码不可信**：`CGSCompleteDisplayConfiguration` 经常返回 1001 但屏幕实际已生效，以轮询状态为准
- **虚拟屏**：所有物理屏消失时 macOS 会插入虚拟屏（vendor="unkn", model="virt"），脚本会显示但标记为 virtual
