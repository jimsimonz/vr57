# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Unreal Engine 5.7 VR project (`vr57`) targeting multiple platforms with primary focus on Meta Quest devices (Quest 2, Quest Pro, Quest 3) and PC VR using OpenXR. The project uses the VR Template as its foundation and includes advanced OpenXR features through a custom plugin.

## Engine and Platform Configuration

- **Engine Version**: Unreal Engine 5.7
- **Primary VR Runtime**: OpenXR with hand tracking and eye tracking extensions
- **Target Platforms**: Windows (DX12/SM6), Android (Meta Quest), Linux (Vulkan SM6), and various console platforms
- **Build Settings**: BuildSettingsVersion.V6

## Key Commands

### Building the Project

```bash
# Generate Visual Studio project files (run after modifying .uproject or .Build.cs files)
# Right-click vr57.uproject in Windows Explorer -> "Generate Visual Studio project files"

# Build from Visual Studio
# Open vr57.sln and build the desired configuration (Development Editor, Development, Shipping, etc.)

# Build from command line (requires UE5 build tools in PATH)
# For editor builds:
"C:\Program Files\Epic Games\UE_5.7\Engine\Build\BatchFiles\Build.bat" vr57Editor Win64 Development "F:\UnrealProjects\vr57\vr57.uproject"

# For game builds:
"C:\Program Files\Epic Games\UE_5.7\Engine\Build\BatchFiles\Build.bat" vr57 Win64 Development "F:\UnrealProjects\vr57\vr57.uproject"
```

### Running the Editor

```bash
# Launch Unreal Editor (double-click vr57.uproject or use command line)
"C:\Program Files\Epic Games\UE_5.7\Engine\Binaries\Win64\UnrealEditor.exe" "F:\UnrealProjects\vr57\vr57.uproject"
```

### Packaging for Meta Quest

```bash
# Package from Editor: File -> Package Project -> Android (ASTC)
# Ensure "Package for Meta Quest" is enabled in Project Settings -> Android
```

## Project Architecture

### Module Structure

**Primary Module: `vr57`**
- Location: `Source/vr57/`
- Type: Runtime module
- Dependencies: Core, CoreUObject, Engine, InputCore
- Main module implementation in `vr57.cpp` using `FDefaultGameModuleImpl`

**Build Configuration**
- Main build rules: `Source/vr57/vr57.Build.cs`
- Game target: `Source/vr57.Target.cs` (Type: Game)
- Editor target: `Source/vr57Editor.Target.cs` (Type: Editor)
- PCH Usage: UseExplicitOrSharedPCHs

### Plugin Architecture

**OpenXRExtended Plugin** (`Plugins/OpenXRExtended/`)
- Custom plugin that exposes advanced OpenXR features not available in UE5's base OpenXR implementation
- Created by: Yannick Comte (Demonixis)
- Contains two modules:
  - `OpenXRExtended`: Core extended OpenXR functionality (AppSW, FFR)
  - `XRRefreshRate`: Refresh rate control for VR displays
- Dependencies: Requires OpenXR plugin to be enabled

When modifying OpenXR functionality:
1. Check if the feature belongs in the OpenXRExtended plugin or base project code
2. Advanced OpenXR extensions (AppSW, FFR, refresh rate) should go in the plugin
3. Game-specific VR logic should go in the main vr57 module

### Content Organization

- `Content/VRTemplate/`: Base VR template maps and blueprints
- `Content/XRFramework/`: Custom XR framework with BP_XRGameMode as the global default game mode
- `Content/XRMannequins/`: VR character/mannequin assets
- `Content/Weapons/`: Weapon interaction system
- `Content/VRSpectator/`: Spectator mode functionality
- `Content/LevelPrototyping/`: Level design prototyping assets

**Important**: The default game mode is set to `/Game/XRFramework/Blueprints/BP_XRGameMode.BP_XRGameMode_C` in Config/DefaultEngine.ini

### Rendering Configuration

The project uses forward rendering optimized for VR:
- Forward shading enabled
- Mobile multi-view and instanced stereo enabled for performance
- Virtual shadow maps enabled
- Lumen for dynamic GI (r.DynamicGlobalIlluminationMethod=1)
- Ray tracing enabled for high-end platforms
- Auto exposure, motion blur, and AO disabled for VR comfort
- TAA (Anti-aliasing method 3) for quality

### OpenXR Configuration

Active OpenXR features (vr57.uproject):
- OpenXR base runtime
- OpenXREyeTracker
- OpenXRHandTracking

OpenXR settings (Config/DefaultEngine.ini):
- Inverted alpha for correct rendering (xr.OpenXRInvertAlpha=True)
- Fixed foveated rendering enabled (bIsFBFoveationEnabled=True)

### Android/Meta Quest Configuration

Key settings in Config/DefaultEngine.ini:
- Minimum SDK: 32, Target SDK: 32
- Supported devices: Quest 2, Quest Pro, Quest 3
- bPackageForMetaQuest=True
- No ES3.1 builds (only Vulkan mobile)
- Mobile anti-aliasing: TemporalAA

## Adding New C++ Classes

When adding new C++ classes to the project:

1. Create header (.h) and implementation (.cpp) files in `Source/vr57/`
2. Ensure proper module API macro usage: `VR57_API` for public classes
3. Include appropriate UE headers (`CoreMinimal.h` at minimum)
4. Rebuild the project to compile new classes
5. If adding new module dependencies, update `vr57.Build.cs`:
   - Add to `PublicDependencyModuleNames` for public dependencies
   - Add to `PrivateDependencyModuleNames` for private dependencies

## Custom Collision Channel

The project defines a custom trace channel:
- **3DWidget** (ECC_GameTraceChannel1): Used for 3D UI widget interaction in VR, defaulting to Ignore

When adding VR UI elements, ensure they're set to respond to the 3DWidget channel.

## Development Notes

### VR-Specific Considerations

- Always test with VR headset when modifying interaction, rendering, or performance-related code
- The project supports client-side navigation (bAllowClientSideNavigation=True)
- Alpha propagation is enabled for proper transparency in mobile VR

### Plugin Development

If modifying the OpenXRExtended plugin:
- Plugin source: `Plugins/OpenXRExtended/Source/`
- After changes, regenerate project files and rebuild both modules
- The plugin has its own module dependencies defined in its .uplugin file

### Platform-Specific Paths

Windows builds use DX12/SM6, Linux uses Vulkan SM6. When writing shader or rendering code, ensure compatibility across these RHIs.
