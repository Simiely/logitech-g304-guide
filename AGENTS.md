# AGENTS.md · 项目规则

> 📌 **文档基线**：2026-10-26（commit `待回填`）完成四件套重写
> **更新文档后，请更新此行**（日期 + 新 commit hash），并在 CHANGELOG 追加版本

## 技术栈

- 目标设备：罗技 G304 LIGHTSPEED **经典款**（AA 电池 / HERO 12K / 99 g / 250 h 续航）
  - **不覆盖**新款 G304 X（内置 290 mAh 锂电 / HERO 25K / 57 g / 106 h）——两者驱动支持不同
- 驱动：**LGS 9.04.49 x64**（最终版；2022-05 构建，官方已停更）
- 系统：Windows 10 / 11 x64
- 本仓库是**纯文档**：无代码、无构建、无依赖、无 CI

## 关键坑

- **LGS 最终版是 9.04.49，不是 9.02.65**。9.02.65 已从 CDN 下架（403）；9.02.74 是 Win7/8 分支
- **OMM 的无版本号旧直链给的是 2022 年 `1.1.6805.0`**。新构建用 `OnboardMemoryManager_<版本>.exe` 命名
- **「板载内存模式」与「自动游戏检测」是 LGS 首页同一个滑块的两档，互斥**，只能二选一
- **软件模式能用的功能 ≠ 能写进板载**。例：中键「双击」写板载后失效（社区实测）
- **LGS 自启在 `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` 的 `Launch LCore`**，
  不在 HKCU Run、不在启动文件夹、也不在计划任务里

## 约定

- 一律简体中文；文件名、命令、路径用反引号
- 公开仓库隐私红线：不写本机绝对路径、账号名、凭据；安装目录统一写作 `%ProgramFiles%\Logitech Gaming Software\`
- 一坑一篇，格式与 knowledge-base 完全一致：`TL;DR / 问题 / 根因 / 解决 / 预防`
- 社区实测结论必须标注来源性质（社区实测 / 官方文档），不要混为一谈

## 常用命令

```powershell
# 核对 LGS 是否在跑（主进程是 LCore，不是 LGS）
Get-Process LCore

# 权威枚举全部开机自启项（含 Run 键 + 启动文件夹）
Get-CimInstance Win32_StartupCommand | Select-Object Name, Location, Command

# 下载包三项校验
Get-Item <file> | Select-Object Length
Get-AuthenticodeSignature <file> | Select-Object Status, SignerCertificate
Get-FileHash <file> -Algorithm SHA256
```

## 详细规则（按需 @引用）

暂无。AGENTS.md 未超过 150 词阈值，未触发 `rules/` 拆分。
