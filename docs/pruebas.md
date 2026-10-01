# Plan y ejecución de pruebas (QA)

> **Estado de este documento:** el plan (secciones 1–4) está redactado **antes de ejecutar**. Los campos
> *Resultado real*, *Estado*, *Evidencia*, *Autor/fecha* y *SHA* de cada caso están en `PENDIENTE` y se
> llenan **sólo después de ejecutar** el caso. Un caso bloqueado no cuenta como ejecutado.

## 0. Entorno (llenar con salida real)

| Dato | Valor |
|---|---|
| Equipo / SO | MacBook Air M1 (`air-de-luis`) · macOS 26.6.2 (build 25G83), arm64 |
| Android Studio | `<versión>` |
| JDK usado por Gradle | `<java -version>` (el CI usa JDK 21) |
| Android SDK / compileSdk | `<SDK Manager>` · el proyecto usa compileSdk/targetSdk 36, minSdk 24 |
| Dispositivo / AVD principal | AVD `Pixel_8` ("Pixel 8 API 37.1") · API 37 · modelo `sdk_gphone16k_arm64` (imagen 16 KB) · emulator-5554 |
| Dispositivo / AVD secundario (TC-07) | `<opcional>` |
| Repositorio / fork | https://github.com/LuisRan/PolitecnicoOpenWorld |
| SHA base (`main` sincronizado) | `<git rev-parse HEAD antes de ramificar>` (referencia vista el 1-oct-2026: `7ed325393f82872c2be94ff2ada46948efa19152`) |
| SHA probado (después) | `c9647041c1da769ddc2722e409850206f93fe2cb` (commits: `206f87a4` fix, `c9647041` test) |
| Versión de la app | `versionName <leer en app/build.gradle.kts / About>` (la base declara 1.0.0.18) |
| Dependencias que limitan las pruebas | `MAPS_API_KEY=DEFAULT_API_KEY` (marcador), sin `google-services.json`: no se prueban mapas ni Firebase. El **anfitrión LAN real** requiere un 2.º dispositivo en la misma red: no se dispone de él, por lo que la unión exitosa extremo a extremo **queda fuera de este QA** (ver limitaciones). |

## 1. Cambio bajo prueba

- **Comportamiento actual (base):** en *Street Fighter → MULTIJUGADOR → "…o únete tecleando la IP"*, el botón
  **UNIRSE** se habilita si el texto tiene exactamente 3 puntos. Con `...`, `a.b.c.d` o `999.999.999.999`
  se puede pulsar y la app intenta conectar (3 intentos con 1 s de pausa) antes de mostrar error de red.
- **Comportamiento esperado:** UNIRSE sólo se habilita con una IPv4 válida; el campo sólo acepta dígitos y
  puntos; si el texto parece completo pero es inválido se muestra *"IP no válida. Ejemplo: 192.168.1.5"*.
- **Usuario afectado:** jugador que se une a la sala LAN tecleando la IP del anfitrión (p. ej. error de dedo).
- **Archivos:** `SfLanAddress.kt` (nuevo), `SfOnlineOverlays.kt`, `values/strings.xml`, `values-en/strings.xml`,
  `SfLanAddressTest.kt` (nuevo).
- **Fuera de alcance:** IPv6, nombres de host, puerto personalizado (el puerto es fijo), el flujo de autodescubrimiento
  LAN, el código de sala de 4 caracteres, mapas, servidores, la copia `shared/…/StreetFighterScreen.kt`.

### Criterios de aceptación

| ID | Tipo | Criterio observable |
|---|---|---|
| CA-1 | Éxito | Con una IPv4 válida (p. ej. `192.168.1.5`) UNIRSE está habilitado y al pulsarlo la app inicia la conexión (pantalla CONECTANDO…) sin mensaje de validación. |
| CA-2 | Límite | Con entrada que no es IPv4 válida (`...`, `999.999.999.999`, `1.2.3.4.5`, `192.168.1`) UNIRSE permanece deshabilitado y **no** se inicia conexión; si el texto parece completo e inválido se muestra el mensaje de error. |
| CA-3 | Límite | El campo sólo conserva dígitos y puntos (máx. 15): letras, espacios y símbolos tecleados o pegados se descartan. |

### Riesgos

