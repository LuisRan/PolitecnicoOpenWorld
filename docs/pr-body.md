## What changed?

- Added `SfLanAddress` (pure Kotlin, no Android deps) with `isValidIpv4`, `sanitize` and `shouldShowError`.
- `OnlineMenuOverlay` (Street Fighter → MULTIPLAYER → "join by typing the IP"):
  - the field now keeps only digits and dots (max 15 chars) instead of just `trim()`;
  - the **JOIN** button is enabled only for a valid IPv4 (previously: any text with exactly 3 dots, e.g. `...`, `a.b.c.d`);
  - when the text looks complete but is invalid, the field shows an error with a new string (`sf_lan_ip_invalid`, es/en).
- Added `SfLanAddressTest` (11 JVM unit tests).

Scope: Android only. 5 files. No map, network, permission, dependency or architecture changes.
Out of scope: IPv6/hostnames, custom port, LAN auto-discovery cards, the 4-char room code, the `shared/.../StreetFighterScreen.kt` copy.

## Why?

Issue: https://github.com/LuisRan/PolitecnicoOpenWorld/issues/1

On the base commit `7ed325393f82872c2be94ff2ada46948efa19152` the JOIN button enabled for inputs such as `...`, `a.b.c.d` or `999.999.999.999` and started a connection that could not succeed (3 attempts with pauses) before showing a network error.

Acceptance criteria:
- **CA-1 (success):** `192.168.1.5` enables JOIN and starts the connection.
- **CA-2 (boundary):** `...`, `999.999.999.999`, `1.2.3.4.5`, `192.168.1` keep JOIN disabled, no connection is started, and a complete-looking invalid value shows the error message.
- **CA-3 (boundary):** letters/spaces/symbols are dropped; max 15 chars.

## QA evidence

- **Full test plan, fiches, findings and verdict:** [docs/pruebas.md](https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/docs/pruebas.md)
- **Delivery index (team, SHAs, matrix, checks, review):** [README.md](https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/README.md)
- **All evidence files by test case ID:** [evidencias/](https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/evidencias)
- **Issue text and CI/review log:** [docs/issue.md](https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/docs/issue.md) · [docs/ci-y-revision.md](https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/docs/ci-y-revision.md)

**Environment:** macOS 26.6.2 (Apple M1) · JDK = Android Studio JBR, Gradle 9.5.0 · AVD Pixel 8, API 37.1 (English UI; Spanish in TC-07).
**SHA base:** `7ed325393f82872c2be94ff2ada46948efa19152` · **SHA tested:** `c9647041c1da769ddc2722e409850206f93fe2cb` · **Final delivered SHA:** the same (no commits after QA; the academic branch only adds docs/evidence and is not part of this PR).

### Risks covered

| Risk | Impact | Case |
|---|---|---|
| R1 a valid IPv4 is rejected (0 / 255 limits) | High | TC-01, TC-04 |
| R2 regression in the room code (4 chars), CREATE SERVER, Cancel | Medium | TC-05 |
| R3 lost state / broken navigation (Back) | Medium | TC-06 |
| R4 error text clipped with enlarged text / TalkBack | Medium | not executed (see limitations) |
| R5 wrong/missing language string | Low | TC-07 |

### Results (executed on the tested SHA)

| Case | Category | Covers | Base (before) | Change (after) | Status |
|---|---|---|---|---|---|
| TC-01 | Happy path | CA-1, R1 | JOIN enabled → network error | same | Passed |
| TC-02 | Boundary (defect) | CA-2 | **DEF-01:** JOIN enabled with `...` | JOIN disabled | Passed |
| TC-03 | Boundary / invalid input | CA-2, CA-3 | **DEF-01:** JOIN enabled with `999.999.999.999` | error shown, JOIN disabled | Passed |
| TC-04 | Boundary / extreme values | CA-1, CA-2, R1 | n/a | `255.255.255.255` valid; `.500` error; typing `192.168.1.` no error | Passed |
| TC-05 | Regression | R2 | room code, CREATE SERVER | identical | Partial |
| TC-06 | Navigation / state | R3 | n/a | Back + re-enter: clean field, no crash | Partial |
| TC-07 | Compatibility (language) | R5 | n/a | Spanish error message shown, JOIN disabled | Partial |
| TC-08 | Automated + local checks | CA-1..3, R1 | n/a | `SfLanAddressTest` 11/11; `:app` 136 + `:shared` 217 tests, 0 failures | Passed |

