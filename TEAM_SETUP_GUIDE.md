# MATLAB Agentic Toolkit — Team Setup & Demo Guide

This guide walks your team through installing and using the MATLAB Agentic Toolkit with Claude Code on Windows. Follow the steps in order. By the end, Claude Code will have a live connection to MATLAB and expert MATLAB skills built in.

---

## What Is This?

The MATLAB Agentic Toolkit connects Claude Code (AI coding agent) to MATLAB through the Model Context Protocol (MCP). This means Claude can:

- **Run MATLAB code** and return results
- **Execute tests** and report structured results
- **Perform static analysis** on MATLAB files
- **Detect installed toolboxes** and versions
- **Open MATLAB apps** like Filter Analyzer, Region Analyzer, Image Segmenter
- **Write idiomatic MATLAB code** using built-in expert skills (signal processing, image processing, app building, and more)

**Architecture:**
```
Claude Code (CLI)  <-->  MCP Server (matlab-mcp-core-server.exe)  <-->  MATLAB R20xx
```

---

## Prerequisites

Before starting, make sure you have:

- [ ] **MATLAB R2020b or later** installed (guide tested with R2026a)
- [ ] **Windows 10/11** (x64)
- [ ] **Internet connection** (for downloads)
- [ ] **Anthropic account** (free or Pro at claude.ai)

---

## Step 1 — Install Node.js

Claude Code CLI requires Node.js.

1. Go to **https://nodejs.org**
2. Click **"Windows Installer (.msi)"** — use the LTS version
3. Run the installer, keep all defaults (make sure **"Add to PATH"** is checked)
4. On the **"Tools for Native Modules"** screen — **leave the checkbox unchecked**
5. Complete the installation
6. **Close and reopen PowerShell**, then verify:

```powershell
node --version
npm --version
```

Both should print version numbers (e.g. `v24.15.0` and `11.12.1`).

---

## Step 2 — Install Claude Code CLI

```powershell
npm install -g @anthropic-ai/claude-code
```

Verify:
```powershell
claude --version
```

Expected output: `2.1.143 (Claude Code)` or newer.

---

## Step 3 — Download the MATLAB MCP Server Binary

