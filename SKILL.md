---
name: paper-plugin-dev
description: This skill should be used when the user asks to create, develop, or debug PaperMC/Minecraft server plugins. Covers Java basics for beginners, Paper API, Bukkit API, event systems, command systems (Brigadier), scheduling, Persistent Data Containers, Adventure text, configuration, and plugin debugging. Triggers on: "Paper plugin", "Bukkit plugin", "Minecraft plugin", "papermc", "spigot plugin", "/paper", "/bukkit", or when writing Java code that imports org.bukkit or io.papermc.paper.
version: 3.0.0
---

# Paper Plugin Development Skill

Comprehensive guide for developing Minecraft server plugins with the Paper API. Covers everything from Java basics to advanced Paper features.

## When to Apply

- User asks to create a Paper/Bukkit/Spigot plugin
- User asks about Minecraft plugin development patterns
- User imports `org.bukkit.*` or `io.papermc.paper.*` in Java code
- User asks about events, commands, scheduling, PDC, or Adventure text in plugin context
- User needs help with plugin.yml, build.gradle.kts, or project setup
- User asks to debug a Paper plugin error

---

## 1. Environment & Project Setup

### Required Tools

| Tool | Purpose | Download |
|------|---------|----------|
| JDK 21+ | Paper requires Java 21 | https://adoptium.net |
| IntelliJ IDEA | Recommended IDE (Community free) | https://jetbrains.com/idea |
| VSCode (alternative) | Lighter-weight IDE option | https://code.visualstudio.com |
| Paper Server | Local test server | https://papermc.io/downloads |

### IntelliJ Plugin (strongly recommended)

In IntelliJ: **File → Settings → Plugins → Search "Minecraft Development" → Install → Restart IDE**

This plugin provides:
- New Project wizard with Paper platform pre-configured
- plugin.yml / paper-plugin.yml autocomplete and validation
- Gradle/Maven build template generation
- Inline API documentation

### Option A: IntelliJ Wizard (easiest for beginners)

1. File → New → Project → **Minecraft** (left sidebar)
2. Platform: **Paper**, fill in Group ID, Artifact ID, Plugin Name, Main Class
3. Build System: **Gradle** (recommended) or **Maven**
4. Click Create — everything is generated automatically

### Option B: Gradle (manual, recommended)

#### settings.gradle.kts
```kotlin
rootProject.name = "my-plugin"
```

#### build.gradle.kts
```kotlin
plugins {
    java
    id("xyz.jpenilla.run-paper") version "2.3.1"  // optional: auto-download test server
}

group = "com.example"
version = "1.0.0"

java {
    toolchain.languageVersion = JavaLanguageVersion.of(21)
}

repositories {
    mavenCentral()
    maven("https://repo.papermc.io/repository/maven-public/")
}

dependencies {
    // MUST use compileOnly — the server provides the API at runtime
    // NEVER use implementation — it will cause class conflicts!
    compileOnly("io.papermc.paper:paper-api:1.21.1-R0.1-SNAPSHOT")
}

tasks.withType<JavaCompile> {
    options.encoding = "UTF-8"
}

tasks {
    runServer {
        minecraftVersion("1.21.1")
    }
}
```

### Option C: Maven (pom.xml)

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>myplugin</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>21</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <repositories>
        <repository>
            <id>papermc</id>
            <url>https://repo.papermc.io/repository/maven-public/</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>io.papermc.paper</groupId>
            <artifactId>paper-api</artifactId>
            <version>1.21.1-R0.1-SNAPSHOT</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <finalName>${project.artifactId}-${project.version}</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.13.0</version>
                <configuration>
                    <source>21</source>
                    <target>21</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**Maven scope rule**: Use `<scope>provided</scope>` — same as Gradle's `compileOnly`. The server provides the API at runtime.

### VSCode Setup (alternative IDE)

If using VSCode instead of IntelliJ, install these extensions:
- **Extension Pack for Java** (vscjava.vscode-java-pack)
- **Gradle for Java** (vscjava.vscode-gradle) — if using Gradle
- **Maven for Java** (vscjava.vscode-maven) — if using Maven

Then create the project with `gradle init --type basic --dsl kotlin` (or manually).

### Standard Project Structure

```
my-plugin/
├── src/main/java/com/example/myplugin/
│   └── MyPlugin.java              # Main class
├── src/main/resources/
│   ├── plugin.yml                 # Plugin metadata (REQUIRED)
│   └── config.yml                 # Default config (optional)
├── build.gradle.kts               # or pom.xml for Maven
└── settings.gradle.kts
```

### Building

| Command | Build System |
|---------|-------------|
| `./gradlew build` | Gradle — output in `build/libs/` |
| `mvn package` | Maven — output in `target/` |
| `./gradlew runServer` | Gradle — also starts a test server |
| `./gradlew clean build` | Gradle — clean rebuild |

### Version Compatibility

| Minecraft | Java | Paper API |
|-----------|------|-----------|
| 1.21+ | JDK 21 | `1.21.x-R0.1-SNAPSHOT` |
| 1.20.5-1.20.6 | JDK 21 | `1.20.6-R0.1-SNAPSHOT` |
| 1.17-1.20.4 | JDK 17 | Match server version |
| 1.8.8-1.16 | JDK 8 | Use Spigot repo (Paper stopped hosting old versions) |

For 1.8.8-1.16, switch to the Spigot repository:
```
https://hub.spigotmc.org/nexus/content/repositories/snapshots/
```
And use artifact `org.spigotmc:spigot-api:1.8.8-R0.1-SNAPSHOT`.

### Kotlin Project Setup

If the user wants Kotlin instead of Java, use Gradle with Kotlin DSL:

```kotlin
plugins {
    kotlin("jvm") version "2.0.0"
    id("xyz.jpenilla.run-paper") version "2.3.1"
}

group = "com.example"
version = "1.0.0"

repositories {
    mavenCentral()
    maven("https://repo.papermc.io/repository/maven-public/")
}

dependencies {
    compileOnly("io.papermc.paper:paper-api:1.21.1-R0.1-SNAPSHOT")
}

java {
    toolchain.languageVersion = JavaLanguageVersion.of(21)
}

tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile> {
    kotlinOptions.jvmTarget = "21"
}
```

**Building Kotlin plugins**: `./gradlew build` produces two JARs in `build/libs/`:
- `plugin-1.0.0.jar` — plugin code only (use when other Kotlin plugins exist on the server)
- `plugin-1.0.0-all.jar` — includes Kotlin stdlib (use when this is the only Kotlin plugin; otherwise conflicts)

---

## 2. Plugin Architecture Patterns

### Manager Pattern (recommended for medium/large plugins)

Split features into Manager classes, each responsible for one area. A central `PluginMgr` coordinates them:

```java
// Base Manager class
public abstract class Manager {
    protected final JavaPlugin plugin;

    public Manager(JavaPlugin plugin) {
        this.plugin = plugin;
    }

    public abstract void register();  // Register listeners, commands, tasks
}

// Example feature manager
public class KillManager extends Manager {
    private final NamespacedKey killsKey;
    private final NamespacedKey deathsKey;

    public KillManager(JavaPlugin plugin) {
        super(plugin);
        this.killsKey = new NamespacedKey(plugin, "kills");
        this.deathsKey = new NamespacedKey(plugin, "deaths");
    }

    @Override
    public void register() {
        plugin.getServer().getPluginManager().registerEvents(new KillListener(this), plugin);
        // Register commands via LifecycleEvents...
    }

    public int getKills(Player player) {
        return player.getPersistentDataContainer()
            .getOrDefault(killsKey, PersistentDataType.INTEGER, 0);
    }
    // ...
}

// Central coordinator
public class PluginMgr extends Manager {
    private final KillManager killManager;
    private final HomeManager homeManager;

    public PluginMgr(JavaPlugin plugin) {
        super(plugin);
        this.killManager = new KillManager(plugin);
        this.homeManager = new HomeManager(plugin);
    }

    @Override
    public void register() {
        killManager.register();
        homeManager.register();
    }

    public KillManager getKillManager() { return killManager; }
    public HomeManager getHomeManager() { return homeManager; }
}

// In main class:
public class MyPlugin extends JavaPlugin {
    private PluginMgr pluginMgr;

    @Override
    public void onEnable() {
        pluginMgr = new PluginMgr(this);
        pluginMgr.register();
    }

    public PluginMgr getPluginMgr() { return pluginMgr; }
}
```

### ItemsUtils Pattern (centralize item creation)

