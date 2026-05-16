# Paper Plugin Dev — Claude Code 技能

[English](README.md) | 中文版

一个全面的 [Claude Code](https://claude.ai/code) 技能，用于开发 PaperMC 我的世界服务器插件。覆盖从 Java 基础到 Paper 高级特性的全部内容。

## 技能覆盖范围

| 主题 | 说明 |
|-------|-------------|
| **项目搭建** | Gradle + Maven + Kotlin DSL、VSCode/IntelliJ、JDK 21+ 配置 |
| **插件架构** | Manager 模式、ItemsUtils 工厂、PluginInstance 单例 |
| **插件生命周期** | `onLoad()` → `onEnable()` → `onDisable()`、主类模式 |
| **plugin.yml / paper-plugin.yml** | 元数据、依赖关系、权限、加载顺序 |
| **事件系统** | 监听器、优先级、取消事件、自定义事件 |
| **命令系统** | Brigadier — BasicCommand 和命令树两种 API |
| **调度器** | 延迟任务、重复任务、异步任务、tick 计时 |
| **持久化数据容器 (PDC)** | 在玩家、实体、物品、区块上附加自定义数据 |
| **配置文件** | config.yml 读取/写入/重载、ConfigManager 模式 |
| **Adventure 文本** | Component、MiniMessage、ActionBar、Title |
| **常用 API** | 玩家、世界、物品、位置、合成配方 |
| **Data Components (1.20.5+)** | 现代物品数据系统 |
| **AsyncChatEvent** | 基于 Adventure 的现代聊天处理 |
| **完整模板** | 含事件、命令、PDC、配置的完整可用插件 |
| **调试技巧** | 堆栈阅读、本地测试服务端、远程调试、Paper 调试命令 |
| **README 与许可证** | 插件 README 模板、GPLv3/MIT 许可指导 |
| **DRY gradle.properties** | 统一版本/名称管理的单源模式 |
| **单元测试** | JUnit 5 + Mockito 测试 Paper 插件 |
| **发布 (CI/CD)** | GitHub Actions、通过 mc-publish 发布到 Modrinth/CurseForge |
| **高级主题** | Paperweight Userdev (NMS)、Registry API (1.21+)、数据库 (HikariCP) |
| **计分板与 BossBar** | 计分板侧边栏、名称下方、Tab 列表、BossBar API、队伍前缀/后缀 |
| **自定义 GUI 菜单** | 基础库存、点击处理、CustomInventoryHolder、GUI 框架 |
| **生态集成** | Vault 经济、PlaceholderAPI、BungeeCord/Velocity 插件消息 |
| **最佳实践** | 24 条清单，覆盖现代 Paper 插件开发全部方面 |

## 安装方式

### 方式 A：直接安装（推荐）

```bash
# 将技能文件复制到 Claude Code 的 skills 目录
cp SKILL.md ~/.claude/skills/paper-plugin-dev/SKILL.md
```

然后重启 Claude Code 或执行 `/reload-skills`。

### 方式 B：手动设置

1. 创建目录：`mkdir -p ~/.claude/skills/paper-plugin-dev/`
2. 将 `SKILL.md` 下载到该目录
3. 重启 Claude Code

## 使用方式

安装后，技能会在以下情况自动触发：

- 要求 Claude Code 创建 Paper/Bukkit/Spigot 插件时
- 编写导入了 `org.bukkit.*` 或 `io.papermc.paper.*` 的 Java 代码时
- 询问我的世界插件开发模式时
- 需要帮助处理事件、命令、PDC、Adventure 文本或调度器时
- 调试 Paper 插件错误时

也可以手动调用：

```
/paper-plugin-dev
```

或者直接说：

- "帮我创建一个给玩家发金币的 Paper 插件"
- "帮我调试这个 Bukkit 插件的 NullPointerException"
- "Paper 里面怎么用 Brigadier 命令？"
- "用 PDC 做一个击杀/死亡统计系统"

## 环境要求

- **Claude Code**（任意较新版本）
- 技能面向 **PaperMC**（1.21+），但大部分模式同样适用于 Spigot/Bukkit

## 为什么需要这个技能

我的世界插件开发已经有了很大的变化。现代 Paper 使用：

- **Adventure** 文本库（不再是传统的 ChatColor / § 颜色码）
- **Brigadier** 命令框架（不再是旧的 onCommand）
- **Persistent Data Container**（不再是直接操作 NBT 反射）
- **Data Components**（1.20.5+，替代物品上的 NBT 标签）
- **异步 API** 用于传送、区块加载等操作

这个技能编码了所有这些现代最佳实践，让 Claude Code 能够帮你从第一天就写出正确、地道的 Paper 插件——即使你刚接触 Java。

## 许可证

MIT — 自由使用、修改、分享。

## 参与贡献

发现 Bug 或想添加更多模式？欢迎提 PR！本技能维护在 [github.com/TiQIMENG/paper-plugin-dev](https://github.com/TiQIMENG/paper-plugin-dev)。