| ID | Riesgo | Impacto | Caso que lo cubre |
|---|---|---|---|
| R1 | La validación rechaza una IP **válida** (p. ej. límites 0/255) y el jugador ya no puede unirse por LAN | Alto: se pierde una función | TC-01, TC-04 |
| R2 | Regresión en otros caminos del mismo menú (código de sala de 4 caracteres, CREAR SERVIDOR, Cancelar) | Medio | TC-05 |
| R3 | Pérdida de texto/estado o navegación rota (Atrás, rotación, segundo plano) con el campo modificado | Medio | TC-06 |
| R4 | El mensaje de error se corta o es ilegible con texto ampliado, o TalkBack no lo anuncia | Medio (accesibilidad) | **Ninguno** (riesgo aceptado: no se probó; el cambio no toca fuentes ni tamaños de texto) |
| R5 | Cadena/idioma: el mensaje no existe o aparece en el idioma equivocado; comportamiento sin red | Bajo | TC-07 |
| R6 | Pegar `192.168.1.5:7777` produce `192.168.1.57777` al descartar `:` (el puerto no está soportado) | Bajo, conocido | TC-03 (observación) |

### Trazabilidad criterio/riesgo → casos

| | TC-01 | TC-02 | TC-03 | TC-04 | TC-05 | TC-06 | TC-07 | TC-08 |
|---|---|---|---|---|---|---|---|---|
| CA-1 | ● | | | ● | | | ● | ● |
| CA-2 | | ● | ● | ● | | | | ● |
| CA-3 | | | ● | | | | | ● |
| R1 | ● | | | ● | | | | ● |
| R2 | | | | | ● | | | |
| R3 | | | | | | ● | | |
| R4 | _sin cobertura (aceptado)_ | | | | | | | |
| R5 | | | | | | | ● | |

## 2. Convenciones de ejecución

- Cada caso se ejecuta **ANTES** (SHA base) y **DESPUÉS** (SHA probado) en el mismo AVD y con los mismos datos.
- Datos ficticios únicamente. No se usan cuentas, claves ni IPs reales.
- Evidencia: `evidencias/TC-xx_antes_nn.png|mp4|txt` y `evidencias/TC-xx_despues_nn.png|mp4|txt`.
- Capturas: `adb exec-out screencap -p > evidencias/TC-xx_<antes|despues>_01.png`.
- Cada evidencia indica qué se hizo, qué se observó, SHA y entorno (se anota en la ficha del caso).
- Cómo llegar: Menú principal → deslizar hacia abajo → **TITULACIÓN POR COMBATE** → (menú de modos) deslizar hacia abajo → **MULTIPLAYER / MULTIJUGADOR** → deslizar hacia abajo hasta *"…o únete tecleando la IP"* (campo *Host IP / IP del anfitrión*). OJO: el botón MULTIPLAYER del menú principal es el del mundo abierto ("Connect to Server"), **no** es éste.
- **Idioma del AVD: inglés.** Textos de la UI en las capturas: MULTIPLAYER, Host IP, JOIN, error *"Invalid IP. Example: 192.168.1.5"* (en español: UNIRSE, IP del anfitrión, *"IP no válida. Ejemplo: 192.168.1.5"*).

## 3. Fichas de casos

> Resultado esperado = comportamiento **tras el cambio** (columna DESPUÉS). La columna ANTES documenta lo que
> realmente ocurre en la base; si en la base se observa otra cosa, se anota lo observado (no se fuerza).

