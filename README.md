<div align="center">

**Instituto Politécnico Nacional**
**Escuela Superior de Cómputo**

Ingeniería en Sistemas Computacionales · Plan 2020

**Desarrollo de aplicaciones móviles nativas**

**Primer examen parcial — Pull Request con aseguramiento de calidad**
Periodo 2027-1 · Fecha de entrega: jueves 1 de octubre de 2026

</div>

| | |
|---|---|
| **Alumno** | Rangel Mata José Luis |
| **Grupo** | 7CV4 |
| **Asignatura** | Desarrollo de aplicaciones móviles nativas |
| **Profesor** | Gabriel Hurtado Avilés |

<!-- La boleta y demás datos de identificación escolar se registran en Classroom, no en este repositorio (lo pide el examen). -->

> ⚠️ Plantilla en construcción: todo lo marcado `PENDIENTE` o `<...>` se reemplaza por datos **reales** antes de entregar.
> Este índice y la carpeta `docs/` y `evidencias/` **no forman parte del diff** del PR a POW.

## 1. Equipo y usuarios de GitHub

| Integrante | Usuario de GitHub | PR |
|---|---|---|
| Rangel Mata José Luis | `LuisRan` | https://github.com/gabrielhuav/PolitecnicoOpenWorld/pull/176 |
| `<compañero(s) del equipo, si aplica>` | `<usuario>` | `<enlace>` |

## 2. Objetivo y alcance

Preparar, probar y someter a revisión un cambio pequeño en
[gabrielhuav/PolitecnicoOpenWorld](https://github.com/gabrielhuav/PolitecnicoOpenWorld/) con un proceso de QA reproducible.

**Cambio:** validación de la IP del anfitrión en el menú online de *Street Fighter* (modo LAN). En la base, el botón
**UNIRSE** se habilitaba con cualquier texto de 3 puntos (`...`, `a.b.c.d`, `999.999.999.999`) y lanzaba una conexión
condenada a fallar. Ahora sólo se habilita con una IPv4 válida, el campo admite dígitos y puntos (máx. 15) y se muestra un
mensaje de error cuando el texto parece completo pero es inválido.

- **Archivos (5):** `SfLanAddress.kt` (nuevo), `SfOnlineOverlays.kt`, `values/strings.xml`, `values-en/strings.xml`, `SfLanAddressTest.kt` (nuevo).
- **Fuera de alcance:** IPv6/nombres de host, puerto personalizado, autodescubrimiento LAN, código de sala, mapas, servidores, arquitectura.
- **Criterios de aceptación:** CA-1 (éxito), CA-2 y CA-3 (límite) — detalle en [`docs/pruebas.md`](docs/pruebas.md#criterios-de-aceptación).

## 3. Trazabilidad

| Elemento | Valor |
|---|---|
| Issue | https://github.com/LuisRan/PolitecnicoOpenWorld/issues/1 |
| Draft PR (hacia `gabrielhuav/PolitecnicoOpenWorld` `main`) | https://github.com/gabrielhuav/PolitecnicoOpenWorld/pull/176 |
| Rama de trabajo | `fix/sf-lan-ip-validation` |
| **SHA base** | `<git rev-parse HEAD antes de ramificar>` (referencia: `7ed325393f82872c2be94ff2ada46948efa19152`) |
| **SHA probado** | `c9647041c1da769ddc2722e409850206f93fe2cb` |
| **SHA final entregado** | `c9647041c1da769ddc2722e409850206f93fe2cb` · relación con el probado: **el mismo** (sin commits posteriores al QA) |
| Registro de ejecución inicial (base) | [`docs/pruebas.md`](docs/pruebas.md) §3, columna ANTES |

Título y descripción del PR (en inglés): [`docs/pr-description.md`](docs/pr-description.md) · Texto de la issue: [`docs/issue.md`](docs/issue.md).

## 4. Matriz de pruebas

Plan, fichas por caso, resultados y hallazgos: **[`docs/pruebas.md`](docs/pruebas.md)**.

| Caso | Categoría | Estado | Evidencia |
|---|---|---|---|
| TC-01 | Ruta feliz | Aprobado | `evidencias/TC-01_*` |
| TC-02 | Límite (defecto de la base) | Aprobado | `evidencias/TC-02_*` |
| TC-03 | Límite: entrada inválida | Aprobado | `evidencias/TC-03_*` |
| TC-04 | Límite: valores extremos | Aprobado | `evidencias/TC-04_*` |
| TC-05 | Regresión | Aprobado parcial | `evidencias/TC-05_*` |
| TC-06 | Navegación y estado | Aprobado parcial (Atrás + reentrada) | `evidencias/TC-06_*` |
| TC-07 | Compatibilidad/entorno | Aprobado parcial (idioma español) | `evidencias/TC-07_*` |
| TC-08 | Automatizado (unitarias + local) | Aprobado | `evidencias/TC-08_*` |

## 5. Verificaciones automáticas (CI) y revisión

[`docs/ci-y-revision.md`](docs/ci-y-revision.md): qué comprueba cada check, su estado con SHA y enlace al registro, la revisión
de un compañero, las respuestas y el estado final del PR.

## 6. Bitácora

[`docs/bitacora.md`](docs/bitacora.md): commits, casos ejecutados, enlace a la revisión que hice y **declaración de uso de IA**.

## 7. Conclusiones

**Se recomienda integrar con reservas.** El defecto (UNIRSE habilitado con IPs inválidas) se reprodujo en la base y queda corregido; lo válido se conserva y las pruebas automáticas pasan. Reservas: la accesibilidad con texto ampliado no se probó (riesgo R4 aceptado: el cambio no modifica fuentes ni tamaños de texto) y TC-05, TC-06 y TC-07 quedaron parciales; no se probó un anfitrión LAN real. Detalle en [`docs/pruebas.md`](docs/pruebas.md) §7.

## 8. Referencias

- Repositorio original: https://github.com/gabrielhuav/PolitecnicoOpenWorld/
- Workflow revisado: `.github/workflows/pr-quality-gate.yml` (commit base)
- `PolitecnicoOpenWorld/README for IAS/SETUP_PC_NUEVA.md` (preparación del build) y `09_CONVENTIONS_GOTCHAS.md`
- Enunciado del examen: *AppMovilesExamenParcial1 2027-1* (Classroom)