```java
public final class ItemsUtils {

    private ItemsUtils() {} // Utility class

    public static ItemStack createItem(Material material, int amount, String name, List<String> lore) {
        ItemStack item = ItemStack.of(material, amount);
        item.editMeta(meta -> {
            meta.displayName(Component.text(name, NamedTextColor.WHITE));
            if (lore != null && !lore.isEmpty()) {
                meta.lore(lore.stream()
                    .map(l -> Component.text(l, NamedTextColor.GRAY))
                    .collect(Collectors.toList()));
            }
        });
        return item;
    }

    public static ItemStack createSkull(int amount, String name, List<String> lore, String owner) {
        ItemStack skull = ItemStack.of(Material.PLAYER_HEAD, amount);
        skull.editMeta(meta -> {
            meta.displayName(Component.text(name, NamedTextColor.WHITE));
            if (lore != null && !lore.isEmpty()) {
                meta.lore(lore.stream()
                    .map(l -> Component.text(l, NamedTextColor.GRAY))
                    .collect(Collectors.toList()));
            }
            if (meta instanceof SkullMeta skullMeta) {
                skullMeta.setOwningPlayer(Bukkit.getOfflinePlayer(owner));
            }
        });
        return skull;
    }
}
```

---

## 3. Plugin Main Class & Lifecycle

```java
package com.example.myplugin;

import io.papermc.paper.plugin.lifecycle.event.types.LifecycleEvents;
import org.bukkit.plugin.java.JavaPlugin;

public class MyPlugin extends JavaPlugin {

    private static MyPlugin instance;

    public static MyPlugin getInstance() { return instance; }

    // Called during server startup, before worlds load
    @Override
    public void onLoad() {
        instance = this;
    }

    // Called after worlds load — register events, commands, tasks here
    @Override
    public void onEnable() {
        saveDefaultConfig();  // copy config.yml from JAR if absent

        // Register event listeners
        getServer().getPluginManager().registerEvents(new PlayerListener(this), this);

        // Register commands (Brigadier)
        getLifecycleManager().registerEventHandler(LifecycleEvents.COMMANDS, event -> {
            var commands = event.registrar();
            commands.register("mycommand", "Description", new MyCommand());
        });

        getLogger().info("MyPlugin v" + getDescription().getVersion() + " enabled!");
    }

    // Called on server shutdown — save data, close connections
    @Override
    public void onDisable() {
        saveConfig();
        getLogger().info("MyPlugin disabled.");
    }
}
```

**Lifecycle order:** `onLoad()` → worlds load → `onEnable()` → server runs → `onDisable()`

| Operation | onLoad | onEnable | onDisable |
|-----------|--------|----------|-----------|
| Register events/commands | no | yes | no |
| Access players/worlds | no | yes | avoid |
| Save data | no | no | yes |
| Read config | no | yes | yes |

---

## 4. Plugin Metadata Files

### plugin.yml (standard, always works)

```yaml
name: MyPlugin
version: "1.0.0"
main: com.example.myplugin.MyPlugin
api-version: "1.21"
description: Plugin description
author: YourName
# authors: [Alice, Bob]
website: https://example.com

# Hard dependencies (must be present)
depend: [Vault, WorldEdit]
# Soft dependencies (optional)
softdepend: [LuckPerms]
# Load before these plugins
loadbefore: [OtherPlugin]
# Startup priority: STARTUP or POSTWORLD
load: POSTWORLD

permissions:
  myplugin.admin:
    description: Admin permission
    default: op       # true = everyone, op = ops only, false = no one
  myplugin.use:
    description: Basic usage permission
    default: true
```

### paper-plugin.yml (Paper-specific, richer dependency control)

```yaml
name: MyPlugin
version: "1.0.0"
main: com.example.myplugin.MyPlugin
api-version: "1.21"
dependencies:
  server:
    Vault:
      load: BEFORE    # BEFORE or AFTER
      required: true  # true = hard dep, false = soft dep
    LuckPerms:
      load: BEFORE
      required: false
```

If both files exist, Paper prefers `paper-plugin.yml`.

---

## 5. Event System

Events are the core mechanism. Paper broadcasts an event for everything that happens in the game; your plugin listens and reacts.

### Listener Class

```java
import org.bukkit.event.EventHandler;
import org.bukkit.event.EventPriority;
import org.bukkit.event.Listener;
import org.bukkit.event.player.PlayerJoinEvent;
import org.bukkit.event.player.PlayerQuitEvent;
import org.bukkit.event.block.BlockBreakEvent;
import net.kyori.adventure.text.Component;
import net.kyori.adventure.text.format.NamedTextColor;

public class PlayerListener implements Listener {

    private final MyPlugin plugin;

    public PlayerListener(MyPlugin plugin) {
        this.plugin = plugin;
    }

    @EventHandler
    public void onPlayerJoin(PlayerJoinEvent event) {
        event.joinMessage(
            Component.text("+ ", NamedTextColor.GREEN)
                .append(Component.text(event.getPlayer().getName(), NamedTextColor.YELLOW))
                .append(Component.text(" joined!", NamedTextColor.WHITE))
        );
        event.getPlayer().sendMessage(
            Component.text("Welcome back, " + event.getPlayer().getName() + "!", NamedTextColor.GOLD)
        );
    }

    @EventHandler
    public void onPlayerQuit(PlayerQuitEvent event) {
        event.quitMessage(
            Component.text("- " + event.getPlayer().getName() + " left.", NamedTextColor.GRAY)
        );
    }

    // HIGH priority, skip if already cancelled
    @EventHandler(priority = EventPriority.HIGH, ignoreCancelled = true)
    public void onBlockBreak(BlockBreakEvent event) {
        plugin.getLogger().info(event.getPlayer().getName() + " broke " + event.getBlock().getType());
    }

    // MONITOR priority: read-only, for logging only
    @EventHandler(priority = EventPriority.MONITOR, ignoreCancelled = true)
    public void onBlockBreakLog(BlockBreakEvent event) {
        // Log the final outcome after all plugins have processed this event
    }
}
```

### Event Priorities

```
LOWEST → LOW → NORMAL (default) → HIGH → HIGHEST → MONITOR
```

- **NORMAL**: Use for most cases
- **HIGHEST/LOWEST**: Use when you need to run before/after other plugins
- **MONITOR**: Read-only; never modify the event here

### Cancelling Events

```java
@EventHandler
public void onBlockBreak(BlockBreakEvent event) {
    if (event.getBlock().getType() == Material.DIAMOND_ORE && !event.getPlayer().isOp()) {
        event.setCancelled(true);
        event.getPlayer().sendMessage(Component.text("You cannot break diamond ore!", NamedTextColor.RED));
    }
}
```

### Common Events

```
PlayerJoinEvent            PlayerQuitEvent          PlayerDeathEvent
PlayerRespawnEvent         PlayerMoveEvent          PlayerInteractEvent
PlayerItemPickupEvent      PlayerDropItemEvent      AsyncChatEvent
PlayerCommandPreprocessEvent
BlockBreakEvent            BlockPlaceEvent          BlockRedstoneEvent
EntityDamageEvent          EntityDamageByEntityEvent  EntityDeathEvent
CreatureSpawnEvent         ServerLoadEvent
```

### Custom Events

```java
public class PlayerEarnCoinsEvent extends Event implements Cancellable {

    private static final HandlerList HANDLER_LIST = new HandlerList();

    public static HandlerList getHandlerList() { return HANDLER_LIST; }
    @Override public HandlerList getHandlers() { return HANDLER_LIST; }

    private final Player player;
    private int amount;
    private boolean cancelled = false;

    public PlayerEarnCoinsEvent(Player player, int amount) {
        this.player = player;
        this.amount = amount;
    }

    public Player getPlayer() { return player; }
    public int getAmount() { return amount; }
    public void setAmount(int amount) { this.amount = amount; }
    @Override public boolean isCancelled() { return cancelled; }
    @Override public void setCancelled(boolean cancel) { this.cancelled = cancel; }
}
```

**Fire the event:**
```java
PlayerEarnCoinsEvent event = new PlayerEarnCoinsEvent(player, coins);
player.getServer().getPluginManager().callEvent(event);
if (!event.isCancelled()) {
    // Use event.getAmount() — may have been modified by another plugin
    giveCoins(player, event.getAmount());
}
```

---

## 6. Command System (Brigadier)

### Option A: BasicCommand Interface (simpler)

