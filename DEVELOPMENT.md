# DEVELOPMENT.md · 架构说明与问题记录

## 项目概览

纯文档仓库。管理对象是「一只 G304 该用什么驱动、怎么配、哪些坑会反复踩」。

不涉及代码，不修改硬件固件，只固化软件层的选型结论与验证方法。

## 架构说明：三套软件的分层关系

罗技目前有三套能配置 G 系列鼠标的软件。它们不是新旧替代关系，而是**分层**的：

| 软件 | 定位 | 关键能力 | 关键限制 |
|---|---|---|---|
| **LGS** | 旧一代**完整驱动** | 板载模式开关、宏引擎、自动游戏检测、输入分析 | 只支持 2023 年前的老设备；已停更 |
| **G HUB** | 新一代**完整驱动** | 功能最全、社区配置、**唯一的固件更新途径** | 体积/常驻开销大；联网校验 |
| **OMM** | **板载内存读写小工具**（不是驱动） | 免安装、零常驻、跨机改板载配置、接收器配对 | 无宏、无 per-app、无固件更新 |

**依赖链**（重要）：

```
完整驱动（LGS 或 G HUB）
   └─ 负责：切模式、写板载槽位、宏、固件更新
        └─ 开启「板载内存模式」后
             └─ OMM 才认得到设备，可独立改写板载槽位
```

OMM 官方 FAQ 原文：*"The Onboard Memory Manager software **relies on G HUB for functionality**."*
所以 OMM **不是**「装它就能彻底摆脱 G HUB」的工具 —— 它省不掉你先装一套完整驱动这一步。

## 模式取舍：板载 vs 自动游戏检测

两者是 LGS 首页**同一个蓝色滑块**的左右两档，互斥：

| | 板载内存模式（拨左） | 自动游戏检测（拨右） |
|---|---|---|
| 设置存在哪里 | 鼠标内部 | 电脑上 |
| 每程序独立配置 | 不支持 | 支持 |
| 完整宏 | 不支持（受限） | 支持 |
| 后台常驻 | 不需要 | 必须 |
| 换电脑即插即用 | 支持 | 不支持（需装 LGS） |

补充事实：**G304 只有 1 个板载槽位**（官方规格：內建記憶體 1 個設定檔），所以板载模式下连「多套配置手动切」都做不到。唯一近似折中是存 5 档 DPI 用 DPI 键循环，但那只切灵敏度、切不了按键。

**选择判据**：只需回答一个问题 ——「会不会把这只鼠标带到别的机器上用？」

- 不会 → 拨自动游戏检测，白拿 per-app 与完整宏（常驻代价已在付）
- 会 → 留板载

---

## 关键问题与方案

### 问题 1：LGS 版本号被普遍写错（9.02.65 vs 9.04.49）

**TL;DR**：LGS 的最终版是 **9.04.49 x64**，9.02.65 是更早的旧构建且已从官方 CDN 下架。

- **问题**：中文下载站与大量教程都写「罗技 G304 驱动 v9.02.65」，并标注「2026-08 更新」；照此下载会拿到旧包，甚至拿到 403 的死链。
- **根因**：那些站点的「更新日期」是页面刷新时间，不是版本发布时间；LGS 自 2022 年后就没再发版。
- **解决**：直接用官方 CDN 的最终构建：
  `https://download01.logi.com/web/ftp/pub/techsupport/gaming/LGS_9.04.49_x64_Logitech.exe`
  实测 `HTTP 200`、`Content-Length: 125787696`、`FileVersion 9.04.49`。
  该版本更新日志含 *Updated drivers for Windows 10 and 11 code integrity (HVCI) compatibility* —— 这是老驱动能在 Win11 正常工作的关键。
- **预防**：判断罗技驱动的真实版本，看**文件属性里的 FileVersion**，不要看下载站的日期；`9.02.74` 是 Win7/8 分支，Win10/11 一律选 `9.04.49 x64`。

### 问题 2：下载到的官方安装包如何确认没被替换

**TL;DR**：官方 CDN 文件可用「文件大小 + 数字签名 + SHA256」三项交叉确认，缺一不可。

- **问题**：第三方站点广泛转存罗技安装包，且常夹带捆绑；同名文件难以肉眼分辨。
- **根因**：文件名不含校验信息，而 `Content-Length` 与签名的验证需要主动做。
- **解决**：先 `curl -sIL <url>` 拿 `Content-Length`，下载后比对本地 `Length`；再验签名与哈希：

```powershell
$f = '<下载的 exe>'
(Get-Item $f).Length
(Get-AuthenticodeSignature $f) | Select-Object Status, @{n='Signer';e={$_.SignerCertificate.Subject}}
(Get-FileHash $f -Algorithm SHA256).Hash
```

  LGS 9.04.49 x64 基准值：`125787696` 字节 / `Valid` / `CN=Logitech Inc` /
  `6DB9F6442D46FBDE0953F8BFBC36BD714E5E31A62F927563594CAFD60724D3E0`
