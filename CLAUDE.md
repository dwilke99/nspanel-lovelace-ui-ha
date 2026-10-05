# NSPanel Lovelace UI – eigener Home-Assistant-Fork

Startkontext für Claude Code. Die Datei entstand am 5. Oktober 2026 aus der Übergabe eines längeren Chats, in dem ein Sonoff NSPanel von ioBroker auf Home Assistant umgezogen wurde. Alles Nötige steht hier.

Details zum Heimnetz (Adressen, Topics, Benutzer, lokale Konfiguration) gehören nicht in dieses öffentliche Repository und werden hier bewusst nicht aufgeführt.

## 1. Worum es geht

Dirk betreibt ein Sonoff NSPanel (EU-Modell) mit Tasmota und der Oberfläche „NSPanel Lovelace UI". Früher lief es an ioBroker, jetzt an Home Assistant. Dabei hat sich gezeigt:

- Die Home-Assistant-Variante von joBr99 bekommt nur noch Fehlerkorrekturen. Display-Firmware und Funktionsumfang stehen seit 2024 still.
- Die ioBroker-Variante wird aktiv weiterentwickelt und hat eine deutlich neuere Display-Firmware (5.x).

**Ziel:** ein eigener Fork des Home-Assistant-Backends, der mit der neueren Display-Firmware aus der ioBroker-Linie zusammenarbeitet. Dirk ist Eigentümer des Forks und testet am echten Panel. Claude schreibt den Code.

## 2. Getroffene Entscheidungen

- **Eigener Fork auf GitHub** von `joBr99/nspanel-lovelace-ui`, öffentlich, als persönliches Projekt ohne Support-Zusage. Die Lizenz bleibt GPL-3.0.
- **Name des Forks:** `nspanel-lovelace-ui-ha`, angelegt als `dwilke99/nspanel-lovelace-ui-ha`. Abgezweigt von `main` des Originals auf Stand `1b08656` (06.08.2026).
- **Nur das Repository umbenennen.** Der Ordner `apps/nspanel-lovelace-ui` und die Datei `nspanel-lovelace-ui.py` behalten ihre Namen, damit Änderungen vom Original sauber übernommen werden können und Dirks `apps.yaml` gültig bleibt.
- **`hacs.json`:** Anzeigename „NSPanel Lovelace UI Backend (Fork)", damit er sich vom Original („NSPanel Lovelace UI Backend") unterscheidet.
- **Display-Firmware und Berry-Treiber bleiben beim ioBroker-Team.** Der Fork ändert daran nichts, sondern legt sich auf eine Version fest und hebt sie nur bewusst an.
- **Ziel-Firmware ist 61** (Release 5.1.1 der ioBroker-Linie), festgelegt am 5. Oktober 2026. Startmeldung `event,startup,61,eu,5.1.1`. Das ioBroker-Skript im Repository (`ioBroker/NsPanelTs.ts`) zielt auf dieselbe Version und gehört zum Berry-Treiber 10.
- **Entwicklung in Claude Code (Cloud), Betrieb über einen separaten Chat.** Siehe Abschnitt 6.
- **Änderungen am Upstream-Code klein halten.** Je weniger Dateien des Originals der Fork anfasst, desto leichter lassen sich Korrekturen von joBr99 übernehmen.

Noch offen:

- Ob der Fork auf dem AppDaemon-Backend (`apps/`) aufbaut oder auf dem neuen Add-on (`nspanel-lovelace-ui/`). Bisherige Empfehlung: AppDaemon-Backend, weil es bei Dirk läuft und das Add-on laut Entwickler Alpha ist.

## 3. Was über die Repositories bekannt ist

Alle Angaben in diesem Abschnitt stammen aus den Git-Historien, abgerufen am 5. Oktober 2026.

### joBr99/nspanel-lovelace-ui (GPL-3.0)