### TC-01 — Ruta feliz: IPv4 válida habilita UNIRSE y arranca la conexión
- **Cubre:** CA-1, R1 · **Autor/fecha:** Rangel Mata José Luis · 2026-10-01 · **SHA:** antes `7ed32539` / después `c9647041` · **App:** v1.0.0.18 (pre-alpha, visible en el menú)
- **Dispositivo/API/config:** `PENDIENTE` (AVD principal, idioma inglés, fuente 100 %)
- **Precondiciones/datos:** app instalada desde la rama; sin anfitrión LAN en la red; dato `192.168.1.5`.
- **Pasos:** 1) Abrir TITULACIÓN POR COMBATE → MULTIJUGADOR. 2) Tocar el campo "IP del anfitrión". 3) Teclear `192.168.1.5`. 4) Observar el botón UNIRSE. 5) Pulsar UNIRSE.
- **Esperado:** el campo acepta el texto sin mensaje de error; UNIRSE habilitado; tras pulsar aparece CONECTANDO… y, al no existir anfitrión, termina en la pantalla de error de red (comportamiento correcto y preexistente del flujo).
- **Resultado real (antes, SHA base, 1-oct-2026 12:07):** `192.168.1.5` se acepta, **JOIN habilitado**; al pulsarlo pasa a la pantalla de error *NO WI-FI CONNECTION — "Red local: no se pudo conectar (failed to connect to /192.168.1.5 (port 47645) from /10.0.2.16 … ENETUNREACH (Network is unreachable))"*. Es el comportamiento correcto sin anfitrión (el AVD está en una red NAT 10.0.2.x y no alcanza 192.168.1.5). **(después, SHA `c9647041`, 12:34):** `192.168.1.5` → **JOIN habilitado, sin mensaje de validación**; al pulsarlo el mismo error de red (`ENETUNREACH`, puerto 47645) que en la base → **comportamiento conservado**.
- **Evidencia DESPUÉS:** [IP válida, JOIN habilitado](../evidencias/TC-01_despues_01_ip_valida_join_habilitado.png) · [error de red sin anfitrión](../evidencias/TC-01_despues_02_error_red_sin_anfitrion.png)
- **Evidencia ANTES:** [IP válida, JOIN habilitado](../evidencias/TC-01_antes_01_ip_valida_join_habilitado.png) · [error de red sin anfitrión](../evidencias/TC-01_antes_02_error_red_sin_anfitrion.png)
- **Estado:** **Aprobado** · **Defecto:** `—` · **Decisión:** el cambio cumple el criterio; sin defecto nuevo

### TC-02 — Límite: `...` no habilita UNIRSE (defecto de la base)
- **Cubre:** CA-2 · **Autor/fecha:** Rangel Mata José Luis · 2026-10-01 · **SHA:** antes `7ed32539` / después `c9647041` · **App:** v1.0.0.18 (pre-alpha, visible en el menú) · **Dispositivo/API:** AVD Pixel_8 · API 37 · inglés · fuente 100 %
- **Precondiciones/datos:** menú online abierto; dato `...`.
- **Pasos:** 1) Teclear `...` en "IP del anfitrión". 2) Observar UNIRSE. 3) Intentar pulsar UNIRSE. 4) Anotar qué ocurre y cuánto tarda en aparecer el error (cronómetro).
- **Esperado (después):** UNIRSE deshabilitado, no cambia de pantalla, no se abre conexión (sin logs `SF-NET … intento`).
- **Resultado real (antes):** observado en la grabación `11.58.31` (AVD Pixel_8 API 37, inglés, SHA base): con `...` escrito en *Host IP* el botón **JOIN se ve habilitado**; al pulsarlo pasa a *CONNECTING… / Connecting to the local server…* durante unos 3 s (estimado del video) y termina en *NO WI-FI CONNECTION — "Red local: no se pudo conectar (...)"*. **Defecto reproducido en la base.** · **(después, SHA `c9647041`, 12:30):** con `...` en *Host IP* el botón **JOIN queda deshabilitado** (igual que con el campo vacío) y no aparece mensaje de error (el texto termina en punto: aún se considera "escribiendo"). Un botón deshabilitado no recibe toques y `onClick` además está protegido por `isValidIpv4`. [captura](../evidencias/TC-02_despues_01_campo_con_puntos_join_deshabilitado.png)
- **Evidencia ANTES (real, 1-oct-2026 12:01, SHA base):** [campo vacío, JOIN deshabilitado](../evidencias/TC-02_antes_01_campo_vacio_join_deshabilitado.png) · [`...` con JOIN habilitado](../evidencias/TC-02_antes_02_puntos_join_habilitado.png) · [error tras pulsar JOIN](../evidencias/TC-02_antes_03_error_tras_join.png) · [video](../evidencias/TC-02_antes_video.mov). Comparación: con el campo vacío JOIN se ve apagado; con `...` se ve activo.
- **Estado:** **Aprobado** (JOIN deshabilitado; no se capturó un toque sobre el botón apagado: un botón deshabilitado no recibe toques y `onClick` está además protegido) · **Defecto:** `DEF-01` (ver §5) · **Decisión:** el cambio cumple el criterio; sin defecto nuevo

