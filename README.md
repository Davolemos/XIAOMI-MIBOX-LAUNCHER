# XIAOMI-MIBOX-LAUNCHER

Quitar la publicidad y el bloatware de una **Xiaomi Mi Box S** y poner una pantalla de inicio
estilo Apple TV, por Wi‑Fi, sin root y sin desinstalar nada (todo se puede deshacer).

Basado en [CleanYourTV](https://github.com/Gathin23/cleanyourtv) (licencia MIT, ver `LICENSE`),
recortado a lo que necesita una Mi Box:

| Carpeta | Qué hay |
|---|---|
| `launcher/` | El launcher Android (Java). `launcher/app/src/main/res/values/config.xml` ya viene ajustado para Mi Box |
| `tools/` | Scripts para desactivar / restaurar paquetes y hablar con la Mi Box por ADB |
| `examples/xiaomi-mi-box-s/` | Lista de bloatware de la **Mi Box S 1ª gen** (MDZ-22-AB, Android TV 9) |
| `examples/xiaomi-mi-box-s-2nd-gen/` | Lista de la **Mi Box S 2ª gen** (MiTV-AFKR0, Google TV 11) |
| `themes/` | Temas (fondos, colores) que se cambian sin recompilar |
| `PROMPT.md` | El prompt original para hacerlo con Claude Code paso a paso |

> **¿Cuál Mi Box tengo?** Ajustes → Preferencias del dispositivo → Información → Modelo.
> `MIBOX4` / `MDZ-22-AB` = 1ª gen. `MiTV-AFKR0` (con Google TV) = 2ª gen.

## Importante: hay que hacerlo desde tu casa

La Mi Box solo es accesible desde tu red Wi‑Fi. Necesitas un ordenador (Mac, Windows o Linux)
**en la misma Wi‑Fi** que la Mi Box. No se puede hacer desde la nube.

La forma más fácil: abre **Claude Code en tu ordenador** (app de escritorio o terminal), clona este
repo y pega esto:

```
Clona este repo y úsalo. Quiero quitar el bloatware de mi Xiaomi Mi Box S <1ª/2ª gen> y poner
el launcher de launcher/. Sigue PROMPT.md, usa tools/ y examples/xiaomi-mi-box-s*/debloat.txt.
Pregúntame antes de desactivar apps que pueda usar.
```

Tarda unos 15 minutos. Si prefieres hacerlo a mano, sigue los pasos de abajo.

## 1. En la Mi Box

1. Ajustes → Preferencias del dispositivo → Información → pulsa OK sobre **Compilación** 7 veces.
2. Atrás → **Opciones de desarrollador** → activa **Depuración USB** (y "Depuración ADB/red" si aparece).
3. Anota la IP: Ajustes → Red e Internet → tu Wi‑Fi.
4. Aviso: mientras la depuración está activa, el puerto USB de la Mi Box se queda **sin corriente**
   (un disco o altavoz USB deja de funcionar). Vuelve al normal al apagar la depuración.

## 2. En tu ordenador

Necesitas Python 3, Java 17 y el Android SDK (lo instala Android Studio).

```bash
git clone https://github.com/davolemos/xiaomi-mibox-launcher.git
cd xiaomi-mibox-launcher
python3 -m venv tools/venv && tools/venv/bin/pip install -r tools/requirements.txt
export TV_HOST=192.168.1.50          # la IP de tu Mi Box
tools/venv/bin/python tools/tv.py 'getprop ro.product.model'   # acepta "¿Permitir depuración USB?" en la tele
```

## 3. Elige tu dock

Edita `launcher/app/src/main/res/values/config.xml`, sección `dock_apps`. Viene con:
Netflix, YouTube, Disney+, Spotify. Para ver los nombres de paquete de tus apps:

```bash
tools/venv/bin/python tools/tv.py 'pm list packages -3'
```

## 4. Quitar bloatware

Abre la lista de tu modelo y **comenta con `#`** cualquier línea que quieras conservar
(por ejemplo Chromecast `mediashell`, el asistente de voz `katniss`, o Prime Video).

```bash
# 1ª gen
tools/disable.sh examples/xiaomi-mi-box-s/debloat.txt
# 2ª gen
tools/disable.sh examples/xiaomi-mi-box-s-2nd-gen/debloat.txt
```

El script guarda una copia del estado antes de tocar nada y se salta el launcher original hasta
que el nuevo esté instalado.

## 5. Instalar el launcher nuevo

```bash
launcher/build-and-install.sh
```

Luego **vuelve a ejecutar el paso 4** para desactivar el launcher original (y en la 2ª gen el
asistente `setupwraith`). En la 2ª gen puede que la Mi Box te pida otra vez la contraseña del Wi‑Fi
una vez; es normal.

Si el botón Home del mando no abre el launcher: `HOME_KEY_SERVICE=1 launcher/build-and-install.sh`.

## 6. Cerrar al terminar (seguridad)

En la Mi Box: desactiva **Depuración USB**, revoca las autorizaciones de depuración y reinicia.
El puerto 5555 queda abierto en tu red mientras la depuración esté activa.

## Deshacer todo

```bash
# 1ª gen
tools/restore.sh examples/xiaomi-mi-box-s/debloat.txt
# 2ª gen
tools/restore.sh examples/xiaomi-mi-box-s-2nd-gen/debloat.txt com.google.android.apps.tv.launcherx
```

Para reactivar una sola app: `tools/venv/bin/python tools/tv.py 'pm enable com.paquete.de.la.app'`.

Problemas frecuentes: `docs/TROUBLESHOOTING.md`.
