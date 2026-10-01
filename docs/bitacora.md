# Bitácora individual

**Integrante:** Rangel Mata José Luis · **GitHub:** `LuisRan` · **Grupo:** 7CV4

> Completa con datos reales: tus propios commits, los casos que **tú** ejecutaste y el enlace a la revisión que hiciste.

## Commits

| SHA | Mensaje | Qué aporta |
|---|---|---|
| `206f87a4` | `fix: validate IPv4 before enabling LAN join in Street Fighter` | Validador `SfLanAddress`, ajuste de `OnlineMenuOverlay` y cadenas es/en |
| `c9647041c1da769ddc2722e409850206f93fe2cb` | `test: add unit tests for LAN IPv4 validation` | `SfLanAddressTest` (11 pruebas) |
| `PENDIENTE` | commits de QA/documentación/correcciones de revisión | |

## Casos ejecutados (por mí)

| Caso | Fecha | SHA | Estado | Evidencia |
|---|---|---|---|---|
| TC-01, TC-02, TC-03, TC-04, TC-08 | 2026-10-01 | `c9647041` | Aprobados | `evidencias/` |
| TC-05 | 2026-10-01 | `c9647041` | Aprobado parcial | `evidencias/` |
| TC-06 | 2026-10-01 | `c9647041…` | Aprobado parcial (Atrás + reentrada) | `evidencias/TC-06_despues_*` |
| TC-07 | 2026-10-01 | `c9647041…` | Aprobado parcial (idioma español) | `evidencias/TC-07_despues_01_*` |

## Revisión técnica que hice a otro trabajo

Enlace: `PENDIENTE`

## Uso de herramientas de IA (declaración)

- **Claude (Anthropic), en modo Cowork:** exploró el repositorio, propuso la contribución (validación de la IP LAN),
  redactó el código del validador, el ajuste de la interfaz, las pruebas unitarias, y las plantillas de la documentación
  (plan de pruebas, issue, descripción del PR, README) y la guía de ejecución.
- **Lo que hice yo:** `PENDIENTE — describe con tus palabras: fork, clon, ejecución en Android Studio/emulador, capturas
  antes/después, commits, push, apertura del PR, revisión entre compañeros y llenado de resultados reales.`
- Las evidencias de ejecución son reales y las produje yo en mi equipo; los resultados no fueron generados por la IA.
- Debo poder explicar: qué hacía el código antes (`count { it == '.' } == 3`), por qué se eligió un validador puro,
  qué hace `sanitize`/`shouldShowError`, por qué se rechazan ceros a la izquierda y qué limitaciones quedan.
