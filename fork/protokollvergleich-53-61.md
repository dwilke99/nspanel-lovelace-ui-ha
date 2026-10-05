# Protokollvergleich Display-Firmware 53 → 61

Stand: 5. Oktober 2026. Verglichen wird die HA-Firmware 53 (`HMI/n2t-out/`) mit der Firmware 61 = Release 5.1.1 aus der ioBroker-Linie (`fork/n2t-out-61/`, Herkunft in `fork/README.md`). Maßstab ist das AppDaemon-Backend in `apps/nspanel-lovelace-ui/` auf dem Stand dieses Forks.

Einstufung:

- **unverändert**: Das Backend funktioniert so, wie es ist.
- **anpassen**: 61 hat Format oder Verhalten geändert, das Backend muss nachziehen.
- **neu**: gibt es nur in 61, das Backend nutzt es heute nicht.

Belege: `61:Datei:Zeile` steht für `fork/n2t-out-61/Datei.txt`, `53:Datei:Zeile` für `HMI/n2t-out/Datei.txt`. Backend-Pfade sind relativ zu `apps/nspanel-lovelace-ui/`. Am Panel ist nichts davon getestet, alles stammt aus dem Quelltext.

## 1. Ergebnis in Kürze

Transport, Rahmen (55 BB, Länge, CRC), Startmeldung, die globalen Befehle und die Formate fast aller bestehenden Seiten sind gleich geblieben. Das Backend kann mit 61 starten und den Bildschirmschoner sowie die meisten Karten anzeigen. Für einen sauberen Betrieb muss es an acht Stellen angepasst werden:

| # | Was | Wo im Backend | Folge ohne Anpassung |
|---|---|---|---|
| 1 | Neues Event `buttonPress3` (langer Druck) wie `buttonPress2` behandeln | `luibackend/mqtt.py:65` | Langer Druck auf Navigationspfeile und Grid-Kacheln bewirkt nichts |
| 2 | `popupNotify`: zwei Felder einfügen, Antworten `button1`/`button2` statt `no`/`yes` | `luibackend/pages.py:1096`, `luibackend/updater.py:35`, `luibackend/mqtt.py:70-72`, `luibackend/controller.py:251` | Update-Dialog zeigt Unsinn und lässt sich nicht bestätigen |
| 3 | `popupSlider` für `number`-Einträge unterstützen | `luibackend/controller.py:194` (`detail_open`), neuer Generator, Handler `positionSlider1-3` | Antippen einer input_number-Zeile öffnet ein leeres Popup |
| 4 | `popupFan`: Icon in Feld 2 senden | `luibackend/pages.py:1007` | Icon im Lüfter-Popup verschwindet |
| 5 | `pageOpenDetail` robuster auswerten | `luibackend/mqtt.py:79-80` | `popupColor` ohne Entität führt zu IndexError; `popupInSel` mit leerer Liste erzeugt Endlosschleife |
| 6 | Nachrichtenlängen begrenzen | `luibackend/pages.py` (Media, Thermo-Popup, InSel) | Puffer in 61 kleiner, lange Nachrichten werden abgeschnitten |
| 7 | Leerer Tipp im `screensaver2` (`buttonPress2,,button`) wie `bExit` behandeln | `luibackend/controller.py` (`button_press`) | Tipp auf leeren Slot weckt das Panel womöglich nicht |
| 8 | Zielversion 61 und Firmware-URL im Updater | `nspanel-lovelace-ui.py:35-46` | Kein Update-Hinweis, Flash-URL zeigt auf 53 |

Für Punkt 8 gilt: Solange `desired_display_firmware_version` auf 53 steht und das Panel 61 meldet, bleibt das Backend still. `luibackend/updater.py:65` prüft nur auf `<`, ein Rückflashen auf 53 findet nicht statt. Firmware 61 lässt sich also von Hand flashen, ohne dass das Backend dagegen arbeitet.

Verhaltensänderungen ohne Bruch (Abschnitt 6), neue Seiten (Abschnitt 7) und offene Fragen (Abschnitt 9) stehen weiter unten.

