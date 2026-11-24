# ClipAngel Linux clone

An **unofficial, source-informed Linux reimplementation targeting ClipAngel 2.23 (https://github.com/tormozit/ClipAngel)**.

<img width="100%" alt="screenshot" src="https://github.com/user-attachments/assets/ef6d018f-02dd-426e-8b3a-c4391cac7421" />
<img width="100%" alt="screenshot-dark" src="https://github.com/user-attachments/assets/f9843797-4f1a-4f81-b87b-d15d07b9b534" />
<br><br>
This is an actual desktop application and Debian package, not a screenshot mock-up, web wrapper, or Wine launcher. It recreates the classic two-pane interface, original artwork, menu structure, clipboard history and many local workflows. **It is not a feature-complete or pixel-perfect reproduction of the Windows application.** Read `docs/FEATURE-PARITY.md` before migrating.

## Install

The supplied binary is for **amd64 / x86-64**. It targets **Linux Mint 21.2, Ubuntu 22.04/24.04 and Debian 12/13**, with a graphical **X11 session**. Those distribution targets are dependency/ABI targets, not a claim that clean installations on each distribution were tested.

```sh
sudo apt update
sudo apt install ./clipangel_2.23.0~linux4_amd64.deb
clipangel
```

Use `apt install ./...`, not just `dpkg -i`, so the distribution resolves dependencies. An enabled distribution repository and network access may be needed. Ubuntu provides the Pillow ImageTk dependency in its `universe` component. The package does not bundle Python, Tk, Pillow, X11 libraries or a browser engine.

Do not run the application as root. After installing, start **ClipAngel** from the application menu or run `clipangel` as your regular desktop user.

Default shortcuts: **Alt+V** opens history, **Alt+B** opens favorites, **Enter** pastes, **Ctrl+Enter** pastes plain text, **Ctrl+Shift+Enter** pastes as one line. Copy something in another application before opening history so the paste destination can be identified. Shortcut conflicts are reported in the status bar.

## Pillow dependency fix (preview 2)

Preview 1 required `python3-pil (>= 9.1)` and `python3-pil.imagetk (>= 9.1)`.
Linux Mint 21.2 uses the Ubuntu 22.04 package base, which provides Pillow
9.0.1, so those requirements could not be met
from that release's normal repositories. Preview 2 adds a resampling-API
compatibility layer and lowers both minimum versions to **9.0.1**. Python
**3.10 or newer** and the other dependencies are still required.

Install the new package rather than forcing the old package's dependencies.
Installing Pillow with pip does not satisfy the package manager's named
`python3-pil` dependency. Do not mix packages from different Debian/Ubuntu
releases to work around this issue.

On Ubuntu, the ImageTk package is in `universe`. If unavailable, enable the
normal Ubuntu component and retry (these commands are not for Debian):

```sh
sudo add-apt-repository universe
sudo apt update
sudo apt install ./clipangel_2.23.0~linux4_amd64.deb
```

For unresolved dependencies, collect:

```sh
cat /etc/os-release
dpkg --print-architecture
apt-cache policy python3 python3-pil python3-pil.imagetk
```

See `docs/DEPENDENCY-FIX.md` for the change and verification limits.

## Menu and installation fixes (preview 4)

This build fixes the disappearing Clip menu and context menus that remained
on screen after an outside click. Opening a long menu near a screen edge must
not accidentally invoke the entry that Tk moves under the cursor. Mouse,
keyboard, submenu and outside-click behavior are covered by XTest input tests.
The same opening-click protection applies to right-click menus. Preview/image
menus are reused, and hiding the window or rebuilding menus releases their grab.
A shutdown error with pending child-widget timers is also fixed.

`unrtf` is now **optional** (`Suggests`, not `Depends`). Without it, RTF clips
use the built-in plain-text preview; their original RTF payload is still stored,
copied and exported. Install `unrtf` only for approximate formatted RTF previews:

```sh
sudo apt install unrtf
```

Help → Diagnostics reports whether the converter is available. To upgrade an
unconfigured older package after a failed `dpkg -i`, use the new package:

```sh
clipangel --quit
sudo apt update
sudo apt --fix-broken install ./clipangel_2.23.0~linux4_amd64.deb
clipangel
```

No uninstall or data reset is needed. History, favorites, theme, paste preferences
and the user's optional login-startup entry are retained. This is not a fully
self-contained/offline installer; the other system dependencies are unchanged.
See `docs/LINUX4-RELEASE.md` and `docs/TEST-REPORT.md`.

## Retained from preview 3

Automatic paste uses Ctrl+Shift+V for GNOME Terminal and Konsole and Ctrl+V for
ordinary applications. Select **Settings → General → Paste keystroke → Auto**.
The earlier default migrates automatically; deliberate non-default overrides
are preserved.

Use **Window → Dark theme**, or **Settings → Appearance → Use dark theme**, to
switch without restarting. The preference persists and does not alter clipboard
payloads. Desktop title bars remain controlled by your window manager.

Login startup is optional. Enable **Window → Start at login (minimized)** or use:

```sh
clipangel --autostart on
clipangel --autostart status
# Disable later:
clipangel --autostart off
```

These commands run as your regular desktop user. They manage only ClipAngel's
per-user XDG startup entry. Startup is off by default; installing the package
never enables it or starts capture. Quit the old instance before upgrading;
existing clipboard history and preferences are retained.

The baseline label is corrected to 2.23 as requested; the supplied Windows
snapshot itself has not been replaced. See `docs/LINUX3-RELEASE.md` for the
assembly-metadata discrepancy, migration details and current test limitations.

## Desktop compatibility

Full clipboard monitoring, global shortcuts and focus-checked paste-back use X11, XFixes and XTest. An X11 login session is the supported target for this build. In a Wayland session the application can run through XWayland, but clipboard visibility and global shortcuts are not reliable across native Wayland applications. Automatic paste is deliberately disabled there. Native Wayland data-control and GlobalShortcuts portal integrations are **not implemented**. A desktop shortcut invoking `clipangel --show` can still open the window; paste manually.

An XEmbed system-tray implementation is included. Actual tray visibility depends on the desktop's tray support; the automated test environment did not have a tray host. The window remains recoverable from the taskbar when neither a working history shortcut nor a tray is available.

## Included functionality

- Persistent SQLite history for text, HTML, RTF, images and file references; favorites, used flags, retention and duplicate handling.
- Original icons and classic list/preview layout; Unicode search, mark/type/date filtering, optional columns and source metadata.
- Original-format/plain-text/one-line/file paste, automatic destination-aware or manually configured paste keystroke, multi-clip join, transformations, text editing, rename, comparison and undo deletion.
- Safe local HTML subset preview, optional formatted RTF conversion through `unrtf` with a built-in text fallback, image zoom, import/export, optional QR generation and browser translation with confirmation.
- Source-derived `.cac` XML import/export, read-only import of unencrypted original SQLite databases, and lossless Linux JSON export for retained MIME payloads.
- Configurable shortcuts, application exclusions, capture pause, explicit capture, optional per-user login startup, saved dark/light themes and a single-instance command line.

The local history, exports, backups and temporary files are **not encrypted**. Directories are created with mode 0700 and files with mode 0600. Password-manager exclusion hints are respected when present; this cannot guarantee that passwords will never be captured. Pause monitoring before copying sensitive data.

## Data and removal

Default settings: `~/.config/clipangel/settings.json`.

Default history: `~/.local/share/clipangel/history.sqlite3` (with SQLite WAL/SHM while running). XDG_CONFIG_HOME and XDG_DATA_HOME are respected. Imports create a backup under the data directory. Explicit Open/Save/Paste-file actions can create files under `temporary-files/`; these are retained and can be removed after closing applications that use them.

```sh
# Run with a completely separate settings/history directory:
clipangel --data-dir "$HOME/clipangel-test"

# Stop the running default instance:
clipangel --quit

# Remove the application; existing personal history is deliberately preserved:
sudo apt remove clipangel
```

To remove saved data, quit ClipAngel and explicitly delete its configuration/data directories and `~/.config/autostart/clipangel.desktop`, where applicable. Package removal never deletes personal clipboard history automatically.

## Build from source

```sh
sudo apt install build-essential libx11-dev libxfixes3 libxtst6 \
  python3 python3-tk python3-pil python3-pil.imagetk python3-qrcode \
  xdg-utils dpkg-dev
# Optional formatted RTF preview:
sudo apt install unrtf
make
python3 -m clipangel
make deb
```

The build does not fetch dependencies or require administrator rights. `make deb` writes a package into `dist/`. Build natively on another architecture; the distributed `.so` and `.deb` are amd64. **Run `make clean && make deb` on a different architecture**, so the included amd64 library is not reused. The C adapter uses stable XFixes/XTest ABI declarations and runtime libraries; extension development headers are not required.

## Test

```sh
sudo apt install xvfb openbox
xvfb-run -a -s '-screen 0 1280x900x24' sh -c \
  'openbox --config-file /etc/xdg/openbox/rc.xml >/tmp/clipangel-test-wm.log 2>&1 & wm=$!; sleep 1; python3 -m unittest discover -s tests -v; r=$?; kill "$wm"; exit "$r"'
```

Without DISPLAY, `make test` runs the core tests and skips X11/GUI tests. All tests use temporary, synthetic data. See `docs/TEST-REPORT.md` for the observed results and untested areas.

## Implementation and license

Python 3.10+, Tk/ttk, SQLite and Pillow, with a small C X11 adapter. No .NET or Windows binaries are required. Original artwork is copied from `Picture/`; menu translations are extracted from `Main*.resx`. Linux-specific dialogs remain English. Original source references and deliberate differences are listed in `docs/FEATURE-PARITY.md`.

GPL version 3. Original ClipAngel attribution: Tormozit and contributors. The original `LICENSE` is retained. This build is not endorsed by upstream. No original history database, remote-service credentials, signing keys or Windows assemblies are redistributed.
