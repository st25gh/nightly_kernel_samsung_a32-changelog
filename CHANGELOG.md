# Nightly Kernel Changelog

**Version:** r1
**Series:** nightcord_yuki

## 2026-08-11

### Fixed
- **Fixed random soft reboots (YouTube, MTP, idle)** — enabled `CONFIG_FTRACE` which activates the full tracing infrastructure (`CONFIG_TRACING`, `CONFIG_TRACEPOINTS`, `CONFIG_RING_BUFFER`, `CONFIG_EVENT_TRACING`). Dozens of MTK drivers (fpsgo/xgf, cmdq, connectivity, performance manager) depend on `tracepoint_probe_register()`, `for_each_kernel_tracepoint()`, and other tracing functions. Without the tracing subsystem, these drivers silently failed to register their hooks, causing broken CPU/GPU frequency management, display command queue instability, and watchdog-triggered reboots.
- **Fixed bootloop caused by XZ-compressed ramdisk** — enabled `CONFIG_RD_XZ` so the kernel can decompress the initial ramdisk image. Previously the kernel panicked with "Could not decompress initial ramdisk image."
- **Enabled `CONFIG_DMA_CMA` for USB gadget** — allows the USB gadget driver to allocate contiguous DMA memory for MTP bulk transfers.
- **Fixed kernel link errors when CONFIG_TRACING is disabled** — added `#ifdef CONFIG_TRACING` / `#ifdef CONFIG_TRACEPOINTS` guards with no-op stubs to kernel headers (fallback paths, not normally reached now that FTRACE is enabled):
  - `include/linux/trace_events.h`: wrapped `event_trace_printk()` with a no-op fallback
  - `include/linux/kernel.h`: added `trace_puts()`, `do_trace_printk()`, and `__trace_printk_check_format()` stubs
  - `include/linux/tracepoint.h`: wrapped `tracepoint_probe_register/unregister` and `for_each_kernel_tracepoint` with `-ENOSYS` stubs
  - `drivers/misc/mediatek/connectivity/common/connectivity_build_in_adapter.c`: guarded `tracing_record_cmdline()` call
  - `drivers/misc/mediatek/met_drv/met_api.c`: guarded `tracing_record_cmdline()` call
- **Exported tracing symbols** — added `EXPORT_SYMBOL_GPL(tracing_record_cmdline)` in `kernel/trace/trace.c` and `EXPORT_SYMBOL()` for `perfmgr_trace_count/printk/log` in MTK performance driver to resolve cross-driver link errors.

### Changed
- **CI: zip filename now includes profile** — naming changed from `nightly_yuki-r1-(sha)-(device)-(buildtype)` to `nightly_yuki-r1-(sha)-(device)-(profile)-(buildtype)`. Applies to zip filename, artifact name, and release tag in both `build.yml` and `build-ksu.yml`.
- **CI: build all 3 profiles on release** — when build_type is 'release', build all 3 profiles (battery, balance, perf) in a matrix. When 'debug', build only the selected profile.
