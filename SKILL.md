---
name: paper-plugin-dev
description: This skill should be used when the user asks to create, develop, or debug PaperMC/Minecraft server plugins. Covers Java basics for beginners, Paper API, Bukkit API, event systems, command systems (Brigadier), scheduling, Persistent Data Containers, Adventure text, configuration, and plugin debugging. Triggers on: "Paper plugin", "Bukkit plugin", "Minecraft plugin", "papermc", "spigot plugin", "/paper", "/bukkit", or when writing Java code that imports org.bukkit or io.papermc.paper.
version: 1.0.0
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

| Tool | Purpose |
|------|---------|
| JDK 21+ | Paper requires Java 21+ |
| IntelliJ IDEA | Recommended IDE (Community Edition is free) |
| Paper Server | Local test server |

### Gradle Build Script (build.gradle.kts)

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

### Standard Project Structure

```
my-plugin/
├── src/main/java/com/example/myplugin/
│   └── MyPlugin.java              # Main class
├── src/main/resources/
│   ├── plugin.yml                 # Plugin metadata
│   └── config.yml                 # Default config (optional)
├── build.gradle.kts
└── settings.gradle.kts
```

Build with `./gradlew build`, output JAR in `build/libs/`.

---

## 2. Plugin Main Class & Lifecycle

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

## 3. Plugin Metadata Files

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

## 4. Event System

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

## 5. Command System (Brigadier)

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

## 6. Scheduler — Timed & Repeating Tasks

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

## 7. Persistent Data Container (PDC)

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

## 8. Configuration (config.yml)

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

## 9. Adventure Text & UI

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

## 10. Common API Operations

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

## 11. Data Component API (1.20.5+)

Modern item data system replacing NBT tags:

```java
// Check if item has custom name
boolean hasName = itemStack.hasData(DataComponentTypes.CUSTOM_NAME);

// Set enchantments
itemStack.setData(DataComponentTypes.ENCHANTMENTS,
    ItemEnchantments.itemEnchantments().add(Enchantment.SHARPNESS, 5).build());
```

---

## 12. AsyncChatEvent (Modern Chat Handling)

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

## 13. Complete Plugin Template

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

## 14. Debugging

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

## 15. Java Basics for Beginners

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

## 16. Best Practices Summary

1. **compileOnly for paper-api** — never bundle the API into your JAR
2. **Separate listeners, commands, and config into distinct classes** — don't put everything in the main class
3. **Use Adventure Components, not legacy ChatColor** — Paper APIs expect Components
4. **Use PDC, not NBT reflection** — PDC is the stable, official API for custom data
5. **Async tasks for I/O** — database, HTTP, and file operations must run off the main thread
6. **teleportAsync over teleport** — async teleport avoids blocking the main thread
7. **saveDefaultConfig() in onEnable** — ensures users get a default config file
8. **Use NamespacedKey with your plugin name** — prevents data key conflicts between plugins
9. **Check null before using getPlayer()** — players can be offline/null
10. **Register commands via LifecycleEvents.COMMANDS** — this is the modern Paper approach

---

## References

- Paper Docs: https://docs.papermc.io/paper/dev/
- Paper API Javadoc: https://jd.papermc.io/paper/
- Adventure Docs: https://docs.advntr.dev/
- Paper GitHub: https://github.com/PaperMC/paper
- SpigotMC Forums: https://www.spigotmc.org/forums/spigot-plugin-development.52/
