# VenusOS Installation auf einem Raspberry PI 4
## VenusOS installieren
- VenusOS Image herunterladen: https://updates.victronenergy.com/feeds/venus/release/images/raspberrypi4/
- auf eine SD Karte flashen (zB mit `balenaEtcher` oder dem `Raspberry Pi Imager`
- SSH im VenusOS aktivieren (direkt auch gleich `Remote Zugriff` aktivieren)
- admin/superuser PW erstellt ([Victron Anleitung "Root access"](https://www.victronenergy.com/live/ccgx:root_access))

## Shelly Pro3EM Integration
- [Anleitung & Skript](https://github.com/fabian-lauer/dbus-shelly-3em-smartmeter/), dass die Daten vom VenusOS aus dem ShellyPro3EM abgefragt werden
  - siehe Diskussion im [Forum](https://community.victronenergy.com/questions/125793/shelly-3em-smartmeter-with-venusos-cerbo-gx.html)
  - funktioniert aber **NUR** für den Shelly 3EM und **NICHT für die Pro Variante!!!**
  - jemand hat einen [Fix](https://github.com/funkmaster86/dbus-shelly-pro-3em-smartmeter) für die **Pro Variante** gemacht
  - aber [Issue #1](https://github.com/funkmaster86/dbus-shelly-pro-3em-smartmeter/issues/1) beachten, es war noch ein Fehler im run Skript

&rarr;  Der ShellyPro3EM taucht nun in der `Device List` in der `Remote Console` auf
![Shelly Pro3EM in Device List der Remote Console](https://github.com/CommentSectionScientist/VenusOs/blob/main/RemoteConsole.png?raw=true)
![VRM Protal mit Shelly Pro3EM](https://github.com/CommentSectionScientist/VenusOs/blob/main/VRM.png?raw=true)

## Shelly Plus1PM Integration
### Node-Red
- in VenusOS Mqtt aktivieren
- auf dem Shelly Mqtt aktivieren
- Shelly als PV-Inverter registrieren: ![Flow](https://github.com/CommentSectionScientist/VenusOs/blob/main/SetupShellyPvInverter.json)
- Daten von Shelly Plus1PM holen & Daten nach VenusOS per mqtt schreiben: ![Flow](https://github.com/CommentSectionScientist/VenusOs/blob/main/DataShellyPvInverter.json)
- benötigte Nodes:
  - node-red (mqtt)
  - node-red-contrib-shelly (könnte man theoretisch auch mit plain mqtt von node-red machen)
### Skirpt (veraltet)
- [Anleitung & Skript](https://github.com/Halmand/dbus-shelly-1pm-and-pm1-Plus-pvinverter-multi-instance)
- Der Wert `/Position` muss noch anders gesetzt werden
  - der Shelly Pro3EM ist bereits auf Position 0, dh der Shelly Plus1PM muss auf Position 1 sein, sonst wird das Plus1PM nicht als PV-Inverter angezeigt
  - ganz korrekt ist die Anordnung so nicht, aber besser als das man das Plus1PM überhaupt nicht sieht
  - das ist nötig, weil noch keine Batterie mit Muliplus und dann ESS vorhanden ist, dort kann man dann das Shelly Pro3EM korrekt als Grid hinterlegen, dann muss das **Shelly Plus1PM auf Position 0** gesendet werden!

## Speicher + Multiplus Integration
- BMV + Multiplus mittels Adapter Kabel an VenusOs anschließen
- Multiplus mittels VEConfigure konfigurieren
### Problem mit den Phasen
- Der Speicher ist physisch auf L3 angeschlossen, bei Victron muss es aber immer auf L1 angeschlossen sein
- Phasen softwaretechnisch im Pro3EM Python Skript ändern L1>L2, L2>L3, L3>L1 
```
self._dbusservice['/Ac/L2/Voltage'] = meter_data['em:0']['a_voltage']
self._dbusservice['/Ac/L3/Voltage'] = meter_data['em:0']['b_voltage']
self._dbusservice['/Ac/L1/Voltage'] = meter_data['em:0']['c_voltage']
```
- Nicht vergessen, im Plus1PM in der Config nun die Phase auf L1 zu ändern, sonst wird dier Erzeugung der PV auf die falsche Phase gerechnet (Falls das Skript und nicht Node-Red verwendet wurde!)
 
![VRM Protal mit Speicher](https://github.com/CommentSectionScientist/VenusOs/blob/main/VRM_mit_Speicher.png)
 
## OpenWb Ladenstation Integration
IN ARBEIT!!!
-> TODO: Besser per Node-Red
- Per [angepassten Skript](https://github.com/CommentSectionScientist/dbus-evcharger-openwb) die Daten von OpenWB über mqtt auslesen und dann nach VenusOs schreiben
- [Paho Mqtt Python](https://github.com/eclipse/paho.mqtt.python)
- Daten verarbeiten
- Daten an VenusOs senden
- ...?

## Useability
### Automatisch Skript starten
- Nach einem Neustart lief das Skript im VenusOS welches die Daten vom ShellyPro3EM abfrägt nicht automatisch los
- mittels `crontab` automatisch bei Neustart das install.sh Skript ausführen (eigentlich müsste man nur den Service starten, aber kA wie das geht, install.sh macht das auch)
- `crontab -e` &rarr; Zeile `@reboot sh /data/dbus-shelly-pro-3em-smartmeter/install.sh` einfügen und speichern (SRTG+C &rarr; `:wq` &rarr; ENTER)
### Zugriff von VRM (Internet) auf das VenusOS im Netzwerk ermöglichen
- in der `Remote Console`: `Settings` &rarr; `VRM online portal` &rarr; `VRM Portal ID` notieren
- in [VRM Online](https://vrm.victronenergy.com) einloggen und neue Installation hinzufügen
  - `Color Control GX` auswählen und die `VRM Portal ID` eingeben (In der `Remote Console` muss vorher `Remote Zugriff` aktiviert sein, siehe oben)