### TC-03 — Límite: letras, fuera de rango y sobrante
- **Cubre:** CA-2, CA-3, R6 · **Autor/fecha:** Rangel Mata José Luis · 2026-10-01 · **SHA:** antes `7ed32539` / después `c9647041` · **App:** v1.0.0.18 (pre-alpha, visible en el menú) · **Dispositivo/API:** AVD Pixel_8 · API 37 · inglés · fuente 100 %
- **Datos:** `a.b.c.d`; `999.999.999.999`; `1.2.3.4.5`; pegar `ip: 192.168.1.5 ` (con texto y espacio); pegar `192.168.1.5:7777`.
- **Pasos:** 1) Teclear `a.b.c.d`. 2) Borrar y teclear `999.999.999.999`. 3) Borrar y teclear `1.2.3.4.5`. 4) Pegar `ip: 192.168.1.5 `. 5) Pegar `192.168.1.5:7777`. Tras cada paso anotar texto visible, botón y mensaje.
- **Esperado (después):** `a.b.c.d` → el campo queda `...` y UNIRSE deshabilitado; `999.999.999.999` y `1.2.3.4.5` → mensaje "IP no válida. Ejemplo: 192.168.1.5" y UNIRSE deshabilitado; `ip: 192.168.1.5 ` → `192.168.1.5` y UNIRSE habilitado; `192.168.1.5:7777` → `192.168.1.57777` con mensaje de error (**limitación R6**, el puerto no está soportado).
- **Resultado real (antes, SHA base, 1-oct-2026 12:02–12:03):** `a.b.c.d` → el campo lo acepta y **JOIN se habilita**; al pulsarlo: *NO WI-FI CONNECTION — "Red local: no se pudo conectar (a.b.c.d)"*. `999.999.999.999` → **JOIN se habilita**; al pulsarlo: *NO WI-FI CONNECTION — "… (999.999.999.999)"*. Defecto reproducido. No se prueba `1.2.3.4.5` en la base: con 4 puntos el JOIN ya quedaba deshabilitado (no es defecto de la base); ese dato y los pegados (`ip: 192.168.1.5 `, `192.168.1.5:7777`) se ejecutan sólo DESPUÉS. **(después, SHA `c9647041`):** `999.999.999.999` → mensaje rojo *"Invalid IP. Example: 192.168.1.5"* y **JOIN deshabilitado** · `1.2.3.4.5` → mismo mensaje y JOIN deshabilitado · `192.168.1.5:7777` pegado → queda `192.168.1.57777` con error y JOIN deshabilitado (limitación R6 confirmada: se descarta `:`). `a.b.c.d` y el pegado `ip: 192.168.1.5 `: `PENDIENTE de captura`.
- **Evidencia DESPUÉS:** [999…](../evidencias/TC-03_despues_01_999_error_join_deshabilitado.png) · [1.2.3.4.5](../evidencias/TC-03_despues_02_12345_error_join_deshabilitado.png) · [ip:puerto pegado](../evidencias/TC-03_despues_03_ip_puerto_pegado_error.png)
- **Evidencia ANTES:** [a.b.c.d JOIN habilitado](../evidencias/TC-03_antes_01_abcd_join_habilitado.png) · [a.b.c.d error](../evidencias/TC-03_antes_02_abcd_error.png) · [999… JOIN habilitado](../evidencias/TC-03_antes_03_999_join_habilitado.png) · [999… error](../evidencias/TC-03_antes_04_999_error.png)
- **Estado:** **Aprobado** (`a.b.c.d` y el pegado `ip: 192.168.1.5 ` no tienen captura manual útil —el resultado visible es sólo `...`/`192.168.1.5`—; quedan cubiertos por `SfLanAddressTest.sanitize_keeps_only_digits_and_dots`, 11/11 en verde en TC-08) · **Defecto:** `DEF-01` · **Decisión:** el cambio cumple el criterio; sin defecto nuevo

