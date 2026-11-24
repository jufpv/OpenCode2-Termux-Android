# FluxLinux 2.0.1 — correctif du serveur X

APK corrigé de [FluxLinux v2.0.1](https://github.com/zenithblue-oss/fluxlinux/releases/tag/v2.0.1) (`com.ivarna.fluxlinux`). Le bureau ne démarrait pas : le serveur X intégré (Termux-X11) quittait avant de créer le socket `X0`.

## Cause

Le journal s’arrête ici :

```text
Loader: Loading classes from .../base.apk
Failed to load native library. Did you install the right apk? Try the universal one.
FluxLinux: [ERROR] X server did not create .../tmp/.X11-unix/X0 within 15s (or exited).
```

Ce n’est pas un mauvais ABI ni un problème VirGL. `CmdEntryPoint` charge `libXlorie.so` (le serveur X) puis abandonne. Dans l’APK v2.0.1 :

1. `libXlorie.so` est **compressé** dans `lib/arm64-v8a/` (`extractNativeLibs=true`, pour pouvoir exécuter `libbash.so` / `libproot.so`). Le linker ne peut pas mapper une entrée zip compressée, donc le chemin `base.apk!/lib/arm64-v8a/libXlorie.so` échoue.
2. Le code bascule alors vers `/data/data/com.ivarna.fluxlinux/lib/libXlorie.so`, une copie **inscriptible** dans les données de l’application. Avec `targetSdk` 36, Android refuse `System.load()` sur un fichier inscriptible (`Attempt to load writable file`). SELinux refuse aussi d’exécuter un `app_data_file`.

Le même échec est rapporté sur les issues [#54](https://github.com/zenithblue-oss/fluxlinux/issues/54) et [#59](https://github.com/zenithblue-oss/fluxlinux/issues/59). L’audio et VirGL, plus haut dans le journal, ne sont pas en cause.

## APK

| | |
| --- | --- |
| Fichier | `dist/FluxLinux-2.0.1-x11fix.apk` |
| SHA-256 | `9c7d02884a067fb8e79436a863e7b88fd8f7c9a01b016f19493a09e62cc67b78` |
| Signature SHA-256 | `A6:F0:1B:DA:FE:21:4F:CA:E1:A8:AD:8B:59:B4:00:3C:D4:15:CB:5E:E5:D5:B0:4A:37:A9:8A:52:E1:60:BC:BB` |

La signature n’est pas celle du projet upstream. Android refuse d’installer cet APK par-dessus la version officielle.

### Installation

1. Désinstaller FluxLinux. **Cela efface la distribution déjà installée** (`/data/data/com.ivarna.fluxlinux`, y compris le rootfs). Une sauvegarde `adb backup` ne se restaure pas avec une autre signature.
2. Installer `dist/FluxLinux-2.0.1-x11fix.apk`.
3. Rouvrir l’application et réinstaller la distribution, puis lancer le bureau.

Au démarrage, le journal doit contenir une ligne `FluxLinux: loaded libXlorie.so from ...` puis `host X0 socket ready`. S’il échoue encore, la ligne `FluxLinux: dlopen failed for ...` donne l’erreur réelle du linker (l’ancien message masquait cette cause).

## Correctif

- `libXlorie.so` est stocké non compressé et aligné sur 16 Ko, pour que le linker puisse le mapper dans `base.apk`.
- `CmdEntryPoint` essaie, dans l’ordre : cette entrée de l’APK, `TERMUX_X11_LIB_PATH`, la bibliothèque déjà extraite à côté de `base.apk` (`lib/arm64/libXlorie.so`, lecture seule), puis en dernier recours la copie sous `/data/data` après `chmod 0444`.
- `start_gui.sh` (proot) et `start_gui_chroot.sh` ne copient plus la bibliothèque dans un dossier inscriptible.

Les autres `.so` (`libbash.so`, `libproot.so`, Pulse) restent compressés : ils doivent continuer à être extraits pour être exécutés.

Sources : tag upstream `v2.0.1` (GPL-3.0) et le correctif dans `tools/` (`x11_load_methods.smali`, `build_fixed_apk.py`).

## Reconstruire

À partir de l’APK officiel `app-release.apk` :

```bash
export FLUX_SRC_APK=/chemin/vers/app-release.apk
python3 tools/build_fixed_apk.py
```

Il faut `baksmali` / `smali` 3.0.10, `zipalign` et `apksigner` (build-tools 35) sur `PATH` ou via les variables `BAKSMALI`, `SMALI`, `ZIPALIGN`, `APKSIGNER`.