```java
import io.papermc.paper.command.brigadier.BasicCommand;
import io.papermc.paper.command.brigadier.CommandSourceStack;
import net.kyori.adventure.text.Component;
import net.kyori.adventure.text.format.NamedTextColor;
import org.bukkit.entity.Player;
import org.jetbrains.annotations.NotNull;
import java.util.Collection;
import java.util.List;

public class HelloCommand implements BasicCommand {

    @Override
    public void execute(@NotNull CommandSourceStack source, @NotNull String[] args) {
        if (!(source.getSender() instanceof Player player)) {
            source.getSender().sendMessage(Component.text("Players only!", NamedTextColor.RED));
            return;
        }
        if (args.length == 0) {
            player.sendMessage(Component.text("Hello, " + player.getName() + "!", NamedTextColor.GREEN));
        } else {
            player.sendMessage(Component.text("Hello, " + args[0] + "!", NamedTextColor.AQUA));
        }
    }

    @Override
    public @NotNull Collection<String> suggest(@NotNull CommandSourceStack source, @NotNull String[] args) {
        if (args.length == 1) {
            return source.getSender().getServer().getOnlinePlayers()
                .stream().map(Player::getName).toList();
        }
        return List.of();
    }

    @Override
    public String permission() {
        return "myplugin.hello";
    }
}
```

**Register in onEnable():**
```java
getLifecycleManager().registerEventHandler(LifecycleEvents.COMMANDS, event -> {
    event.registrar().register("hello", "Greeting command", new HelloCommand());
});
```

### Option B: Brigadier Command Tree (for complex commands)

```java
import com.mojang.brigadier.arguments.IntegerArgumentType;
import com.mojang.brigadier.arguments.StringArgumentType;
import io.papermc.paper.command.brigadier.Commands;
import net.kyori.adventure.text.Component;
import net.kyori.adventure.text.format.NamedTextColor;

getLifecycleManager().registerEventHandler(LifecycleEvents.COMMANDS, event -> {
    Commands commands = event.registrar();

    commands.register(
        Commands.literal("myplugin")
            // /myplugin info
            .then(Commands.literal("info")
                .executes(ctx -> {
                    ctx.getSource().getSender().sendMessage(
                        Component.text("MyPlugin v1.0 - by Alice", NamedTextColor.AQUA));
                    return com.mojang.brigadier.Command.SINGLE_SUCCESS;
                })
            )
            // /myplugin give <amount>   (1-100)
            .then(Commands.literal("give")
                .requires(src -> src.getSender().hasPermission("myplugin.give"))
                .then(Commands.argument("amount", IntegerArgumentType.integer(1, 100))
                    .executes(ctx -> {
                        int amount = IntegerArgumentType.getInteger(ctx, "amount");
                        if (ctx.getSource().getSender() instanceof Player player) {
                            player.sendMessage(Component.text("Gave " + amount + " items!", NamedTextColor.GREEN));
                        }
                        return com.mojang.brigadier.Command.SINGLE_SUCCESS;
                    })
                )
            )
            // /myplugin say <message...>
            .then(Commands.literal("say")
                .then(Commands.argument("message", StringArgumentType.greedyString())
                    .executes(ctx -> {
                        String message = StringArgumentType.getString(ctx, "message");
                        ctx.getSource().getSender().getServer().broadcast(
                            Component.text("[Broadcast] " + message, NamedTextColor.YELLOW));
                        return com.mojang.brigadier.Command.SINGLE_SUCCESS;
                    })
                )
            )
            .build(),
        "MyPlugin main command"
    );
});
```

### Argument Types

| Type | Usage | Description |
|------|-------|-------------|
| `IntegerArgumentType.integer()` | `/cmd <num>` | Integer, can set range |
| `FloatArgumentType.floatArg()` | `/cmd <float>` | Floating point |
| `StringArgumentType.word()` | `/cmd <word>` | Single word (no spaces) |
| `StringArgumentType.string()` | `/cmd <"text">` | Quoted string |
| `StringArgumentType.greedyString()` | `/cmd <all text>` | Everything after the command |
| `BoolArgumentType.bool()` | `/cmd <true/false>` | Boolean |
| `EntityArgumentType.player()` | Player selector | Player name or selector |

---

## 7. Scheduler — Timed & Repeating Tasks

Paper uses **ticks**. 1 second = 20 ticks.

| Time | Ticks |
|------|-------|
| 1 second | 20 |
| 5 seconds | 100 |
| 30 seconds | 600 |
| 1 minute | 1,200 |
| 5 minutes | 6,000 |

### Delayed Task (run once)

```java
// 5 seconds later
getServer().getScheduler().runTaskLater(this, () -> {
    player.sendMessage(Component.text("5 seconds passed!", NamedTextColor.AQUA));
}, 100L);  // 5 * 20

// Using Tick utility (Paper-specific, more readable):
import io.papermc.paper.util.Tick;
import java.time.Duration;

long delay = Tick.tick().fromDuration(Duration.ofSeconds(5));
getServer().getScheduler().runTaskLater(this, task, delay);
```

### Repeating Task

```java
// Every 30 seconds
BukkitTask task = getServer().getScheduler().runTaskTimer(this, () -> {
    Bukkit.broadcast(Component.text("Server tip: follow the rules!", NamedTextColor.YELLOW));
}, 0L, 600L);  // start immediately, repeat every 30s

// Cancel
task.cancel();
// Or cancel all tasks of this plugin:
getServer().getScheduler().cancelTasks(this);
```

### Async Tasks (for database, HTTP, file I/O)

```java
getServer().getScheduler().runTaskAsynchronously(this, () -> {
    // Thread pool — do NOT call Bukkit API here!
    String data = readFromDatabase();

    // Switch back to main thread for Bukkit API:
    getServer().getScheduler().runTask(this, () -> {
        Bukkit.broadcast(Component.text("Data loaded: " + data));
    });
});
```

---

## 8. Persistent Data Container (PDC)

PDC attaches custom data to game objects (players, entities, items, chunks, worlds). It persists automatically with the object.

### Storing Data

```java
import org.bukkit.NamespacedKey;
import org.bukkit.persistence.PersistentDataType;

NamespacedKey key = new NamespacedKey(plugin, "kill_count");
player.getPersistentDataContainer().set(key, PersistentDataType.INTEGER, 0);
```

### Reading Data

```java
// With default value (preferred):
int kills = player.getPersistentDataContainer()
    .getOrDefault(killsKey, PersistentDataType.INTEGER, 0);

// Nullable get (returns null if absent):
Integer kills = player.getPersistentDataContainer().get(killsKey, PersistentDataType.INTEGER);

// Check existence:
boolean has = player.getPersistentDataContainer().has(killsKey, PersistentDataType.INTEGER);

// Remove:
player.getPersistentDataContainer().remove(killsKey);
```

### Supported Data Types

`BYTE`, `SHORT`, `INTEGER`, `LONG`, `FLOAT`, `DOUBLE`, `STRING`, `BYTE_ARRAY`, `INT_ARRAY`, `LONG_ARRAY`, `TAG_CONTAINER`

### PDC on Items

```java
ItemStack sword = ItemStack.of(Material.DIAMOND_SWORD);
sword.editMeta(meta -> {
    meta.getPersistentDataContainer().set(key, PersistentDataType.BOOLEAN, true);
});

// Reading from an item:
if (item.hasItemMeta()) {
    boolean isSpecial = item.getItemMeta().getPersistentDataContainer()
        .getOrDefault(key, PersistentDataType.BOOLEAN, false);
}
```

### Complete Example: Kill/Death Tracker

```java
public class KillTracker implements Listener {
    private final NamespacedKey killsKey;
    private final NamespacedKey deathsKey;

    public KillTracker(MyPlugin plugin) {
        this.killsKey = new NamespacedKey(plugin, "kills");
        this.deathsKey = new NamespacedKey(plugin, "deaths");
    }

    @EventHandler
    public void onDeath(PlayerDeathEvent event) {
        Player victim = event.getEntity();

        // Increment deaths
        int deaths = victim.getPersistentDataContainer()
            .getOrDefault(deathsKey, PersistentDataType.INTEGER, 0);
        victim.getPersistentDataContainer().set(deathsKey, PersistentDataType.INTEGER, deaths + 1);

        // Increment killer's kills
        if (victim.getKiller() instanceof Player killer) {
            int kills = killer.getPersistentDataContainer()
                .getOrDefault(killsKey, PersistentDataType.INTEGER, 0);
            killer.getPersistentDataContainer().set(killsKey, PersistentDataType.INTEGER, kills + 1);

            killer.sendActionBar(Component.text("Kill +1! Total: " + (kills + 1), NamedTextColor.GREEN));
        }
    }

    public int getKills(Player player) {
        return player.getPersistentDataContainer().getOrDefault(killsKey, PersistentDataType.INTEGER, 0);
    }

    public int getDeaths(Player player) {
        return player.getPersistentDataContainer().getOrDefault(deathsKey, PersistentDataType.INTEGER, 0);
    }
}
```

