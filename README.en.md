# Minecraft Demo
[![en](https://img.shields.io/badge/lang-en-blue.svg)](README.en.md)
[![en](https://img.shields.io/badge/lang-zh--CN-red.svg)](README.md)

**Written during my second year of high school, just take a look if you're interested. No longer maintained.**

This project was created using `Unity 2021.3.8f1c1`.

**The following images are actual in-game screenshots.**

![In-game Screenshot (1920x1080)](/Recordings/Capture.png)

![Biome Border (1920x1080)](/Recordings/biome.png)

![Classic Matchbox House (1920x1080)](/Recordings/house.png)

![Matchbox House Interior (1920x1080)](/Recordings/house_inside.png)

# Features

* Infinite world with randomly generated terrain
* Biomes (Savanna, Desert, Forest, Beach, and MORE+)
* Caves and ore veins (Diamond, Coal, Gold, and MORE+)
* Lighting, ambient occlusion, shadows
* Glowstone and torches
* Swaying leaves
* Trees
* Slabs (merging currently not supported)
* Slime blocks with elastic collision (Custom physics system instead of Unity's built-in one)
* Explosive TNT (with particle effects)
* Gravity-affected sand and gravel
* Flowing water and lava (limited maximum spread, so one block of water won’t flood the world 😂)
* Block orientation (rotation)
* Day-night cycle
* URP rendering pipeline + full PBR materials ([Supports NVIDIA's Minecraft PBR texture pack](https://www.nvidia.cn/geforce/guides/minecraft-rtx-texturing-guide/))
* Post-processing effects
* Customizable resource packs
* Highly configurable block, biome, and item data
* Some ~~bugs~~ features

# Notice

* The texture pack used in this project was downloaded from [“Minecraft” Windows 10 RTX Beta: PBR Rendering Texture Q&A and Free Resource Pack Download (nvidia.cn)](https://www.nvidia.cn/geforce/news/minecraft-with-rtx-beta-your-pbr-questions-answered/), specifically `RTX Vanilla Conversion`, created by u/TheCivilHulk. It can also be downloaded from [TheCivilHulk/Minecraft-RTX-Vanilla-Conversion-and-Patches (github.com)](https://github.com/TheCivilHulk/Minecraft-RTX-Vanilla-Conversion-and-Patches).
* Updates will be sporadic!

# Guide

## How To Play In Unity?

1. Open the `SinglePlayer` scene
2. Click the Play button
3. Follow the on-screen key prompts to play

## Asset Management

In the editor, two asset loading modes are supported:

* Load from `AssetDatabase` (useful for development, no need to build `AssetBundle`, modifications take effect immediately)
* Load from `AssetBundle` files (useful for testing, requires building `AssetBundle`)

You can switch between these modes in Unity’s menu under `Minecraft-Unity/Assets/Load Mode`. Both modes share the same API, so the higher-level logic remains unaffected.

Example Code:

```c#
// Ensure that the scene has an AssetManagerUpdater component attached with proper settings, or manually initialize AssetManager.
using Minecraft.Assets; // Everything is in this namespace.

class Example : MonoBehaviour
{
    // Direct asset path, e.g., Assets/MyFolder/MyAsset.asset.
    public string AssetPath;

    // Asset reference (recommended).
    // This appears as a normal Object reference in the Inspector.
    // Internally stores the asset GUID, so it remains valid even if moved or renamed.
    public AssetPtr Asset;

    private IEnumerator Start()
    {
        // Fully asynchronous architecture.
        // No need to worry about which AssetBundle the asset belongs to.
        AsyncAsset asset = AssetManager.Instance.LoadAsset<GameObject>(Asset); // Can also use AssetPath.
        yield return asset; // Wait for asset to load.

        GameObject go = asset.GetAssetAs<GameObject>(); // Retrieve the object.

        // Do something.

        AssetManager.Instance.Unload(asset); // Unload asset (can also unload by name).
    }
}
```

This is just the basic usage. Check the source code for more APIs (since this isn’t the main focus here).

To build `AssetBundle`, click `Minecraft-Unity/Assets/Build AssetBundles` in Unity’s menu. It will automatically package assets for the current platform and place them in the designated path.

## Configuration

Block data, item data (not yet used), and biome data are configured in a custom editor.

Open it by clicking `Minecraft-Unity/MC Config Editor` in Unity’s menu. The interface looks like this:

![Editor - Blocks](EditorRecordings/mc_config_editor_block.png)

![Editor - Items](EditorRecordings/mc_config_editor_item.png)

![Editor - Biomes](EditorRecordings/mc_config_editor_biome.png)

### Tips

1. Use the dropdown menu on the left of the toolbar to switch between editing Blocks, Items, and Biomes.
2. Click the `+` or `-` buttons in the toolbar to add or remove data entries (Block / Item).
3. The right side of the toolbar includes: Search box, Save button, and Settings button.
4. The Settings button allows you to set the save path. Make sure to configure this before using it.
5. Pre-configured files are stored in `Assets/Minecraft Default PBR Resources/Tables/Configs`.
6. If you don’t want to start from scratch, set your path to the one in Tip #5 after opening the editor.
7. The saved files include many items; those prefixed with `[editor]` are only used in Unity Editor.
8. For details about the `[editor]` files, check the source code (not enough space here, Fermat’s Last Theorem vibes 😆).
9. If you have any questions, feel free to open an issue!

## Lua Codes

All Lua scripts are stored in `Assets/Minecraft Default PBR Resources/Lua Scripts`. `main.lua` serves as the entry point, and `cleanup.lua` handles resource cleanup. To modify these settings, find `Lua Manager` in the Hierarchy.

The `Minecraft.Lua` namespace provides empty interfaces like `ILuaCallCSharp`, `ICSharpCallLua`, and `IHotfixable`, which should be familiar to those using xlua. For example, implementing `ILuaCallCSharp` is equivalent to adding `XLua.LuaCallCSharpAttribute` to the class.

### Write Custom Block Behaviours

First, create a Lua file and set its AssetBundleName.

For example, defining sand:

```lua
require "block" -- Import block behavior API
local gravity = require "blocks.templates.gravity" -- Import gravity behavior template
sand = create_block_behaviour(gravity) -- Inherit behavior
-- Note: The variable 'sand' must be global and match the name in the editor config.
```

After writing it, don’t forget to `require` it in `main.lua`. The `block` module provides:

```lua
-- File path: Assets/Minecraft Default PBR Resources/Lua Scripts/block.lua

local behaviour = {} -- Default behavior

-- Called when block behavior is registered
function behaviour:init(world, block)
    self.world = world
    self.__block = block
    print("init block behaviour: " .. block.InternalName)
end
```

This allows for defining behaviors like classes. Templates `blocks.templates.gravity` and `blocks.templates.fluid` are available for gravity and fluid behaviors.

## Terrain Generation

Terrain generation config files are in `Assets/Minecraft Default PBR Resources/WorldGen`. Modify parameters or add generators to change terrain.

# End

If you have any questions, open an issue, and I'll respond when I can.

# References

* Various projects and resources listed.
* [TrueCraft](https://github.com/ddevault/TrueCraft)
* [xLua](https://github.com/Tencent/xLua)
* [MineCase](https://github.com/dotnetGame/MineCase)
* [MineClone-Unity](https://github.com/bodhid/MineClone-Unity)
* [MinecraftClone](https://github.com/Shedelbower/MinecraftClone)
* [Making a Minecraft Clone](https://www.shedelbower.dev/projects/minecraft_clone/)
* [Minecraft_Wiki](https://minecraft-zh.gamepedia.com/Minecraft_Wiki)
* [Super Hacker Moderator Xu's CSDN Blog (炒鸡嗨客协管徐的CSDN博客)](https://blog.csdn.net/xfgryujk)
