# Nightly Kernel Changelog

**Version:** r1
**Series:** nightcord_yuki

## 2026-08-11

### Fixed
- **Fixed bootloop caused by XZ-compressed ramdisk** — enabled `CONFIG_RD_XZ` so the kernel can decompress the initial ramdisk image. Previously the kernel panicked with "Could not decompress initial ramdisk image."

### Changed
- **CI: zip filename now includes profile** — naming changed from `nightly_yuki-r1-(sha)-(device)-(buildtype)` to `nightly_yuki-r1-(sha)-(device)-(profile)-(buildtype)`. Applies to zip filename, artifact name, and release tag in both `build.yml` and `build-ksu.yml`.

### Fixed
- **Fixed kernel link errors when CONFIG_TRACING is disabled** — added `#ifdef CONFIG_TRACING` / `#ifdef CONFIG_TRACEPOINTS` guards with no-op stubs to kernel headers:
  - `include/linux/trace_events.h`: wrapped `event_trace_printk()` with a no-op fallback
  - `include/linux/kernel.h`: added `trace_puts()`, `do_trace_printk()`, and `__trace_printk_check_format()` stubs
  - `include/linux/tracepoint.h`: wrapped `tracepoint_probe_register/unregister` and `for_each_kernel_tracepoint` with `-ENOSYS` stubs
  - `drivers/misc/mediatek/connectivity/common/connectivity_build_in_adapter.c`: guarded `tracing_record_cmdline()` call
  - `drivers/misc/mediatek/met_drv/met_api.c`: guarded `tracing_record_cmdline()` call
- **Exported tracing symbols** — added `EXPORT_SYMBOL_GPL(tracing_record_cmdline)` in `kernel/trace/trace.c` and `EXPORT_SYMBOL()` for `perfmgr_trace_count/printk/log` in MTK performance driver to resolve cross-driver link errors.
- **Defconfig: disabled non-functional TRACING and TRACEPOINTS** — these are hidden bools that can only be activated via `select` from `CONFIG_FTRACE`. Marked all dependent tracing configs as disabled to match actual build state.
