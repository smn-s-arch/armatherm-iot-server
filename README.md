# Installation Armatherm IoT-Server für LoRaWAN

## Systemanforderungen

OS: Ubuntu Server 22.04 LTS

Hardware: 2-core CPU, 64GB HDD, 2048MB RAM, für Umgebungen bis 500 Sensoren. 

Portfreigaben: Port 8080 + 8090 für Chirpstack, Port 9090 für Thingsboard, Port 1883 für MQTT

Zugand ins Internet für den Server, sowie das MTCAP-Gateway. (Nur für die Installation. Danach kann der Betrieb ohne Internet erfolgen)

---

## Installation

Via SSH auf dem Server einloggen:
```bash
ssh <username>@<ip-des-ubuntu-servers>
```
Alle nachfolgenden Befehle werden immer in der Shell auf dem Server ausgeführt.

### 1. Vorbereitung: 
Die folgenden Repos müssen in das home Verzeichnis des Servers geklont werden:
- [armatherm-chirpstack](www.github.com/smn-s-arch/armatherm-chirpstack) 

    ```bash
    git clone https://github.com/smn-s-arch/armatherm-chirpstack.git
    ```

- [armatherm-thingsboard](www.github.com/smn-s-arch/armatherm-thingsboard)

    ```bash
    git clone https://github.com/smn-s-arch/armatherm-thingsboard.git
    ```

- [armatherm-mtcap](www.github.com/smn-s-arch/armatherm-mtcap)
    
    ```bash
    git clone https://github.com/smn-s-arch/armatherm-mtcap.git
    ```
---

### 2. Installation Chirpstack-Netzwerkserver
Der Chirpstack Netzwerkserver empfängt die Daten per MQTT von dem Gateway und verwaltet das LoRa Netzwerk. Eine ausführliche Dokumentation ist zu finden unter: [Chirspstack.io](www.chirpstack.io/docs)

Die Installation erfolgt über ein Installations Script `install.sh`

Im Rahmen der Installation werden folgende Pakete installiert: 
- mosquitto
- mosquitto-clients
- redis-server
- redis-tools
- postgresql
- apt-transport-https
- dirmngr
- chirpstack-gateway-bridge
- chirpstack
- chirpstack-rest-api

Um die Installation zu starten rufe das eben geklonte repo **armatherm-chirpstack** auf. 

```bash
cd ~/armatherm-chirpstack/
```
Das Skript muss ausführbar gemacht werden mit:
```bash
chmod +x install.sh
```
Danach kann die Installation gestartet werden:
```bash
sudo ./install.sh
```
Hier müssen nun einige Parameter für die Installation eingegeben werden. 
1. `ChirpStack DB username` Dies ist der User der für die Chirpstack Datenbank angelegt wird. Bspw. *chirpstack*
2. `ChirpStack DB password` Das Passwort für die Chirpstack Datenbank. Es sollte möglichst sicher sein.

Wenn die Installation erfolgreich abgeschlossen wurde, findet man in dem Verzeichnis eine `chirpstack_install_info.txt` mit Informationen zur Installation, den Zugangsdaten, sowie der aktuellen Netwwerkkonfiguration. 

Das Passwort und der Username sollten sicher abgespeichert werden, falls man später in der DB etwas anpassen muss. Für die Benutzung des Chirpstack Servers sind sie allerdings nicht notwendig. Hierfür werden Logins in einem Späteren Schritt erstellt.

---

### 3. Installation Thingsboard Plattform
Über die IoT Plattform Thingsboard werden die Daten, welche vom Chirpstack Server gesammelt werden grafisch aufbereitet, sodass der Enduser eine Übersichtliche Oberfläche zum Ablesen der Messdaten erhält. Ebenfalls können hier die Messgeräte konfiguriert werden.

Die Installation erfolgt über ein Installations Script `install.sh`

Im Rahmen der Installation werden folgende Pakete installiert:
- openjdk-17-jdk
- thingsboard

Um die Installation zu starten rufe das eben geklonte repo **armatherm-thingsboard** auf. 

```bash
cd ~/armatherm-thingsboard/
```
Das Skript muss ausführbar gemacht werden mit:
```bash
chmod +x install.sh
```
Danach kann die Installation gestartet werden:
```bash
sudo ./install.sh
```
Hier müssen nun einige Parameter für die Installation eingegeben werden. 
1. `Thingsboard DB username` Dies ist der User der für die Chirpstack Datenbank angelegt wird. Bspw. *thingsboard*

2. `Thingsboard DB password` Das Passwort für die Chirpstack Datenbank. Es sollte möglichst sicher sein. 

