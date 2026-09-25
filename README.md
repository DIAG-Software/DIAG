# DIAG — Diagnostic Interface for Active Gamers

Version 1.1.0 Alpha. Compiled for testing September 25, 2026. All implemented features are included.

For another PC, use `releases/1.1.0 Alpha/DIAG-Setup-1.1.0-Alpha-x64.exe`. It includes the runtime, a branded setup wizard, optional shortcuts/startup, and sensor setup. See `INSTALLER.md`. The instructions below also describe running the editable development copy.

## Launch and first-time setup

Double-click `Launch-DIAG.vbs` or `Start-Local.cmd`. DIAG runs from its system-tray icon. Click the icon for a QR code and dashboard controls; choose **Open dashboard** to open Chrome.

The first tray launch requests Windows administrator approval to install the signed PawnIO hardware-sensor driver and DIAG's elevated sensor helper. The helper is installed under Program Files and registered as a per-user scheduled task with highest privileges. Subsequent tray launches start sensors automatically without another approval prompt. The HTTP dashboard continues running as the ordinary user. This development installer requires the installing account to be an administrator using its own UAC approval; installing with a different administrator's credentials is not supported yet.

If first-time setup was canceled, **Install sensor helper** on the PC dashboard retries it. Changes to the protected helper during development require reinstalling that helper. The helper has no network listener and does not accept commands from paired devices. It collects only while DIAG's local activity lease is present; its small supervisor remains idle between app sessions.

The development launcher finds Node.js on PATH, `runtime/node.exe`, or this PC's bundled Codex runtime. The standalone installer includes its own Node.js runtime; another PC needs no Node installation or development tools. The Alpha installer is currently unsigned.

## Dashboard

- **CPU, RAM and GPU:** current utilization bars labeled 0–100%, plus labeled two-minute usage graphs. History fills while the dashboard is open.
- **Temperatures:** CPU package/main reading and supported GPU temperatures. Individual CPU cores are omitted from the UI. Celsius/Fahrenheit selection is saved in each browser.
- **FPS:** PresentMon captures app-present intervals without injecting into games. Shows a two-second average and frame interval, with an app selector. Auto chooses the busiest rendering swapchain; it does not identify which app is a game. Stopped apps disappear after four seconds. This is application-present FPS, not monitor refresh rate or guaranteed displayed/generated-frame FPS.
- **Fans:** every fan sensor exposed by LibreHardwareMonitor, NVIDIA's available fan percentage, and a direct read-only MSI ACPI WMI v2 adapter on supported MSI laptops. MSI Center is not required. Fan channels are labeled Fan 1/2 rather than guessing CPU/GPU assignment. Firmware zero channels are only displayed after they have supplied a nonzero tachometer reading, since unused channels also return zero.
- **Displays:** monitor model, current resolution, integer refresh rate reported by Windows, and available rates at the same resolution/color depth/scan type. A higher listed rate produces a recommendation to review Display settings. NVIDIA's display driver supplies VRR capability where available. FreeSync certification, G-SYNC Compatible certification and enabled/in-game VRR state are not inferred from generic VRR capability; unverified fields say so.
- **Graphics drivers:** an on-demand, read-only check. NVIDIA GeForce models are matched exactly to the official WHQL Game Ready catalog. AMD/Intel are checked for compatible display-driver offers from the configured Windows Update source; this does not establish whether the newest AMD/Intel vendor package is installed. Official vendor pages remain available. No driver is downloaded or installed automatically. Laptop manufacturers may recommend a different driver.
- **Network:** connected physical Ethernet and Wi-Fi adapters, link speed, live download/upload Mbps and a two-minute graph with a labeled scale. Both are shown if both are connected. Link speed is not an internet speed test.
- **Battery health:** appears only when a battery is detected. Charge and power state refresh every 30 seconds. Design/full-charge capacities refresh from Windows' battery report every ten minutes. Health is full-charge divided by design capacity. A reported zero cycle count may mean the firmware does not supply a meaningful count.
- **My Rig:** CPU/cores, GPU and VRAM, installed drivers, RAM modules and configured transfer rate, motherboard, BIOS, Windows, physical storage models/capacities/type, displays and network controllers (including disconnected controllers). Copy includes these details. Serial numbers and product keys are omitted.

## Cleanup tools

**Free up memory** directly uses Windows' EmptyWorkingSet API on eligible background processes belonging to the current user and session. The foreground app, detected rendering processes, their child processes, Explorer and DIAG's own processes are protected. It does not close apps or purge the entire standby cache. Output reports the measured increase in available physical memory, before/after amounts and number of trimmed apps. Other activity can affect the measurement, and apps can load pages back as needed. No automatic or periodic cleanup runs.

