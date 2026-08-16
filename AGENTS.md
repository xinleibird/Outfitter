# Outfitter — Agent Notes

Turtle WoW（基于 1.12 经典客户端）的装备管理插件。`## Interface: 11200`。
纯 Lua，无构建/测试/lint 流水线——改完肉眼检查即可。

## 目录结构

- `Outfitter.toc` — 插件清单。改 `## Interface:` / `## Version:` 在这里。加载顺序敏感：libs → strings → `BossTrash.lua` → `Outfitter.lua` → `Outfitter.xml`。
- `Outfitter.lua` — 主模块（~7.3k 行）。全局函数命名 `Outfitter_<模块>_<动作>`（如 `Outfitter_WearOutfit`），字符串/常量 `Outfitter_c<驼峰>`。
- `Outfitter.xml` — 帧/模板（`OutfitterFrame`、`OutfitterCurrentOutfit`、物品行、对话框）。
- `Bindings.xml` — `OUTFITTER_OUTFIT1..10` → `Outfitter_WearBoundOutfit(N)`。新增快捷键在这里加，不要写在 Lua 里。
- `OutfitterStrings.lua` — 简体中文字符串（唯一 locale 文件，enUS 不支持）。
- `BossTrash.lua` — 简体中文首领/小怪怪物名表。
- `Libs/AceLibrary`、`Libs/AceOO-2.0`、`Libs/AceEvent-2.0` — 内嵌 Ace2。**不要**再去拉新版本；**不要**加 `LibStub`（vanilla 1.12 没有）。
- `Textures/` — `.blp` 图片，由 XML 引用。

## 约定

- **只能用 vanilla 1.12 API**。`arg1`、`this`、`getglobal(...)` 是有意保留的，不要重构掉。
- **全局命名空间**仅暴露文档化的 `Outfitter_…` 与 `Outfitter_c…` 前缀（以及模块状态 `gOutfitter_<state>`）。仅 Lua 5.0 语法：不用 `goto`、`//`、位运算。
- **常量/字符串**写在 `OutfitterStrings.lua`。新增 `Outfitter_c…` 时直接加在这里；缺键会渲染成 `nil` 而不报错。
- **`BossTrash` 表项**直接加进 `BossTrash.lua`（中文怪物名表）。
- **版本号**：同时改 `Outfitter.toc` 的 `## Version:` 与 `OutfitterStrings.lua` 的 `Outfitter_cVersion`。
- **特殊套装**（Boss/Lvl63/Trash/Critter/BeastTrash/UndeadTrash/DemonTrash/Riding/Dining/BG…）统一在 `Outfitter_TargetChangedDelayedEvent`（`Outfitter.lua:1058`）里切换。新增"目标驱动"套装挂在这里。
- **登录期间的装备变更**必须走 `Outfitter_StartStartupSafeWindowGate`，**不要**在 `PLAYER_ENTERING_WORLD` 直接调 `EquipItem*`。
- **Riding/ArgentDawn 与 Smart outfits 一致**：只在首次初始化（`gOutfitter_Settings.Outfits` 为 nil）时扫描背包创建并自动填充装备；之后每次登录**不自动重建、不重扫**。装备变化靠手动右键"重建"（REBUILD，走 `Outfitter_GenerateSmartOutfit`）补。**不要**再加回"每次登录缺失即自动重建"的逻辑。
- **外观集成**（如 `Outfitter_pfUISkin`）必须用 `IsAddOnLoaded(...)` 守门，对应插件不存在时静默 no-op。

## Slash / 快捷键

- `/outfitter wear <outfit>` / `unwear <outfit>` / `toggle <outfit>` / `summary` — 见 `Outfitter_ExecuteCommand`（`Outfitter.lua:1240`）。
- `OUTFITTER_OUTFIT1..10` 快捷键按用户顺序映射前 10 个套装。

## 验证

没有自动化测试。手动验证流程：

1. 进 1.12（或 Turtle WoW）客户端，`/reload` 生效。
2. 手动触发相关事件：
   - `/outfitter toggle <outfit>` 走换装逻辑。
   - 选63+ 级怪 / 小动物 / 野兽 / 亡灵 / 恶魔 目标 → 触发 `Outfitter_TargetChangedDelayedEvent`。
   - 上马 / 进战场 / 坐下进食 → Riding/Dining/BG 套装。
   - 登出再登录 → 启动安全窗 gate。
3. `/console scriptErrors 1` 看 Lua 报错。
4. 切 zhCN 客户端重新加载，确认 UI 标签都正常解析（无 `nil`）。

## 不要碰

- `## Interface: 11200` — 改了就跑不动 vanilla 客户端。
- `Libs/*` — 内嵌版本，升级另开 PR。
- `Outfitter.lua:5367` 处的 `if false and IsAddOnLoaded("TankPoints")` — 故意死代码，留着。