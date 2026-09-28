# Schnellstart: Tang Nano 20K v3923 mit BL616 und USB-Tastatur

Stand: 23.09.2026 · Befehle für macOS und das Terminal

Diese Anleitung installiert die vorhandene USB-Version für das Tang Nano 20K **v3923**. Der derzeit veröffentlichte FPGA-Bitstream enthält die neue F3-Funktion noch nicht, weil dafür ein neuer FPGA-Build mit Gowin EDA benötigt wird. openFPGALoader kann fertige Bitstreams übertragen, aber keine Bitstreams erzeugen.

Während eines Schreibvorgangs weder Stromversorgung noch USB-Verbindung trennen. Bei einer Fehlermeldung abbrechen und Ursache prüfen.

## Vorbereitung

Die folgenden Variablen an die eigenen Verzeichnisse anpassen:

```sh
REPO="/Pfad/zu/Z1013-Tang-Nano-20K"
BLFLASH="/Pfad/zu/BLFlashCommand-macos"
OPENFPGALOADER="/Pfad/zu/openFPGALoader"
BACKUP_DIR="$HOME/Documents/VHDL"
```

Der BL616-Flashbefehl verwendet auf Intel-Werkzeuge angewiesenen Macs gegebenenfalls Rosetta. Vorher kontrollieren, dass das Programm lokal ausführbar ist und seine Hilfe angezeigt werden kann.

## 1. BL616-Firmware für die USB-Tastatur

Wenn die passende Z1013-Companion-Firmware bereits installiert ist, kann dieser Abschnitt übersprungen werden.

### Downloadmodus und Port

1. Tang Nano 20K abziehen.
2. Den mit **UPDATE** beschrifteten BL616-Taster gedrückt halten.
3. Das Board direkt per USB-Datenkabel mit dem Mac verbinden.
4. UPDATE loslassen. Nicht den FPGA-Benutzertaster verwenden.
5. Den neu erschienenen Bootloader-Port anzeigen:

   ```sh
   ls /dev/cu.usbmodem*
   ```

Den zum Tang gehörenden Port einsetzen. `/dev/cu.usbmodem3101` ist nur ein Beispiel. Keine `usbserial`-Schnittstelle wählen.

```sh
BL616_PORT="/dev/cu.usbmodem3101"
```

### Vollständige Sicherung

Vor dem ersten Schreiben auf einem Board den kompletten 4-MiB-BL616-Flash sichern:

```sh
mkdir -p "$BACKUP_DIR"
"$BLFLASH" \
  --interface=uart \
  --baudrate=2000000 \
  --port="$BL616_PORT" \
  --chipname=bl616 \
  --flash --read \
  --start=0x0 \
  --len=0x400000 \
  --file="$BACKUP_DIR/BL616_Backup_$(date +%Y%m%d_%H%M%S).bin"
```

Nur nach erfolgreicher Sicherung fortfahren. Antwortet der Bootloader nicht mehr, das Board erneut mit gedrücktem UPDATE-Taster anschließen und den Port kontrollieren.

### Companion-Firmware schreiben

Die Repository-Konfiguration schreibt ausschließlich die Companion-Firmware ab Adresse `0x40000`. Dazu in das Release-Verzeichnis wechseln, damit `secondary_only.ini` die nebenliegende Datei `companion_z1013_v3923.bin` findet:

```sh
cd "$REPO/release"
"$BLFLASH" \
  --interface=uart \
  --baudrate=2000000 \
  --port="$BL616_PORT" \
  --chipname=bl616 \
  --cpu_id= \
  --config="secondary_only.ini"
```

**Kein `--whole_chip` verwenden.** Der ursprüngliche BL616-Bereich unterhalb `0x40000` muss erhalten bleiben. Erfolgreiche Programmierung und SHA-Prüfung im Protokoll abwarten. Danach das Board abziehen und ohne UPDATE wieder verbinden.

SHA-256 der vorgesehenen v3923-Companion-Firmware:

```text
79c05c7b2260a70897cba16da0e03695e2c12516dc461178006604ca7f1f8441  companion_z1013_v3923.bin
```

## 2. FPGA-Bitstream mit USB-Tastatur

Das Tang Nano 20K normal und ohne UPDATE-Taster mit dem Mac verbinden. Gowin Programmer schließen, damit nur ein Programm auf den Adapter zugreift.

Zuerst das Board erkennen:

```sh
"$OPENFPGALOADER" -b tangnano20k --detect
```

Nur wenn das passende Board erkannt wurde, den fertigen Bitstream dauerhaft in den FPGA-Konfigurationsflash schreiben und prüfen:

```sh
"$OPENFPGALOADER" \
  -b tangnano20k \
  -f --external-flash --verify \
  --offset 0x000000 \
  "$REPO/release/z1013_usb_v3923.fs"
```

SHA-256 des derzeit veröffentlichten USB-Bitstreams:

```text
c71664c4957aa939212a7a2673d739e10e2fceecfcf3d0653f4cd38c4ab7592f  z1013_usb_v3923.fs
```

Nach erfolgreichem Schreiben **und** erfolgreicher Verifikation das Board kurz abziehen und wieder verbinden. Die USB-Tastatur über einen geeigneten USB-C-Hub anschließen.

Im vorhandenen Bitstream gelten:

- F1: Verzeichnis anzeigen
- F2: Datei laden
- F9 bis F12: CPU-Geschwindigkeit

F3 steht erst in einem später neu erzeugten und getesteten FPGA-Bitstream zur Verfügung. Dafür im letzten Befehl lediglich den Pfad zur neuen `.fs`-Datei einsetzen.

## Weitere Informationen

- [Ausführliche BL616-Anleitung](README.md)
- [USB-Tastatur und Einschränkungen](../docs/USB_KEYBOARD.md)
- [BL616-Companion-Anpassungen](../docs/BL616_COMPANION.md)
- [openFPGALoader-Anleitung](../docs/OPENFPGALOADER.md)
- [Wiederherstellung](recovery/README.md)

Mit der Erstellung dieser Anleitung wurde kein Board programmiert.
