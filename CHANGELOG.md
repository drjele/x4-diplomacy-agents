# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### Changed

- Standardize repository settings, development checks, code style and documentation.
- Detailed Steam Workshop description, covering every setting, where to change it, and the required and optional dependencies.

### Added

- `publish.sh update` options `--minor`, `--namedesc` and `--readback`, for an update that leaves the version number alone, one that also pushes the name and description to Steam, and one that writes Steam's own text back into `content.xml.steam`.

### Fixed

- `publish.sh` passes `-batchmode`, so an upload no longer waits for a keypress a non-interactive run cannot give it.
- `publish.sh` runs the workshop tool inside the Steam snap's mount namespace. The snap has a private `/tmp`, so the Steam client IPC the tool needs is unreachable from outside it and the upload failed on a Steamworks assertion.
- `publish.sh` shows the tool's output, which Proton otherwise discards, and reads success or failure out of it rather than out of an exit code Proton does not pass on.
- `publish.sh` restores the local installation even when the upload fails, instead of leaving it holding the staged copy with the Workshop id in it.
- `publish.sh update` sends the preview image too, so a refreshed `extension/preview.jpg` reaches the Workshop item instead of leaving the one from the first upload in place.

## [v1.0.0] - 2026-08-30 - Initial release

### Added

- Configurable agent experience multiplier, injured-agent experience, risk scaling, death prevention, and agent gender selection.
- Missing gender-specific character variants for supported Teladi-race factions and the Quettanauts.
- Optional in-game controls through SirNukes Mod Support APIs, with vanilla-equivalent defaults and file-based configuration fallbacks.
- Existing-save support, X4 9.00 compatibility, and installation and Steam Workshop publishing helpers.

[v1.0.0]: https://github.com/drjele/x4-diplomacy-agents/releases/tag/v1.0.0
