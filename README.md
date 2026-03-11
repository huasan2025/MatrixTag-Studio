# DMLabelTool

Cross-platform industrial `Data Matrix` label generator for macOS and Windows.
Repository: https://github.com/huasan2025/DMLabelTool

This project supports:
- GUI mode for operations users
- CLI mode for batch automation
- Fixed-size industrial label rendering (`40x30 mm`, `300 DPI`)
- Data Matrix + 3-line code layout
- Safe output folder strategy (`PREFIX-YYYYMMDD`, auto suffix `(1)`, `(2)`...)

## Features

- Single-batch GUI workflow:
  - Line 1: prefix (manual input)
  - Line 2: middle code (default `4000`)
  - Line 3: start serial (default `0035`)
  - Quantity as total count
- CLI range workflow:
  - Separate ranges per prefix (`--ld-range`, `--rd-range`, `--fd-range`, `--bd-range`)
- Deterministic file naming:
  - Filename = full code string
  - No silent rename inside one batch
- Persistent GUI settings:
  - Output root and optional font path saved in user config file

## Project Structure

- `src/dm_label_tool/core.py`: label/DM generation and validation
- `src/dm_label_tool/gui.py`: Tkinter GUI
- `src/dm_label_tool/cli.py`: CLI parsing and execution
- `src/dm_label_tool/config.py`: user config persistence
- `dmlabeltool.py`: preferred local launcher
- `dm_label_generator.py`: backward-compatible legacy launcher
- `examples/`: reference images

## Requirements

- Python `3.10+`
- `Pillow`
- `pylibdmtx`
- Runtime `libdmtx` shared library

## Runtime Dependencies (Check First, Then Install)

`DMLabelTool` uses `pylibdmtx`, which requires the native `libdmtx` runtime.

### Windows: check before install

1. Open PowerShell in your `DMLabelTool.exe` folder.
2. Check whether `libdmtx` DLL is already available:

```powershell
Get-ChildItem .\*.dll, .\libs\*.dll -ErrorAction SilentlyContinue | Where-Object { $_.Name -match "dmtx" }
where.exe libdmtx.dll
where.exe libdmtx-64.dll
where.exe liblibdmtx-64.dll
```

If any command finds a valid DLL path, you may already be done.

3. Check whether VC++ runtime is already installed:

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\VisualStudio\12.0\VC\Runtimes\x64" -ErrorAction SilentlyContinue
```

If output contains `Installed : 1`, VC++ 2013 x64 runtime is already installed.

### Windows: install (only if missing)

1. Download `libdmtx` DLL:
- [barcode-reader-dlls Releases](https://github.com/NaturalHistoryMuseum/barcode-reader-dlls/releases)
- Download `liblibdmtx-64.dll` for 64-bit Windows.

2. Put DLL next to `DMLabelTool.exe` (recommended).
3. For compatibility, duplicate it to these names in the same folder:
- `libdmtx.dll`
- `libdmtx-64.dll`

4. If VC++ runtime is missing, install:
- [Microsoft Visual C++ 2013 Redistributable](https://www.microsoft.com/en-us/download/details.aspx?id=40784)
- Install `vcredist_x64.exe` on 64-bit Windows.

5. Restart `DMLabelTool.exe`.

### macOS: check before install

1. Open Terminal.
2. Check Homebrew and `libdmtx`:

```bash
brew --version
brew list libdmtx
```

If `brew list libdmtx` succeeds, `libdmtx` is already installed.

3. Optional runtime self-check:

```bash
python3 -c "from pylibdmtx.pylibdmtx import encode; print('pylibdmtx/libdmtx OK')"
```

### macOS: install (only if missing)

1. Install Homebrew (if not installed):
- [Homebrew official site](https://brew.sh/)

2. Install `libdmtx`:

```bash
brew install libdmtx
```

3. Install Python dependencies:

```bash
python3 -m pip install -r requirements.txt
```

4. Verify:

```bash
python3 -c "from pylibdmtx.pylibdmtx import encode; print('pylibdmtx/libdmtx OK')"
```

## Quick Start (GitHub Release Users)

### Windows (DMLabelTool.exe)

1. Download and unzip `DMLabelTool-windows.zip` from Releases.
2. In the extracted folder, complete the checks/installation in `Runtime Dependencies` above.
3. Double-click `DMLabelTool.exe`.
4. Smoke test:
- Prefix: `LD`
- Middle code: `4000`
- Start serial: `0035`
- Quantity: `1`

### macOS (DMLabelTool.app / local run)

1. Download and unzip release artifact from Releases.
2. Complete the checks/installation in `Runtime Dependencies` above.
3. If the app bundle is blocked by Gatekeeper, remove quarantine:

```bash
xattr -dr com.apple.quarantine /path/to/DMLabelTool.app
```

4. Launch app, or run local GUI:

```bash
python3 dmlabeltool.py --gui
```

## Quick Start

### 1) Install

```bash
cd /path/to/DMLabelTool
python3 -m pip install -r requirements.txt
```

### 2) Launch GUI

```bash
python3 dmlabeltool.py --gui
```

If you run `python3 dmlabeltool.py`, GUI is the default unless CLI flags are provided.

### 3) Run CLI

```bash
python3 dmlabeltool.py --cli \
  --ld-range 75-80 \
  --rd-range 100-120 \
  --middle-code 4000 \
  --output ./dist_output
```

## Packaging (PyInstaller)

Install build dependency:

```bash
python3 -m pip install pyinstaller
```

Build GUI app:

```bash
pyinstaller packaging/dm_label_tool_gui.spec
```

Build CLI executable:

```bash
pyinstaller --name dm-label-cli --onefile dmlabeltool.py
```

## GitHub Releases (MVP)

The workflow in `.github/workflows/release.yml` builds artifacts on:
- `macos-latest`
- `windows-latest`

Trigger by tag:

```bash
git tag v0.1.0
git push origin v0.1.0
```

## FAQ

### GUI does not start and shows dependency error

`pylibdmtx` could not load `libdmtx`. Install runtime dependencies first.

### Why new folder name has `(1)` suffix

To avoid mixing multiple runs from the same day.  
Example: `LD-20260311`, then `LD-20260311(1)`.

### Is quantity interpreted as range span or total count?

Total count.  
`start=0035`, `quantity=1000` produces `...0035` through `...1034`.

## License

MIT. See `LICENSE`.