**Korrektur zur Übergabe:** Firmware 61 hat bei `dimmode` keinen zusätzlichen Parameter, sondern einen weniger. Das fünfte Feld (`featNewSliders`, Schalter für `popupLightNew`) wertet 61 nicht mehr aus. Das neue Licht-Popup heißt jetzt `popupLight2` und wird über den Eintragstyp `light2` geöffnet. Auch `screensaver2` gab es schon in 53, neu ist nur `screensaver3`.

## 2. Start, Treiber und Updater

### Startablauf

- 53 startet direkt mit `pageStartup`. 61 zeigt zuerst `pageSplash`, eine Animation von etwa 4,5 Sekunden ohne Befehlsverarbeitung (`61:Program.s:23`, `61:pageSplash:52-85`). Die Startmeldung kommt entsprechend später. Befehle aus dieser Zeit bleiben im Puffer, bis `pageStartup` sie abarbeitet. **unverändert**, das Backend braucht nichts.
- Startmeldung (**unverändert**):

  | Index | 53 (`53:pageStartup:171`) | 61 (`61:pageStartup:160`) |
  |---|---|---|
  | 2 | Version `53` | Version `61` |
  | 3 | Modell `eu` | Modell `eu` |
  | 4 | – | Release `5.1.1` |

  `luibackend/mqtt.py:49-58` liest nur Index 2 und 3, das zusätzliche Feld stört nicht.

### Globale Befehle

Diese Befehle versteht jede Seite. Neue globale Befehle gibt es nicht.

| Befehl | 53 → 61 | Backend | Einstufung |
|---|---|---|---|
| `time~T~add` | gleich | `luibackend/pages.py:110-121` | **unverändert** |
| `date~D` | gleich | `luibackend/pages.py:123-134` | **unverändert** |
| `timeout~N` | gleich | `luibackend/pages.py` (render_card) | **unverändert** |
| `pageType~Ziel~a2~a3~a4` | gleich, mehr Ziele | `luibackend/pages.py:136-139` | **unverändert** |
| `dimmode~sleep~normal~bco~font~feat` | Feld 5 entfällt | `luibackend/controller.py:89-93` | **unverändert**, `featureExperimentalSliders` wirkt nicht mehr |

Neue `pageType`-Ziele in 61: `pageSplash`, `screensaver3`, `cardSchedule`, `cardGrid3`, `cardThermo2`, `cardLChart2`, `popupInSel`, `popupFan`, `popupShutter2`, `popupLight2`, `popupTimer`, `popupSlider`, `popupColor`. Entfallen ist `popupLightNew`. Alle Ziele, die das Backend heute sendet, gibt es weiterhin.

### Berry-Treiber

| | joBr99 `tasmota/autoexec.be` | ticaki `tasmota/berry/10` | ticaki `tasmota/berry/11` (Beta) |
|---|---|---|---|
| Version | 9 | 10 | 11 |

- Bei allen gleich: Abfrage per `GetDriverVersion`, Antwort `{"nlui_driver_version":"N"}`, Befehle `CustomSend`, `FlashNextion`, `FlashNextionAdv0-6`, `UpdateDriverVersion`, gleiches Framing, 115200 Baud.
- Treiber 10 meldet den Flash-Fortschritt als Text mit `"done"` am Ende und schaltet beim Flashen `Rule3` ab und danach wieder ein. Treiber 11 flasht robuster.
- Folgerung: Für das Protokoll mit 61 ist kein Treiberwechsel nötig. Das ist abgeleitet, nicht getestet. Die ioBroker-Linie liefert 61 zusammen mit Treiber 10 aus.

### Updater