---

## 9. Configuration (config.yml)

### Default config.yml (in src/main/resources/)

```yaml
welcome-message: "&aWelcome, %player%!"
features:
  welcome-enabled: true
  kill-counter: true
settings:
  max-homes: 3
  coin-rate: 1.5
blocked-commands:
  - "/op"
  - "/stop"
```

### Reading Config

```java
@Override
public void onEnable() {
    saveDefaultConfig();  // Copy from JAR if not present

    String welcome = getConfig().getString("welcome-message", "&aWelcome!");
    boolean welcomeOn = getConfig().getBoolean("features.welcome-enabled", true);
    int maxHomes = getConfig().getInt("settings.max-homes", 3);
    double rate = getConfig().getDouble("settings.coin-rate", 1.0);
    List<String> blocked = getConfig().getStringList("blocked-commands");
}
```

### Writing & Reloading

```java
getConfig().set("settings.max-homes", 5);
saveConfig();   // Persist to disk

reloadConfig(); // Reload from disk, discarding in-memory changes
```

### ConfigManager Pattern (recommended for larger plugins)

```java
public class PluginConfig {
    private final MyPlugin plugin;
    private String welcomeMessage;
    private boolean welcomeEnabled;
    private int maxHomes;

    public PluginConfig(MyPlugin plugin) {
        this.plugin = plugin;
        reload();
    }

    public void reload() {
        plugin.reloadConfig();
        this.welcomeMessage = plugin.getConfig().getString("welcome-message", "&aWelcome!");
        this.welcomeEnabled = plugin.getConfig().getBoolean("features.welcome-enabled", true);
        this.maxHomes = plugin.getConfig().getInt("settings.max-homes", 3);
    }

    public String getWelcomeMessage() { return welcomeMessage; }
    public boolean isWelcomeEnabled() { return welcomeEnabled; }
    public int getMaxHomes() { return maxHomes; }
}
```

---

## 10. Adventure Text & UI

Paper uses the **Adventure** library for rich text. Never use legacy `§` color codes or `ChatColor`.

### Basic Components

```java
import net.kyori.adventure.text.Component;
import net.kyori.adventure.text.format.NamedTextColor;
import net.kyori.adventure.text.format.TextColor;
import net.kyori.adventure.text.format.TextDecoration;

// Colored text
player.sendMessage(Component.text("Red text", NamedTextColor.RED));
player.sendMessage(Component.text("Custom color", TextColor.fromHexString("#FF6B35")));

// Chaining
Component msg = Component.text()
    .content("Prefix")
    .color(NamedTextColor.GRAY)
    .append(Component.text("[Alert] ", NamedTextColor.RED).decorate(TextDecoration.BOLD))
    .append(Component.text("Something happened!", NamedTextColor.YELLOW))
    .build();
player.sendMessage(msg);
```

### MiniMessage (simpler tag-based syntax)

```java
import net.kyori.adventure.text.minimessage.MiniMessage;
import net.kyori.adventure.text.minimessage.tag.resolver.Placeholder;

MiniMessage mm = MiniMessage.miniMessage();

// Basic tags
player.sendMessage(mm.deserialize("<red>Red</red> <green>Green</green>"));
player.sendMessage(mm.deserialize("<gradient:red:blue>Gradient text</gradient>"));
player.sendMessage(mm.deserialize("<bold><italic>Bold italic</italic></bold>"));

// Clickable & hoverable
player.sendMessage(mm.deserialize(
    "<click:run_command:'/spawn'><hover:show_text:'Click to teleport'>[Spawn]</hover></click>"
));

// Placeholder substitution (safe against injection)
player.sendMessage(mm.deserialize(
    "Welcome, <player>! You have <coins> coins.",
    Placeholder.unparsed("player", player.getName()),
    Placeholder.unparsed("coins", String.valueOf(1000))
));
```

### Action Bar & Title

```java
import net.kyori.adventure.title.Title;
import java.time.Duration;

// ActionBar (small text above hotbar)
player.sendActionBar(Component.text("HP: " + player.getHealth(), NamedTextColor.RED));

// Title (large screen title)
player.showTitle(Title.title(
    Component.text("WELCOME!", NamedTextColor.GOLD),
    Component.text("To the server", NamedTextColor.YELLOW),
    Title.Times.times(
        Duration.ofMillis(500),   // fade in
        Duration.ofSeconds(3),    // stay
        Duration.ofMillis(500)    // fade out
    )
));
```

---

## 11. Common API Operations

### Players

```java
Player player = Bukkit.getPlayer("Steve");
Player player = Bukkit.getPlayer(uuid);
Collection<? extends Player> online = Bukkit.getOnlinePlayers();

// Messages
player.sendMessage(Component.text("Hello!"));
player.kick(Component.text("You were kicked!", NamedTextColor.RED));

// Teleport
player.teleport(location);
player.teleportAsync(location).thenAccept(success -> {
    if (success) getLogger().info("Teleported!");
});

// Health & food
player.setHealth(20.0);
player.setFoodLevel(20);

// Game mode
player.setGameMode(GameMode.CREATIVE);

// Permissions
boolean can = player.hasPermission("myplugin.admin");
boolean isOp = player.isOp();

// Inventory
player.getInventory().addItem(ItemStack.of(Material.DIAMOND, 64));

// Sound
player.playSound(player.getLocation(), Sound.ENTITY_PLAYER_LEVELUP, 1.0f, 1.0f);
```

### Worlds

```java
World world = Bukkit.getWorld("world");
World nether = Bukkit.getWorld("world_nether");

world.spawnEntity(location, EntityType.ZOMBIE);
world.createExplosion(location, 4.0f, false, false);
world.spawnParticle(Particle.HEART, location, 10);
world.setType(location, Material.DIAMOND_BLOCK);
Material type = world.getBlockAt(location).getType();
```

### Items

```java
// Create
ItemStack sword = ItemStack.of(Material.DIAMOND_SWORD);

// Edit meta
sword.editMeta(meta -> {
    meta.displayName(Component.text("Excalibur", NamedTextColor.AQUA));
    meta.lore(List.of(
        Component.text("Legendary weapon", NamedTextColor.GRAY),
        Component.text("Attack +100", NamedTextColor.GREEN)
    ));
});

// Enchants
sword.editMeta(meta -> meta.addEnchant(Enchantment.SHARPNESS, 5, true));

// Give to player
player.getInventory().addItem(sword);

// Check type
if (item.getType() == Material.DIAMOND_SWORD) { /* ... */ }
```

### Locations

```java
Location loc = new Location(world, x, y, z);
Location loc = new Location(world, x, y, z, yaw, pitch);

Location playerLoc = player.getLocation();
double x = playerLoc.getX();
World w = playerLoc.getWorld();

double dist = loc1.distance(loc2);
Location above = playerLoc.clone().add(0, 2, 0);
```

### Recipes

```java
NamespacedKey key = new NamespacedKey(this, "custom_sword");
ShapedRecipe recipe = new ShapedRecipe(key, ItemStack.of(Material.DIAMOND_SWORD));
recipe.shape(" D ", " D ", " S ");
recipe.setIngredient('D', Material.DIAMOND);
recipe.setIngredient('S', Material.STICK);
getServer().addRecipe(recipe);
```

---

## 12. Data Component API (1.20.5+)

Modern item data system replacing NBT tags:

```java
// Check if item has custom name
boolean hasName = itemStack.hasData(DataComponentTypes.CUSTOM_NAME);

// Set enchantments
itemStack.setData(DataComponentTypes.ENCHANTMENTS,
    ItemEnchantments.itemEnchantments().add(Enchantment.SHARPNESS, 5).build());
```

---

## 13. AsyncChatEvent (Modern Chat Handling)

```java
import io.papermc.paper.event.player.AsyncChatEvent;
import net.kyori.adventure.text.serializer.plain.PlainTextComponentSerializer;

@EventHandler
public void onChat(AsyncChatEvent event) {
    String plain = PlainTextComponentSerializer.plainText().serialize(event.message());

    if (plain.contains("badword")) {
        event.setCancelled(true);
        event.getPlayer().sendMessage(Component.text("Please be respectful!", NamedTextColor.RED));
        return;
    }

    // Custom chat formatting
    event.renderer((source, sourceDisplayName, message, viewer) ->
        Component.text("[Player] ", NamedTextColor.GRAY)
            .append(sourceDisplayName.color(NamedTextColor.YELLOW))
            .append(Component.text(": ", NamedTextColor.WHITE))
            .append(message)
    );
}
```