### TC-04 — Límite: escritura progresiva, borrado y valor máximo
- **Cubre:** CA-1, CA-2, R1 · **Autor/fecha:** Rangel Mata José Luis · 2026-10-01 · **SHA:** antes `7ed32539` / después `c9647041` · **App:** v1.0.0.18 (pre-alpha, visible en el menú) · **Dispositivo/API:** AVD Pixel_8 · API 37 · inglés · fuente 100 %
- **Pasos:** 1) Teclear `192.168.1.` carácter por carácter: confirmar que **no** aparece error mientras se escribe. 2) Completar `192.168.1.5`: UNIRSE se habilita. 3) Añadir un dígito más (`192.168.1.50` → `192.168.1.500`): aparece error y UNIRSE se deshabilita. 4) Borrar todo: el error desaparece. 5) Teclear `255.255.255.255`: UNIRSE habilitado (límite superior válido); no se pulsa.
- **Esperado:** error sólo cuando el texto parece completo e inválido; `255.255.255.255` aceptado.
- **Resultado real (después, SHA `c9647041`):** `192.168.1.` (escritura progresiva) → **sin mensaje de error** y JOIN apagado · `192.168.1.5` → JOIN habilitado (ver TC-01) · `192.168.1.500` → mensaje de error y JOIN deshabilitado · `255.255.255.255` → **JOIN habilitado** (límite superior válido). El borrado total no se capturó: el error depende sólo del texto actual (`SfLanAddressTest.error_is_hidden_while_typing_and_for_valid_input`). (Antes: no aplica comparación, el comportamiento de la base ya se documentó en TC-02/TC-03.)
- **Evidencia DESPUÉS:** [progresivo `192.168.1.`](../evidencias/TC-04_despues_03_progresivo_sin_error.png) · [255.255.255.255](../evidencias/TC-04_despues_01_255_join_habilitado.png) · [192.168.1.500](../evidencias/TC-04_despues_02_500_error_join_deshabilitado.png)
- **Estado:** **Aprobado** · **Defecto:** — · **Decisión:** el cambio cumple el criterio · **Defecto:** `PENDIENTE` · **Decisión:** `PENDIENTE`

### TC-05 — Regresión: resto del menú online sigue funcionando
- **Cubre:** R2 · **Autor/fecha:** Rangel Mata José Luis · 2026-10-01 · **SHA:** antes `7ed32539` / después `c9647041` · **App:** v1.0.0.18 · **Dispositivo/API:** AVD Pixel_8 · API 37 · inglés
- **Pasos:** 1) En el campo "Código" teclear `ab1` → UNIRSE (de código) deshabilitado; teclear `ab12` → se muestra `AB12` en mayúsculas y UNIRSE se habilita; no pulsar. 2) Confirmar que el campo Código no admite más de 4 caracteres. 3) Pulsar CREAR SERVIDOR: observar el resultado (IP local o aviso de "sin red") y salir. 4) Reabrir el menú y pulsar Cancelar: vuelve al selector de Street Fighter.
- **Esperado:** idéntico en ANTES y DESPUÉS (el cambio no toca estos controles).
- **Resultado real (antes, SHA base, 1-oct-2026 12:08–12:09):** Código `ab1` → se muestra `AB1` en mayúsculas y el JOIN del código queda **apagado**; `ab12` → `AB12` y JOIN **habilitado**. CREATE SERVER abre *LOCAL SERVER — "Your room is open…"* con las IPs locales `10.0.2.16 · 10.0.2.15` y botón Cancel. No se capturó en la base el límite de 4 caracteres ni Cancel/volver (se ejecutan en DESPUÉS y se comparan por observación). **(después, SHA `c9647041`, parcial):** `ab1` → `AB1` con JOIN del código **apagado** (igual que antes) · CREATE SERVER → *LOCAL SERVER* con `10.0.2.16 · 10.0.2.15` (igual que antes). No se capturaron DESPUÉS `ab12`, el límite de 4 caracteres ni Cancel (el diff no toca el campo Code ni su botón: `onValueChange = { code = it.uppercase().take(4) }` queda idéntico).
- **Evidencia DESPUÉS:** [AB1](../evidencias/TC-05_despues_01_codigo_AB1_join_deshabilitado.png) · [CREATE SERVER](../evidencias/TC-05_despues_02_crear_servidor_ip_local.png)
- **Evidencia ANTES:** [AB1](../evidencias/TC-05_antes_01_codigo_AB1_join_deshabilitado.png) · [AB12](../evidencias/TC-05_antes_02_codigo_AB12_join_habilitado.png) · [CREATE SERVER](../evidencias/TC-05_antes_03_crear_servidor_ip_local.png)
- **Estado:** **Aprobado parcialmente** (AB1 y CREATE SERVER idénticos al ANTES; faltan capturas de `ab12`, límite y Cancel) · **Defecto:** `PENDIENTE` · **Decisión:** `PENDIENTE`