- `nspanel-lovelace-ui.py:35-37`: `desired_display_firmware_version` von 53 auf 61, `version` auf `"v5.1.1"`.
- Firmware-URLs der ioBroker-Linie (`ioBroker/NsPanelTs.ts:228-231`): `http://nspanel.de/nspanel-v5.1.1.tft`, `http://nspanel.de/nspanel-us-l-v5.1.1.tft`, `http://nspanel.de/nspanel-us-p-v5.1.2.tft`. Der Adapter nennt für us-p inzwischen 5.1.3. Nur `http`, der Treiber kann kein `https`.
- Über die Option `displayURL-EU` (bzw. `-US-L`, `-US-P`) lässt sich die URL schon heute in `apps.yaml` setzen.
- Der Flash-Befehl `FlashNextion` (`luibackend/mqtt.py:114`) entspricht `FlashNextionAdv0`, das die ioBroker-Linie nutzt. **unverändert**.
- Laut `ioBroker/NsPanelTs.ts:22` lässt Tasmota 15.1.0 kein `FlashNextion` zu. Gemeldet ist das nur für diese eine Version. Ob das auf dem Panel installierte Tasmota 15.5.0 betroffen ist, ist offen.

## 3. Bildschirmschoner

| Seite | Befehle | Einstufung |
|---|---|---|
| `screensaver` | `weatherUpdate`, `statusUpdate`, `color`, `notify`, `wake`, `timeout`: Felder gleich | **unverändert** |
| `screensaver2` | Felder gleich, Puffer 1935 → 1800 Zeichen | **unverändert**, Länge von `weatherUpdate` prüfen |
| `screensaver3` | neue „EasyView"-Variante von `screensaver`, gleiche Nachrichten, ohne Vorhersage 4 (Felder 27–30 und `color` 9/13 werden ignoriert), Statusicons nur als Farbbalken | **neu** |

- Events `bExit,<Taps>`, `swipeUp/Down/Left/Right`, `sleepReached`: **unverändert**.
- `screensaver2`: Ein Tipp auf einen leeren Eintrag sendete in 53 `bExit`. In 61 sendet er `event,buttonPress2,,button`. Das Backend leert im Bildschirmschoner die Entitäts-IDs (`luibackend/pages.py:153`, `mask`), deshalb sind die Slots immer leer. **anpassen**: leere ID mit `button` im Bildschirmschoner wie `bExit` behandeln.
- `screensaver3` unterstützen: in `luibackend/pages.py:856` die Liste um `"screensaver3"` erweitern, mehr ist nicht nötig.

## 4. Karten

### Gemeinsam: Navigation und langer Druck

- Die Navigationspfeile senden in 61 erst beim Loslassen, nicht mehr beim Drücken. Ein Druck ab 500 ms sendet `event,buttonPress3,<nav>,button` statt `buttonPress2`. Das betrifft alle Karten (`cardAlarm`, `cardChart`, `cardEntities`, `cardGrid`, `cardGrid2`, `cardGrid3`, `cardLChart`, `cardMedia`, `cardPower`, `cardQR`, `cardSchedule`, `cardThermo`, `cardThermo2`).
- In `cardGrid`, `cardGrid2` und `cardGrid3` sendet auch ein langer Druck auf eine Kachel `buttonPress3` (z. B. `61:cardGrid:303`). In 53 hat ein langer Druck auf einen Schalter geschaltet, in 61 passiert mit dem heutigen Backend nichts.
- **anpassen**: `buttonPress3` in `luibackend/mqtt.py:65` wie `buttonPress2` behandeln. Genau so macht es `ioBroker/NsPanelTs.ts:4798-4799`. Der ioBroker-Adapter von ticaki nutzt den langen Druck auf die Pfeile zum Seitenwechsel statt zum Blättern; das wäre eine spätere Verfeinerung.

### Eintragsformat `entityUpd`

Kopf und Einträge sind auf allen bestehenden Karten **unverändert**: Index 1 Titel, 2–7 linker und 8–13 rechter Navigationseintrag, ab Index 14 die Einträge mit je sechs Feldern `type~internalName~icon~iconColor~displayName~optionalValue` (cardPower: sieben Felder, siehe unten).

| Typ | optionalValue | 53 | 61 |
|---|---|---|---|
| `shutter` | `iconUp\|iconStop\|iconDown\|stUp\|stStop\|stDown` | ✓ | ✓ |
| `shutter2` | wie `shutter`, öffnet `popupShutter2` | – | **neu** |
| `light`, `switch`, `fan` | `0`/`1` | ✓ | ✓ |
| `light2` | `0`/`1`, öffnet `popupLight2` | – | **neu** |
| `text`, `timer` | Text | ✓ | ✓ |
| `button`, `input_sel` | Text, antippbar | ✓ | ✓ |
| `number` | `val\|min\|max`, öffnet in 61 `popupSlider` | ✓ | ✓ |

