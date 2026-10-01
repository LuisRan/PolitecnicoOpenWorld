# Verificaciones automáticas (CI) y revisión del PR

> Todo en `PENDIENTE` hasta tener datos reales. Un bloqueo externo documentado (p. ej. GitHub exige que el
> mantenedor autorice workflows de un fork) **no** se presenta como prueba aprobada.

## 1. Qué comprueba el workflow `PR Quality Gate` (revisión del commit base `7ed3253`)

Archivo: `.github/workflows/pr-quality-gate.yml`. Se activa en PR hacia `main` con cambios en `PolitecnicoOpenWorld/**`
(`opened`, `synchronize`, `reopened`). Usa JDK 21.

| Job / paso | Qué comprueba | Qué NO cubre |
|---|---|---|
| `unit-tests` · *Nombres de test compatibles con Kotlin/Native* (`tools/check_kmp_test_names.sh`) | `grep` de nombres de test con `( ) ,` entre backticks en `shared/src/commonTest` (rompen iOS/Kotlin Native) | No compila iOS; mi test está en `:app` y no usa backticks |
| `unit-tests` · *Build debug APK + run unit tests* (`:app:assembleDebug :app:testDebugUnitTest :shared:testAndroidHostTest`) | Que la app compile (Hilt, manifest, recursos) y pasen las pruebas unitarias JVM de `:app` y `:shared` | No ejecuta la app ni UI; no prueba mapas/Firebase (`MAPS_API_KEY` puede ir vacío y se omite `google-services.json`); no sustituye el QA manual |
| `detekt` (CLI 1.23.8, bloqueante con baseline) | Issues **nuevos** de análisis estático en `app/src/main/java` y `shared/src/commonMain/kotlin` | No analiza tests de `:app`; la deuda previa está perdonada por `config/detekt/baseline.xml` |

## 2. Registro de verificaciones del PR (llenar)

| Verificación | Estado | SHA / ejecución | Enlace al registro | Notas |
|---|---|---|---|---|
| unit-tests (kmp names + build + tests) | PENDIENTE | PENDIENTE | PENDIENTE | |
| detekt | PENDIENTE | PENDIENTE | PENDIENTE | |
| Local: `check_kmp_test_names.sh` | OK | `c9647041` | `../evidencias/TC-08_resumen.txt` | |
| Local: `./gradlew :app:assembleDebug :app:testDebugUnitTest :shared:testAndroidHostTest` | BUILD SUCCESSFUL (1m 21s) | `c9647041` | `../evidencias/TC-08_gradle.txt` | `SfLanAddressTest`: 11/11; `:app` 136 y `:shared` 217 pruebas, 0 fallos |
| Local: detekt (mismo CLI y config) | Sin issues nuevos (exit 0) | `c9647041` | `../evidencias/TC-08_resumen.txt` | |

Si un check falla: causa localizada en el registro, relación con el cambio, corrección con commit nuevo, repetición del check.
Prohibido desactivar pruebas/checks/reglas ni ampliar el baseline. Si el fallo ya existe en la base: marcarlo **preexistente** con evidencia.

## 3. Revisión por un compañero

- Revisor (usuario de GitHub): `PENDIENTE` · SHA revisado: `PENDIENTE` · Enlace a la revisión: `PENDIENTE`
- Caso(s) reproducido(s) por el revisor: `PENDIENTE`
- El autor del PR **no** aprueba su propio trabajo.

| # | Observación del revisor (línea/comportamiento, observado, recomendación) | Respuesta del autor | Commit / evidencia | Nueva ejecución |
|---|---|---|---|---|
| 1 | PENDIENTE | PENDIENTE | PENDIENTE | PENDIENTE |

## 4. Revisión que yo hago a otro compañero (requisito de cada integrante)

- Trabajo revisado (PR/equipo): `PENDIENTE`
- Diff leído y criterios contrastados con la evidencia: `PENDIENTE`
- Caso reproducido en el SHA revisado: `PENDIENTE`
- Comentario técnico publicado (enlace): `PENDIENTE`

## 5. Estado final del PR

Estado (Draft / Ready for review / Merged): `PENDIENTE` · Dictamen de calidad: ver `pruebas.md` §7.
