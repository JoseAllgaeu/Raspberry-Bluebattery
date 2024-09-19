# Raspberry-Bluebattery
Beschreibung

POC für eine Darstellung von Messdaten aus dem Batteriecomputer “Bluebattery” mit Grafana.

Ausstattung

Raspberry Zero 2 WH

Raspberry OS: Version 11 - Lite - Bookworm 64-bit

64 MB SD Karte

InfluxDB Version 1.8.10 - 64-bit

Mosquitto Version  2.0.11

Telegraf 1.31.1-1 - 64-bit

Docker

einen BlueBattery Batteriecomputer von Kai Scheffler
https://www.blue-battery.com/solar-und-batteriecomputer?Kategorie=Batteriecomputer

Python Programm “BlueBattery” von Daniel Fett
https://github.com/danielfett/bluebattery.py

Grafana Online

Abfolgebeschreibung

Der Raspberry nimmt die Daten auf, die der Bluebattery Batteriecomputer per Bluetooth versendet. Hierzu dient das Pyhton Programm “BlueBattery” von Daniel Fett: “Software for interacting with the BlueBattery line of battery computers for RVs”.

Dieses Pyhton Programm verschickt wiederum die Messdaten weiter per MQTT an den MQTT-Brocker “Mosquitto”. Mosquitto reicht die MQTT Daten an Telegraf weiter. Telegraf “fischt” die notwendigen Messwerte heraus und trägt diese in einer InfluxDB Datenbank ein.

Um die Messdaten zu visualisieren bietet sich sehr gut Grafana an. Grafana kann ebenfalls auf demselben Raspberry installiert werden. Alles befindet sich nun lokal. Falls ein Zugriff von Extern erfolgen soll, ist ein VPN zum Router im Camper einzurichten.

Es gibt noch eine andere Möglichkeit mit Grafana Online. Grafana Online kann nicht ohne weiteres auf die lokalen Influx Daten zugreifen. Es gibt hierzu die Möglichkeit ohne VPN oder ähnlich von Extern auf die Influx Datenbank zuzugreifen. Diese Funktion nennt sich: “Private data source connect (PDC) enables you to securely connect your Grafana Cloud stack to data sources hosted on a private network.” 

Realisiert wird die Möglichkeit durch einen auf dem Raspberry zu installierenden Docker Container mit dem selben Namen: “Private Data Source Connect”.
https://grafana.com/docs/grafana-cloud/connect-externally-hosted/private-data-source-connect/configure-pdc/
https://grafana.com/docs/grafana-cloud/connect-externally-hosted/private-data-source-connect/scalability-and-security/

Dieser lokal installierte Container baut aus dem lokalem Netzwerk eine verschlüsselte SSH Verbindung zu Grafana Cloud auf. Kurzum, über Grafana Cloud können die Messdaten nun visualisiert werden. Grafana Cloud ist in der kleinsten Ausprägung kostenfrei.

Screenshots