### TC-06 — Navegación y estado: Atrás, rotación y ciclo de vida
- **Cubre:** R3 · **Autor/fecha:** Rangel Mata José Luis, 2026-10-01 (≈12:57) · **SHA:** `c9647041c1da769ddc2722e409850206f93fe2cb` · **App:** build debug del SHA probado · **Dispositivo/API:** AVD Pixel 8, API 37.1 (horizontal)
- **Pasos:** 1) Con `192.168.1.5` escrito pulsar **Atrás** del sistema: anotar adónde regresa. 2) Reabrir MULTIJUGADOR, teclear `192.168.1.5`, pulsar UNIRSE y, en la pantalla de error, pulsar el botón para volver y luego Atrás. 3) Rotar el AVD (Ctrl+←/→ en el emulador) con `192.168.1.5` escrito: anotar si el texto se conserva y si el botón sigue bien. *(Si la pantalla fija la orientación, registrarlo.)* 4) Con texto escrito pulsar Inicio, esperar 10 s y volver a la app: anotar el estado.
- **Esperado:** navegación consistente en ANTES y DESPUÉS. El campo usa `remember` (no `rememberSaveable`) igual que antes del cambio: **no se promete** que el texto sobreviva a la recreación; se documenta lo observado.
- **Pasos efectivamente ejecutados (parcial):** con `999.999.999.999` escrito (error visible y UNIRSE apagado) se salió de la pantalla con **Atrás** y se volvió a abrir el mismo menú. No se probó rotación, segundo plano ni el flujo `192.168.1.5` → pantalla de error → Atrás (pasos 2–4).
- **Resultado real (después):** el error y el texto se muestran con el valor inválido; al reabrir el menú el campo aparece **vacío**, sin error y con UNIRSE apagado, sin cierres ni pantallas rotas. Es coherente con el diseño esperado (`remember`, el estado no se conserva al salir). No hay captura ANTES de este caso: la equivalencia con la base se apoya en que el cambio no toca la navegación ni el tipo de estado.
- **Estado:** **Aprobado parcialmente** (sólo Atrás + reentrada; rotación y segundo plano **no ejecutados**) · **Evidencia:** [error antes de Atrás](../evidencias/TC-06_despues_01_error_antes_de_atras.png) · [campo limpio al reentrar](../evidencias/TC-06_despues_02_campo_limpio_al_reentrar.png) · **Defecto:** ninguno nuevo; OBS-01 visible en la primera captura (el texto de error de 2 líneas empuja el botón Cancel hacia el borde inferior) · **Decisión:** se acepta con reserva (ver §7).

### TC-07 — Compatibilidad/entorno: idioma español y sin red
- **Cubre:** R5, CA-1 · **Autor/fecha:** Rangel Mata José Luis, 2026-10-01 (≈12:59) · **SHA:** `c9647041c1da769ddc2722e409850206f93fe2cb` · **App:** build debug del SHA probado · **Dispositivo/API:** AVD Pixel 8, API 37.1 (horizontal) (opcional: repetir en un 2.º AVD de otro tamaño/API)
- **Pasos:** 1) Cambiar el idioma del AVD de inglés a **Español**. 2) Teclear `999.999.999.999`: el mensaje debe decir "IP no válida. Ejemplo: 192.168.1.5" (en inglés ya se vio en los casos anteriores). 3) Activar modo avión, teclear `192.168.1.5` y pulsar UNIRSE: anotar el comportamiento (error de red amigable, sin cierre inesperado). 4) Restaurar idioma y red.
- **Esperado:** cadena en español presente; sin cierres inesperados sin red.
- **Pasos efectivamente ejecutados (parcial):** idioma del AVD cambiado a **Español** (pasos 1–2). Con `999.999.999.999` escrito se comprobó el texto en pantalla. El modo avión (paso 3) **no** se probó.
- **Resultado real (después):** el menú completo aparece en español (ANFITRIÓN, BUSCAR RIVAL, UNIRSE, «IP del anfitrión»); el campo muestra error con el mensaje **«IP no válida. Ejemplo: 192.168.1.5»** y UNIRSE queda apagado. Cumple lo esperado para la cadena en español `sf_lan_ip_invalid`. Se observa de nuevo OBS-01 (el error de 2 líneas deja «Cancelar» al borde inferior).
- **Estado:** **Aprobado parcialmente** (idioma español comprobado; comportamiento sin red/modo avión **no ejecutado**) · **Evidencia:** [idioma ES](../evidencias/TC-07_despues_01_idioma_es_ip_no_valida.png) · **Defecto:** ninguno nuevo · **Decisión:** se acepta con reserva (ver §7).