### Je Karte

| Karte | Eingehend | Ausgehend | Einstufung |
|---|---|---|---|
| `cardEntities` | gleich (4 Einträge) | `OnOff`, `number-set`, `up/stop/down` gleich | **unverändert** bis auf Navigation; Verhaltensänderungen siehe Abschnitt 6 |
| `cardGrid` / `cardGrid2` | gleich (6 bzw. 8 Einträge), Icon-Schrift `¬N` wird je Kachel zurückgesetzt | Kachel: Popups für `light`/`fan`/`number` nach 500 ms, sonst `buttonPress3` | **anpassen** (buttonPress3) |
| `cardQR` | gleich | gleich | **unverändert** |
| `cardPower` | gleich (7 Felder je Eintrag) | nur Navigation | **unverändert** |
| `cardMedia` | Felder gleich, **Puffer 750 → 600 Byte** | `media-*`, `volumeSlider` gleich; −/+ springen um 5 | **anpassen** (Länge) |
| `cardAlarm` / `cardUnlock` | gleich | gleich | **unverändert** |
| `cardThermo` | gleich, Puffer bleibt 750 | `hvac_action`, `tempUpd`, `tempUpdHighLow` gleich | **unverändert** |
| `cardChart` | gleich, Puffer 275 → 1750, leere Ticks zeigen „No Data" | nur Navigation | **unverändert** |
| `cardLChart` | gleich, Puffer 512 → 1750 | nur Navigation | **unverändert**, vom Backend aber nie unterstützt |

**cardMedia im Detail:** Ein realistisches Beispiel mit sechs Einträgen kam auf rund 560 Byte, das ist knapp unter der neuen Grenze. Titel und Interpret sollten gekürzt werden (der ioBroker-Adapter begrenzt auf 36 bzw. 28 Zeichen). Neu und optional: Steht im Icon-Feld des ersten Eintrags `logo-spotify`, `logo-sonos`, `logo-alexa`, `logo-mpd`, `logo-bose`, `logo-volumio` oder `logo-dnla`, zeigt die Firmware ein Logo (`61:cardMedia:1151-1203`).

## 5. Popups

| Popup | Eingehend | Ausgehend | Einstufung |
|---|---|---|---|
| `popupLight` | gleich (11 Felder) | gleich | **unverändert** |
| `popupShutter` | gleich (19 Felder inkl. Tilt) | gleich | **unverändert** |
| `popupTimer` | gleich, Puffer 960 → 900 | gleich | **unverändert** |
| `popupThermo` | gleich, **Puffer 500 → 400** | gleich | **anpassen** (Länge) |
| `popupFan` | **Feld 2 (Icon) wird jetzt ausgewertet** | gleich | **anpassen** |
| `popupInSel` | gleich, **Puffer 960 → 900, Optionsliste 900 → 840** | neues `pageOpenDetail` mit `bModeNext` | **anpassen** |
| `popupNotify` | **Felder verschoben** | **`button1/2/3` statt `no/yes`** | **anpassen** |
| `popupLight2` | wie `popupLight` mit Abweichungen | wie `popupLight` | **neu** |
| `popupShutter2` | 22 Felder | `up/stop/down`, `positionSlider`, `button1-3Press` | **neu** |
| `popupSlider` | bis zu 3 Regler | `positionSlider1-3` | **neu**, aber nötig (siehe Punkt 3) |
| `popupColor` | nur `entityUpdateDetail~<id>` | `pageOpenDetail` ohne Entität | **neu**, nicht nutzen |
| `popupLightNew` | – | – | **entfallen**, Nachfolger `popupLight2` |

### popupNotify

| Index | 53 (`53:popupNotify:264-319`) | 61 (`61:popupNotify:321-363`) |
|---|---|---|
| 1–7 | entn, Überschrift, Farbe, b1, b1-Farbe, b2, b2-Farbe | gleich |
| 8 | Text | **b3-Text** |
| 9 | Textfarbe | **b3-Farbe** |
| 10 | Timeout | Text |
| 11 | Schriftart | Textfarbe |
| 12 | Icon | Timeout |
| 13 | Icon-Farbe | Schriftart |
| 14 | – | Icon (leer blendet aus) |
| 15 | – | Icon-Farbe |

