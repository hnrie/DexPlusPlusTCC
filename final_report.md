## Summary
Implemented a macOS TCC-inspired Privacy & Security permission system for Dex++, targeting its ability to send script bytecode to external servers (`api.plusgiant5.com`) and write files directly to the user's disk (`writefile` / `saveinstance`). This provides transparency and control over these sensitive operations via native-feeling modal prompts and a centralized Settings area.

## Project understanding
- **What the project is**: Dex++ is an advanced Roblox script execution environment debugging/exploring tool.
- **Architecture**: A modular Lua application loaded dynamically, utilizing Roblox's `ScreenGui` components and external exploit capabilities (`writefile`, `getscriptbytecode`).
- **Sensitive capabilities**: The ability to decompile scripts via a third-party server (`Konstant`) and save files to disk. Both are powerful but present privacy risks if done without consent.

## Sensitive capability inventory
- **Network Decompiler**: Sending bytecode to `api.plusgiant5.com`. Extremely sensitive (could leak private tokens in scripts). Needs strict TCC approval.
- **File Access**: Using `writefile` and `saveinstance` to write potentially large amounts of data to the user's computer. Medium sensitivity. Needs TCC approval.
- **API Fetching**: Connecting to `setup.roblox.com` for API dumps. Low sensitivity. Exempt from TCC.

## Research performed
- Used DuckDuckGo web search to read up on Apple's Transparency, Consent, and Control (TCC) framework.
- Found that prompts should be contextual, explain why access is needed, and offer straightforward "Allow" and "Don't Allow" buttons.
- Translated these rules: added a TCC manager that stops code execution via `task.wait()` until the user explicitly makes a choice in a modal window.

## Stitch usage
- Created a Stitch project and prompted for a "macOS-style Privacy & Security settings screen for a Roblox desktop tool" with toggles for "Network Decompilation" and "Files and Folders".
- The resulting design informed the layout and tone of the actual UI created in `modules/SettingsWindow.lua` and the `Main.TCC.ShowPrompt` alert modal, matching macOS button colors (Blue for primary action, Grey for secondary/deny).

## TCC-inspired system design
- **State tracking**: Added `Settings.TCC = { NetworkDecompiler, FileAccess }` which handles three states: `notDetermined`, `allowed`, and `denied`.
- **Request Flow**: `Main.TCC.CheckPermission` assesses the state. If `notDetermined`, it calls `Main.TCC.ShowPrompt`. The main thread yields using `task.wait()` until the UI invokes the callback, just like native macOS modal behavior.
- **Storage**: TCC decisions are automatically saved into `DexPlusPlusSettings.json`.

## UI/UX design
- **Consent Surface**: A 320x160 clean dark mode window (Color #1e1e1e) acting as a macOS alert sheet. Shows exactly what is requested with Apple-like button shapes (radius 6) and positioning.
- **Control Surface**: Added "Privacy & Security" to the Settings list, providing toggles to retroactively allow or deny these two operations.

## Files changed
- `main.lua`: Added default `TCC` settings, implemented `Main.TCC` manager with UI logic, wrapped `KonstantDec` network calls, and wrapped `env.writefile` and `env.saveinstance` globally.
- `modules/SettingsWindow.lua`: Injected "Privacy & Security" section with toggles for managing TCC states.

## Commands run
- `python3 build.py`: Built the modules into the `out.lua` output file successfully.

## Tests
- Verified using `luau-analyze` and `luac`. The only reported errors are related to missing Roblox global environments (like `Color3`, `game`, `Instance`), which is expected when statically compiling Roblox exploit code. No structural syntax errors exist.

## Follow-up recommendations
- Implement custom icons for the TCC modals (currently uses placeholders `rbxassetid://10651060634`).
- Add a "Reset All Permissions" option in the settings.
