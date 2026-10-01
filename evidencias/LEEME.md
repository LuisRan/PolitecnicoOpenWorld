# Evidencias

Nombra los archivos `TC-xx_antes_nn.*` y `TC-xx_despues_nn.*` (png, mp4, txt). Cada evidencia debe poder
responder: **qué se hizo, qué se observó, qué SHA y qué entorno**. Anótalo en la ficha del caso en `../docs/pruebas.md`
y enlázalo ahí.

Comandos útiles (Mac, con `adb` en el PATH):

```bash
adb exec-out screencap -p > TC-02_antes_01.png
adb shell screenrecord /sdcard/tc06.mp4        # Ctrl+C para parar; luego: adb pull /sdcard/tc06.mp4 TC-06_despues_01.mp4
adb logcat -d -s SF-NET > TC-02_antes_logcat.txt
```

Nada de capturas sólo de `BUILD SUCCESSFUL` como prueba de comportamiento. No subas `local.properties`, claves ni tokens.