| Ordner | Inhalt | Aktivität |
|---|---|---|
| `apps/nspanel-lovelace-ui/` | AppDaemon-Backend für Home Assistant, Unterordner `luibackend/` | 7 Commits seit v4.7.4, zuletzt 26.07.2026 |
| `HMI/` | Quellen der HA-Display-Firmware, Textauszüge unter `HMI/n2t-out/` | letzte inhaltliche Änderung November 2024 |
| `tasmota/` | Berry-Treiber `autoexec.be` | unverändert seit September 2023 |
| `nspanel-lovelace-ui/` | Neuschreibung als Home-Assistant-Add-on ohne AppDaemon, Version 4.7.91 | 17 Commits in 2026, zuletzt April |
| `ioBroker/` | TypeScript-Skript für ioBroker, HMI-Quellen nur bis 4.5.0 | sehr aktiv |
| `docs/`, `docs-standalone/` | Dokumentation zu AppDaemon-Backend und Add-on | |

- Releases: v4.7.4 (02.08.2025), v4.7.3 (31.07.2025), v4.7.2 (12.06.2025), v4.7.0 (28.05.2025).
- Im Backend steht fest: `desired_display_firmware_version = 53`, `desired_tasmota_driver_version = 8` (`apps/nspanel-lovelace-ui/nspanel-lovelace-ui.py`). Der Versionsstring im Code lautet `v4.7.3`, auch im Release v4.7.4.
- Auf `main`, aber in keinem Release: „Use kelvin instead of Mireds for color_temp (#1435)" vom 17.04.2026 und „Fix off-by-one crash in forecast (#1440)" vom 24.05.2026. Beide sind im Fork enthalten.
- Das Add-on wird vom Entwickler selbst als Alpha bezeichnet. Nutzer berichten in Issue #1058 von Hängern und Speicherlecks. Es nutzt eine eigene Konfigurationsdatei `panels.yaml`.

### ticaki/ioBroker.nspanel-lovelace-ui (MIT)

- Aktueller ioBroker-Adapter, Release v1.1.2 vom 18.09.2026.
- `HMI/` enthält die Firmware-Quellen bis `nspanel-v5.1.1.HMI`, dazu US-Varianten. Das sind Dateien des Nextion-Editors, vermutlich binär. Ob es Textauszüge gibt, ist nicht geprüft.
- `HMI/Readme.md` dokumentiert das Protokoll zwischen Backend und Display.
- `tasmota/berry/9`, `10` und `11` enthalten die Berry-Treiber dieser Linie.
- Seitentypen im Adapter unter `src/lib/pages/`: Alarm, Chart (Balken und Linie), Entities, Grid, Media, Menu, Popup, Power, QR, Schedule, Thermo, Thermo2, Screensaver.

### Protokoll, soweit bekannt

- Transport ist in beiden Linien gleich: Backend sendet an `cmnd/<topic>/CustomSend`, das Display antwortet auf `tele/<topic>/RESULT` als `{"CustomRecv":"..."}`.
- Startmeldung der HA-Firmware: `event,startup,53,eu`. Startmeldung der ioBroker-Firmware 5.0.0: `event,startup,59,eu,5.0.0`.
- Die 5.x-Firmware kennt zusätzliche Seitentypen (`screensaver2`, `screensaver3`) und einen zusätzlichen Parameter bei `dimmode` für ein neues Licht-Popup.
- Firmware 61 ist Release 5.1.1. Belegt durch `fork/n2t-out-61/pageStartup.txt` (`tVersion` = 61, `tRelease` = 5.1.1) und `ioBroker/NsPanelTs.ts` (`desired_display_firmware_version = 61`, `tft_version = 'v5.1.1'`).
- Textauszüge der Firmware 61 liegen unter `fork/n2t-out-61/`, Herkunft in `fork/README.md`. Der Vergleich der Nachrichtenformate 53 gegen 61 steht in `fork/protokollvergleich-53-61.md`.

### Orientierung im Code (AppDaemon-Backend)

- `apps/nspanel-lovelace-ui/nspanel-lovelace-ui.py`: Einstieg, Klasse `NsPanelLovelaceUIManager`, gewünschte Firmware- und Treiberversion.
- `luibackend/mqtt.py`: Empfang von `CustomRecv`, Auswertung von `event,startup,...` und den übrigen Ereignissen.
- `luibackend/updater.py`: Vorab-Check der Versionen, stößt bei Bedarf das Flashen an.
- `luibackend/pages.py`: baut die Nachrichten an das Display (`pageType`, `entityUpd`, `entityUpdateDetail`, ...). Hier wird die Anpassung an die 5.x-Firmware hauptsächlich stattfinden.
- `luibackend/controller.py`: Kartenwechsel, Tastendrücke, Zustandsänderungen aus Home Assistant.
- `HMI/n2t-out/*.txt`: Textauszüge der HA-Firmware 53, eine Datei pro Seite oder Popup. Maßgeblich dafür, wie die Firmware die Nachrichten auswertet.
- `fork/n2t-out-61/*.txt`: dasselbe für die Ziel-Firmware 61. `diff HMI/n2t-out/X.txt fork/n2t-out-61/X.txt` zeigt, was sich an einer Seite geändert hat.
- `ioBroker/NsPanelTs.ts` und ticakis Adapter (`src/lib/pages/`): funktionierende Backends für Firmware 61, nützlich als Vorlage für Nachrichtenformate.

## 4. Offene Fehler im aktuellen Betrieb

### Helligkeitsregler im Licht-Popup fehlt (ungeklärt)

- Dirk meldet: Beim Antippen von „Ankleide" erscheint im Popup kein Schieberegler.
- Die Entität ist eine Hue-Raumgruppe mit `supported_color_modes: [brightness]`. Eingeschaltet meldet sie Helligkeit 127.
- Das Backend sendet korrekt, im ausgeschalteten Zustand: `entityUpdateDetail~<id>~~17299~0~0~disable~disable~Color~Farbtemperatur~Helligkeit~disable`. Feld 5 ist die Helligkeit.
- Laut `HMI/n2t-out/popupLight.txt` blendet die Firmware den Regler nur aus, wenn Feld 5 den Wert `disable` hat. Außerdem verarbeitet das Popup die Nachricht nur, wenn die Entitätskennung mit der des geöffneten Popups übereinstimmt.
- Nach Datenlage müsste der Regler also erscheinen. Ein Foto oder eine Beschreibung des Popups steht von Dirk noch aus. Der Fehler ist nicht reproduziert.

### Farbtemperatur-Regler fehlt (Ursache bekannt, im Fork behoben)

- Betrifft Lampen mit `color_temp`, zum Beispiel `light.bad_wanne_decke_white`.
- Das Backend v4.7.4 liest `color_temp`, `min_mireds` und `max_mireds`. Home Assistant 2026.9 liefert nur noch die Kelvin-Attribute. Das Backend schickt deshalb `unknown`, und die Firmware zeigt den Regler nicht.
- Auf `main` des Originals ist das seit April 2026 behoben (#1435). Der Fork enthält die Korrektur (`color_temp_kelvin` in `luibackend/pages.py` und `luibackend/controller.py`). Am Panel noch nicht bestätigt.

## 5. Stolpersteine, die schon Zeit gekostet haben

- **AppDaemon lädt Konfigurationsänderungen nicht zuverlässig nach.** Beim Hinzufügen eines neuen Abschnitts in `apps.yaml` stieg das automatische Nachladen mit `KeyError` aus. Nach Änderungen das Add-on neu starten.
- **`secrets.yaml` darf keine eingerückten Zeilen haben.** Sonst meldet AppDaemon nur „Configuration file must be a dictionary".
- **`FlashNextion` einzeln senden.** Zusammen mit einem `Backlog`, das auf `Restart 1` endet, geht der Befehl verloren.
- **Flash-Fortschritt** kommt auf `tele/<topic>/RESULT` als `{"Flashing":{"complete": N, ...}}`. Der Flash der HA-Firmware dauerte rund acht Minuten.
- **HACS legt den Installationsordner nach dem Repository-Namen an** (`appdaemon/apps/<repo-name>/`). Original und Fork dürfen nicht gleichzeitig installiert sein, sonst liegt dasselbe Modul doppelt vor.
- **Die AppDaemon-Konfiguration liegt seit Add-on-Version 15 außerhalb des HA-Konfigurationsordners**, unter `/addon_configs/a0d7b954_appdaemon/`. In `appdaemon.yaml` muss `app_dir` auf `/homeassistant/appdaemon/apps/` zeigen.

## 6. Arbeitsweise

- **Claude Code in der Cloud hat keinen Zugriff auf Dirks Home Assistant oder das Panel.** Es sieht nur das Repository. Logs lesen, AppDaemon neu starten und Flashen laufen über einen separaten Chat in der Claude-App, der mit Dirks Rechner verbunden ist.
- **Ablauf pro Änderung:**
  1. Kurze Notiz, was sich ändern soll.
  2. Umsetzung in einem eigenen Zweig, Pull Request in den Fork.
  3. Dirk zieht den Stand per HACS, startet AppDaemon neu und testet am Panel.
  4. Zusammenführen und mit einer Versionsnummer markieren.
  5. Etwa monatlich die Korrekturen vom Original übernehmen (`git remote add upstream https://github.com/joBr99/nspanel-lovelace-ui`, dann `main` von `upstream` mergen).
- **Kleine, allgemeingültige Korrekturen** können zusätzlich als Pull Request an joBr99 gehen. Der Entwickler nimmt fremde Fixes an.
- **Dirk schreibt Deutsch**, meist kurz. Antworten bitte auf Deutsch, Ergebnis zuerst.
- **Das Display kann niemand außer Dirk sehen.** Jede Änderung an der Darstellung braucht seine Rückmeldung oder ein Foto.
- **Keine Heimnetz-Details ins Repository.** IP-Adressen, MQTT-Topics, Benutzernamen und Dirks `apps.yaml` bleiben draußen, auch in Commit-Nachrichten und Pull Requests.

### Test-Checkliste fürs Panel

Nach jeder Änderung am Backend:

1. AppDaemon startet ohne Fehler, im Log steht „Started" für die Panel-App.
2. Das Panel meldet `event,startup,...` und der Vorab-Check der Versionen läuft durch.
3. Bildschirmschoner zeigt Uhrzeit, Datum und Wetter.
4. Antippen führt zur ersten Karte, alle Einträge sind sichtbar.
5. Ein Licht lässt sich über den Schalter in der Zeile schalten.
6. Das Licht-Popup öffnet sich und zeigt Schalter und Helligkeitsregler. Der Regler ändert die Helligkeit.
7. Bei einer Lampe mit Farbtemperatur erscheint der zweite Regler.
8. Nach 20 Sekunden ohne Eingabe kehrt das Panel zum Bildschirmschoner zurück.

## 7. Nächste Schritte

1. ~~**Dirk:** Fork anlegen und umbenennen, GitHub unter claude.ai/code verbinden.~~ Erledigt.
2. ~~**Claude Code, erste Aufgaben im Fork:** diese Datei als `CLAUDE.md` ablegen, `hacs.json` mit eigenem Anzeigenamen versehen, im README kenntlich machen, dass es ein persönlicher Fork ohne Support ist.~~ Erledigt.
3. ~~**Protokollvergleich** 53 gegen 61, Seite für Seite.~~ Erledigt, siehe `fork/protokollvergleich-53-61.md`.
4. **Entscheidung** über den Umfang auf Basis dieser Liste. Die Zielversion 61 steht fest.
5. **Betriebsseite (separater Chat):** HACS auf den Fork umstellen, das Regler-Problem klären, die Diagnose-Protokollierung (`quiet: false`) wieder abschalten, später die Firmware 61 zum Test flashen.