### TC-08 — Automatizado: pruebas unitarias y verificaciones locales
- **Cubre:** CA-1, CA-2, CA-3, R1 · **Autor/fecha:** `PENDIENTE` · **SHA:** `PENDIENTE`
- **Pasos:** ejecutar desde `PolitecnicoOpenWorld/PolitecnicoOpenWorld`: `bash tools/check_kmp_test_names.sh` y `./gradlew :app:assembleDebug :app:testDebugUnitTest :shared:testAndroidHostTest --stacktrace`; luego detekt (ver `LEEME_PRIMERO.md` §6).
- **Esperado:** `BUILD SUCCESSFUL`; `SfLanAddressTest` con **11** pruebas en verde; detekt sin issues nuevos.
- **Resultado real (1-oct-2026, SHA `c9647041c1da769ddc2722e409850206f93fe2cb`):** `check_kmp_test_names.sh` OK; `./gradlew :app:assembleDebug :app:testDebugUnitTest :shared:testAndroidHostTest --stacktrace` → **BUILD SUCCESSFUL in 1m 21s**; `:app` 136 pruebas (0 fallos, incluye las **11** de `SfLanAddressTest`), `:shared` 217 pruebas (0 fallos); detekt local (CLI del repo, config + baseline) sin issues nuevos. Los warnings del log son preexistentes.
- **Estado:** Aprobado · **Evidencia:** [log completo](../evidencias/TC-08_gradle.txt), [resumen](../evidencias/TC-08_resumen.txt) · **Nota:** una captura de `BUILD SUCCESSFUL` **no** sustituye a TC-01…TC-07.

## 4. Matriz resumen (llenar al terminar)

| Caso | Categoría | Criterio/riesgo | ANTES (base) | DESPUÉS | Estado | Evidencia |
|---|---|---|---|---|---|---|
| TC-01 | Ruta feliz | CA-1, R1 | JOIN activo → error de red | JOIN activo → mismo error de red | Aprobado | TC-01_antes_*, TC-01_despues_* |
| TC-02 | Límite / defecto | CA-2 | **Defecto:** JOIN activo con `...` | JOIN apagado | Aprobado | TC-02_antes_*, TC-02_despues_01 |
| TC-03 | Límite / entrada inválida | CA-2, CA-3, R6 | **Defecto:** JOIN activo con `a.b.c.d` y `999…` | `999…`/`1.2.3.4.5` con error y JOIN apagado; `a.b.c.d` y pegados cubiertos por pruebas unitarias | Aprobado | TC-03_antes_*, TC-03_despues_* |
| TC-04 | Límite / valores extremos | CA-1, CA-2, R1 | n/a | `192.168.1.` sin error; `.500` error; `255.255.255.255` válido | Aprobado | TC-04_despues_* |
| TC-05 | Regresión | R2 | AB1/AB12/CREATE SERVER | AB1 y CREATE SERVER idénticos | Aprobado parcial | TC-05_antes_*, TC-05_despues_* |
| TC-06 | Navegación y estado | R3 | — | Atrás + reentrada: campo limpio, sin cierres (rotación/segundo plano no probados) | Aprobado parcial | TC-06_despues_* |
| TC-07 | Compatibilidad/entorno | R5, CA-1 | — | Español: mensaje «IP no válida…» y UNIRSE apagado (sin red no probado) | Aprobado parcial | TC-07_despues_01 |
| TC-08 | Automatizado | CA-1…3, R1 | n/a | Aprobado (136 `:app` + 217 `:shared`, 0 fallos) | Aprobado | [resumen](../evidencias/TC-08_resumen.txt) |

