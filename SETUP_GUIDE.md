# Ebutale Balancing Mod — Setup Guide

Complete walkthrough for getting the mod building, running, and integrated with JEI.

---

## 1. Prerequisites

### 1a. Java Development Kit (JDK 21)

NeoForge 1.21.1 requires Java 21. The Minecraft launcher bundles its own runtime, but mod **development** needs a full JDK.

**Install via Homebrew (macOS):**
```bash
brew install openjdk@21
```

**Symlink for system discovery** (requires admin password):
```bash
sudo ln -sfn /opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-21.jdk
```

If you can't run sudo, Gradle can still find the JDK — just set `JAVA_HOME` before building:
```bash
export JAVA_HOME="/opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home"
```

To make this permanent, add the export line to `~/.zshrc`.

**Verify:**
```bash
$JAVA_HOME/bin/java -version
# openjdk version "21.0.10" ...
```

### 1b. IDE (Recommended)

- **IntelliJ IDEA Community** (free) — best NeoForge support, understands Gradle natively
- Or **VS Code** with the Java Extension Pack

### 1c. Git

```bash
git --version
# Should already be installed on macOS
```

### 1d. Modrinth App

The Modrinth App (`/Applications/Modrinth App.app`) is used for testing the built mod in a managed Minecraft instance with other mods installed. It is **not** needed for development builds — `./gradlew runClient` handles that.

Initial Modrinth App setup:
1. Open the app and log in with your Microsoft/Xbox account
2. Create a new instance: select **Minecraft 1.21.1** with **NeoForge** as the loader
3. Add **JEI** and **The Ebutale** mods from the Modrinth mod browser
4. This instance is where you drop your built `.jar` for integration testing

---

## 2. Clone & First Build

```bash
cd ~/Documents/Minecraft\ modding
git clone https://github.com/TeamHusmann/The-Ebutale-Balancing-1.21.X.git
cd The-Ebutale-Balancing-1.21.X
```

**First build** (downloads Gradle, NeoForge, Minecraft sources, JEI — takes ~2 minutes):
```bash
export JAVA_HOME="/opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home"
./gradlew build
```

If it succeeds, the mod jar is at `build/libs/ebutalebalancing-1.0.0.jar`.

**Run Minecraft in dev mode** (launches MC 1.21.1 with the mod + JEI loaded):
```bash
./gradlew runClient
```

---

## 3. Project Structure (Current)

All template boilerplate has been cleaned up. This is the current layout:

```
The-Ebutale-Balancing-1.21.X/
├── build.gradle                           # Build config, dependencies (JEI + The Ebutale)
├── gradle.properties                      # Versions, mod metadata, JEI version
├── settings.gradle                        # Plugin repos (foojay JDK resolver)
├── libs/
│   └── the_ebutale-1.1.s1-neoforge-1.21.1.jar  # The Ebutale mod (dev dependency)
├── SETUP_GUIDE.md                         # This file
├── src/
│   ├── main/
│   │   ├── java/net/zeifrein/ebutalebalancing/
│   │   │   ├── EbutaleBalancing.java      # Main @Mod entrypoint
│   │   │   ├── EbutaleBalancingClient.java # Client-side setup (config screen)
│   │   │   ├── Config.java                # Mod config (enableJeiInfo toggle)
│   │   │   └── integration/jei/
│   │   │       └── EbutaleBalancingJeiPlugin.java  # JEI plugin
│   │   ├── resources/
│   │   │   ├── assets/ebutalebalancing/lang/
│   │   │   │   └── en_us.json             # English translations
│   │   │   └── data/ebutalebalancing/recipe/  # Balancing recipes (26 files)
│   │   │       ├── blasting_*.json        # Blast furnace recipes for ores (12)
│   │   │       ├── smelting_*.json        # Furnace cooking for foods (7)
│   │   │       └── campfire_*.json        # Campfire cooking for foods (7)
│   │   └── templates/META-INF/
│   │       └── neoforge.mods.toml         # Mod metadata + dependencies
│   └── generated/resources/               # Output from data generators
└── gradle/                                # Gradle wrapper (9.2.1)
```

### Key files at a glance

