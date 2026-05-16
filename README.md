# Paper Plugin Dev — Claude Code Skill

[中文版](README_CN.md) | English

A comprehensive [Claude Code](https://claude.ai/code) skill for developing Minecraft server plugins with the Paper API. Covers everything from Java basics to advanced Paper features.

## What This Skill Covers

| Topic | Description |
|-------|-------------|
| **Project Setup** | Gradle + Maven + Kotlin DSL, VSCode/IntelliJ, JDK 21+ configuration |
| **Plugin Architecture** | Manager pattern, ItemsUtils factory, PluginInstance singleton |
| **Plugin Lifecycle** | `onLoad()` → `onEnable()` → `onDisable()`, main class patterns |
| **plugin.yml / paper-plugin.yml** | Metadata, dependencies, permissions, load order |
| **Event System** | Listeners, priorities, cancellation, custom events |
| **Command System** | Brigadier — both `BasicCommand` and command tree APIs |
| **Scheduler** | Delayed tasks, repeating tasks, async tasks, tick timing |
| **Persistent Data Container (PDC)** | Custom data on players, entities, items, chunks |
| **Configuration** | config.yml read/write/reload, ConfigManager pattern |
| **Adventure Text** | Components, MiniMessage, ActionBar, Titles |
| **Common API** | Players, worlds, items, locations, recipes |
| **Data Components (1.20.5+)** | Modern item data system |
| **AsyncChatEvent** | Modern chat handling with Adventure |
| **Complete Templates** | Full working plugin with events, commands, PDC, and config |
| **Debugging** | Stacktrace reading, local test server, remote debugging, Paper debug commands |
| **README & Licensing** | Plugin README template, GPLv3/MIT guidance |
| **DRY gradle.properties** | Single-source version/name management |
| **Unit Testing** | JUnit 5 + Mockito for Paper plugin testing |
| **Publishing (CI/CD)** | GitHub Actions, Modrinth/CurseForge via mc-publish |
| **Advanced Topics** | Paperweight Userdev (NMS), Registry API (1.21+), Database (HikariCP) |
| **Scoreboard & BossBar** | Sidebar, below-name, tab list, BossBar API, team prefix/suffix |
| **Custom Inventory / GUI** | Basic inventories, click handlers, CustomInventoryHolder, GUI libraries |
| **Ecosystem Integrations** | Vault economy, PlaceholderAPI, BungeeCord/Velocity plugin messaging |
| **Best Practices** | 24-item checklist covering all aspects of modern Paper plugin development |

## Installation

### Option A: Direct Install (Recommended)

```bash
# Copy the skill file into your Claude Code skills directory
cp SKILL.md ~/.claude/skills/paper-plugin-dev/SKILL.md
```

Then restart Claude Code or run `/reload-skills`.

### Option B: Manual Setup

1. Create the directory: `mkdir -p ~/.claude/skills/paper-plugin-dev/`
2. Download `SKILL.md` into that directory
3. Restart Claude Code

## Usage

Once installed, the skill triggers automatically when you:

- Ask Claude Code to create a Paper/Bukkit/Spigot plugin
- Write Java code that imports `org.bukkit.*` or `io.papermc.paper.*`
- Ask about Minecraft plugin development patterns
- Need help with events, commands, PDC, Adventure text, or scheduling
- Debug Paper plugin errors

You can also invoke it explicitly with:

```
/paper-plugin-dev
```

Or just say things like:

- "Create a Paper plugin that gives players coins for breaking blocks"
- "Debug this NullPointerException in my Bukkit plugin"
- "How do I use Brigadier commands in Paper?"
- "Add a kill/death tracker using PDC"

## Requirements

- **Claude Code** (any recent version)
- The skill is for **PaperMC** (1.21+) but most patterns work with Spigot/Bukkit too

## Why This Skill Exists

Minecraft plugin development has evolved significantly. Modern Paper uses:

- **Adventure** text library (not legacy ChatColor / § codes)
- **Brigadier** command framework (not old onCommand)
- **Persistent Data Container** (not raw NBT reflection)
- **Data Components** (1.20.5+, replacing NBT tags on items)
- **Async APIs** for teleport, chunk loading, etc.

This skill encodes all of those modern best practices so Claude Code can help you write correct, idiomatic Paper plugins from day one — even if you're new to Java.

## License

MIT — use it, modify it, share it.

## Contributing

Found a bug or want to add more patterns? PRs welcome! This skill is maintained at [github.com/TiQIMENG/paper-plugin-dev](https://github.com/TiQIMENG/paper-plugin-dev).