AsyncChatEvent is async — use `getScheduler().runTask()` to access Bukkit API.

---

## 14. Complete Plugin Template

Here is a complete, working plugin demonstrating all core concepts:

```java
// === StatsPlugin.java (Main Class) ===
package com.example.statsplugin;

import io.papermc.paper.command.brigadier.Commands;
import io.papermc.paper.plugin.lifecycle.event.types.LifecycleEvents;
import net.kyori.adventure.text.Component;
import net.kyori.adventure.text.format.NamedTextColor;
import org.bukkit.Bukkit;
import org.bukkit.plugin.java.JavaPlugin;

public class StatsPlugin extends JavaPlugin {

    private static StatsPlugin instance;
    private StatsListener statsListener;

    public static StatsPlugin getInstance() { return instance; }

    @Override
    public void onEnable() {
        instance = this;
        saveDefaultConfig();

        statsListener = new StatsListener(this);
        getServer().getPluginManager().registerEvents(statsListener, this);

        getLifecycleManager().registerEventHandler(LifecycleEvents.COMMANDS, event -> {
            event.registrar().register("stats", "View your stats", new StatsCommand(this));
        });

        long interval = 20L * 60 * 5; // every 5 minutes
        getServer().getScheduler().runTaskTimer(this, () -> {
            String tip = getConfig().getString("broadcast-tip", "Use /stats!");
            Bukkit.broadcast(Component.text(tip, NamedTextColor.AQUA));
        }, interval, interval);

        getLogger().info("StatsPlugin enabled!");
    }

    @Override
    public void onDisable() {
        getLogger().info("StatsPlugin disabled.");
    }

    public StatsListener getStatsListener() { return statsListener; }
}
```

```java
// === StatsListener.java ===
package com.example.statsplugin;

import net.kyori.adventure.text.Component;
import net.kyori.adventure.text.format.NamedTextColor;
import org.bukkit.NamespacedKey;
import org.bukkit.entity.Player;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.entity.PlayerDeathEvent;
import org.bukkit.event.player.PlayerJoinEvent;
import org.bukkit.persistence.PersistentDataType;
import net.kyori.adventure.title.Title;
import java.time.Duration;

public class StatsListener implements Listener {

    private final StatsPlugin plugin;
    private final NamespacedKey killsKey;
    private final NamespacedKey deathsKey;

    public StatsListener(StatsPlugin plugin) {
        this.plugin = plugin;
        this.killsKey = new NamespacedKey(plugin, "kills");
        this.deathsKey = new NamespacedKey(plugin, "deaths");
    }

    @EventHandler
    public void onJoin(PlayerJoinEvent event) {
        Player player = event.getPlayer();
        String welcome = plugin.getConfig().getString("welcome-message", "Welcome, %player%!")
            .replace("%player%", player.getName());

        player.showTitle(Title.title(
            Component.text("Welcome!", NamedTextColor.GOLD),
            Component.text(welcome, NamedTextColor.YELLOW),
            Title.Times.times(Duration.ofMillis(300), Duration.ofSeconds(2), Duration.ofMillis(500))
        ));

        int kills = getKills(player);
        int deaths = getDeaths(player);
        if (kills > 0 || deaths > 0) {
            player.sendMessage(Component.text(
                "Your record: " + kills + " kills / " + deaths + " deaths", NamedTextColor.GRAY));
        }
    }

    @EventHandler
    public void onDeath(PlayerDeathEvent event) {
        Player victim = event.getEntity();

        int deaths = getDeaths(victim);
        victim.getPersistentDataContainer().set(deathsKey, PersistentDataType.INTEGER, deaths + 1);

        if (victim.getKiller() instanceof Player killer) {
            int kills = getKills(killer);
            killer.getPersistentDataContainer().set(killsKey, PersistentDataType.INTEGER, kills + 1);
            killer.sendActionBar(Component.text("Kill +1! Total: " + (kills + 1), NamedTextColor.GREEN));
        }
    }

    public int getKills(Player player) {
        return player.getPersistentDataContainer().getOrDefault(killsKey, PersistentDataType.INTEGER, 0);
    }

    public int getDeaths(Player player) {
        return player.getPersistentDataContainer().getOrDefault(deathsKey, PersistentDataType.INTEGER, 0);
    }
}
```

```java
// === StatsCommand.java ===
package com.example.statsplugin;

import io.papermc.paper.command.brigadier.BasicCommand;
import io.papermc.paper.command.brigadier.CommandSourceStack;
import net.kyori.adventure.text.Component;
import net.kyori.adventure.text.format.NamedTextColor;
import org.bukkit.Bukkit;
import org.bukkit.entity.Player;
import org.jetbrains.annotations.NotNull;
import java.util.Collection;
import java.util.List;

public class StatsCommand implements BasicCommand {

    private final StatsPlugin plugin;

    public StatsCommand(StatsPlugin plugin) { this.plugin = plugin; }

    @Override
    public void execute(@NotNull CommandSourceStack source, @NotNull String[] args) {
        Player target;
        if (args.length >= 1) {
            target = Bukkit.getPlayerExact(args[0]);
            if (target == null) {
                source.getSender().sendMessage(Component.text("Player not found: " + args[0], NamedTextColor.RED));
                return;
            }
        } else if (source.getSender() instanceof Player player) {
            target = player;
        } else {
            source.getSender().sendMessage(Component.text("Console: /stats <player>", NamedTextColor.RED));
            return;
        }

        StatsListener stats = plugin.getStatsListener();
        int kills = stats.getKills(target);
        int deaths = stats.getDeaths(target);
        double kd = deaths == 0 ? kills : (double) kills / deaths;

        source.getSender().sendMessage(Component.empty());
        source.getSender().sendMessage(
            Component.text("=== ", NamedTextColor.DARK_GRAY)
                .append(Component.text(target.getName() + "'s Stats", NamedTextColor.GOLD))
                .append(Component.text(" ===", NamedTextColor.DARK_GRAY))
        );
        source.getSender().sendMessage(
            Component.text("  Kills: ", NamedTextColor.GRAY)
                .append(Component.text(kills, NamedTextColor.GREEN))
        );
        source.getSender().sendMessage(
            Component.text("  Deaths: ", NamedTextColor.GRAY)
                .append(Component.text(deaths, NamedTextColor.RED))
        );
        source.getSender().sendMessage(
            Component.text("  K/D: ", NamedTextColor.GRAY)
                .append(Component.text(String.format("%.2f", kd), NamedTextColor.YELLOW))
        );
        source.getSender().sendMessage(Component.empty());
    }

    @Override
    public @NotNull Collection<String> suggest(@NotNull CommandSourceStack source, @NotNull String[] args) {
        if (args.length == 1) {
            return Bukkit.getOnlinePlayers().stream()
                .map(Player::getName)
                .filter(n -> n.toLowerCase().startsWith(args[0].toLowerCase()))
                .toList();
        }
        return List.of();
    }
}
```

```yaml
# === plugin.yml ===
name: StatsPlugin
version: "1.0.0"
main: com.example.statsplugin.StatsPlugin
api-version: "1.21"
```

```yaml
# === config.yml ===
welcome-message: "Welcome, %player%! Have fun!"
broadcast-tip: "Use /stats to check your kill/death record!"
```

---

## 15. Debugging

### Stacktrace Reading

```
[ERROR] Error occurred while enabling MyPlugin v1.0 (Is it up to date?)
java.lang.NullPointerException: Cannot invoke method sendMessage()
        at com.example.myplugin.PlayerListener.onJoin(PlayerListener.java:24)
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        Look for YOUR package name → it tells you the exact file and line
```

### Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `NullPointerException` | Calling method on null object | Add null check |
| `ClassCastException` | Wrong type cast | Use `instanceof` first |
| Plugin gray in /plugins | plugin.yml malformed | Check indentation, colons |
| Events don't fire | Forgot `registerEvents()` | Register in onEnable() |
| `NoSuchMethodError` | API version mismatch | Check api-version matches server |

### Useful Server Commands

```
/plugins              → list plugins (green=ok, red=error)
/version MyPlugin     → plugin version info
/paper dumplisteners PlayerJoinEvent → debug event conflicts
/paper dumpplugins    → plugin load order & dependencies
```

### Remote Debugging

Add JVM args to server start script:
```
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005
```
Then connect via IntelliJ: Run → Attach to Process (Remote JVM Debug on port 5005).

### Local Test Server Setup

**Option A — run-paper Gradle plugin (automated):**
```bash
./gradlew runServer      # Auto-downloads Paper, starts server with your plugin
```