| File | Purpose |
|------|---------|
| `gradle.properties` | All version pins: MC 1.21.1, NeoForge 21.1.222, JEI 19.27.0.340, mod metadata |
| `build.gradle` | JEI Maven repo, JEI + The Ebutale deps, NeoForge plugin config |
| `libs/the_ebutale-*.jar` | The Ebutale mod jar — compile + runtime dependency for dev |
| `neoforge.mods.toml` | Declares The Ebutale (required) and JEI (optional, client-side) as dependencies |
| `EbutaleBalancing.java` | `@Mod` class — registers config and event listeners |
| `EbutaleBalancingClient.java` | `@Mod(dist=CLIENT)` — registers NeoForge config screen |
| `Config.java` | `enableJeiInfo` boolean config option |
| `EbutaleBalancingJeiPlugin.java` | `@JeiPlugin` — JEI discovers this automatically, skeleton for recipe registration |

---

## 4. Changes Made From the MDK Template

These changes have already been applied. Documenting them here for reference:

### 4a. Renamed `ExampleModClient.java` → `EbutaleBalancingClient.java`
- Updated class name to match
- Removed debug logging (`HELLO FROM CLIENT SETUP`, username printing)
- Kept the NeoForge config screen registration

### 4b. Moved `assets/examplemod/` → `assets/ebutalebalancing/`
- The assets directory must match the mod ID for NeoForge to find resources

### 4c. Replaced lang file content
- Removed all `examplemod.*` and `block.examplemod.*` / `item.examplemod.*` placeholder keys
- Added `ebutalebalancing.configuration.*` keys for the config screen

### 4d. Cleaned up `Config.java`
- Removed boilerplate values: `LOG_DIRT_BLOCK`, `MAGIC_NUMBER`, `MAGIC_NUMBER_INTRODUCTION`, `ITEM_STRINGS`
- Added `ENABLE_JEI_INFO` — controls whether JEI info pages are shown

### 4e. Cleaned up `EbutaleBalancing.java`
- Removed boilerplate comments and empty `addCreative` listener
- Removed unused `BuildCreativeModeTabContentsEvent` import
- Added a startup log message

### 4f. Updated `neoforge.mods.toml`
- Added `authors` field (pulls from `gradle.properties`)
- Updated the description text
- Added The Ebutale as a **required** dependency (`modId="the_ebutale"`, ordering AFTER)
- Added JEI as an **optional client-side** dependency (`modId="jei"`, version `[19.27,)`)

### 4g. Updated `build.gradle`
- Added `mod_authors` to the template replacement properties map
- Added Jared's Maven repository (`https://maven.blamejared.com/`)
- Uncommented the three JEI dependency lines:
  - `compileOnly` for `jei-1.21.1-common-api` and `jei-1.21.1-neoforge-api`
  - `localRuntime` for `jei-1.21.1-neoforge` (available in dev, not shipped with the mod)

### 4h. Updated `gradle.properties`
- Added `jei_version=19.27.0.340`
- Added `mc_version=1.21.1` (used by JEI dependency coordinates)

### 4i. Created JEI plugin skeleton
- New package: `integration.jei`
- `EbutaleBalancingJeiPlugin.java` implements `IModPlugin` with `@JeiPlugin`
- Skeleton `registerCategories()`, `registerRecipes()`, `registerRecipeCatalysts()` methods with commented examples
- JEI discovers the plugin automatically at runtime via the annotation

### 4j. Added The Ebutale mod jar as a dev dependency
- Downloaded from Modrinth: `the_ebutale-1.1.s1-neoforge-1.21.1.jar` (v1.1 snapshot 1, "The Rebuilt Update")
- Placed in `libs/` directory
- Added to `build.gradle` as `implementation files("libs/the_ebutale-1.1.s1-neoforge-1.21.1.jar")`
- This makes The Ebutale's items/blocks available at compile time and loads the mod during `./gradlew runClient`
- **To update:** download a new jar from https://modrinth.com/mod/the-ebutale, replace the file in `libs/`, update the filename in `build.gradle`

### 4k. Created 26 balancing recipes
All recipes are JSON data packs in `src/main/resources/data/ebutalebalancing/recipe/`. JEI picks these up automatically — no plugin code needed.

---

## 5. Balancing Recipes Added

### The Ebutale Mod Analysis

The Ebutale (mod ID: `the_ebutale`, v1.1.s1) is a MCreator mod by SocialTag that adds:
- **The Eburis dimension** (accessed via Rift Opener + Anomaly Researcher NPC + Dimensional Sanctum)
- **9 metal tiers:** Eburisian Copper, Tin, Silver, Flimsium, Tritium, Mytrium, Vervonite, Cryonium, Pyronium
- **6 stone types** with full block sets (cobbled, bricks, slabs, stairs, walls, tiles, grit)
- **12 wood types** (softwood + hardwood) with tools, bows, block sets
- **12 mobs**, 7+ armor sets, 15+ food items, 3 glass types with 16 dye colors each
- **1,450 existing recipes** — ALL vanilla types (shaped, shapeless, smelting, stonecutting, smoking)
- **Zero custom machines or recipe types** — everything uses vanilla crafting stations

