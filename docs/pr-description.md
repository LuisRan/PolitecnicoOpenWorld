# Pull Request (texto en inglés, para pegar en GitHub)

**Title:** `fix: validate IPv4 before enabling LAN join in Street Fighter online menu`

**Body:**

```markdown
## What changed?

- Added `SfLanAddress` (pure Kotlin, no Android deps) with `isValidIpv4`, `sanitize` and `shouldShowError`.
- `OnlineMenuOverlay` (Street Fighter → MULTIPLAYER → "join by typing the IP"):
  - the field now keeps only digits and dots (max 15 chars) instead of just `trim()`;
  - the **JOIN** button is enabled only for a valid IPv4 (previously: any text with exactly 3 dots, e.g. `...`, `a.b.c.d`);
  - when the text looks complete but is invalid, the field shows an error with a new string
    (`sf_lan_ip_invalid`, es/en).
- Added `SfLanAddressTest` (11 JVM unit tests).

Scope: Android only. 5 files. No map, network, permission, dependency or architecture changes.
Out of scope: IPv6/hostnames, custom port, LAN auto-discovery cards, the 4-char room code, the
`shared/.../StreetFighterScreen.kt` copy.

## Why?

Tracked in https://github.com/LuisRan/PolitecnicoOpenWorld/issues/1.

On the base commit `7ed325393f82872c2be94ff2ada46948efa19152` the JOIN button enabled for inputs such as `...`, `a.b.c.d` or
`999.999.999.999` and started a connection that could not succeed (3 attempts with pauses) before showing a
network error.

Acceptance criteria:
- **CA-1 (success):** `192.168.1.5` enables JOIN and starts the connection.
- **CA-2 (boundary):** `...`, `999.999.999.999`, `1.2.3.4.5`, `192.168.1` keep JOIN disabled, no connection is started,
  and a complete-looking invalid value shows the error message.
- **CA-3 (boundary):** letters/spaces/symbols are dropped; max 15 chars.

## QA evidence

- Test plan, results, defects and quality verdict: <LINK to docs/pruebas.md at the delivered SHA>
- Before/after screenshots and logs linked per test case ID (TC-01…TC-08): <LINK to evidencias/>
- Environment: macOS 26.6.2 (Apple M1), JDK = Android Studio JBR (Gradle 9.5.0), AVD Pixel 8, API 37.1 (English UI; Spanish checked in TC-07).
- SHA tested: `c9647041c1da769ddc2722e409850206f93fe2cb` · final delivered SHA: the same (no commits after QA).
- Local checks: `./gradlew :app:assembleDebug :app:testDebugUnitTest :shared:testAndroidHostTest` → BUILD SUCCESSFUL (:app 136 tests incl. 11 new, :shared 217, 0 failures);
  detekt (repo config + baseline) → no new issues; `tools/check_kmp_test_names.sh` → OK.
- CI (PR Quality Gate): <links to the unit-tests and detekt runs, or the documented external blocker>.

Not covered: joining a real LAN host (needs a second device on the same network); maps/Firebase (placeholder
`MAPS_API_KEY`, no `google-services.json`); rotation/background lifecycle, no-network (airplane mode) behaviour, and large-font/TalkBack checks (not executed, listed as limitations in the test plan). Back navigation (TC-06) and the Spanish locale (TC-07) were checked only partially.

## Risk / rollback

- Risk: a valid IPv4 being rejected. Mitigated by boundary unit tests (`0.0.0.0`, `255.255.255.255`) and TC-01/TC-04.
  Octets with leading zeros (e.g. `192.168.01.5`) are rejected on purpose (ambiguous/octal in some parsers).
- Known limitation: pasting `ip:port` drops the `:` (the port is fixed, not supported by this field).
- Rollback: `git revert 206f87a4` (UI + strings) and, optionally, the test commit; no data, network or
  permission migration is involved.
```