**Clean up disk** opens the classic Windows Disk Cleanup dialog (`cleanmgr.exe`). The user chooses files and confirms deletion there. **Display settings** opens Windows Advanced display settings. Paired devices can run Game On, memory cleanup, driver and Windows update checks, network speed tests, and the listed Windows tools. Windows dialogs open on the gaming PC. Sensor installation and pairing management remain local-owner actions. The browser FPS test stays PC-only so a phone does not report its own rendering speed as PC performance.

## Second screen

Phone/tablet/PC access is always enabled. Scan the automatically displayed QR code, or type the LAN address (http://IP:port) shown in the tray panel or Second screen into another device's browser and enter the displayed 8-digit pairing code. The code has no expiration and accepts spaces or a hyphen. New code, disconnect-all, or restart invalidates it. Copy pairing link copies to the current device's clipboard only. No IP dropdown is needed; DIAG selects a local network address. Devices need to be on the same reachable private network. A VPN, guest-network isolation, firewall, or multiple networks may still require troubleshooting.

Codes and connected sessions have no time-based expiration. Codes can connect multiple devices. **New pairing code** replaces the dashboard's code. **Disconnect all devices** revokes existing sessions and makes a fresh code available. Restarting DIAG resets all access; use the tray's Open dashboard command for a fresh local link.

This development preview uses HTTP on a trusted local network, not encrypted HTTPS. Pairing secrets stay local and QR codes are rendered locally. Treat pairing links as passwords. Do not publish runtime files or QR screenshots.

## Development and limitations

`node server.js` starts LAN mode; `node server.js --local` is available for developer tests. `DIAG_PORT` changes the default 47831. Run `node --test tests/*.js` for automated checks.

CPU/RAM poll every two seconds, NVIDIA every four seconds, hardware sensors every three seconds, displays/battery charge every 30 seconds and inventory every two minutes. Network sampling uses measured time between byte counters. No universal temperature alarm threshold or invented overall health score is applied.

The primary GPU utilization card currently uses NVIDIA SMI. Broader AMD/Intel live-GPU coverage, AMD/Intel vendor-latest matching, certified-monitor lookup, exact fractional refresh rates, gaming-overhead benchmarks, encrypted pairing, persistent device approvals, and a signed packaged installer remain future work. Real iPhone/Android and additional PC models have not been validated.

See `VALIDATION.md` for actual checks and `THIRD-PARTY.md` for dependencies and protocol references.

## Power mode

The dashboard shows the active power plan, Windows power mode, power source, energy saver and the plan's plugged-in/battery settings. Performance recommendations explain relevant tradeoffs. Power settings opens Windows settings; DIAG does not change the plan automatically. Firmware and Windows mode can override effective behavior.

The elevated sensor supervisor runs through a hidden launcher, so normal background collection does not leave a console window open.

## FPS animation test

Test FPS runs a foreground canvas animation in the Windows PC's dashboard browser: one second to warm up and eight seconds to sample requestAnimationFrame intervals. It shows a rolling live rate, followed by average FPS and milliseconds per frame. Cancel or hiding the tab stops the test. This measures browser animation scheduling, not a game's performance or physical display output; monitor refresh is shown separately under Displays. Paired devices cannot run this PC test. The result is separate from PresentMon app capture.

## Internet speed test

Run speed test opens an animated dialog showing real download samples, idle HTTP latency, latency variation and concurrent HTTP latency during download requests. The moving dots indicate activity; the line graph plots measured cumulative download throughput. View test reopens the dialog. Close hides it without canceling; Cancel stops the transfer.

Tests run on demand from the Windows PC dashboard or a paired device against Cloudflare's documented download endpoint (https://github.com/cloudflare/speedtest). The test runs on the gaming PC even when started from a paired device. Seven idle samples follow a warm-up request. The download phase targets ten seconds but stops at 250 MB; the overall timeout is 45 seconds. Mbps and decimal MB/s describe the same measured throughput. Latency is HTTP response time rather than ICMP or game-server ping. Network traffic, short samples, endpoint choice and connection warm-up affect results; loaded latency may be lower than the earlier idle sample.

The latest 100 completed, canceled and failed tests persist in runtime/speed-test-history.json. Previous available results are preserved. History includes timestamps, speed, idle and loaded latency, variation and payload size. No result is uploaded by DIAG. Cloudflare necessarily receives the speed-test requests. Tests can affect active gaming traffic.

## Storage

Fixed volumes show used/free/total capacity, percentage used, file system and a warning below 10% free space. Physical drive cards show model, type, bus, capacity and Windows-reported health where exposed. Physical drives and volumes are listed separately; no unverified mapping or SSD wear percentage is inferred. Storage and My Rig panels size independently.

## Windows updates

Check for updates queries the configured Windows Update service through the Windows Update Agent API for available, non-hidden software updates. Results include titles, KB references, categories, severity and maximum download size where reported, along with pending-update restart status. Partial or failed scans are distinct from a successful empty result. Open Windows Update launches Windows settings for installation. DIAG does not install updates or restart Windows. Managed policy and rollout availability remain controlled by Windows.

## Game Time

The large centered button beneath the header opens the pre-game checklist. It verifies graphics drivers and Windows software updates, releases eligible background-app memory, silently cleans eligible local temporary files older than seven days, and checks each monitor's maximum Windows-reported rate at its current resolution. Update availability, incomplete checks, pending restart and higher refresh rates are distinct from OK. No updates are installed and no display modes are changed. Checks are initiated from the authenticated PC dashboard. Cleanup is rate-limited and cannot overlap another Game Time run.

Silent cleanup only examines this account's standard local Temp directory, skips reparse points and locked/protected files, and never recursively deletes directories. Personal folders and the Recycle Bin are outside its scope. The standalone Clean up disk button in Storage still opens the classic Windows dialog. Free up memory is at the bottom of Memory; Quick Tools has been removed. All action buttons share the same color, with dimming for disabled controls.

Game Time includes a short explanation below the launch button. Checklist rows use prominent OK badges and highlight installed/available driver versions, measured memory availability increase, reclaimed disk space and current monitor refresh rates. When every check returns OK, a separate completion dialog says All checks passed and offers Let's play or Review results. The message is based on those checks and is shown once per completed run in the browser session; incomplete, warning, restart or unverifiable results suppress it.


**Disk optimization**, next to Clean up disk in Storage, opens Windows Optimize Drives (dfrgui.exe) on the gaming PC. Paired devices may open it too. Review and start optimization in that Windows dialog; it is not added to the automatic Game On cleanup.

Windows tool buttons bring their windows to the foreground, restoring an existing minimized window when needed. PowerShell helpers stay hidden. If Windows prevents focus, DIAG displays a message pointing to the taskbar.

## User-defined warning and critical thresholds

| Reading | Warning | Critical |
|---|---|---|
| CPU temperature | 80°C to below 95°C | 95°C and above |
| GPU temperature | 70°C to below 90°C | 90°C and above |
| RAM usage | 80% to below 90% | 90% and above |
| Drive free space | Above 10%, up to 15% | 10% or less |
| Each reported fan | 100 RPM to below 1,000 RPM | Below 100 RPM |
| Completed download speed test | 20 Mbps to below 100 Mbps | Below 20 Mbps |
| Battery charge, if detected | 10% to below 25% | Below 10% |
| Graphics driver / Windows updates | — | Confirmed update available |

Thresholds use raw readings without rounding, so fractional values have no gaps. Temperature classification uses Celsius even in Fahrenheit display mode. Fan percentages are not evaluated as RPM. Network status uses the last completed speed test, labeled with its date. Battery charge is separate from capacity health. Missing/stale sensor readings are unverified, never assumed healthy.

Game On includes CPU/GPU temperature, RAM usage, each exposed fan RPM, each drive's free space, last completed download test and detected battery charge. These checks update while the checklist runs, using the latest available readings. Warning, critical and unverified results prevent the all-checks-passed dialog. The speed test is not started automatically. Existing cleanup and update checks continue as before.

## Threshold Overrides editor
Open Settings in the left menu to find the temperature display toggle and the full Threshold Overrides editor. Set Warning and Critical values for CPU/GPU temperature (always entered in Celsius), RAM usage, free disk space, fan RPM, download Mbps and battery charge. Update-available severity can be Warning or Critical. Save overrides stores settings in runtime/threshold-overrides.json and shares them with all paired devices. Restore defaults loads the original values into the editor; Save applies them. Reload values discards unsaved edits. Invalid ranges are rejected. Settings cannot be changed during a running Game On checklist, and saving clears previous checklist results so they are not mistaken for checks under the new thresholds. Changes from another device are detected before saving.

PC uptime is monitored live and included in Game On. Defaults: Warning at 3 days (72 hours), Critical at 5 days (120 hours). Both values are editable in Threshold Overrides. Uptime uses the Windows uptime reported by the collector, displays days/hours/minutes, and prompts for a manual restart without restarting automatically. Existing saved overrides are preserved when the uptime setting is added.

## Connection stability
DIAG automatically runs a hidden, lightweight ICMP monitor while the companion is running. It probes Cloudflare DNS (1.1.1.1) and the detected IPv4 default gateway about every two seconds with a 32-byte payload and one-second timeout. The last ten minutes show current/average/peak/p95 latency, mean absolute variation between consecutive successful replies (jitter), replied/missed probe counts, and ICMP loss. Local probe errors are separate from missed replies. Red graph markers are misses, amber markers are local errors, and gaps are not interpolated. Target changes reset that target's history. History is in memory and resets when DIAG restarts. Paired devices can pause, resume and clear it. Monitoring is independent of the bulk download test; during that test the display notes that the connection is under download load. Some destinations block or deprioritize ICMP, so no-reply results are not proof of internet or game-packet loss. Default-gateway monitoring is not a game-server route trace.
The header now displays the installed package version and live connection status. The former intro slogans and footer are removed; uptime remains in its dedicated section.

## Gaming session recording

Use **Record Gaming Session** at the upper right of the dashboard, or the recording controls in Gaming View. DIAG saves performance telemetry every two seconds in the Windows companion, independently of the dashboard tab. Switch to the game or monitor from a paired phone. Outside Gaming View, the recording dialog cannot be dismissed using Escape or a Close button while recording; reopening the dashboard restores it. Gaming View keeps recording controls inline so monitoring remains visible. Closing the browser does not stop the companion's recording. Stop it with **Stop recording & view report**.

**Mark the Moment** requires a short reason and saves a timestamped user-requested notation. The report attaches recorded samples within 15 seconds before and after each marker (limited by session boundaries and collection availability). Reports show summaries, threshold observations, trends, frame-source identities, network probe totals, and paged readings. Expand a marker to inspect its nearby readings. **Reporting > Gaming Sessions** retains report history.

Installed reports are saved under `%LOCALAPPDATA%\DIAG\reports\sessions` on the gaming PC: dated HTML (readable offline), JSON (compiled report) and JSONL (full timeline). No automatic expiration or deletion. The HTML is already saved locally; **Save a copy of report** downloads an additional copy to the browser's device. Recordings write incrementally and periodically flush to disk. A subsequent launch recovers unfinished timelines as interrupted reports, ignoring a torn final line. A disk-write failure stops recording and displays an error; saved data is retained. Available disk space limits recording length.

Captured metrics include CPU load/main temperature, RAM use, available GPU load/temperature/VRAM/power/clock/fan percentage, supported fan RPM and power/clock sensors, fixed-volume free space, battery charge when detected, uptime, network adapter throughput, and gateway/Cloudflare ICMP probes. Settings context includes displays, power plan, update-scan results and threshold overrides. Recording resumes lightweight network monitoring if paused; no automatic speed test, update scan, cleanup or settings optimization is performed.

Frame data uses the dashboard's selected rendering stream, or Auto (busiest rendering app at each sample). Auto can change sources; reports record identities. PresentMon must supply frames from an active rendering app. Unavailable/stale readings remain unavailable. Frame rates and frame intervals are sampled rolling windows, not a complete per-frame trace, video capture or a 1% low calculation. Rolling p95 and longest-frame metrics require the updated sensor helper from this source tree; older installed helpers expose mean intervals only. Threshold counts reflect observations, not exact durations. ICMP loss is missed probes, not measured game-packet loss. Full raw readings remain on disk; compiled trend buckets are bounded for long sessions, and timeline pages contain at most 120 samples.

## My Rig and Rig Card

My Rig is a dedicated left-menu page above Ask Your AI. Rig Card asks for a gamer handle, previews the approved Violet design and downloads a 2000 x 1200 PNG. Edit handle changes the title; Copy Text separately copies the detailed hardware summary. Card storage totals below 1.1 TB use decimal GB. The PC illustration is decorative. Downloads go to the browser device; cards are not published automatically.

The dashboard opens on Overview. Clicking a left-menu section starts at its first tab. Displays remains under System Health. Gaming View and Focus recording controls include the red recording icon; native tray menus use the current package version. See docs/RELEASE-NOTES.md for cumulative history.

## Gaming View

Choose up to 12 telemetry tiles, change their order and size, choose desktop column count and save the layout in your browser. Focus hides navigation; Fullscreen uses the browser capability. Recording, Mark the Moment and report controls stay within the view. Missing or stale readings remain unavailable. Use the same dashboard on a second monitor or a paired phone, tablet or other PC.

## Ask Your AI

Build a local prompt from a specific question, selected hardware, live readings and optional saved-session or marked-moment context. Review it, then copy or download the text for your preferred AI. DIAG neither sends the prompt automatically nor supplies an AI answer.

## Close selected apps

On the gaming PC, select eligible apps manually in Memory or the optional Game On step. DIAG sends normal close requests and does not force termination. Apps may remain open or prompt to save. Skip or cancel the selection if needed. Paired devices cannot list or close apps. This is separate from Free up memory.