### Gaps Identified

1. **No blast furnace recipes** — all ore smelting is furnace-only, no 2x-speed blasting alternative
2. **No furnace/campfire cooking** — all food cooking is smoker-only, missing standard furnace and campfire
3. These are standard vanilla recipe types that players expect to work

### Recipes Created (26 total)

#### Blast Furnace — Ore Processing (12 recipes)
Blast furnace processes ores at 2x speed (100 ticks vs 200). Each ore has two input variants: the ore block and the raw item.

| Recipe | Input | Output |
|--------|-------|--------|
| `blasting_eburisian_copper_from_ore` | Eburisian Copper Ore | Eburisian Copper Ingot |
| `blasting_eburisian_copper_from_raw` | Raw Eburisian Copper | Eburisian Copper Ingot |
| `blasting_eburisian_tin_from_ore` | Eburisian Tin Ore | Eburisian Tin Ingot |
| `blasting_eburisian_tin_from_raw` | Raw Eburisian Tin | Eburisian Tin Ingot |
| `blasting_eburisian_silver_from_ore` | Eburisian Silver Ore | Eburisian Silver Ingot |
| `blasting_eburisian_silver_from_raw` | Raw Eburisian Silver | Eburisian Silver Ingot |
| `blasting_flimsium_from_ore` | Flimsium Ore | Flimsium Ingot |
| `blasting_flimsium_from_raw` | Raw Flimsium | Flimsium Ingot |
| `blasting_tritium_from_ore` | Tritium Ore | Tritium Ingot |
| `blasting_tritium_from_raw` | Raw Tritium | Tritium Ingot |
| `blasting_mytrium_from_ore` | Mytrium Ore | Mytrium Ingot |
| `blasting_mytrium_from_raw` | Raw Mytrium | Mytrium Ingot |

*Note: Cryonium, Vervonite, and Pyronium are crafted from shards/components — not smelted from ore — so no blasting recipe applies.*

#### Furnace Cooking — Food (7 recipes)
Standard furnace smelting for food items (200 ticks, 0.35 XP). The Ebutale only had smoker recipes.

| Recipe | Input | Output |
|--------|-------|--------|
| `smelting_eburisian_mutton` | Raw Eburisian Mutton | Cooked Eburisian Mutton |
| `smelting_eburisian_pork_loin` | Raw Eburisian Pork Loin | Cooked Eburisian Pork Loin |
| `smelting_silkie_chicken` | Raw Silkie Chicken | Cooked Silkie Chicken |
| `smelting_eburisian_beef` | Raw Eburisian Beef | Eburisian Steak |
| `smelting_venison` | Raw Venison | Venison Steak |
| `smelting_mighty_chuck` | Raw Mighty Chuck | Mighty Chuck Steak |
| `smelting_silkie_egg` | Silkie Egg | Fried Silkie Egg |

#### Campfire Cooking — Food (7 recipes)
Campfire cooking (600 ticks, 0.35 XP, no fuel needed). Matches vanilla campfire behavior.

| Recipe | Input | Output |
|--------|-------|--------|
| `campfire_eburisian_mutton` | Raw Eburisian Mutton | Cooked Eburisian Mutton |
| `campfire_eburisian_pork_loin` | Raw Eburisian Pork Loin | Cooked Eburisian Pork Loin |
| `campfire_silkie_chicken` | Raw Silkie Chicken | Cooked Silkie Chicken |
| `campfire_eburisian_beef` | Raw Eburisian Beef | Eburisian Steak |
| `campfire_venison` | Raw Venison | Venison Steak |
| `campfire_mighty_chuck` | Raw Mighty Chuck | Mighty Chuck Steak |
| `campfire_silkie_egg` | Silkie Egg | Fried Silkie Egg |

### How these appear in JEI

All 26 recipes are standard vanilla recipe types. JEI automatically detects them and shows them in the appropriate categories:
- **Blast furnace** icon → blasting recipes (press R on an ingot to see)
- **Furnace** icon → smelting food recipes
- **Campfire** icon → campfire cooking recipes
- Players can also press **U** on a raw item to see all ways to cook/process it

