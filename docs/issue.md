# Texto de la issue (para abrir en TU fork)

**Título:** `Street Fighter LAN: el botón UNIRSE se habilita con IPs inválidas (...) / (a.b.c.d)`

**Cuerpo:**

## Comportamiento actual
En *TITULACIÓN POR COMBATE → MULTIJUGADOR (menú de modos) → "…o únete tecleando la IP"* el botón **UNIRSE** se habilita
cuando el texto tiene exactamente 3 puntos (`SfOnlineOverlays.kt`: `lanIp.count { it == '.' } == 3`).
Con `...`, `a.b.c.d` o `999.999.999.999` el botón queda activo y, al pulsarlo, la app inicia una conexión que
no puede funcionar (3 intentos con pausa) antes de mostrar el error de red.

## Comportamiento esperado
UNIRSE sólo se habilita con una IPv4 válida (4 octetos 0–255). El campo sólo admite dígitos y puntos.
Si el texto parece completo pero es inválido se muestra un mensaje de error.

## Usuario afectado
Jugador que se une por LAN tecleando la IP del anfitrión y se equivoca al escribirla.

## Archivos previstos
`SfOnlineOverlays.kt`, `SfLanAddress.kt` (nuevo), `values/strings.xml`, `values-en/strings.xml`,
`SfLanAddressTest.kt` (nuevo).

## Fuera de alcance
IPv6, nombres de host, puerto personalizado, autodescubrimiento LAN, código de sala, mapas, servidores.

## Criterios de aceptación
- [ ] **CA-1 (éxito):** con `192.168.1.5` UNIRSE está habilitado y al pulsarlo inicia la conexión.
- [ ] **CA-2 (límite):** con `...`, `999.999.999.999`, `1.2.3.4.5` o `192.168.1` UNIRSE está deshabilitado, no se inicia
      conexión y, si el texto parece completo e inválido, se muestra el mensaje de error.
- [ ] **CA-3 (límite):** letras, espacios y símbolos se descartan; máximo 15 caracteres.

## Verificación previa
Reproducido en la versión base `7ed325393f82872c2be94ff2ada46948efa19152` (AVD Pixel 8, API 37.1): con `...` y con `999.999.999.999` el botón UNIRSE aparece habilitado; al pulsarlo se intenta conectar y aparece la pantalla de error de red («NO WI-FI CONNECTION»). Capturas: adjuntar `evidencias/TC-02_antes_01_campo_vacio_join_deshabilitado.png`, `TC-02_antes_02_puntos_join_habilitado.png` y `TC-02_antes_03_error_tras_join.png` arrastrándolas al cuerpo de la issue.

> Antes de abrir: busca en las issues del proyecto original (`gabrielhuav/PolitecnicoOpenWorld`) si ya existe una
> similar para no duplicar. Enlázala desde el PR con su URL completa.