- Die Antworten lauten in 61 `buttonPress2,<id>,notifyAction,button1|button2|button3` (`61:popupNotify:174/200/226`), in 53 `no|yes`.
- Das Backend nutzt `popupNotify` für den Update-Dialog (`luibackend/updater.py:54/61/74`) und für Sensor-Hinweise im Alarm (`luibackend/controller.py:408`). Mit 61 landet der Meldungstext auf Knopf 3, und „Yes" wird nie erkannt (`luibackend/mqtt.py:70-72` prüft auf `yes`).
- **anpassen**: nach Feld 7 zwei leere Felder einfügen, `button2` wie `yes` und `button1` wie `no` behandeln. Die Knöpfe liegen in 61 zu dritt nebeneinander. Das ioBroker-Skript lässt b2 leer und legt den zweiten Knopf auf b3 (`ioBroker/NsPanelTs.ts:3330-3352`), das wäre optisch die bessere Wahl. Die Doku `docs/notifications.md` und eigene Automationen, die das Popup per MQTT öffnen, nutzen noch das alte Format.

### popupFan

In 53 war das Lesen von Feld 2 auskommentiert, in 61 ist es aktiv (`61:popupFan:447`). Das Backend sendet dort einen Leerstring (`luibackend/pages.py:1007`), das Icon verschwindet. **anpassen**: das Icon der Entität in Feld 2 senden.

### popupInSel

- Optionen wählen sendet weiter `mode-<typ>,<idx>`. Neu: Nach jeder Auswahl schließt das Popup sich selbst (`buttonPress2,popupInSel,bExit`).
- Neu: Beim Blättern sendet das Popup `event,pageOpenDetail,popupInSel,<id>,bModeNext,<seite>` (`61:popupInSel:750`). Ist die Optionsliste leer, klickt die Firmware selbst auf „weiter" (`61:popupInSel:857-861`). Das Backend beantwortet jedes `pageOpenDetail` mit einer neuen Detailnachricht. Bei leerer Liste (z. B. leere `effect_list`, Mediaplayer ohne `source_list`) entsteht so ein endloses Hin und Her.
- **anpassen**: `pageOpenDetail` mit mehr als vier Teilen nicht beantworten oder nur bei nicht leerer Liste; Optionsliste auf 840 Byte begrenzen.

### popupSlider

- Öffnet sich in 61 von selbst beim Antippen einer `number`-Zeile in `cardEntities`, bei langem Druck in den Grid-Karten und für `number`-Einträge in `cardMedia`. Meldet sich mit `pageOpenDetail,popupSlider,<id>` (`61:popupSlider:42`). `detail_open` kennt das nicht, das Popup bleibt leer.
- `entityUpdateDetail` mit bis zu drei Reglern, Basis b = 2, 11, 20:

  | Index | Bedeutung |
  |---|---|
  | b | Überschrift |
  | b+1 / b+2 | Icon „−" / „+" |
  | b+3 | aktueller Wert |
  | b+4 / b+5 | min / max |
  | b+6 | Nullpunkt |
  | b+7 | Schrittweite |
  | b+8 | `enable` (blendet nur ein, nie aus) |

- Antwort: `buttonPress2,<id>,positionSlider1|2|3,<wert>`. Nur ganze Zahlen; Dezimalschritte müsste das Backend skalieren.
- **anpassen**: Zweig in `detail_open`, Generator aus min/max/step der Entität, Handler `positionSliderN` → `set_value`. Alternative: input_number nicht mehr als `number` senden.

### popupLight2 (Nachfolger von popupLightNew)

- Großer senkrechter Helligkeitsregler, öffnet sich über den Eintragstyp `light2`.
- Meldet sich als `pageOpenDetail,popupLight,<id>` (`61:popupLight2:28`), das Backend antwortet also ohne Änderung.
- Abweichungen zu `popupLight`: Feld 2 und 8–10 werden ignoriert, Feld 3 ist die Füllfarbe des Reglers.
- **neu**: Das Backend müsste nur `light2` statt `light` als Typ senden (`luibackend/pages.py:298-299`), z. B. per Option.