- **预防**：三项里任何一项不符就直接删掉重下，不要「先装上试试」。

### 问题 3：OMM 直链下到的是 2022 年的旧构建

**TL;DR**：OMM 的**无版本号**旧直链长期挂着 `v1.1.6805.0`（2022 年）；当前版本走**带版本号**的文件名。

- **问题**：从 `.../Onboard Memory Manager.exe` 下载，签名有效、确实是罗技正品，但 `FileVersion` 只有 `1.1.6805.0`，而官方页面标注的是 `2.6.1749`（2026-02-02）。
- **根因**：罗技 CDN 上新构建改用「带版本号」命名，老的无版本号文件成了历史遗留，不会随版本更新。
- **解决**：用带版本号的直链
  `https://download01.logi.com/web/ftp/pub/techsupport/gaming/OnboardMemoryManager_2.6.1749.exe`
  实测 `Content-Length: 13064344`、`FileVersion 2.6.1749.0`。
- **预防**：凡是「官网直链」，一律用**文件属性版本号**与页面标注比对，不要只看域名和签名。签名只能证明「是罗技出的」，不能证明「是最新的」。

### 问题 4：OMM 做不到「不同程序切不同鼠标配置」

**TL;DR**：结构上不可能 —— OMM 不常驻、没有后台进程，没人监听前台窗口。

- **问题**：希望切到不同软件时自动换一套 DPI/按键，问 OMM 能否实现。
- **根因**：per-app 切换的前提是有常驻进程持续检测前台应用。OMM 的设计目标是「写完就关、零常驻、可离线」，与这个前提直接冲突。能做到的只有 LGS 的自动游戏检测和 G HUB 的 per-game profile。
- **解决**：需要 per-app 就拨到自动游戏检测并让 LGS 常驻；不需要则留在板载模式。二者互斥，不能兼得。
- **预防**：选软件前先问「要不要 per-app」。这一条会直接决定后面所有配置步骤，先定再动手，避免配完才发现方向错。

### 问题 5：板载模式存不下「靠软件模拟的功能」

**TL;DR**：软件模式里能用的功能，不等于能写进板载；写进去后失效是最常见的困惑。

- **问题**：在 G HUB/LGS 里把中键设为「双击」工作正常，切到板载模式并写入后，中键直接失效；LGS 的板载按键列表里也根本没有「双击」选项。
- **根因**：双击这类动作不是鼠标 MCU 原生指令，是驱动在主机侧模拟出来的。板载只有一颗 MCU 读 EEPROM 里的原生指令集，无法承载软件层逻辑。
- **解决**：需要这类功能就留在自动游戏检测模式（LGS 常驻）；坚持板载就把按键限定为原生动作（左/右/中键、侧键、DPI 切换、媒体键、单键）。
- **预防**：**写完板载必须实测一遍**（关掉驱动、按一遍所有按键）。宏同理：板载模式只支持 No Repeat + 文本 + 修饰键的极简宏，含 emoji、Repeat While Holding、Toggle、Sequence 的宏在板载模式下不工作。

### 问题 6：LGS 开机自启项「找不到」，其实是读取方式导致误判

**TL;DR**：LGS 自启挂在 `HKLM\...\Run` 的 `Launch LCore`，用表格方式渲染注册表多值项会被截断而漏看。

- **问题**：查 HKCU Run / 启动文件夹 / 计划任务都没有罗技项，一度判断「LGS 没注册开机自启」。
- **根因**：注册表 `Run` 键有多个值，用 `Select-Object * -ExcludeProperty PS*` + `Out-String` 渲染成固定宽度表格时会静默截断，只显示第一行。自启项实际存在于 `HKLM` 侧。
- **解决**：权威枚举用 `Get-CimInstance Win32_StartupCommand`，一次列全并带 `Location` / `Command`：

```powershell
Get-CimInstance Win32_StartupCommand | Select-Object Name, Location, Command | Format-List
```

  实测 LGS 占两处：
  1. `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` → `Launch LCore` = `%ProgramFiles%\Logitech Gaming Software\LCore.exe /minimized`
  2. 服务 `LogiRegistryService`（`StartMode=Auto`），位于 `%ProgramFiles%\Logitech Gaming Software\Drivers\APOService\`
- **预防**：**LGS 安装后默认就是开机自启，不需要额外设置**。想验证是否常驻，搜进程 `LCore`（不是 `LGS`）。另外任何「查注册表没查到」的结论，都改用 `Format-List` 或 CIM 重查一遍再说。
