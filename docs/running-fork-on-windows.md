# Running a Horizon fork on Windows (agent guide)

This page is for agents and developers who want to run **a fork** of Horizon on Windows, including forks that changed launcher/account behavior.

## What you need

From the main project requirements (`README.md` / launcher docs):

- **Windows 10 or 11** (x64), or **Windows 11 on Arm** (ARM64).
- A GPU with **Vulkan or DirectX 12** (as required by the launcher/release notes).
- **Your own copy of Minecraft: Java Edition 26.3** (the version Horizon pins) to prepare game resources/assets. Horizon does **not** distribute Mojang game files.

## Option 1: Use a fork-built release (recommended when available)

If the fork has CI/release artifacts:

1. Open the fork on GitHub → **Actions** (`launcher`/release workflows) or **Releases**.
2. Download the Windows artifact:
   - x64: `horizon-launcher-windows-x64.zip` or `horizon-launcher-setup.exe`
   - ARM64: `horizon-launcher-windows-arm64.zip` or `horizon-launcher-setup-arm64.exe`
3. Extract/run the launcher.
4. Prepare resources from your local Minecraft install as the launcher/project requires (`horizon-prepare` flow). The launcher needs prepared resources in the expected Horizon data location.

## Option 2: Build the launcher from source on Windows

Prerequisites (typical):

- **Rust toolchain** (stable/new enough for the workspace; install via rustup).
- **Visual Studio Build Tools / MSVC** with C++ build tools (required for `x86_64-pc-windows-msvc` / `aarch64-pc-windows-msvc`).
- **Git**.
- Project prerequisites as documented in `README.md`/`CONTRIBUTING.md` if you also need full game resource preparation.

Build:

```powershell
# From the repo root
cd launcher
cargo build --release
```

Or target explicitly:

```powershell
cargo build --release --target x86_64-pc-windows-msvc
# ARM64 (on ARM64 host/toolchain):
cargo build --release --target aarch64-pc-windows-msvc
```

The binary is produced under `launcher\target\<target>\release\horizon-launcher.exe` (or the default target folder).

## Resource preparation (still required)

Running the native client still needs **prepared resources** built from your own Minecraft 26.3 install (checked against `research-lock.json`). Removing an account/ownership gate does **not** provide game assets.

Follow the repo’s preparation instructions (`README.md`, `docs/prepare-performance.md`, `horizon-prepare` docs) and ensure the prepared output is available where Horizon expects it.

## Forks that make Microsoft sign-in optional

If your fork removes the launcher’s Microsoft-account/Minecraft-ownership gate (as in this change set):

- **Playing/starting the native client offline** may work without signing in, once resources are prepared.
- **Microsoft sign-in can remain optional** for flows that don’t require it.
- **Mojang downloads / entitlement-gated download paths still require an owning Microsoft/Minecraft account** (do not treat offline play as authorization to download game files you don’t own).
- **Online-mode servers** still require proper authentication.

## Safety / isolation (for automated agents)

When testing, avoid writing to real user data. Use isolated directories/environment variables where supported (see `AGENTS.md`), e.g. set launcher/game homes to temp directories and disable self-update where applicable:

```powershell
$env:HORIZON_LAUNCHER_HOME = "$env:TEMP\horizon-launcher-test"
$env:HORIZON_HOME = "$env:TEMP\horizon-game-test"
$env:HORIZON_LAUNCHER_NO_SELF_UPDATE = "1"
```

Adjust to actual supported variables in the fork; verify names in code/docs.

## Troubleshooting

- **Missing GPU/API support**: ensure Vulkan/DX12-capable drivers are installed.
- **Missing prepared resources**: run the preparation flow from your own Minecraft install.
- **MSVC/linker errors**: install/update Visual Studio C++ build tools and Windows SDK.
- **Download/entitlement failures**: expected if not signed in with an owning account on paths that still require Mojang entitlement checks.

## Relevant docs

- [`README.md`](../README.md)
- [`launcher/README.md`](../launcher/README.md)
- [`docs/microsoft-account.md`](microsoft-account.md)
- [`docs/launcher-commands.md`](launcher-commands.md)
- [`AGENTS.md`](../AGENTS.md)
