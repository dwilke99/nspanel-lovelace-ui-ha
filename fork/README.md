# Fork-Material

Dieser Ordner enthält Dateien, die es nur in diesem Fork gibt. Der übrige Repository-Inhalt folgt dem Original `joBr99/nspanel-lovelace-ui`. Wer Änderungen vom Original übernimmt, muss hier nichts zusammenführen.

## `n2t-out-61/`

Textauszüge der Display-Firmware 61 (Release 5.1.1) aus der ioBroker-Linie. Sie entsprechen `HMI/n2t-out/` für die HA-Firmware 53 und zeigen, wie die Firmware Nachrichten auswertet und welche Events sie sendet.

- Quelle: `HMI/nspanel-v5.1.1.HMI` aus `ticaki/ioBroker.nspanel-lovelace-ui`, Commit `524db23` vom 23.09.2026 (MIT-Lizenz).
- Werkzeug: `linux/Nextion2Text.py` aus `joBr99/Nextion2Text`, Commit `b0ba044`.
- Aufruf wie im Workflow `.github/workflows/nextion2text.yml` für `HMI/n2t-out/`:

  ```sh
  python Nextion2Text.py -c ignore-id.py -p font -d -i nspanel-v5.1.1.HMI -o n2t-out-61
  ```

  `ignore-id.py` ist die base64-kodierte Datei aus demselben Workflow.

Kontrolle: Derselbe Aufruf auf `HMI/nspanel.HMI` erzeugt eine Ausgabe, die mit `HMI/n2t-out/` identisch ist. Unterschiede zwischen den beiden Ordnern sind also echte Firmware-Unterschiede.

Die Dateien werden nicht von Hand bearbeitet. Bei einer neuen Zielversion den Ordner neu erzeugen.

## `protokollvergleich-53-61.md`

Vergleich der Nachrichtenformate zwischen Firmware 53 und 61, Seite für Seite, mit Einstufung „unverändert", „anpassen" oder „neu".
