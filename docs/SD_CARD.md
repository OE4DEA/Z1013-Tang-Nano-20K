# FAT32-microSD und Z1013-Dateien / FAT32 microSD and Z1013 files

[Deutsch](#deutsch) · [English](#english)

## Deutsch

Die Karte benötigt eine MBR-Partitionstabelle und eine FAT32-Partition. Dateien liegen ohne Unterverzeichnisse im Wurzelverzeichnis. Kurze FAT-Namen im Format 8.3 sind am zuverlässigsten.

## 9-Byte-Kopf

Vor den eigentlichen Nutzdaten steht:

| Byte | Inhalt |
|---|---|
| 0–2 | ASCII `@DD` (`40 44 44`) |
| 3–4 | Anfangsadresse, Low-Byte zuerst |
| 5–6 | Endadresse einschließlich, Low-Byte zuerst |
| 7–8 | Startadresse, Low-Byte zuerst |
| ab 9 | Nutzdaten |

Der Z1013 zeigt den Namen aus dem FAT32-Verzeichniseintrag an. Nur `.COM`-Dateien werden nach dem Laden automatisch gestartet; andere Dateiendungen dürfen auch ohne ausführbare Startadresse verwendet werden.

## Monitorbefehle

- `@DD` – Verzeichnis anzeigen
- `@DL` – Datei laden; `.COM` wird automatisch gestartet
- `@DS ANFANG ENDE START` – Speicherbereich als Datei sichern
- `@DK` – Datei nach Sicherheitsabfrage löschen

Die Verzeichnisanzeige pausiert passend zum 32×32-Zeichenbild und kann mit Enter fortgesetzt oder mit Esc beendet werden.

`@DS` und `@DL` unterstützen `0000h` bis einschließlich `D000h`.
Beispiel: `@DS 0000 D000`, anschließend einen freien Namen wie `SPEICHER.BIN`
eingeben. Die Datei enthält 53.249 Nutzbytes und den 9-Byte-Kopf. `.BIN` wird
beim Laden nicht automatisch ausgeführt. Bei `.COM` muss zusätzlich eine
Startadresse innerhalb des gespeicherten Bereichs angegeben werden.

Der Monitor-Stack wird während des Speicherns und Ladens aus der niedrigen
Speicherseite nach `DF00h..DFFFh` verlegt. Dieser Arbeitsbereich darf dabei
nicht anderweitig verwendet werden. `D000h` ist jetzt normales RAM; der
schreibgeschützte SD-Testzugang beginnt bei `D001h`, der vollständige
SD-Puffer bleibt bei `E100h..E2FFh`.

Gesichert wird der aktuell eingeblendete Speicher. Im Grafikmodus betrifft
`B000h..CFFFh` den ausgewählten Bild- oder Farbspeicher. Zum Zurückladen muss
derselbe Modus bzw. dieselbe Bank eingestellt sein. Diese Funktion ist kein
vollständiger Snapshot mit allen Speicherbänken, CPU- und Hardware-Zuständen.

## Mitgelieferte Spiele

`PACMAN.COM`, `KIKSTART.COM` und `PUNIVERS.COM` besitzen bereits einen gültigen 9-Byte-Kopf. Sie werden unverändert in das Wurzelverzeichnis der FAT32-Karte kopiert. Das Werkzeug `@DS` darf auf diese fertigen Dateien nicht noch einmal angewendet werden, weil dadurch ein zweiter Kopf entstehen würde.

## TBDOS

Für TBDOS wird `programs/TBDOS/DOS.COM` in das Wurzelverzeichnis der Karte kopiert. Die übrigen Dateien aus `programs/TBDOS/` mit den Endungen `.BIN` und `.OVL` müssen im Verzeichnis `/DOS/` auf der Karte liegen. Die vollständige Anleitung steht in [programs/TBDOS/README.md](../programs/TBDOS/README.md).

---

## English

The card needs an MBR partition table and a FAT32 partition. Store files in the root directory without subdirectories. Short 8.3 FAT filenames are the most reliable choice.

### 9-byte header

Every Z1013 file starts with this header before its payload:

| Byte | Contents |
|---|---|
| 0–2 | ASCII `@DD` (`40 44 44`) |
| 3–4 | Load address, low byte first |
| 5–6 | Inclusive end address, low byte first |
| 7–8 | Start address, low byte first |
| 9 onward | Program data |

The Z1013 displays the name from the FAT32 directory entry. Only `.COM` files start automatically after loading. Other extensions may be used without an executable start address.

### Monitor commands

- `@DD` – display the directory
- `@DL` – load a file; `.COM` files start automatically
- `@DS BEGIN END START` – save a memory range as a file
- `@DK` – delete a file after confirmation

The directory display pauses to fit the 32 × 32 character screen. Press Enter to continue or Esc to stop.

`@DS` and `@DL` support `0000h` through `D000h`, inclusive. For example,
enter `@DS 0000 D000`, then an unused name such as `MEMORY.BIN`. The file
contains 53,249 payload bytes plus the 9-byte header. `.BIN` is not executed
after loading; `.COM` requires a start address inside the saved range.

The live monitor stack is temporarily moved to `DF00h..DFFFh` while saving
or loading. Reserve this scratch area. `D000h` is ordinary RAM; the read-only
SD diagnostic alias starts at `D001h`, while the full sector buffer remains
at `E100h..E2FFh`.

The file contains the currently mapped memory. In graphics mode,
`B000h..CFFFh` selects video or colour RAM; restore with the same mode and
bank selected. This is not a complete snapshot of all memory banks, CPU
registers or hardware state.

### Included games

`PACMAN.COM`, `KIKSTART.COM`, and `PUNIVERS.COM` already contain a valid 9-byte header. Copy them unchanged to the root directory of the FAT32 card. Do not process these ready-made files with the `@DS` tool again, because that would add a second header.

### TBDOS

For TBDOS, copy `programs/TBDOS/DOS.COM` to the card root. The remaining `.BIN` and `.OVL` files from `programs/TBDOS/` must be stored in the card's `/DOS/` directory. See [programs/TBDOS/README.md](../programs/TBDOS/README.md) for the complete instructions.
