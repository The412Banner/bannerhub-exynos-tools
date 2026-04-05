ExynosTools v1.3.0 (Stable) – Advanced Vulkan Wrapper for Xclipse (Exynos 2400+)
ExynosTools provides a production-grade Vulkan wrapper that intercepts key functions, announces safe extensions, includes full BC4/BC5 emulation via compute shader (BC6H/BC7 ready), enhanced Xclipse auto-detection, an on-screen HUD, and a unified profile system.

💡 Compatible with Winlator Bionic, DXVK 1.10.x/2.x (depending on compatibility), VKD3D-Proton, and Zink.

✅ New in v1.3.0 (Stable)
- **Full BCn emulation**: BC4/BC5 fully functional with embedded SPIR-V shaders; BC6H/BC7 prepared for future versions.
- **On-screen HUD**: Enable with `EXYNOSTOOLS_HUD=1` to show FPS and real-time stats.
- **Unified profile system**: Migrated from `.env` to `.conf` with consistent syntax and better organization.
- **Improved GPU detection**: Updated device IDs for Xclipse 920, 940, and future variants.
- **Robust error handling**: Complete validation and Vulkan resource cleanup.
- **Improved build system**: Automatic compilation of GLSL shaders to SPIR-V and embedding as C headers.

🔧 Requirements
- C compiler, CMake >= 3.15, `tar` with zstd support.
- `glslc` (Shaderc) for compiling GLSL shaders to SPIR-V.
- `xxd` for embedding shaders as C headers.

🚀 Build & Package (CMake)
1) Linux (host x86_64 with NDK r25b for Android arm64):
```
bash scripts/build_and_package.sh
```
The artifact will be at `artifacts/exynostools-android-arm64.tar.zst` with the structure required by Winlator.

Alternative — Meson + Ninja (Android cross-compile):
```
meson setup build-android --cross-file=android/arm64.txt -Dbuildtype=release
ninja -C build-android
```

📦 Installation in Winlator Bionic 10.1+
- Copy `exynostools-android-arm64.tar.zst` to:
  `/storage/emulated/0/Android/data/com.winlator/files/drivers/`
- In Winlator, open your container and select the driver if applicable. Winlator automatically picks up libraries in `usr/lib`.

📦 BannerHub / GameHub — File Locations

When installed as a BannerHub component, the archive extracts into GameHub's private storage.
`{filesDir}` = `/data/data/com.xj.gamehub/files/`

| File | Installed path |
|---|---|
| `libxeno_wrapper.so` | `{filesDir}/xj_install/component/exynostools/<version>/usr/lib/libxeno_wrapper.so` |
| `performance_mode.conf` | `{filesDir}/xj_install/component/exynostools/<version>/etc/exynostools/performance_mode.conf` |
| `meta.json` | `{filesDir}/xj_install/component/exynostools/<version>/usr/share/meta.json` |
| `profiles/winlator/*.env` | Not used in BannerHub (Winlator-specific config, ignored) |

GameHub's `WineComponentInstaller` extracts the `.tar.zst` directly into `xj_install/component/<name>/<version>/` — the archive layout maps 1:1 to the install path.

**Injection at launch:** BannerHub injects the `.so` via `LD_PRELOAD` into Wine's environment before startup:
```
LD_PRELOAD=.../xj_install/component/exynostools/<version>/usr/lib/libxeno_wrapper.so
```

**Note:** `performance_mode.conf` contains a hardcoded Winlator path for `shader_cache_dir`. When running under BannerHub this should point to `{filesDir}/xj_winemu/` instead.

ℹ️ Technical Notes
- **Interception**: `vkGetInstanceProcAddr`, `vkGetDeviceProcAddr`, `vkCreateInstance`, `vkEnumeratePhysicalDevices`, `vkGetPhysicalDeviceProperties`, `vkGetPhysicalDeviceFeatures2`, `vkEnumerateDeviceExtensionProperties`, `vkCreateDevice`, `vkCreateSwapchainKHR`, `vkQueuePresentKHR`.
- **Extension patching**: Virtually adds `VK_EXT_descriptor_indexing`, `VK_EXT_robustness2`, `VK_KHR_shader_float16_int8`, `VK_KHR_dynamic_rendering` when safe; patches `vkGetPhysicalDeviceFeatures2`.
- **BCn emulation**: BC4/BC5 fully functional (compute GLSL→SPIR-V embedded); BC6H/BC7 prepared.
- **GPU detection**: Improved Xclipse heuristic (Samsung vendorID + known name/deviceID), with `EXYNOSTOOLS_FORCE` and `EXYNOSTOOLS_WHITELIST` overrides.
- **Config**: reads `etc/exynostools/performance_mode.conf` for global performance settings.
- **Per-app profiles**: unified `.conf` system in `etc/exynostools/profiles/` or via `EXYNOSTOOLS_APP_PROFILE`.
- **HUD**: enable with `EXYNOSTOOLS_HUD=1` for FPS overlay.
- **FPS logging**: enable with `EXYNOSTOOLS_LOG_FPS=1` for performance logs.

🧩 Xclipse Status
- Many Xclipse devices lack native BC4+ support; compute emulation improves compatibility (BC4/BC5 already present, BC6H/BC7 to follow).
- `supportsDynamicRendering` may require emulation for DXVK 2+. Community-linked public references on known issues.
