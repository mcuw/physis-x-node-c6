# FSD-NODE-001 – ESP32-C6 Thread Node

- Status: Entwurf / Spike-Baseline
- Version: 0.1
- Zugehörige System-FSD: [FSD-001](https://github.com/mcuw/physis-x-system-manager/blob/main/documents/FSD-001-system.md)

## 1. Zweck und Umfang

Spezifikation des ESP32-C6-basierten Physis-X Thread-Endgeräts auf Basis des bestehenden `esp32-thread-coap-light`-Projekts. Das Gerät nimmt am durch einen separaten TBR bereitgestellten Thread-Netz teil und stellt seine Gerätefunktionen über IPv6/CoAP bereit.

Nicht im Umfang: Border-Router-Funktion, zentrale Geräteverwaltung, System-Manager-Simulation.

## 2. Hardware- und Softwarebasis

- MCU: ESP32-C6.
- Funkrolle: Thread-Endgerät.
- Ausgangsbasis: [esp32-thread-coap-light](https://github.com/espressif/esp32-thread-coap-light).
- Buildsystem: ESP-IDF; genaue Version/Commit ist im Spike festzuschreiben.
- Abhängigkeiten, Partitionstabelle, Flashgröße, PSRAM (falls vorhanden), Pinbelegung und Board-Revision müssen im Build-/Hardwareprotokoll dokumentiert werden.
- Bestehende CoAP-Light-/WS2812-Funktion dient als Referenzaktor und ist nicht der generische Physis-X-Gerätevertrag.

## 3. Funktionale Anforderungen

- NODE-F-001: Firmware MUSS auf der festgelegten C6-Hardware reproduzierbar bauen und flashen.
- NODE-F-002: Gerät MUSS als Thread-Endgerät einem bestehenden Thread-Netz beitreten können.
- NODE-F-003: Gerät DARF nicht unbeabsichtigt ein eigenes Thread-Netz als Leader aufbauen.
- NODE-F-004: Join-/Commissioning-Status MUSS diagnostizierbar sein.
- NODE-F-005: Netzwerkidentität und notwendige Zugangsdaten MÜSSEN gemäß Sicherheitskonzept persistent und geschützt verwaltet werden.
- NODE-F-006: Nach Node-Neustart MUSS das Gerät ohne erneute manuelle Provisionierung wieder beitreten können, sofern Credentials und Netzwerk gültig sind.
- NODE-F-007: Nach temporärem TBR-/Netzausfall MUSS ein definierter Rejoin-/Recovery-Pfad greifen.
- NODE-F-008: Gerät MUSS stabile Geräte-ID, Gerätetyp, Firmwareversion und Fähigkeiten bereitstellen.
- NODE-F-009: Unterstützte Status-, Telemetrie-, Konfigurations- und Kommandooperationen MÜSSEN über den abgestimmten Vertrag erreichbar sein.
- NODE-F-010: Nicht unterstützte oder ungültige Requests MÜSSEN mit definiertem Fehler beantwortet werden.

## 4. Logische API

Vorläufige Ressourcenhypothesen:
- `/physis/device`
- `/physis/capabilities`
- `/physis/status`
- `/physis/telemetry`
- `/physis/config`
- `/physis/commands`

Diese Ressourcen sind mit der zentralen FSD und dem System-Manager-Spike abzustimmen. Noch offen: JSON/CBOR, Observe, Payload-Grenzen, Authentisierung, DTLS-/OSCORE- oder alternatives Sicherheitsprofil, Fehlercodes, Versionierung und Idempotenz.

## 5. Laufzeit- und Fehlerverhalten

- Thread-Verbindung, IPv6-Adresse und CoAP-Serverzustand müssen separat diagnostizierbar sein.
- Ein CoAP-Fehler darf nicht ohne weitere Diagnose als Thread-Ausfall klassifiziert werden.
- Bei Verlust der Verbindung zum TBR muss die lokale Gerätefunktion in einem definierten Zustand bleiben.
- Wiederholte Kommandos müssen gemäß ihrer Semantik sicher behandelbar sein.
- Logs dürfen keine Thread-Netzwerk-Credentials oder Geheimnisse offenlegen.

## 6. Spike-Arbeitspakete und Abhängigkeiten

1. NODE-1 Firmwarebasis: Quellbaum, Lizenzhinweise, ESP-IDF, Komponenten, sdkconfig, Partitionen, Pinout und Clean Build.
2. NODE-2 Thread Join/Recovery: Joiner/Commissioning, Credential-Persistenz, Neustart und TBR-Ausfall.
3. NODE-3 Physis-X-CoAP-Vertrag: Ressourcen, Kodierung, Fehler, Payload und Security mit System Manager abstimmen.

NODE-2 hängt von TBR-1 und TBR-2 ab. NODE-3 hängt von der vorläufigen Vertragsdefinition SM-1 ab und muss mit realem TBR/Node validiert werden.

## 7. Abnahmetests

| ID | Test | Kriterium |
|---|---|---|
| NODE-AT-001 | Clean Build | Dokumentierte Toolchain baut Firmware aus sauberem Checkout |
| NODE-AT-002 | Flash/Boot | Firmware startet auf der dokumentierten Board-Revision ohne Reset-Schleife |
| NODE-AT-003 | Join | C6 tritt dem vorgesehenen Thread-Netz bei |
| NODE-AT-004 | Persistence | Neustart erfordert keine erneute manuelle Provisionierung |
| NODE-AT-005 | Recovery | Node erholt sich nach TBR-/Netzunterbrechung |
| NODE-AT-006 | IPv6/CoAP | Vereinbarte Ressourcen sind vom Integrationstest erreichbar |
| NODE-AT-007 | Contract | Gültige und ungültige Requests liefern spezifikationsgemäße Antworten |
| NODE-AT-008 | Identity | Geräte-ID bleibt über Neustart und IP-Änderung stabil |

## 8. Offene Punkte

- Exakte Board-Revision und Pinbelegung.
- IDF-Version und kompatible OpenThread-Komponenten.
- Commissioning- und Credential-Provisionierungsverfahren.
- Geräte-ID-Erzeugung und -Persistenz.
- Finale Ressourcen, Payloads und Sicherheitsprofil.
- Verhalten bei Factory Reset und Geräteentzug.

## 9. Traceability

Systemanforderungen: SYS-F-001 bis SYS-F-035, SYS-S-001 bis SYS-S-004. Die konkreten NODE-Anforderungen sind nach Abschluss der Spikes gegen die zentrale FSD zu mappen.
