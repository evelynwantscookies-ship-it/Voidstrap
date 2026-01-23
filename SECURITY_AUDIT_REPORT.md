# Security Audit Report - Voidstrap

**Date:** 2026-01-23
**Auditor:** Claude Code (Automated Security Scan)
**Repository:** Voidstrap (Roblox Launcher Fork)

---

## Executive Summary

This security audit analyzed the Voidstrap codebase for potentially malicious code patterns including backdoors, data exfiltration, credential theft, cryptocurrency mining, and other malware indicators.

**Overall Assessment: MEDIUM RISK** - The codebase appears to be a legitimate Roblox launcher application with no obvious malware. However, several **security concerns** were identified that users should be aware of.

---

## Critical Findings

### 1. ARBITRARY CODE EXECUTION via LuaScriptManager (HIGH RISK)

**Location:** `Bloxstrap/LuaScriptManager.cs:140-198`

The application includes a Lua scripting engine that can:
- Execute arbitrary Lua scripts from `autoexecute.lua`
- **Load and execute arbitrary DLLs** via the `load()` function
- Register arbitrary DLL functions into the Lua environment via `loadfunc()`

```csharp
// Line 161: Loads ANY DLL from the Voidstrap directory
Assembly assembly = Assembly.LoadFrom(dllPath);
// Line 188: Executes the Main() method
mainMethod.Invoke(null, ...);
```

**Risk:** If an attacker can place a malicious DLL in the Voidstrap directory, they can achieve full code execution on the user's system when Lua scripting is enabled.

**Mitigation:** This feature is opt-in (`EnableLuaScripting` setting), but users should be warned about the risks.

---

### 2. SUSPICIOUS EXTERNAL DOWNLOAD URL (MEDIUM RISK)

**Location:** `Bloxstrap/Utility/DarkTexturesMod.cs:12`

```csharp
private static readonly string DownloadUrl = "https://cocajola.com/wp-content/uploads/2024/09/dark-textures-rivals.zip";
```

**Risk:** Downloads content from a third-party domain (`cocajola.com`) that is not associated with Roblox or the Voidstrap project. This could potentially be used to deliver malicious payloads.

**Recommendation:** Verify the legitimacy of this URL or host content on official project infrastructure.

---

### 3. RUNTIME C# CODE COMPILATION (MEDIUM RISK)

**Location:** `Bloxstrap/UI/ViewModels/Settings/PluginsViewModel.cs:388-424`

The plugins system (when `ENABLE_ROSLYN` is defined) can compile and execute arbitrary C# code at runtime:

```csharp
// Line 416: Loads compiled assembly from memory
return Assembly.Load(ms.ToArray());
```

**Risk:** While behind a compile flag, this feature allows arbitrary code execution through user-provided plugins.

---

### 4. MULTIPLE GITHUB RAW CONTENT SOURCES (LOW-MEDIUM RISK)

**Locations:** Multiple files

The application fetches configuration and data from various GitHub raw URLs:
- `raw.githubusercontent.com/KloBraticc/` - Skybox assets, news
- `raw.githubusercontent.com/SCR00M/` - Channel lists, FFlags
- `raw.githubusercontent.com/MaximumADHD/` - FFlag tracking
- `raw.githubusercontent.com/LeventGameing/` - Allowlists
- `raw.githubusercontent.com/DynamicFastFlag/` - FVariables

**Risk:** If any of these GitHub accounts are compromised, malicious content could be delivered to users.

---

## Security Positive Findings

### No Evidence of:
- **Keylogging** - No keyboard hook APIs or keystroke capture
- **Credential Theft** - No password/token collection code
- **Cryptocurrency Mining** - No mining-related code patterns
- **Data Exfiltration** - No unauthorized uploads of user data
- **Registry Persistence** - No RunOnce/Startup registry keys for auto-start
- **Hidden Processes** - Process window hiding is only for legitimate installer operations
- **Obfuscated Malware** - No suspicious base64 encoded payloads or obfuscated strings

### Legitimate Features:
- **Activity Tracking** - Reads local Roblox logs for Discord Rich Presence (user-controllable)
- **Telemetry Disabling** - Provides options to block Roblox telemetry (privacy feature)
- **Registry Operations** - Standard uninstall registry entries only
- **Update Mechanism** - Downloads from official GitHub releases only
- **HTTP Communications** - All external connections use HTTPS to legitimate services

---

## Network Endpoints Analysis

### Trusted Endpoints:
| Domain | Purpose |
|--------|---------|
| `api.github.com` | Version checking, updates |
| `raw.githubusercontent.com` | Configuration files |
| `setup.rbxcdn.com` | Official Roblox CDN |
| `clientsettings.roblox.com` | Roblox settings |
| `tr.rbxcdn.com` | Roblox thumbnails |
| `discord.com` | Rich Presence (optional) |

### Questionable Endpoints:
| Domain | Purpose | Concern |
|--------|---------|---------|
| `cocajola.com` | Dark textures mod | Unverified third-party |

---

## Default Privacy Settings

The following features are **enabled by default**:
- Activity Tracking
- Analytics
- Discord Rich Presence
- Background Updates

Users should review settings if they prefer maximum privacy.

---

## Recommendations

1. **For Users:**
   - Do not enable Lua scripting unless you trust all files in your Voidstrap directory
   - Review and disable activity tracking if not needed
   - Be cautious when using the "dark textures" mod feature

2. **For Developers:**
   - Consider removing or sandboxing the DLL loading capability
   - Host mod assets on official infrastructure
   - Implement signature verification for downloaded content
   - Add security warnings for dangerous features

---

## Files Analyzed

- Total C# source files: 325+
- Total XAML files: 208
- Key security-relevant files reviewed:
  - `LuaScriptManager.cs`
  - `PluginsViewModel.cs`
  - `GithubUpdater.cs`
  - `DarkTexturesMod.cs`
  - `ActivityWatcher.cs`
  - `Bootstrapper.cs`
  - `Installer.cs`

---

## Conclusion

Voidstrap appears to be a **legitimate application** with no obvious malware. The identified security concerns are primarily related to **powerful features** (Lua/DLL execution, plugins) that could be abused if an attacker gains write access to the installation directory. Users should be aware of these capabilities and exercise caution.

**This report does not constitute a complete security audit.** A thorough manual review and dynamic analysis would be required for full assurance.
