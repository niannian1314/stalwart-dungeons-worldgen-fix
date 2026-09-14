# 坚固地牢结构生成修复 / Stalwart Dungeons Worldgen Fix

修复 [Stalwart Dungeons](https://www.curseforge.com/minecraft/mc-mods/stalwart-dungeons)（MCreator 制作的地牢模组）在 **Minecraft 1.20.1 – 1.21 / Forge 47+** 下的结构生成 bug。

> 本模组**不含任何 Java 代码**，是一个用 `lowcodefml` 加载的纯数据包 mod，不需要 @Mod 主类。

## 它修了什么

原 Stalwart Dungeons 把地牢生成触发器硬编码成：**每 1 个区块有 0.6% 概率，把触发器丢在 y = random(8, WORLD_SURFACE_WG) 的随机高度**。

下界的 `WORLD_SURFACE_WG` 高度图是基岩天花板（≈ y=127），于是触发器经常：

- 悬空在半空；
- 泡在岩浆湖里；
- 顶到基岩天花板；
- 到处生成漂浮的灵魂砖小平台。

## 修复原理

1. **删除原随机放置**：用 `forge:remove_features` 把 `stalwart_dungeons:awful_dungeons` / `keeping_castle` / `end_dungeon` 三个 placed_feature 从相关群系（下界三个群系 + 末地中部/高地 + the_void）里删掉。Forge 生物群系修改器按相位执行（ADD 先、REMOVE 后），所以无论数据包优先级如何，这一条一定在 Stalwart Dungeons 自己添加之后生效，**不需要改 mod 的 jar**。
2. **用原版 jigsaw 重新放置**：只把 `stalwart_dungeons:awful_dungeon_spawn` 触发器用 `minecraft:jigsaw` 结构重新注册，群系限制为下界 **灵魂沙峡谷 / 诡异森林**，高度 `y 38–50`，`spacing 32 / separation 10`（salt 12887431），比原来稀有得多。`keeping_castle` 与 `end_dungeon` 按原设计保持不自然生成。

**没有改动**：boss、实体、方块、物品、配方、战利品表、地牢 NBT。地牢本体仍可用 `/place template stalwart_dungeons:awful_dungeon` 手动放置，boss 祭坛逻辑（手持下界之星右键）不受影响。

## 依赖

- Minecraft `[1.20.1, 1.21)`
- Forge `[47,)`
- `stalwart_dungeons` `[1.2.8,)`（在其后加载）

## 目录结构

```
├── META-INF/
│   ├── MANIFEST.MF
│   └── mods.toml
├── pack.mcmeta
└── data/
    ├── stalwart_dungeons/worldgen/
    │   ├── structure/awful_dungeon.json          # jigsaw 结构定义，y38–50
    │   ├── structure_set/awful_dungeon.json     # random_spawn, spacing32/sep10
    │   └── template_pool/awful_dungeon_marker.json
    └── stalwart_dungeons_worldgen_fix/forge/biome_modifier/
        ├── remove_random_dungeon_placements_surface.json
        └── remove_random_dungeon_placements_underground.json
```

## 从源码重新打包成 jar

本仓库存放的是数据包源文件（用于审阅/版本管理）。要得到可直接放进 `mods/` 的 jar：把本目录所有文件按现有结构打成 zip，再改后缀为 `.jar` 即可（jar 本质就是 zip）。

PowerShell：
```powershell
Compress-Archive -Path META-INF,data,pack.mcmeta -DestinationPath stalwart_dungeons_worldgen_fix-1.0.0.jar
```

## License

MIT