**Option B — Manual server setup:**
1. Download Paper JAR from https://papermc.io/downloads
2. Place it in a `server/` folder, create a startup script:

   **Windows (run.bat):**
   ```bat
   java -Xms2G -Xmx2G -jar paper-1.21.1-xxx.jar nogui
   pause
   ```

   **macOS/Linux (run.sh):**
   ```bash
   #!/bin/bash
   cd "$(dirname "$0")"
   exec java -Xms2G -Xmx2G -jar paper-1.21.1-xxx.jar nogui
   ```

3. Run the server once — it will stop and create `eula.txt`
4. Edit `eula.txt`: set `eula=true`
5. Put your plugin JAR into `plugins/` folder
6. Start the server again
7. Join with Minecraft client → Multiplayer → Direct Connection → `localhost`

**Verification:**
- Console should show: `[MyPlugin] MyPlugin v1.0.0 enabled!`
- In-game: run `/plugins` — your plugin should appear in **green**
- Red name = plugin failed to load; check console for errors

### Logger

```java
getLogger().info("Info message");
getLogger().warning("Warning message");
getLogger().severe("Severe error!");

// Debug guard
if (getConfig().getBoolean("debug", false)) {
    getLogger().info("[DEBUG] player " + player.getName() + " kills: " + kills);
}
```

---

## 16. Plugin README & Licensing

### README Template

Every plugin repo should include a clear README:

```markdown
# PluginName
Brief description of what your plugin does.

## Features
- Feature 1: Description
- Feature 2: Description

## Commands
| Command | Description | Permission |
|---------|-------------|------------|
| `/cmd1` | What it does | `plugin.cmd1` |
| `/cmd2 <arg>` | What it does | `plugin.cmd2` |

## Permissions
| Permission | Description | Default |
|-----------|-------------|---------|
| `plugin.use` | Basic usage | true |
| `plugin.admin` | Admin commands | op |

## Configuration
Explain config.yml structure and key settings.

## Installation
1. Download the JAR from Releases
2. Place in `plugins/` folder
3. Restart server
4. Configure `plugins/PluginName/config.yml`

## Building from Source
```bash
./gradlew build    # or: mvn package
```
Output in `build/libs/` (or `target/` for Maven).

## Dependencies
- Paper 1.21+ (or Spigot 1.20+)
- Java 21+
- (List any plugin dependencies: Vault, LuckPerms, etc.)

## License
MIT (or GPLv3 — note: Bukkit/Paper API are GPLv3, your plugin may need to match)
```

### Licensing Notes

- **Paper API** and **Spigot API** are licensed under **GPL v3**
- If your plugin links against them, it may need to be GPLv3 too
- MIT is fine for the *code you write*, but be aware of the GPL implications
- Template repos and skill content can use MIT

---

## 17. Java Basics for Beginners

When the user is new to Java, reinforce these fundamentals:

### Variables
```java
int age = 25;
double score = 98.5;
boolean isActive = true;
String name = "Steve";
```

### Conditionals
```java
if (player.getHealth() > 10) {
    player.sendMessage(Component.text("You're healthy!"));
} else if (player.getHealth() > 5) {
    player.sendMessage(Component.text("Careful!"));
} else {
    player.sendMessage(Component.text("Danger!"));
}
```

### Loops
```java
for (int i = 0; i < 10; i++) {
    Bukkit.broadcast(Component.text("Tick: " + i));
}

for (Player p : Bukkit.getOnlinePlayers()) {
    p.sendMessage(Component.text("Announcement!"));
}
```

### OOP: Classes, Inheritance, Interfaces
```java
// Class
public class PlayerData {
    private String name;
    private int level;

    public PlayerData(String name) {
        this.name = name;
        this.level = 1;
    }

    public String getName() { return name; }
    public int getLevel() { return level; }
    public void setLevel(int level) { this.level = level; }
}

// Interface
public interface Warpable {
    void warpToSpawn();
}

// Abstract class
public abstract class GameMode {
    public abstract void onStart();
    public abstract void onEnd();
}

// Listener implements an interface
public class MyListener implements Listener {
    // ...
}
```

### Collections
```java
// List: ordered, allows duplicates
List<String> names = new ArrayList<>();
names.add("Steve");
String first = names.get(0);

// Set: unordered, no duplicates
Set<UUID> banned = new HashSet<>();
banned.add(player.getUniqueId());

// Map: key-value pairs
Map<UUID, Integer> levels = new HashMap<>();
levels.put(player.getUniqueId(), 10);
int level = levels.getOrDefault(player.getUniqueId(), 0);
```

### Lambdas
```java
// Old way
Runnable r = new Runnable() {
    @Override
    public void run() {
        Bukkit.broadcast(Component.text("Timer!"));
    }
};

// Lambda
Runnable r = () -> Bukkit.broadcast(Component.text("Timer!"));

// Common in Paper APIs:
sword.editMeta(meta -> meta.displayName(Component.text("Sword")));
player.teleportAsync(loc).thenAccept(success -> {
    if (success) getLogger().info("Done!");
});
```

---

## 18. DRY gradle.properties Pattern

Instead of hardcoding plugin info in multiple places, define everything once in `gradle.properties`:

```properties
# gradle.properties
group=com.example
pluginName=MyPlugin
version=1.0.0
description=An awesome Paper plugin
author=YourName
apiVersion=1.21
paperVersion=1.21.1-R0.1-SNAPSHOT
```

Then reference in `build.gradle.kts`:
```kotlin
group = project.properties["group"] as String
version = project.properties["version"] as String

dependencies {
    compileOnly("io.papermc.paper:paper-api:${project.properties["paperVersion"]}")
}

tasks.processResources {
    filesMatching("plugin.yml") {
        expand(project.properties)
    }
}
```

And use placeholders in `plugin.yml`:
```yaml
name: ${pluginName}
version: ${version}
main: ${group}.${pluginName}.${pluginName}
api-version: ${apiVersion}
description: ${description}
author: ${author}
```

This follows the **DRY principle** — change a value once in gradle.properties and it updates everywhere.

---

## 19. Unit Testing Paper Plugins

Test Paper plugins with JUnit 5 + Mockito:

```kotlin
// build.gradle.kts additions
dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter:5.11.0")
    testImplementation("org.mockito:mockito-core:5.12.0")
    testImplementation("io.papermc.paper:paper-api:1.21.1-R0.1-SNAPSHOT")
}

tasks.test {
    useJUnitPlatform()
}
```

```java
// src/test/java/com/example/myplugin/MyPluginTest.java
import org.bukkit.Server;
import org.bukkit.plugin.PluginManager;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import static org.mockito.Mockito.*;

class MyPluginTest {
    private MyPlugin plugin;
    private Server mockServer;
    private PluginManager mockPluginManager;

    @BeforeEach
    void setUp() {
        mockServer = mock(Server.class);
        mockPluginManager = mock(PluginManager.class);
        when(mockServer.getPluginManager()).thenReturn(mockPluginManager);

        plugin = mock(MyPlugin.class);
        when(plugin.getServer()).thenReturn(mockServer);
        // Use doCallRealMethod() for methods you want to test directly
    }

    @Test
    void testOnEnable_registersListeners() {
        doCallRealMethod().when(plugin).onEnable();
        plugin.onEnable();
        verify(mockPluginManager, atLeastOnce()).registerEvents(any(), eq(plugin));
    }
}
```

**Testing patterns:**
- **Unit test** listener/command logic by injecting mocked dependencies
- **Integration test** against a real Paper server via `runServer` Gradle task
- **Mock Bukkit/Paper API objects** using `mock()` — Server, Player, World, etc.
- Test event handlers by creating mock events and calling the handler directly

---

## 20. Publishing & Distribution

### GitHub Actions CI/CD

Basic build workflow (`.github/workflows/build.yml`):
```yaml
name: Build
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: 21
          distribution: temurin
      - run: ./gradlew build
      - uses: actions/upload-artifact@v4
        with:
          name: plugin
          path: build/libs/*.jar
```

### Publishing to Modrinth & CurseForge