![image](https://github.com/user-attachments/assets/c1c0ec49-bd06-4826-8df0-60413fc8fae5)

![image](https://github.com/user-attachments/assets/3c9caad7-8803-499e-a11c-e027b69c61a4)

![image](https://github.com/user-attachments/assets/d4261f6e-c875-46a7-89c4-97fb0d7ab994)

![image](https://github.com/user-attachments/assets/ddf65c3c-660c-49b8-a4b7-1a627b520cde)

Nur ein Beispiel visualisierter, aufgezeichneter Daten.

Die Grafana Dashboards:

1

2

3

4

Details

Raspberry OS als 64-bit Lite Version. 

InfluxDB 64 bit. Bitte beim Anlegen der Datenbank die Data-Retention-Policy berücksichtigen. Die Datenbanken werden einfach zu groß. Influx ist bis zu 1 Million Datensätze ausgelegt. Aus eigenen Versuchen heraus ist mit 1,7 Millionen Datensätze die Leistungsgrenze längst erreicht.

Installation des Python Programm “BlueBattery” von Daniel Fett, Beschreibung hier: https://github.com/danielfett/bluebattery.py. Das Python Programm sendet fortlaufend MQTT Daten. Die Erfassungsfrequenz ist idealerweise durch einen Cron-Job in Kombination mit einem Script auf 5 Minuten für eine Dauer von 30 Sekunden eingerichtet. 5 Minuten sind m.E. ausreichend. Bietet zugleich den Vorteil, dass für die Smartphone App “BlueBattery” die Bluetooth Verbindung für diesen Zeitraum belegt ist. Im Gegenzug ist für das Python Programm die Bluetooth Verbindung belegt.

sudo systemctl start bb.service
sleep 33s
sudo systemctl stop bb.service

Der MQTT-Brocker Mosquitto hat keine Besonderheiten. Die Konfiguration:

log_dest file /var/log/mosquitto/mosquitto.log
listener 1883
allow_anonymous true
connection_messages true

Telegraf hat ein paar Besonderheiten. Die Konfiguration für den Broadcast der MQTT Daten, die gespeichert werden sollen, muss beschrieben werden. Hier der MQTT Teil in der Telegraf Konfigurationsdatei. Die Bluetooth MAC Adresse AA:BB:CC:DD:EE  ist entsprechend dem vorhanden Bluebattery Batterie Computer anzupassen.
https://github.com/influxdata/telegraf/tree/master/plugins/inputs/mqtt_consumer

#BBB MQTT Input Configuration

 [[inputs.mqtt_consumer]]
	servers = ["tcp://127.0.0.1:1883"]
	topics = ["service/bluebattery/FC:AA:BB:CC:DD:EE/live/#"]
	data_format = "value"
	data_type = "float"

 [[inputs.mqtt_consumer.topic_parsing]]
	topic = "service/bluebattery/FC:AA:BB:CC:DD:EE/live/solar_charger/max_solar_current_day_A"
	measurement = "measurement/_/_/_/_/_"
	tags = "_/site/uuid/live/messpfad1/field"
	fields = "_/_/_/_/_/value"

Die weitere Konfiguration für Telegraf selbst, Telegraf und InlfuxDB 1.0 hier:
https://github.com/influxdata/telegraf/blob/master/docs/CONFIGURATION.md
https://github.com/influxdata/telegraf/tree/master/plugins/outputs/influxdb

Zur Kontrolle, ob die MQTT Datenkette von dem Batteriecomputer, der Phyton Software über den MQTT-Brocker funktioniert, bietet sich der MQTT-Explorer an. Einfach mit der IP-Adresse / DNS-Namen und Port 1883 mit dem Raspberry verbinden.
Webseite: https://mqtt-explorer.com/

Screenshot
![image](https://github.com/user-attachments/assets/25fa8759-41c8-40e0-a833-bef2182960ae)

Kontrolle ob die Daten in die Influx Datenbank geschrieben wurden

Docker Installation
Eine Anleitung zur Installation findet sich hier: https://docs.docker.com/engine/install/debian/#installation-methods. 
Nachfolgende Scripts: https://docs.docker.com/engine/install/linux-postinstall/ 
und weiter: https://docs.docker.com/engine/install/linux-postinstall/#configure-docker-to-start-on-boot-with-systemd.

Grafana Online, PDC Konfiguration
Unter dem Menüpunkt: Home / Conncection / Private data source connect mit der Schaltfläche “Add new network” ein neues Netzwerk hinzufügen, Namen vergeben.
Docker auswählen:
![image](https://github.com/user-attachments/assets/fa04937c-072c-412a-abdd-f4f1f26ac4a1)

Einen Token generieren:
![image](https://github.com/user-attachments/assets/821d0ee6-5db2-463a-9b92-5ae04b474f3a)

Nun wird ein Token generiert und dieser wird in dieser CMD-Line eingetragen. Diese CMD-Line in die Zwischenablage nehmen und auf dem Raspberry ausführen. Hinweis den Token ist hier im Screenshot nicht übernommen. Statt dessen wird der Platzhalter GCLOUD_PDC_SIGNING_TOKEN gezeigt:
![image](https://github.com/user-attachments/assets/e2bc335a-7d75-4b8f-8f84-70adcdae44c9)
 
Nun auf dem Raspberry mit dem generierten Token die CMD-Line ausführen. Hinweis, der Docker Container ist nur für ein Raspberry OS 64-bit vorgesehen.

sudo docker run --name pdc-agent grafana/pdc-agent:latest -token GCLOUD_PDC_SIGNING_TOKEN -cluster prod-eu-west-2 -gcloud-hosted-grafana-id 683208


Nach erfolgter Installation die Verbindung prüfen:
![image](https://github.com/user-attachments/assets/39aeb09f-0988-4078-a83e-b2f3e6ffd331)

Nun mit “Create a new datasource” weiter:
![image](https://github.com/user-attachments/assets/b24cc1a6-bb77-4aa8-8f86-9ae9011b0003)
 
Auswahl von InfluxDB:
![image](https://github.com/user-attachments/assets/309486b6-6d7c-464a-887e-60b2422838c8)
 
Einen beliebigen Namen für die “Data source” bestimmen:
![image](https://github.com/user-attachments/assets/4477099a-695a-41a8-b6ca-856139837d15)
 
Hier wird nun die IP-Adresse / DNS-Namen des Raspberry mit dem Port 8086 für die InfluxDB Instanz angegeben:
![image](https://github.com/user-attachments/assets/b3465487-c78c-4297-ad1a-40c2ddb2dd6c)

Zuletzt den Datenbank Name, der der InfluxDB bzw. Telegraf Konfiguration als Datenbank zugrunde liegt, in dem Feld “Database” eintragen:
![image](https://github.com/user-attachments/assets/cb7013bb-1b04-4be3-8f18-5cd0b7d01034)

Zuletzt die BlueBattery Dashboards hinzufügen. 
Zuvor mit Notepad / Notepad++ in den Dashboard “JSON”-Dateien die Bluetooth MAC Adresse AA:BB:CC:DD:EE gegen die MAC-Adresse des vorhandenen Bluebattery Batterie Computer ersetzten
Hier eine Beispielszeile, kommt in der “JSON” Datei öfters vor:

"value": "service/bluebattery/FC:AA:BB:CC:DD:EE/live/measurement/battery_voltage_V"

kk