### Key before / after

| Case | Before (base `7ed3253`) | After (`c964704`) |
|---|---|---|
| TC-02 `...` | <img src="https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/evidencias/TC-02_antes_02_puntos_join_habilitado.png?raw=true" width="300"> | <img src="https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/evidencias/TC-02_despues_01_campo_con_puntos_join_deshabilitado.png?raw=true" width="300"> |
| TC-03 `999.999.999.999` | <img src="https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/evidencias/TC-03_antes_03_999_join_habilitado.png?raw=true" width="300"> | <img src="https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/evidencias/TC-03_despues_01_999_error_join_deshabilitado.png?raw=true" width="300"> |
| TC-01 `192.168.1.5` | <img src="https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/evidencias/TC-01_antes_01_ip_valida_join_habilitado.png?raw=true" width="300"> | <img src="https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/evidencias/TC-01_despues_01_ip_valida_join_habilitado.png?raw=true" width="300"> |

More: [TC-04](https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/evidencias/TC-04_despues_01_255_join_habilitado.png) · [TC-05](https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/evidencias/TC-05_despues_02_crear_servidor_ip_local.png) · [TC-06](https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/evidencias/TC-06_despues_02_campo_limpio_al_reentrar.png) · [TC-07](https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/evidencias/TC-07_despues_01_idioma_es_ip_no_valida.png) · [TC-08 summary](https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/evidencias/TC-08_resumen.txt) · [Gradle log](https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/evidencias/TC-08_gradle.txt)

### Findings

- **DEF-01 (fixed):** base enabled JOIN with invalid IPs (only counted 3 dots).
- **OBS-01 (pending, introduced by the change):** the 2-line error text pushes the layout down; Cancel is partially hidden until scrolling.
- **DEF-02 (pre-existing, not fixed):** the LAN connection error is hard-coded in Spanish even with the English UI.

### Checks and CI

- Local: `./gradlew :app:assembleDebug :app:testDebugUnitTest :shared:testAndroidHostTest` → BUILD SUCCESSFUL (1m 21s); detekt (repo config + baseline) → no new issues; `tools/check_kmp_test_names.sh` → OK.
- PR Quality Gate (GitHub): no runs are listed in the Checks tab as of 2026-10-01 (0 checks right after opening the PR from a fork; GitHub may be waiting for maintainer approval to run workflows). Recorded as an external blocker, not as a passed check; this section will be updated with the run links once they exist. Log: [docs/ci-y-revision.md](https://github.com/LuisRan/PolitecnicoOpenWorld/blob/academic/examen-parcial-1/docs/ci-y-revision.md).
- Peer review: requested after QA; the reviewer's comment and my reply will be linked here. The PR stays Draft until then.

### Quality verdict

Recommend merging **with reservations**: the defect was reproduced on the base and is fixed; valid values and the nearby flow keep working; unit tests pass. Remaining risks: OBS-01, port not supported when pasting `ip:port`.

### Not covered / limitations

Joining a real LAN host (needs a second device); maps/Firebase (placeholder `MAPS_API_KEY`, no `google-services.json`); rotation / background lifecycle and airplane mode (TC-06/TC-07 only partial); **accessibility with enlarged text / TalkBack was not executed (R4 accepted, not claimed as passed)**.

## Risk / rollback

- Risk: a valid IPv4 being rejected. Mitigated by boundary unit tests (`0.0.0.0`, `255.255.255.255`) and TC-01/TC-04. Octets with leading zeros (e.g. `192.168.01.5`) are rejected on purpose (ambiguous/octal in some parsers).
- Known limitation: pasting `ip:port` drops the `:` (the port is fixed, not supported by this field).
- Rollback: `git revert 206f87a4` (UI + strings) and, optionally, the test commit; no data, network or permission migration is involved.