### popupShutter2

- Vereinfachte Rollladenseite mit senkrechtem Regler und drei frei belegbaren Knöpfen, ohne Tilt. Öffnet sich über den Eintragstyp `shutter2`, meldet sich als `pageOpenDetail,popupShutter2,<id>`.
- `entityUpdateDetail`: 2 Position oder `disable`, 3 Infotext, 5 Icon, 6–8 Icons auf/stopp/ab, 9–11 deren Status, 12–20 drei Knöpfe (Icon, Farbe, `enable`), 22 `1` = 0 % heißt geschlossen.
- Antworten: `up`, `stop`, `down`, `positionSlider,<wert>`, neu `button1Press` bis `button3Press`.
- **neu**: Typ `shutter2`, Zweig in `detail_open`, Generator, Handler.

### popupLight: Wann sind die Regler sichtbar?

53 und 61 verhalten sich gleich (`53:popupLight:475-557`):

- Beim Öffnen blendet die Seite alle Regler aus. Erst ein `entityUpdateDetail` mit passender ID blendet sie ein.
- Helligkeit (Feld 5): `disable` blendet aus, jeder andere Wert blendet ein. Der Schalterzustand spielt keine Rolle.
- Farbtemperatur (Feld 6): Zahl blendet ein, `unknown` und `disable` blenden aus.
- In 61 wird der Temperatur-Regler intern gespiegelt dargestellt, die Werte auf der Leitung bedeuten dasselbe wie in 53.

## 6. Verhaltensänderungen ohne Bruch

- **input_select in cardEntities:** Ein Tipp auf den Wert öffnet in 61 `popupInSel`, statt die nächste Option zu wählen. Das Backend baut das Popup schon (`luibackend/controller.py:203`), der Zweig mit `select_next` wird aber nicht mehr erreicht.
- **Media, erster Eintrag:** Ein Tipp auf das Media-Icon öffnet `popupInSel` (Quellauswahl). Das zusätzliche Lautsprecher-Item, das das Backend anhängt, wird damit überflüssig.
- **Tipp auf Icon oder Name einer `text`-Zeile** sendet in 61 `buttonPress2,<id>,button`. Das Backend ignoriert das, man könnte es nutzen.
- **Leere Events beim Seitenaufbau:** `cardEntities` und `cardThermo2` senden bei jedem Öffnen `event,buttonPress2,,button`. Das erzeugt nur Logzeilen.
- **popupInSel** schließt sich nach einer Auswahl selbst.

## 7. Neue Seiten

| Seite | Zweck | Was das Backend bräuchte |
|---|---|---|
| `screensaver3` | „EasyView"-Bildschirmschoner | nur den Namen zulassen (`luibackend/pages.py:856`) |
| `cardGrid3` | 4 große Kacheln (2×2) | Kartentyp in `render_card`, `generate_entities_page` passt unverändert, höchstens 4 Einträge |
| `cardSchedule` | 6 Zeilen mit langem Namen und Wert, z. B. Abfahrten | Kartentyp in `render_card`, Format wie `cardEntities`, Werte als Typ `text` |
| `cardThermo2` | Thermostat mit Kreisregler, Ist-Temperatur, Feuchte, Statustext und 8 Entitäts-Knöpfen; nur ein Sollwert, keine hvac-Modus-Knöpfe | eigener Generator (Format unten), `tempUpd` funktioniert schon |
| `popupLight2`, `popupShutter2`, `popupSlider` | siehe Abschnitt 5 | siehe Abschnitt 5 |
| `cardLChart2` | Testseite ohne Befehlsverarbeitung | **nicht verwenden**, das Display bliebe dort hängen |
| `popupColor` | lokaler RGB565-Farbwähler, sendet keinen Wert | **nicht verwenden** |
| `pageSplash` | Startanimation | nichts |

**cardThermo2, `entityUpd`** (`61:cardThermo2:1398-1852`, Puffer 800 Byte):