Das Passwort und der Username sollten sicher abgespeichert werden falls man später in der DB etwas anpassen muss. Für die Benutzung des Chirpstack Servers sind sie allerdings nicht notwendig. Hierfür werden Logins in einem Späteren Schritt erstellt.

---

### Wenn die Installationen erfolgreich durchgelaufen sind, sind folgende Web-Services erreichbar:

Der Chirpstack Netzwerkserver: `http://<HOST-IP>:8080` 

Die Chirpstack REST API: `http://<HOST-IP>:8090`

Die ThingsBoard Plattform: `http://<HOST-IP>:9090`

---

### 4. Konfiguration Multitech Gateway

Auf dem Gateway welches das LoRaWAN bereitstellt muss der `chirpstack-mqtt-forwarder` installiert werden. Eine ausführliche Dokumentation dazu findet sich auf [chirpstack.io](https://www.chirpstack.io/docs/chirpstack-mqtt-forwarder/install/multitech.html#multitech-conduit-ap3)

Für Gateways des Typs Multitech Conduit AP3 ist der Ablauf wie folgt: 

1. Die Spannungsversorgung des Gateways herstellen. 
2. Das Gateway via Ethernet mit einem Computer direk verbinden (Kabel vom Gateway in Netzwerkanschluss des PC)
3. Im Webbrowser die Adresse https://192.168.2.1 aufrufen
4. Den Anweisungen zum Erstellen eines Benutzer folgen. 
5. Die Netwerkeinstellungen entsprechend der benötigten Gegebenheiten konfigurieren. Wenn das Gateway in ein Netzwerk eingebunden werden soll, muss der DHCP-Server des Gateways abgeschaltet werden und die Netzwerkkonfiguration auf *DHCP-Client* eingestellt werden. 
6. **SSH aktivieren** Im Menü auf der linken Seite auf *Administration* und dann auf *Access configuration* klicken. Unter *SSH Settings* die Option SSH auf **Enabled** stellen. Am Ende der Seite Einstellungen speichern mit Klick auf **Submit**
7. Die Einstellungen mit Klick auf **Save & Apply** oben im Fenster übernehmen. 

Das Gateway startet neu. Wenn die Status LED anfängt im Herz-Rythmus zu blinken ist die Konfiguration abgeschlossen. Das Gateway kann nun per LAN-Kabel in das Netwerk eingebunden werden. 

Für den nächsten Schritt wird benötigt:

- Die **IP Adresse** des **Gateways**
- Der **Benutzername** des Gateways
- Das **Passwort** des Gateways
- Die **IP** des **Chirpstack Servers**
- Den **Port** des **MQTT Server** (Standardmäßig 1883)
- Die APP **sshpass**.: 

```bash
sudo apt-get install sshpass
``` 

In das Verzeichnis **armatherm-mtcap** wechseln.
```bash
cd ~/armatherm-mtcap
```
Die Datei `configuration.sh` ausführbar machen,
```bash
chmod +x configuration.sh
```
und anschließend ausführen:
```bash
./configuration.sh
```
Für die Installation des **chirpstack-mqtt-forwarder** die Option ```i``` wählen. Soll nur ein Update der Konfiguration erfolgen, bspw. IP des Servers ändert sich, kann dies anschließend mit der Option ```u``` gemacht werden.

**Die Installation von Chirpstack und ThingsBoard, und die Konfiguration des Gateways sind nun abgeschlossen.** 


# Konfiguration Chirpstack und Thingsboard

## Chirpstack Konfiguration

### 1. Admin User erstellen
In der Weboberfläche von Chirpstack `http://<serverip>:8080` mit user *admin* und passwort *admin* einloggen.

In der linken Spalte unter *Network Server* auf *Users* klicken. Mit Klick auf die blaue Schaltfläche *Add user* einen neuen Benutzer erstellen. 

**E-Mail** und **Passwort** vergeben, und die Optionen ***Is active*** und ***Is admin*** aktivieren. 

Anschließend mit dem neuen Benutzer einloggen und den default Admin account deaktivieren.

        Zugangsdaten gut dokumentieren!

### 2. Tenant erstellen
In der linken Spalte unter *Network Server* auf *Tenants* klicken. Mit Klick auf die blaue Schaltfläche *Add tenant* einen neuen Tenant erstellen. Der Tenant ist in der Regel der Kunde.

***Namen*** eingeben und Option ***Tenant can have Gateways*** auswählen.

### 3. Gateway hinzufügen

In der Chirpstack Weboberfläche im linken Menü unter *Tenant* auf *Gateways* klicken. Über den Button **Add gateway** das Gateway hinzufügen, einen Namen eingeben und die *Gateway ID* eingeben. Die Gateway ID ist zu finden in der Weboberfläche des Gateways im Bereich *LoRaWAN*.
Einrichtung mit Klick auf **Submit** abschließen. Das Gateway sollte nach ein paar Minuten als online angezeigt werden. 

    Sollte das Gateway als nicht erreichbar angezeigt werden, liegt entweder ein Problem mit der Netzwerkkonfiguration des Gateways oder der Konnektivität des MQTT Servers vor. 

### 4. Device profile erstellen
In der Chirpstack Weboberfläche im linken Menü unter *Tenant* auf *Device Profiles* klicken. Über den Button **Add device profile** ein neues Profil erstellen.

Das *Device Profile* anhand des Screenshots für den jeweiligen Gerätetyp einrichten. Unter dem Reiter *Codec* den `payloadcodec.js` einfügen. Die Device-Profiles und payloadcodecs sind im Verzeichnis **armatherm-chirpstack** zu finden. 

### 5. Application erstellen
In der Chirpstack Weboberfläche im linken Menü unter *Tenant* auf *Applications* klicken. Über den Button **Add application** eine neue Application erstellen.

### 6. Geräte hinzufügen
Die eben erstellte Application anklicken. Über den Button **Add device** kann nun das Gerät hinzugefügt werden. Hierfür werden benötigt: 

- Die DEV EUI / Device EUI (zu finden im Geräte-Pass welcher mit dem Gerät geliefert wurde)
- Die APP EUI / Application Key (zu finden im Geräte-Pass welcher mit dem Gerät geliefert wurde)

1. Das Gerät benennen (z.B. Temperatur Messstelle A)
2. Die **Device EUI** eingeben
3. Das **Device profile** auswählen
4. Klick auf **Submit**
5. Den **Application Key** im neuen Fenster eingeben
6. Klick auf **Submit**

Nun kann das Gerät gestartet werden. Es sollte sich nach ein paar Minuten am LoRa-Server anmelden. (Eintrag bei *Last Seen*)

    Schritt 6. mit allen hinzuzufügenden Geräten wiederholen.

## Thingsboard Konfiguration

### Einrichtung Kundenkonto 

In der Weboberfläche von Thingsboard `http://<serverip>:9090` mit user *sysadmin@thingsboard.org* und passwort *sysadmin* einloggen. Als erstes sollte das Passwort des Sysadmin Accounts geändert werden. Dazu oben rechts über die drei Punkte auf *Account* klicken. Unter *Security* ein sicheres Passwort vergeben. Danach das Konto für den Kunden anlegen:

1. Im Menü links den Reiter *Tenants* auswählen. 
2. Neuen Tenant hinzufügen (Klick auf + oben rechts)
3. Daten des Kunden entsprechend eingeben (Tenant profile = Default)
4. Tenant anklicken
5. Im geöffneten Fenster auf **Manage tenant admins** klicken um einen Admin zu erstellen
6. Neuen Tenant-Admin hinzufügen (Klick auf + oben rechts)
7. Daten des Admins eingeben und Activation method *Display activation link* auswählen
8. Den angeizeigten link im Browser öffnen
9. Passwort vergeben


        Zugangsdaten gut dokumentieren!

### Grundkonfiguration im Kundenkonto

#### 1. Rule chains (Regelketten) importieren
Die Rule chains sind zu finden im Verzeichnis **armatherm-thingsboard**. 

1. Im Menü links Reiter *Rule chains* auswählen
2. Neue Rulechain hinzufügen (klick auf + oben rechts)
3. *Import rule chain* auswählen
4. Rule chain wird importiert. Mit Klick auf orangenen Haken unten rechts speichern.
5. Alle rule chains einzeln importieren
6. Die rule chain `Basis Regelkette` zur Root rule chain machen. 

Während des Imports der rule chains gehen einige Verknüpfungen verloren. Diese müssen wiederhergestellt werden. Die Betroffenen Nodes innerhalb der Rule chain sind die, welche auf eine andere Rule chain verknüpfen. Siehe Rulechain-Abb1.png. So wird die Verknüpfung wiederhergestellt: 

1. Rule chain bearbeiten (Rule chain anklicken, dann rundes Symbol mit Stift anklicken)
2. Im geöffneten Fenster im Feld Rule chain* die richtige rule chain auswählen. (Info siehe Feld Name)
3. Änderung bestätigen (Orangener Kreis mit Haken)

**Diese Schritte für alle Rule chain Verknüpfungen wiederholen. Alle Rulechains durchschauen, ob Verknüpfungen zu anderen Rule chains vorhanden sind.**

#### 2. API-Token in Downlink-Rulechain einrichten
Damit die ThingsBoard Plattform Daten an den Chirpstackserver senden kann muss ein API-Token generiert werden:

1. In Chirpstack Weboberfläche einloggen
2. Im Menü unter Tenant auf *API Keys* klicken
3. Button **Add API Key** erstellt einen neuen API-Key
4. Name: **Thingsboard-Downlink**
5. **Submit**
6. Den erhaltenen Key in die Zwischenablage kopieren. 
7. In ThingsBoard die rule chain **Downlink Nachrichten** öffnen
8. Die node **rest api call** bearbeiten
9. Unten im Fenster in der Sektion *Header* bei authorization im Feld Value den API Key eintragen. **Wichtig** darauf achten das dass Prefix **Bearer** stehen bleibt. 

        Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJ....

#### 3. Dashboards importieren
Für die verschiedenen Gerätetypen sind Muster Dashboards abgelegt im Verzeichnis **armatherm-thingsboard\tb-dashboards**
So wird ein Dashboard importiert: 

1. In ThingsBoard Weboberfläche einloggen
2. Im Menü links auf *Dashboards* klicken
3. Klick auf *+* oben rechts
4. *Import dashboard* auswählen
5. Je nach Gerätetyp die richtige Datei auswählen 

#### 4. Device profile erstellen
Es ist empfehlenswert für jeden Gerätetyp ein *Device profile* zu erstellen. 

1. Links im Menü unter *Profiles* auf *Device profiles* klicken
2. Neues Profil anlegen. (*+* oben rechts)
3. *Name* eingeben (i.d.R. Gerätetyp, z.B. LK30)
4. *Default rule chain* auswählen (i.d.R. Basis Regelkette)
5. **Speichern**

#### 5. Device erstellen
Nun kann der LoRa-Sensor hinzugefügt werden. Es ist empfehlenswert die Benennung analog der im Chirpstack Server zu wählen, um eine möglichst einfache und eindeutige Identifizierung sicherzustellen. 

1. Links im Menü unter *Entitys* auf *Devices* klicken
2. Device anlegen (*+* oben rechts)
3. *Namen* eingeben
4. *Device profile* auswählen
5. **Add**
6. Die sich öffnende Seite schließen
7. *Device details* öffnen (Klick auf das angelegte Gerät)
8. oben im Reiter auf *Attributes*
9. Bei *Client attributes* auf *Server attrubutes* umschalten
10. Hier müssen nun die **Attribute** für das jeweilige Gerät erstellt werden. In dem Verzeichnis **armatherm-thingsboard\tb-devices** sind `.txt` Dateien für die verschiedenen Gerätetypen zu finden. Diese enthalten die Attribute welche einzeln angelegt werden müssen:
    1. Klick auf *+*
    2. Key eingeben (z.B. Temperatur)
    3. Datentyp auswählen 
    4. Value eingeben
    5. **Add**
    
    **Wichtig: Kommentare der einzelnen Attribute beachten**

#### 6. ThingsBoard + Chirpstack *Device access* konfigurieren
Damit der ThingsBoard Server Daten vom Chirpstack erhält muss für jedes Gerät ein **access token** eingerichtet werden:

1. In den *Device details* in ThingsBoard unter *Details* Klick auf **Copy access token**.
2. In die Chirpstack Weboberfläche wechseln
3. Über das Menü *Applications* auf das Gerät gehen, welches zu dem eben erstellten Gerät in ThingsBoard gehört.
4. Im Reiter *Configuration* auf den Reiter *Variables* gehen
5. **Add variable**
6. Key: `ThingsBoardAccessToken` Value: Der eben kopierte Access token aus ThingsBoard. Beispiel: `w5IXhiJtecDARMVIFdJG`
7. **Submit**

Das Gerät sollte jetzt seine Daten an ThingsBoard weiterleiten. 

#### 7. Dashboard für das Gerät konfigurieren
In dem Dashboard muss in den einzelnen Widgets das Gerät hinterlegt werden, damit die korrekten Daten angezeigt werden und Einstellungen gemacht werden können: 

1. Dashboard öffnen und auf bearbeiten klicken (Stift oben rechts)
2. Jedes Widget öffnen und bei *Device* das korrekte Gerät auswählen
3. **Achtung: Bei manchen Widgets muss das Gerät an zwei Stellen ausgewählt werden**
4. Klick auf **Apply** oben rechts 
5. Nachdem alle Widgets bearbeitet wurden, Dashboard speichern (Haken oben rechts)

**Die Einrichtung von ThingsBoard und Chirpstack ist jetzt abgeschlossen**