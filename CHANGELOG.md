# CHANGELOG

## v1.0.0 — 2026-10-26

首次建立四件套（README / AGENTS / DEVELOPMENT / CHANGELOG），沉淀经典款 G304 的驱动选型与配置经验。

**选型结论**

- 经典款 G304（AA 电池 / HERO 12K）走 **LGS 9.04.49 x64**，不装 G HUB
- OMM 定位为「板载读写小工具」，不作为 G HUB 的替代品（官方 FAQ 明确其依赖 G HUB）

**实测数据**

- LGS 最终版本号为 `9.04.49`，非普遍流传的 `9.02.65`；后者已从官方 CDN 下架（403）
- OMM 无版本号旧直链给的是 `1.1.6805.0`（2022 构建），当前版本走 `OnboardMemoryManager_<版本>.exe`
- LGS 开机自启注册在 `HKLM\...\Run` 的 `Launch LCore` + `LogiRegistryService` 服务，装完即生效

**两个硬边界**

- 「板载内存模式」与「自动游戏检测」是同一滑块的两档，互斥，不可兼得
- 板载模式存不下靠软件模拟的功能（如中键双击）；写完板载必须实测一遍

**下载校验基准**

- `LGS_9.04.49_x64_Logitech.exe`：125,787,696 字节 / Valid / Logitech Inc /
  `6DB9F6442D46FBDE0953F8BFBC36BD714E5E31A62F927563594CAFD60724D3E0`
- `OnboardMemoryManager_2.6.1749.exe`：13,064,344 字节 / Valid / Logitech Inc /
  `AEC76587F1D07C51667C140C730A38F82675FBE2D898E79413372146B9632358`