No JEI plugin code is required for these recipes — the JSON data packs are sufficient.

---

## 6. JEI Integration — How It Works

### What JEI does for players

JEI (Just Enough Items) adds an in-game item/recipe browser overlay. Players press **R** on any item to see recipes that produce it, or **U** to see recipes that use it as an ingredient.

### What the mod needs to do

**Standard crafting/smelting/smithing recipes** added via JSON data packs are picked up by JEI automatically — no code needed. The JEI plugin is for:

1. **Custom recipe categories** — if the mod adds non-vanilla crafting mechanics (custom machines, altars, etc.), each needs an `IRecipeCategory<T>` so JEI knows how to display it
2. **Item info pages** — `registration.addItemStackInfo(stack, component)` adds a text description page for any item in JEI
3. **Hiding/showing items** — control what appears in the JEI browser
4. **Recipe catalysts** — register which blocks/items open a recipe category (e.g., a custom crafting table)

### Plugin entry point

`EbutaleBalancingJeiPlugin.java` — annotated with `@JeiPlugin`, JEI finds it automatically. Registration methods are called in this order:

1. `registerCategories()` — create and register `IRecipeCategory` instances
2. `registerRecipes()` — feed recipe data to each category
3. `registerRecipeCatalysts()` — associate crafting stations with categories
4. (Optional) `registerGuiHandlers()` — make GUI areas clickable to open JEI
5. (Optional) `registerRecipeTransferHandlers()` — enable the `[+]` button to auto-fill crafting grids

### Adding a custom recipe category (template)

When you need a custom recipe type, create a class implementing `IRecipeCategory<T>`:

```java
package net.zeifrein.ebutalebalancing.integration.jei;

import mezz.jei.api.gui.builder.IRecipeLayoutBuilder;
import mezz.jei.api.gui.drawable.IDrawable;
import mezz.jei.api.recipe.IFocusGroup;
import mezz.jei.api.recipe.RecipeIngredientRole;
import mezz.jei.api.recipe.RecipeType;
import mezz.jei.api.recipe.category.IRecipeCategory;
import mezz.jei.api.helpers.IGuiHelper;
import net.minecraft.network.chat.Component;
import net.zeifrein.ebutalebalancing.EbutaleBalancing;

public class YourRecipeCategory implements IRecipeCategory<YourRecipe> {

    public static final RecipeType<YourRecipe> RECIPE_TYPE =
        RecipeType.create(EbutaleBalancing.MOD_ID, "your_recipe", YourRecipe.class);

    private final IDrawable icon;

    public YourRecipeCategory(IGuiHelper guiHelper) {
        this.icon = guiHelper.createDrawableItemStack(/* your catalyst item */);
    }

    @Override public RecipeType<YourRecipe> getRecipeType() { return RECIPE_TYPE; }
    @Override public Component getTitle() {
        return Component.translatable("gui.ebutalebalancing.category.your_recipe");
    }
    @Override public IDrawable getIcon() { return icon; }
    @Override public int getWidth() { return 176; }
    @Override public int getHeight() { return 85; }

    @Override
    public void setRecipe(IRecipeLayoutBuilder builder, YourRecipe recipe, IFocusGroup focuses) {
        builder.addSlot(RecipeIngredientRole.INPUT, 56, 17)
            .addIngredients(recipe.getIngredient());
        builder.addSlot(RecipeIngredientRole.OUTPUT, 116, 35)
            .addItemStack(recipe.getResultItem());
    }
}
```

Then register it in the plugin:
```java
@Override
public void registerCategories(IRecipeCategoryRegistration registration) {
    var guiHelper = registration.getJeiHelpers().getGuiHelper();
    registration.addRecipeCategories(new YourRecipeCategory(guiHelper));
}

@Override
public void registerRecipes(IRecipeRegistration registration) {
    registration.addRecipes(YourRecipeCategory.RECIPE_TYPE, yourRecipeList);
}
```

---

## 7. Testing

### Dev testing (fast iteration)
```bash
export JAVA_HOME="/opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home"
./gradlew runClient
```
This launches MC with the mod and JEI loaded in a dev environment. Changes to code require restarting, but resource changes (JSON recipes, lang files) can be reloaded in-game with F3+T.

