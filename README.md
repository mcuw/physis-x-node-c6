# Physis-X Node – ESP32-C6

## Zweck und Rolle

Dieses Repository enthält die Firmware eines Physis-X-Thread-Endgeräts auf ESP32-C6-Basis. Es ist **kein Border Router**. Der Node tritt dem vom separaten ESP32-S31 Thread Border Router (TBR) bereitgestellten Thread-Netz bei und stellt seine Gerätefunktionen über einen definierten Anwendungsdienst bereit.

Als technische Ausgangsbasis dient [mcuw/esp32-thread-coap-light](https://github.com/mcuw/esp32-thread-coap-light). Dessen vorhandene Thread-/CoAP-Funktionalität soll möglichst erhalten und schrittweise verallgemeinert werden. Die Light-/WS2812-Funktion ist zunächst ein konkreter Beispiel-Actor, nicht das allgemeine Gerätemodell.

## Ziele des Spikes

1. Vorhandene Firmware, Komponenten und Hardwareannahmen verstehen und reproduzierbar bauen.
2. Thread-Join, Commissioning und Wiederherstellung mit dem S31-TBR nachweisen.
3. Einen kleinen, versionierbaren Physis-X-Gerätevertrag über CoAP validieren.
4. Eine klare Grenze zwischen Geräte-Actor, Netzwerktransport und System Manager schaffen.

## Nicht-Ziele

- Keine Border-Router-Funktionalität im C6-Node.
- Keine System-Manager-Implementierung in diesem Repository.
- Keine Festlegung sämtlicher Produkt-APIs, bevor die Spikes und die FSD abgeschlossen sind.
- Keine ungeprüfte Übernahme boardfremder Pinbelegungen oder WS2812-Hardwareannahmen.

## Spike-Plan und Issues

| Reihenfolge | Issue | Arbeitspaket | Abnahmekriterien |
|---|---|---|---|
| 1 | [C6-1 – Firmwarebasis](https://github.com/mcuw/physis-x-node-c6/issues/1) | Quellbaum, lokale Komponenten, ESP-IDF-Version, sdkconfig, Partitionen, Pinbelegung und Lizenzen inventarisieren; Clean Build und Boot prüfen. | Clean checkout baut reproduzierbar; Firmware startet ohne Reset-Schleife; Build-/Boot-Logs und Dependency-Inventar sind dokumentiert. |
| 2 | [C6-2 – Thread-Join und Recovery](https://github.com/mcuw/physis-x-node-c6/issues/2) | Joiner-/Commissioning-Lifecycle, Dataset/Credentials, persistente Identität, Neustart und TBR-Ausfall testen. | C6 joint reproduzierbar das TBR-Netz; C6-Neustart benötigt kein manuelles Recommissioning; TBR-Neustart hat definiertes Recovery; kein unbeabsichtigtes eigenes Netz. |
| 3 | [C6-3 – Physis-X-CoAP-Vertrag](https://github.com/mcuw/physis-x-node-c6/issues/3) | Identity, Capabilities, Status, Telemetry, Config und Commands als Protokollprobe bewerten; Fehlerantworten, Versionierung und Sicherheit prüfen. | Client kann Identität/Capabilities lesen; Command wird quittiert; Status und Fehlerfälle sind definiert; CoAP/CoAPS-Sicherheitsstatus ist praktisch belegt oder als offenes Risiko markiert. |

## Vorläufige Schnittstellenhypothese

Die folgenden Ressourcen sind Vorschläge für die Protokollprobe, keine bereits endgültig beschlossene FSD:

- `/physis/device` – stabile Geräteidentität und Firmware-/Hardwareinformationen
- `/physis/capabilities` – verfügbare Fähigkeiten und Actor-Typen
- `/physis/status` – aktueller Gerätezustand
- `/physis/telemetry` – Messwerte bzw. Betriebsdaten
- `/physis/config` – unterstützte Konfiguration
- `/physis/commands` – Kommandos und deren Ergebnis

Zu klären sind unter anderem Payload-Format (JSON/CBOR), CoAP Observe, Fehlercodes, Idempotenz, maximale Payload-Größen, Authentisierung/Verschlüsselung und Schema-Versionierung. Eine aktivierte DTLS-/ECJPAKE-Konfigurationsoption allein beweist noch keinen abgesicherten Ende-zu-Ende-Dienst.

## Abhängigkeiten

- C6-1 ist Voraussetzung für C6-2 und C6-3.
- C6-2 benötigt ein lauffähiges S31-TBR aus [physis-x-tbr](https://github.com/mcuw/physis-x-tbr), insbesondere TBR-1/TBR-2.
- C6-3 berücksichtigt den vorläufigen Vertrag aus [physis-x-system-manager](https://github.com/mcuw/physis-x-system-manager), Issue SM-1.
- Ende-zu-Ende-Interop wird gemeinsam mit TBR-3 validiert.

## Technische Risiken und offene Entscheidungen

- ESP-IDF- und Component-Manager-Kompatibilität der bestehenden Abhängigkeiten.
- Joiner-Verfahren, Commissioning-Berechtigung und sichere Übergabe/Rotation von Credentials.
- Persistenz des Thread-Datasets und Verhalten nach Factory Reset.
- IPv6-Adresstypen: stabile Device ID darf nicht mit einem möglicherweise wechselnden RLOC verwechselt werden.
- Tatsächliche Hardwarekompatibilität der WS2812-/GPIO-Ansteuerung.
- CoAP oder CoAPS und die konkrete Schlüssel-/Identitätsverwaltung.

## Definition of Done für den Spike

Die drei Issues sind mit Logs, reproduzierbaren Schritten, Messwerten und offenen Entscheidungen abgeschlossen. Das Ergebnis dient als Input für die FSD; es ist noch keine Produktionsfreigabe.

## Repository-Grenze

Dieses Repository enthält C6-Node-Firmware und gerätespezifische Actor-Implementierungen. TBR-Firmware bleibt in `physis-x-tbr`; zentrale Verwaltung und Simulation bleiben in `physis-x-system-manager`.