Use the [mc-publish](https://github.com/marketplace/actions/mc-publish) GitHub Action:

```yaml
- uses: Kir-Antipov/mc-publish@v4
  with:
    modrinth-id: ${{ vars.MODRINTH_PROJECT_ID }}
    modrinth-token: ${{ secrets.MODRINTH_TOKEN }}
    curseforge-id: ${{ vars.CURSEFORGE_PROJECT_ID }}
    curseforge-token: ${{ secrets.CURSEFORGE_TOKEN }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

Set repository secrets in Settings → Secrets and Variables → Actions:
- `MODRINTH_TOKEN`: Modrinth PAT with "Write projects" + "Create versions" scopes
- `CURSEFORGE_TOKEN`: CurseForge API token
- Variables for project IDs: `MODRINTH_PROJECT_ID`, `CURSEFORGE_PROJECT_ID`

---

## 21. Advanced Topics

### Paperweight Userdev (NMS Access)

For plugins that need access to Minecraft's internal (NMS) code:

```kotlin
plugins {
    id("io.papermc.paperweight.userdev") version "2.0.0-beta.10"
}

dependencies {
    paperweight.paperDevBundle("1.21.1-R0.1-SNAPSHOT")
}
```

This provides Mojang-mapped NMS classes while keeping your plugin compatible.

### Custom Registry API (1.21+)

Paper 1.21+ supports custom enchantments via the Registry API:

```java
// Requires paper-plugin.yml with bootstrapper
// Custom enchantment registration:
import io.papermc.paper.registry.RegistryKey;
import io.papermc.paper.registry.TypedKey;

// Define custom enchantment key
TypedKey<Enchantment> CUSTOM_ENCHANT = TypedKey.create(
    RegistryKey.ENCHANTMENT,
    new NamespacedKey(plugin, "custom_enchant")
);

// Register via RegistryAccess
RegistryAccess registryAccess = RegistryAccess.registryAccess();
// ... full registration code in bootstrapper
```

See [SaberSupe/CustomEnchantExample](https://github.com/SaberSupe/CustomEnchantExample) for a complete example.

### Database Integration

For large data persistence, use HikariCP connection pool:

```kotlin
dependencies {
    implementation("com.zaxxer:HikariCP:5.1.0")
    // Choose one:
    implementation("org.xerial:sqlite-jdbc:3.46.0")     // SQLite (file-based)
    implementation("com.mysql:mysql-connector-j:9.0.0")   // MySQL
}
```

```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:sqlite:plugins/MyPlugin/data.db");
// or: config.setJdbcUrl("jdbc:mysql://localhost:3306/db");
// config.setUsername("user");
// config.setPassword("pass");
HikariDataSource dataSource = new HikariDataSource(config);

// Always use in async tasks:
getServer().getScheduler().runTaskAsynchronously(this, () -> {
    try (Connection conn = dataSource.getConnection();
         PreparedStatement ps = conn.prepareStatement("SELECT * FROM players WHERE uuid = ?")) {
        ps.setString(1, uuid.toString());
        ResultSet rs = ps.executeQuery();
        // ... process results
    } catch (SQLException e) {
        getLogger().severe("Database error: " + e.getMessage());
    }
});
```

---

## 22. Scoreboard, BossBar & Tab List

### Scoreboard (Sidebar)

```java
import org.bukkit.scoreboard.Scoreboard;
import org.bukkit.scoreboard.Objective;
import org.bukkit.scoreboard.DisplaySlot;

ScoreboardManager manager = Bukkit.getScoreboardManager();
Scoreboard board = manager.getNewScoreboard();

Objective obj = board.registerNewObjective("myboard", Criteria.DUMMY,
    Component.text("=== My Server ===", NamedTextColor.GOLD));
obj.setDisplaySlot(DisplaySlot.SIDEBAR);

// Add scores (higher score = higher on the sidebar)
Score score1 = obj.getScore(Component.text("Player:", NamedTextColor.WHITE)
    .append(Component.text(player.getName(), NamedTextColor.YELLOW)));
score1.setScore(5);
Score score2 = obj.getScore(Component.text("Kills: 0", NamedTextColor.GREEN));
score2.setScore(4);
Score score3 = obj.getScore(Component.text("Coins: 100", NamedTextColor.GOLD));
score3.setScore(3);

player.setScoreboard(board);
```

**Dynamic updates** — use a repeating scheduler task:
```java
getServer().getScheduler().runTaskTimer(this, () -> {
    for (Player p : Bukkit.getOnlinePlayers()) {
        Scoreboard sb = p.getScoreboard();
        Objective obj = sb.getObjective("myboard");
        if (obj != null) {
            obj.getScore(Component.text("Online: " + Bukkit.getOnlinePlayers().size()))
                .setScore(1);
        }
    }
}, 0L, 20L); // update every second
```

### BossBar

```java
import org.bukkit.boss.BossBar;
import org.bukkit.boss.BarColor;
import org.bukkit.boss.BarStyle;
import org.bukkit.boss.BarFlag;

// Create
BossBar bar = Bukkit.createBossBar(
    new NamespacedKey(plugin, "my_bar"),
    "Loading...",
    BarColor.BLUE,
    BarStyle.SOLID,
    BarFlag.DARKEN_SKY  // optional flags
);

// Show to all players
bar.setVisible(true);
bar.addPlayer(player);

// Remove from player
bar.removePlayer(player);

// Update
bar.setTitle("Progress: 50%");
bar.setProgress(0.5);  // 0.0 to 1.0
bar.setColor(BarColor.GREEN);
bar.setStyle(BarStyle.SEGMENTED_6);

// Cleanup on plugin disable
bar.removeAll();
```

### Tab List (Player List Header/Footer)

```java
player.sendPlayerListHeaderAndFooter(
    Component.text("Welcome to MyServer!", NamedTextColor.AQUA)
        .appendNewline()
        .append(Component.text("Enjoy your stay!", NamedTextColor.WHITE)),
    Component.text("Players online: " + Bukkit.getOnlinePlayers().size(), NamedTextColor.GRAY)
        .appendNewline()
        .append(Component.text("play.myserver.com", NamedTextColor.YELLOW))
);
```

### Player Name Prefix/Suffix (Team-based)

```java
Scoreboard sb = Bukkit.getScoreboardManager().getMainScoreboard();
Team team = sb.getTeam("admin_team");
if (team == null) team = sb.registerNewTeam("admin_team");
team.prefix(Component.text("[Admin] ", NamedTextColor.RED));
team.color(NamedTextColor.RED); // name color
team.addEntry(player.getName());
```

---

## 23. Custom Inventory / GUI Menus

### Basic Custom Inventory

```java
import net.kyori.adventure.text.Component;
import org.bukkit.inventory.Inventory;
import org.bukkit.inventory.ItemStack;
import org.bukkit.Material;
import org.bukkit.Bukkit;

// Create a 27-slot (3 row) inventory
Inventory gui = Bukkit.createInventory(null, 27,
    Component.text("My GUI", NamedTextColor.GOLD));

// Add items with click handlers
ItemStack diamond = ItemsUtils.createItem(Material.DIAMOND, 1, "Claim Reward", List.of("Click to claim!"));
gui.setItem(13, diamond);  // center slot

player.openInventory(gui);
```

### Inventory Click Handler

```java
import org.bukkit.event.inventory.InventoryClickEvent;

@EventHandler
public void onClick(InventoryClickEvent event) {
    if (!(event.getView().title().equals(Component.text("My GUI", NamedTextColor.GOLD)))) return;
    event.setCancelled(true); // prevent item stealing

    if (!(event.getWhoClicked() instanceof Player player)) return;
    ItemStack clicked = event.getCurrentItem();
    if (clicked == null) return;

    if (clicked.getType() == Material.DIAMOND) {
        player.giveExp(100);
        player.sendMessage(Component.text("Reward claimed!", NamedTextColor.GREEN));
        player.closeInventory();
    }
}
```

### Paper Custom InventoryHolder (Type-safe)

Paper provides a type-safe way to identify inventories:

```java
import org.bukkit.inventory.InventoryHolder;
import org.jetbrains.annotations.NotNull;

public class MyGUI implements InventoryHolder {
    private final Player owner;
    private final Inventory inventory;

    public MyGUI(Player owner) {
        this.owner = owner;
        this.inventory = Bukkit.createInventory(this, 27,
            Component.text("My Custom GUI", NamedTextColor.GOLD));
        // populate items...
    }

    @NotNull
    @Override
    public Inventory getInventory() {
        return inventory;
    }
}

// In InventoryClickEvent handler:
@EventHandler
public void onGUIClick(InventoryClickEvent event) {
    if (event.getInventory().getHolder() instanceof MyGUI gui) {
        event.setCancelled(true);
        // Handle click by checking gui.getInventory() slots
    }
}
```

### GUI Libraries

For complex GUIs, consider these frameworks:
- **Triumph GUI** — annotation-based, pagination, async loading
- **IF** (Inventory Framework) — Kotlin DSL for GUI creation
- **Floodgate GUI** — cross-platform GUIs (Java + Bedrock)

---

## 24. Ecosystem Integrations

### Vault (Economy)

```java
// plugin.yml / paper-plugin.yml dependency:
// depend: [Vault]

import net.milkbowl.vault.economy.Economy;
import org.bukkit.plugin.RegisteredServiceProvider;

private Economy economy;

// In onEnable():
if (!setupEconomy()) {
    getLogger().severe("Vault economy not found! Disabling.");
    getServer().getPluginManager().disablePlugin(this);
    return;
}

private boolean setupEconomy() {
    if (getServer().getPluginManager().getPlugin("Vault") == null) return false;
    RegisteredServiceProvider<Economy> rsp = getServer().getServicesManager()
        .getRegistration(Economy.class);
    if (rsp == null) return false;
    economy = rsp.getProvider();
    return economy != null;
}

// Usage:
@EventHandler
public void onKill(PlayerDeathEvent event) {
    if (event.getEntity().getKiller() instanceof Player killer) {
        economy.depositPlayer(killer, 50.0); // give 50 coins
        killer.sendMessage(Component.text("+50 coins! Balance: " + economy.getBalance(killer),
            NamedTextColor.GREEN));
    }
}
```

### PlaceholderAPI

```java
// plugin.yml dependency:
// softdepend: [PlaceholderAPI]

// Register custom placeholders:
import me.clip.placeholderapi.expansion.PlaceholderExpansion;

public class MyExpansion extends PlaceholderExpansion {

    @Override
    public String getIdentifier() { return "myplugin"; }

    @Override
    public String getAuthor() { return "YourName"; }

    @Override
    public String getVersion() { return "1.0.0"; }

    @Override
    public boolean persist() { return true; }

    @Override
    public String onPlaceholderRequest(Player player, String params) {
        if (player == null) return "";
        if (params.equals("kills")) {
            return String.valueOf(killTracker.getKills(player));
        }
        if (params.equals("coins")) {
            return String.valueOf((int)economy.getBalance(player));
        }
        return null;
    }
}

// In onEnable():
new MyExpansion().register(); // auto-unregisters on disable

// Other plugins can now use: %myplugin_kills%, %myplugin_coins%
```

### Plugin Messaging Channels (BungeeCord/Velocity)

```java
// Send player to another server on the proxy:
import com.google.common.io.ByteArrayDataOutput;
import com.google.common.io.ByteStreams;

// In onEnable():
getServer().getMessenger().registerOutgoingPluginChannel(this, "BungeeCord");

// Send player to a server:
ByteArrayDataOutput out = ByteStreams.newDataOutput();
out.writeUTF("Connect");
out.writeUTF("lobby");  // target server name
player.sendPluginMessage(this, "BungeeCord", out.toByteArray());

// Receive messages from the proxy:
getServer().getMessenger().registerIncomingPluginChannel(this, "BungeeCord", (channel, player, message) -> {
    ByteArrayDataInput in = ByteStreams.newDataInput(message);
    String subChannel = in.readUTF();
    if (subChannel.equals("PlayerCount")) {
        String server = in.readUTF();
        int count = in.readInt();
        getLogger().info("Server " + server + " has " + count + " players.");
    }
});
```

---

## 25. Best Practices Summary

1. **compileOnly/provided for paper-api** — never bundle the API into your JAR; the server provides it
2. **Separate listeners, commands, and config into distinct classes** — use the Manager pattern for medium+ plugins
3. **Use Adventure Components, not legacy ChatColor** — Paper APIs expect Components everywhere
4. **Use PDC, not NBT reflection** — PDC is the stable, official API for custom persistent data
5. **Async tasks for I/O** — database, HTTP, and file operations must run off the main thread
6. **teleportAsync over teleport** — async teleport avoids blocking the main thread
7. **saveDefaultConfig() in onEnable** — ensures users get a default config file on first run
8. **Use NamespacedKey with your plugin name** — prevents data key conflicts between plugins
9. **Check null before using getPlayer()/getPlayerExact()** — players can be offline
10. **Register commands via LifecycleEvents.COMMANDS** — the modern Paper Brigadier approach
11. **Use ItemsUtils or similar factory** — centralize ItemStack creation to avoid duplication
12. **Write a good README** — include commands table, permissions table, config explanation, build instructions
13. **Never call Bukkit API from async threads** — always switch back with `runTask()` when needed
14. **Prefer paper-plugin.yml for new plugins** — richer dependency control than plugin.yml
15. **Use Run-Paper Gradle plugin** — fastest way to spin up a test server during development
16. **DRY: use gradle.properties** — define version/name once, reference via placeholders
17. **Write unit tests** — JUnit 5 + Mockito for listener/command logic
18. **Use GitHub Actions** — automated build, test, and publish workflow
19. **HikariCP for database connections** — always use a connection pool, never raw JDBC
20. **Include LICENSE file** — MIT or GPLv3, know the GPL implications of linking Paper API
21. **Use CustomInventoryHolder** — type-safe GUI identification, not string title matching
22. **Integrate with Vault for economy** — don't hardcode your own economy if Vault exists
23. **Provide PlaceholderAPI placeholders** — allows other plugins to display your plugin's data
24. **Clean up resources in onDisable** — cancel tasks, close DB connections, remove boss bars

---

## References

### Official Documentation
- Paper Docs: https://docs.papermc.io/paper/dev/
- Paper API Javadoc: https://jd.papermc.io/paper/
- Adventure Docs: https://docs.advntr.dev/
- Paper GitHub: https://github.com/PaperMC/paper
- PaperMC Docs Repo: https://github.com/PaperMC/docs

### Community & Tutorials
- SpigotMC Forums (Plugin Dev): https://www.spigotmc.org/forums/spigot-plugin-development.52/
- 15-Minute Plugin Tutorial (kangarko): https://gist.github.com/kangarko/456d9cfce52dc971b93dbbd12a95f43c
- VSCode Paper Plugin Guide (luttje): https://github.com/luttje/papermc-vscode-plugin-dev
- MCBBS 中文教程: https://www.mcbbs.co/thread-3592-1-1.html

### Templates & Examples
- PaperPluginTemplate (Gradle + CI/CD): https://github.com/Atrimilan/PaperPluginTemplate
- PaperPluginTemplate (DRY gradle.properties): https://github.com/Ayydxn/PaperPluginTemplate
- MC-Plugin-Template (Maven + Manager): https://github.com/achillebourgault/MC-Plugin-Template
- minecraft-plugin-template (Spigot/Paper): https://github.com/atlaska826/minecraft-plugin-template
- Kotlin-PaperMC-Template: https://github.com/jcurtis06/Kotlin-PaperMC-Template
- Minecraft-Plugin-Template (Flyway+jOOQ+Triumph): https://github.com/milkdrinkers/Minecraft-Plugin-Template
- paper-plugin-template (paperweight userdev): https://github.com/LINCKODE/paper-plugin-template
- CustomEnchantExample (1.21+ Registry API): https://github.com/SaberSupe/CustomEnchantExample
- MeT-Music (中文 Paper 插件示例): https://github.com/MeTerminator/MeT-Music_MCPlugin

### Libraries & Frameworks
- twilight (Kotlin API: events, scheduler, scoreboard, GUI, DB): https://github.com/flytegg/twilight
- helper by lucko (all-in-one: events, scheduler, GUI, scoreboard, messaging): https://github.com/lucko/helper
- kelp (all-in-one: sidebar, inventory, NPC, commands, config): https://github.com/PXAV/kelp
- SpigotLib (all-in-one: scoreboard, GUI, packet, economy): https://github.com/gyurix/SpigotLib
- ACF — Annotation Command Framework (annotation-based commands): https://github.com/aikar/commands
- cloud — modern command framework (Mojang Brigadier wrapper): https://github.com/Incendo/cloud
- CommandAPI (Brigadier simplifier): https://github.com/JorelAli/CommandAPI
- XSeries (cross-version Material/Sound/Particle): https://github.com/CryptoMorin/XSeries
- AnvilGUI (virtual anvil input): https://github.com/WesJD/AnvilGUI
- libby (runtime dependency loading): https://github.com/Byteflux/libby
- spigot-plugin-resources (curated library list): https://github.com/Paul2708/spigot-plugin-resources

### Ecosystem
- Vault (economy/permissions/chat): https://github.com/MilkBowl/Vault
- PlaceholderAPI: https://github.com/PlaceholderAPI/PlaceholderAPI
- ProtocolLib (packet manipulation): https://github.com/dmulloy2/ProtocolLib
- PacketEvents (modern packet library): https://github.com/retrooper/PacketEvents