### Integration testing (Modrinth App)
1. Build the jar: `./gradlew build`
2. Open Modrinth App → your 1.21.1 NeoForge instance
3. Copy `build/libs/ebutalebalancing-1.0.0.jar` into the instance's `mods/` folder
4. Also ensure **JEI** and **The Ebutale** are installed in the instance
5. Launch and verify the mod list shows Ebutale Balancing with its dependencies

### Verifying JEI integration
1. In-game, open inventory — JEI overlay should appear on the right
2. Search for any Ebutale items
3. Press **R** on an item to see its recipes, **U** to see its uses
4. Standard JSON recipes appear automatically; custom categories appear once registered in the plugin

---

## 8. Common Gradle Commands

```bash
# Always set JAVA_HOME first (or add to ~/.zshrc):
export JAVA_HOME="/opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home"
```

| Command | What it does |
|---------|-------------|
| `./gradlew build` | Compile and package the mod jar into `build/libs/` |
| `./gradlew runClient` | Launch Minecraft with the mod + JEI (dev mode) |
| `./gradlew runServer` | Launch a dedicated server with the mod |
| `./gradlew runData` | Run data generators (recipe/loot table JSON generation) |
| `./gradlew clean` | Delete build artifacts |
| `./gradlew clean build` | Full clean rebuild |
| `./gradlew --refresh-dependencies` | Force re-download all dependencies |

---

## 9. Version Reference

| Component | Version | Where defined |
|-----------|---------|---------------|
| Minecraft | 1.21.1 | `gradle.properties` → `minecraft_version` |
| NeoForge | 21.1.222 | `gradle.properties` → `neo_version` |
| ModDevGradle | 2.0.141 | `build.gradle` → plugin version |
| Gradle | 9.2.1 | `gradle/wrapper/gradle-wrapper.properties` |
| Java (JDK) | 21 | `build.gradle` → `java.toolchain.languageVersion` |
| JEI | 19.27.0.340 | `gradle.properties` → `jei_version` |
| Parchment Mappings | 2024.11.17 | `gradle.properties` → `parchment_mappings_version` |

---

## 10. The Ebutale Mod Reference

- **Mod ID:** `the_ebutale` (confirmed)
- **Version:** 1.1.s1 ("The Rebuilt Update" snapshot 1)
- **Author:** SocialTag
- **Modrinth:** https://modrinth.com/mod/the-ebutale
- **Made with:** MCreator (package: `net.mcreator.theebutale`)
- **License:** All Rights Reserved (no source code available)
- **Recipe types:** ALL vanilla — shaped crafting, shapeless, smelting, stonecutting, smoking
- **No custom machines** — everything uses vanilla crafting table, furnace, smoker, stonecutter

### Key item namespaces (all prefixed `the_ebutale:`)

**Metals:** `eburisian_copper_ingot`, `eburisian_tin_ingot`, `eburisian_silver_ingot`, `flimsium_ingot`, `tritium_ingot`, `mytrium_ingot`, `vervonite_ingot`, `cryonium_ingot`, `pyronium_ingot`

**Raw ores:** `raw_eburisian_copper`, `raw_eburisian_tin`, `raw_eburisian_silver`, `raw_flimsium`, `raw_tritium`, `raw_mytrium`

**Foods (raw):** `raw_eburisian_mutton`, `raw_eburisian_pork_loin`, `raw_silkie_chicken`, `raw_eburisian_beef`, `raw_venison`, `raw_mighty_chuck`, `silkie_egg`

**Foods (cooked):** `cooked_eburisian_mutton`, `cooked_eburisian_pork_loin`, `cooked_silkie_chicken`, `eburisian_steak`, `venison_steak`, `mighty_chuck_steak`, `fried_silkie_egg`

---

## 11. Next Steps

- [x] ~~Verify The Ebutale mod ID~~ — confirmed `the_ebutale`
- [x] ~~Add The Ebutale jar as dev dependency~~ — `libs/` directory
- [x] ~~Add blast furnace recipes for all smeltable ores~~ — 12 recipes
- [x] ~~Add furnace + campfire cooking for all foods~~ — 14 recipes
- [x] ~~Test in-game with `./gradlew runClient`~~ — recipes appear in JEI, confirmed working 2026-04-03
- [ ] Add `addItemStackInfo()` calls in JEI plugin for key Ebutale items (dimension access guide, tier progression, etc.)
- [ ] Identify items that still have NO recipe at all and add crafting recipes for them
- [ ] Consider adding stonecutting shortcuts for Ebutale stone variants
- [ ] Set up Modrinth App test instance for integration testing
- [ ] Talk to Jacob about what other balancing changes are needed
