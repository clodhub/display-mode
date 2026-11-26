# display-mode ownership

Repository: https://github.com/clodhub/display-mode.git. Package `pkg:clod/com.clodhub/display-mode`, provides `display-policy` (`com.clodhub/display-policy`), requires `display-mode-configuration` (`com.clodhub/display-mode-configuration`).

Code: `implementation/web/mode.ts`, `implementation/apple/Sources/DisplayPolicy.swift`, `implementation/android/src/main/java/com/clodhub/display/DisplayPolicy.kt`.
Artefacts: `component.json`, `CONTRACT.md`, `INSTRUCTIONS.md`, `SOURCE.md`, `suitability.json`, `tools.lock.json`, `SOURCE.json`, `interfaces/display-policy.json` (OpenRPC 1.3.2 document of the provided com.clodhub/display-policy interface), `interfaces/required-display-mode-configuration.json` (JSON Schema 2020-12 of the configuration), `tests/`, `visual/`. No data is kept, so there is no `migration.json`.
Server display authentication and private-data redaction belong to the room service, not here.
