# screen-off

macOS 显示器开关 CLI — 通过 ID/序号/名字控制任意显示器的开关状态。

零依赖，只需要系统 Python 3.8+。不需要 root，不需要辅助功能权限。

**基于 [zy0816/ScreenOff](https://github.com/zy0816/ScreenOff)** 的核心私有 API 调用，提取为独立 CLI 工具。

## Install

```bash
git clone https://github.com/mfang0126/screen-off.git
# 直接用，或做个 alias：
alias screen-off='python3 /path/to/screen-off/screen-off.py'
```

## Usage

```bash
# 列出所有显示器（含禁用的）
screen-off --status

# 按 ID 开/关（ID 稳定，不会因开关变化）
screen-off --off 3
screen-off --on 3

# 按序号（# 前缀，对应 --status 的 #列，会随开关变化）
screen-off --off #2

# 按名字匹配
screen-off --off main        # 关主屏
screen-off --off builtin     # 关内建屏
screen-off --off "S2700"     # 模糊匹配

# 强制关（即使它是唯一亮着的屏）
screen-off --off 3 --force

# 重启后保留
screen-off --off 3 --permanent
```

### 目标解析优先级

| 格式 | 匹配方式 | 稳定性 |
|------|----------|--------|
| `3` | 按 Display ID | ✅ 稳定 |
| `#2` | 按列表序号 | ⚠️ 会变 |
| `main` | 主显示器 | ✅ 稳定 |
| `builtin` | 内建显示器 | ✅ 稳定 |
| `S2700` | 模糊匹配名字 | ✅ 稳定 |

### 退出码

| 码 | 含义 |
|----|------|
| 0 | 完成或已在目标状态 |
| 1 | 找不到目标显示器 |
| 2 | 黑屏保护拒绝（加 `--force` 覆盖） |
| 3 | 配置已提交但未生效 |

## How it works

macOS 没有公开 API 控制单个显示器开关。本工具使用 4 个 CoreGraphics/SkyLight 私有函数：

| 函数 | 作用 |
|------|------|
| `CGSGetDisplayList` | 列出全部显示器槽位（含禁用的） |
| `CGSBeginDisplayConfiguration` | 开始配置事务 |
| `CGSConfigureDisplayEnabled` | 启用/禁用显示器（公开 API 无对应物） |
| `CGSCompleteDisplayConfiguration` | 提交配置（返回码不可信，以轮询状态为准） |

关键发现（来自原项目）：
- 内建屏被禁用后从公开 API 消失，只有 `CGSGetDisplayList` 保留槽位
- 所有物理屏消失时 macOS 会插入虚拟屏（vendor=`"unkn"`, model=`"virt"`）
- `CGSCompleteDisplayConfiguration` 经常返回 1001 但屏幕实际已生效

## Requirements

- macOS（SkyLight/CoreGraphics 框架）
- Python 3.8+（系统自带）
- 无需 pip install

## Credit

核心私有 API 调用来自 **[zy0816/ScreenOff](https://github.com/zy0816/ScreenOff)**（MIT License）。该项目还包含一个完整的菜单栏 App（Swift），有更完善的恢复链机制（三级恢复：enable → CGRestorePermanentDisplayConfiguration → sleepWake）。

本项目只提取了 CLI 部分，扩展了目标解析（按 ID/序号/名字），去掉了内建屏限制。

## License

MIT — see [LICENSE](LICENSE)