1. Go to: **https://github.com/matlab/matlab-mcp-core-server/releases/latest**
2. Download: **`matlab-mcp-core-server-win64.exe`**
3. In PowerShell, install it:

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.matlab\agentic-toolkits\bin"
Move-Item "$env:USERPROFILE\Downloads\matlab-mcp-core-server-win64.exe" "$env:USERPROFILE\.matlab\agentic-toolkits\bin\matlab-mcp-core-server.exe"
Unblock-File "$env:USERPROFILE\.matlab\agentic-toolkits\bin\matlab-mcp-core-server.exe"
```

Verify it runs:
```powershell
& "$env:USERPROFILE\.matlab\agentic-toolkits\bin\matlab-mcp-core-server.exe" --version
```

Expected output: `github.com/matlab/matlab-mcp-core-server v0.9.1` or newer.

---

## Step 4 — Connect the MCP Server to Claude Code

This writes the MATLAB MCP server configuration to Claude Code's settings file. Paste this entire block into PowerShell — replace `<YourUsername>` with your Windows username (e.g. `KantikaWongkasem`):

```powershell
$p = "$env:USERPROFILE\.claude\settings.json"
New-Item -ItemType Directory -Force -Path (Split-Path $p) | Out-Null
$s = if (Test-Path $p) { Get-Content $p -Raw | ConvertFrom-Json } else { [PSCustomObject]@{} }
if (!$s.PSObject.Properties['mcpServers']) { $s | Add-Member mcpServers ([PSCustomObject]@{}) }
$s.mcpServers | Add-Member matlab ([PSCustomObject]@{
    command = "C:\Users\<YourUsername>\.matlab\agentic-toolkits\bin\matlab-mcp-core-server.exe"
    args = [object[]]@("--matlab-root","C:\Program Files\MATLAB\R2026a","--matlab-display-mode","desktop")
}) -Force
$s | ConvertTo-Json -Depth 10 | Set-Content $p -Encoding UTF8
Write-Host "Done!"
Get-Content $p
```

> **Note:** If your MATLAB is not in `C:\Program Files\MATLAB\R2026a`, change that path to match your installation (e.g. `R2024b`, `R2025a`).

The output should show a JSON block containing `"matlab"` under `"mcpServers"`.

---

## Step 5 — Install MATLAB Skills as Claude Code Plugins

These skills give Claude expert MATLAB knowledge — it learns how to write idiomatic code, run tests, review code, design filters, process images, and more.

```powershell
claude plugin marketplace add "https://github.com/matlab/matlab-agentic-toolkit"
claude plugin install matlab-core@matlab-agentic-toolkit
claude plugin install toolkit@matlab-agentic-toolkit
claude plugin install signal-processing@matlab-agentic-toolkit
claude plugin install matlab-app-building@matlab-agentic-toolkit
claude plugin install matlab-software-development@matlab-agentic-toolkit
claude plugin install image-processing-and-computer-vision@matlab-agentic-toolkit
claude plugin install reporting-and-database-access@matlab-agentic-toolkit
claude plugin install wireless-communications@matlab-agentic-toolkit
claude plugin install rf-and-mixed-signal@matlab-agentic-toolkit
```

Verify all are installed:
```powershell
claude plugin list
```

You should see 9 plugins, all with status `✓ enabled`.

---

## Step 6 — First Launch & Login

```powershell
claude
```

On first launch, a browser will open for authentication:
1. Sign in with your Anthropic account (claude.ai)
2. Authorize Claude Code
3. The terminal will show: **"Welcome back [name]!"**

---

## Step 7 — Verify the MATLAB Connection

Once inside Claude Code, type:

```
What version of MATLAB is running? List the installed toolboxes.
```

Claude will:
1. Load the `matlab-list-products` skill automatically
2. Detect your MATLAB installation
3. Call `detect_matlab_toolboxes` via MCP
4. Report your MATLAB version and all installed toolboxes

This confirms the full pipeline is working.

---

## Available Skills Reference

| Plugin | Skills Included | What Claude Can Do |
|--------|----------------|-------------------|
| `matlab-core` | Debugging, Testing, Code Review, Live Scripts, List Products, Install Products | Write, test, and review any MATLAB code |
| `toolkit` | Setup & Management | Re-run setup, upgrade MCP server |
| `signal-processing` | Digital Filter Design | Design FIR/IIR filters, open Filter Analyzer |
| `matlab-app-building` | App Designer | Build MATLAB GUI apps with uifigure |
| `matlab-software-development` | Code Modernization | Update deprecated functions, packaging |
| `image-processing-and-computer-vision` | Display Image, Region Analysis | Image segmentation, open Region Analyzer |
| `reporting-and-database-access` | Database Read/Write, DuckDB | Connect to databases, generate reports |
| `wireless-communications` | AWGN, 5G/LTE/WLAN | Simulate wireless channels |
| `rf-and-mixed-signal` | SerDes Modeling | Model RF and mixed-signal systems |

---

## Demo Scripts for Customer Presentations

### Demo 1 — Verify Live MATLAB Connection
```
What version of MATLAB is running? List the installed toolboxes.
```
**Shows:** Live MCP connection, real MATLAB version and toolbox list.

---

### Demo 2 — Signal Processing + Filter Analyzer
```
Design a lowpass FIR filter with 1kHz cutoff at 44.1kHz sample rate,
60dB stopband attenuation. Open the result in Filter Analyzer.
```
**Shows:** Claude writes `designfilt` code, runs it in MATLAB, opens Filter Analyzer with magnitude/phase/group delay plots.

---

### Demo 3 — Image Processing + Region Analyzer
```
Load MATLAB's built-in coins.png image, segment the coins using
thresholding, label each region, and open the result in Region Analyzer.
Show me the area and centroid of each coin.
```
**Shows:** Claude uses Image Processing Toolbox, opens interactive Region Analyzer app.

---

### Demo 4 — Write Code + Generate Tests + Run Tests
```
Write a MATLAB function that computes the RMS value of a signal,
then generate unit tests for it and run them.
```
**Shows:** Full TDD workflow — function authoring, test generation with `matlab.unittest`, test execution and results.

---

### Demo 5 — Code Review
```
Review this MATLAB code for quality issues and suggest improvements:
[paste any .m file content here]
```
**Shows:** Static analysis via `check_matlab_code`, expert feedback on MATLAB best practices.

---

### Demo 6 — Create a Live Script
```
Create a Live Script demonstrating FFT analysis on a sample audio signal
with explanatory text, plots, and results in each section.
```
**Shows:** Generates a `.mlx`-compatible script with formatted sections, opens in MATLAB Live Editor.

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `claude` not recognized | Claude Code not installed or not in PATH | Re-run `npm install -g @anthropic-ai/claude-code`, restart PowerShell |
| `node` not recognized | Node.js not installed or PATH not set | Reinstall Node.js, restart PowerShell |
| MCP server not connecting | Wrong MATLAB root path in settings.json | Re-run Step 4 with correct path |
| Skills not loading | Plugins not installed | Re-run Step 5 plugin installs |
| MATLAB GUI apps don't open in batch mode | `-batch` flag blocks GUI | Claude will automatically retry with `-r` flag or PowerShell `Start-Process` |
| `Invalid configuration` on `claude mcp add-json` | PowerShell JSON escaping | Use the PowerShell block in Step 4 instead |
| Filter Analyzer / Region Analyzer shows error | MATLAB not fully started | Wait a few seconds and ask Claude to try again |

---

## Updating the Toolkit

To get the latest skills and MCP server:

**Update skills** (re-add marketplace to refresh cache):
```powershell
claude plugin marketplace add "https://github.com/matlab/matlab-agentic-toolkit"
```

**Update MCP server binary:**
1. Download the latest `matlab-mcp-core-server-win64.exe` from GitHub releases
2. Replace the existing file at `%USERPROFILE%\.matlab\agentic-toolkits\bin\matlab-mcp-core-server.exe`
3. Run `Unblock-File` on the new binary

---

## Quick Reference Card

| Task | Command |
|------|---------|
| Start Claude Code | `claude` (in any folder) |
| List installed plugins | `claude plugin list` |
| Check MCP servers | `claude mcp list` |
| List MATLAB toolboxes | Ask: *"List my MATLAB toolboxes"* |
| Design a filter | Ask: *"Design a [type] filter with [specs]"* |
| Process an image | Ask: *"Load [image] and [task]"* |
| Write + run tests | Ask: *"Write unit tests for [function] and run them"* |
| Review code | Ask: *"Review this MATLAB code: [paste code]"* |

---

## Resources

- MATLAB Agentic Toolkit: https://github.com/matlab/matlab-agentic-toolkit
- MATLAB MCP Core Server: https://github.com/matlab/matlab-mcp-core-server
- Claude Code documentation: https://docs.anthropic.com/claude-code
- Report issues: https://github.com/matlab/matlab-agentic-toolkit/issues

---

*Guide prepared by the MATLAB Agentic Toolkit setup session — May 2026.*

*MATLAB and Simulink are registered trademarks of The MathWorks, Inc.*