## 5. Hallazgos

| ID | Descripción | Pasos | Esperado vs observado | Severidad | Evidencia | Estado |
|---|---|---|---|---|---|---|
| DEF-01 | Base: UNIRSE se habilita con `...` / `a.b.c.d` / `999.999.999.999` (sólo cuenta 3 puntos) | TC-02, TC-03 | Esperado: deshabilitado. Observado (TC-02, base): con `...` JOIN habilitado → CONNECTING… ≈3 s → error de red. `a.b.c.d` y `999.999.999.999` (TC-03, base): JOIN también habilitado y se intenta conectar → error de red (reproducido) | Baja–media (error de entrada que lleva a una conexión inútil con espera; no hay pérdida de datos) — **justificar con lo observado** | `PENDIENTE` | `PENDIENTE (corregido en la rama si TC-02 DESPUÉS aprueba)` |
| OBS-01 | Introducido por el cambio: con el error visible, el texto de ayuda (*"Invalid IP. Example: 192.168.1.5"*) ocupa 2 líneas bajo el campo (ancho 180 dp); el contenido baja y **Cancel** queda parcialmente tapado por la barra de gestos del AVD en horizontal hasta deslizar (el menú tiene scroll); JOIN se desplaza unos píxeles hacia abajo | TC-03/TC-04 (capturas DESPUÉS) | Esperado: mensaje sin tapar controles. Observado: Cancel parcialmente fuera de vista sin scroll | Baja (cosmético, hay scroll) | [999…](../evidencias/TC-03_despues_01_999_error_join_deshabilitado.png) | **Pendiente** (se documenta; posible mejora futura: mensaje más corto o ancho mayor) |
| DEF-02 | Mensaje de error de conexión LAN mezcla idiomas: con la UI en inglés muestra *"Red local: no se pudo conectar (...)"* (texto fijo en español en `SfLanClient.kt`) | TC-02 (grabación 11.58.31) | Esperado: texto en el idioma de la UI. Observado: español en UI inglesa | Baja (cosmético / i18n) | grabación `11.58.31` | **Preexistente, fuera de alcance** (no se corrige en este PR) |
| — | Cualquier otro hallazgo real encontrado durante la ejecución | | | | | preexistente / pendiente / corregido |

> Si en la ejecución no aparece algún defecto, se documenta qué se probó y qué limitaciones quedan. No inventar fallos.

## 6. Re-ejecución tras correcciones

Si un caso falla o cambia el código después del QA: repetir el caso fallido y la regresión afectada (TC-05),
conservar el resultado inicial y el nuevo, y actualizar la matriz. Registrar la relación **SHA probado → SHA final**:
SHA probado = SHA final entregado = `c9647041c1da769ddc2722e409850206f93fe2cb` (sin commits posteriores al QA; la documentación vive fuera del diff del PR).

## 7. Cierre de calidad (llenar al final)

- ¿Se recomienda integrar el cambio? **Sí, con reservas.** Evidencia: el defecto se reprodujo en la base (TC-02/TC-03, capturas y video) y desaparece con el cambio; los valores válidos siguen aceptados (TC-01, TC-04); `SfLanAddressTest` 11/11 y `:app` 136 + `:shared` 217 pruebas sin fallos, detekt sin issues nuevos (TC-08). Reservas: la accesibilidad con texto ampliado no se probó (riesgo R4 aceptado: el cambio no modifica fuentes ni tamaños de texto), y TC-05, TC-06 y TC-07 quedaron parciales; no se probó la unión a un anfitrión LAN real; CI de GitHub pendiente de registrar.
- Riesgos que permanecen (conocidos): OBS-01 (el mensaje de error de dos líneas empuja el contenido y deja Cancel parcialmente fuera de vista sin deslizar); puerto no soportado al pegar `ip:puerto` (R6); no se probó la unión
  real a un anfitrión LAN (requiere 2.º dispositivo); la copia `shared/…/StreetFighterScreen.kt` conserva la lógica anterior
  (fuera de alcance, Android evaluado); el texto del campo no sobrevive a recreación de actividad (igual que antes del cambio).