| Index | Bedeutung |
|---|---|
| 14 | entn (kommt in `tempUpd` zurück) |
| 15 | Sollwert ×10 |
| 16 / 17 / 18 | min / max / Schritt ×10 |
| 19 | Einheit Sollwert |
| 20 | `0` = aus (Regler grau, Sollwert ausgeblendet) |
| 21–62 | 7 Anzeigeblöcke à 6 Felder; gelesen werden nur Icon/Text und Farbe: Ist-Temperatur, Einheit, Feuchte-Icon, Feuchte, Feuchte-Einheit, Statustext |
| 63–110 | 8 Entitäts-Knöpfe à 6 Felder |
| 111–116 | optionaler 9. Knopf |

Antwort: `buttonPress2,<id>,tempUpd,<soll×10>`, 800 ms nach dem Loslassen. Kein `tempUpdHighLow`, kein `hvac_action`. Ob Ist-Temperatur und Feuchte ebenfalls ×10 erwartet werden, ist aus den Referenz-Backends abgeleitet, nicht aus der Firmware.

## 8. Vorschlag für die Reihenfolge

1. **Grundbetrieb mit 61** (eine Änderung, ein PR): Punkte 1, 2, 4 und 5 aus Abschnitt 1. Das sind kleine, gut testbare Eingriffe. Danach Firmware 61 flashen und mit der Test-Checkliste prüfen.
2. **popupSlider** (Punkt 3), weil die Firmware ihn für jede input_number-Zeile von selbst öffnet.
3. **Längen** (Punkt 6) und **screensaver2** (Punkt 7).
4. **Updater** auf 61 umstellen (Punkt 8), erst wenn 61 am Panel läuft.
5. Neue Seiten nach Bedarf: `screensaver3` und `cardGrid3` sind fast geschenkt; `light2`/`shutter2`, `cardSchedule`, `cardThermo2` sind größere Schritte.

## 9. Offene Punkte und Unsicherheiten

- Wie der Nextion mit zu langen Nachrichten umgeht, ist nicht belegt. Vermutlich wird abgeschnitten, und die hinteren Felder fehlen.
- `screensaver2`: Ob ein Tipp auf einen Slot neben dem neuen `buttonPress2,,button` noch `bExit` auslöst, hängt von der Reihenfolge der Touch-Events ab.
- Die Wisch-Hotspots der Karten klicken die Pfeile nur „gedrückt" an. Da die Pfeile in 61 erst beim Loslassen senden, könnte Wischen zum Blättern wirkungslos sein. Ob die Hotspots erreichbar sind, lässt sich aus den Textauszügen nicht ablesen.
- Die US-Varianten (us-l, us-p) sind nicht geprüft.
- Dass Treiber 9 mit 61 voll funktioniert, ist abgeleitet, nicht getestet.
- Das Add-on unter `nspanel-lovelace-ui/` nutzt dieselben Formate und bräuchte dieselben Anpassungen. Es ist hier nicht weiter betrachtet.

## 10. Nebenbefund: fehlender Helligkeitsregler bei Firmware 53

Zum offenen Fehler aus `CLAUDE.md` (Abschnitt 4): `53:popupLight:475-492` blendet den Helligkeitsregler bei Feld 5 = `0` zwingend ein. An der gesendeten Nachricht liegt es also nicht. Infrage kommen:

1. **Die Detailnachricht kommt beim Popup nicht an.** Dann bleibt der Zustand vom Öffnen stehen, und dort sind alle Regler ausgeblendet. Bei ausgeschaltetem Licht sieht das Popup sonst unauffällig aus. Test: Popup bei eingeschaltetem Licht öffnen. Steht der Schalter im Popup dann auf „aus", kommt die Nachricht nicht an.
2. **`featureExperimentalSliders` ist gesetzt.** Dann öffnet 53 `popupLightNew`, dessen Regler bei Wert 0 praktisch unsichtbar ist (`53:popupLightNew:361-371`). Laut Übergabe ist die Option nicht gesetzt, der Standard ist `False` (`luibackend/config.py:130`). Unwahrscheinlich.
3. Auf dem Panel läuft eine andere TFT als 53.
