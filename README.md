UVR1611 Data Logger Pro
======

Der *UVR1611 Datalogger Pro* ist ein webbasierender Datenlogger für die Universalregelungen UVR1611 und UVR16x2. Die Daten werden mit Hilfe des BL-NET oder CMI über den CAN oder DL abgerufen.

Die Features sind:
* UVR1611 oder UVR16x2
* BL-Net oder CMI
* Datenlogging über CAN-Bus und DL in eine MySQL Datenbank
* Web-Interface
* Animiertes Online-Schema (Elemente und Farben)
* Schema Historie
* Temperaturkurven (Tages und Wochenansicht)
* Leistungskurven (Tages und Wochenansicht)
* Ertragsdiagramme (Monats und Tagesgruppierung)
* Übersichtstabellen zu den Diagrammen
* Öffentliche und private Diagramme
* Monatliche Trendwerte
* Aktuelles Wetter

Vorteile der Anwendung
------

Der BL-Net besitzt nur eine begrenzte Speicherkapazität und die Heizungsdaten müssen deshalb regelmäßig – manuell – ausgelesen werden, damit sie nicht verloren gehen. Der direkte Webzugriff auf den BL-Net durch externe Nutzer ist in mehrfacher Hinsicht unvorteilhaft (Sicherheit und Überlastung). Das Speichern und Auslesen der Daten aus der SQL Datenbank verhindert diese Problematik.

Installation
------

Für die Anwendung wird ein Webserver mit PHP und ein MySQL Datenbank-Server benötigt. Die benötigten Pakete können unter Debian/Ubuntu mit folgendem Befehl installiert werden:

	sudo apt-get install lighttpd php5-cgi mysql-server mysql-client php5-mysql php5-curl
	sudo lighttpd-enable-mod fastcgi-php
	sudo service lighttpd force-reload

Danach kann die Anwendung in den Ordner `/var/www/` kopiert werden. 

Konfiguration
------

Die Konfiguration des Datenloggers erfolgt über die Datei `config/config.ini`:
 
	[mysql]
	server = localhost
	user = uvr1611
	password = uvr1611
	database = uvr1611
	
	[uvr1611]
	logger = "bl-net"
	address = 10.0.0.100
	port = 40000
	reset = false
	blnet_login = false
	http_port = 80
	
	[app]
	name = "Solar/Heizung Datenauswertung"
	email = bertram.winter@gmail.com
	chartcache = 600
	latestcache = 60
	reduction = 2

Im Abschnitt `mysql` befinden sich die Parameter für den Zugang zur Datenbank. Der angebene Datenbank-Benutzer benötigt die Rechte `DELETE`, `EXECUTE`, `INSERT`, `SELECT`, `SHOW VIEW` und `UPDATE` für die Datebank. Im Abschnitt `uvr1611` wird der `logger` angegeben (`bl-net` oder `cmi`). Weiters bedindet sich hier die IP-Adresse für den BL-NET/CMI. Mit dem Schalter `reset` kann das Löschen der Daten vom BL-NET nach dem Logging aktiviert werden. Der Bereich `app` legt Einstellungen zur Anwendung fest. Anwendung fest. Unter `name` kann der Eintrag „Solar/Heizung Datenauswertung“ durch einen individuellen Eintrag ersetzt werden, ebenso der Eintrag unter `email`. `chartcache` und `latestcache` legen den Zeitraum in Sekunden fest, in dem keine neuen Daten vom Bootloader geholt werden. `reduction` reduziert die Daten in den Diagrammen um den angegebenen Faktor.

Datenbank
------

Zum Erstellen der Datenbankstruktur kann das Skript `install.php` verwendet werden. Dieses Skript überprüft die Information zur Datenbankverbindung aus der Datei `config/config.ini`. Nach dem Anlegen der Datenbankstruktur, können Beispiel Daten in die Datenbank geschrieben werden, um die spätere Einrichtung des Datenloggers zu vereinfachen. Nach der Installation, muss die Datei install.php **unbedingt** gelöscht werden, um unberechtigte Zurgriffe auf die Datenbank oder den Datenlogger zu verhindern!

![Installation](./doc/install.png)


Standort Wetter
------

Mit dem Datenlogger können auch aktuelle Wetterinformationen angezeigt werden. Dazu muss in der Datenbank in der Tabelle `t_menu` ein Eintrag vom Typ `weather` erstellt werden. In der Spalte `schema` ist der gewünschte Ort einzutragen. Die Daten werden von [openWeatherMap](http://openweathermap.org) abgerufen.

Automatisches Abrufen der Daten
------
Damit die Daten automatisch vom BL-Net/CMI geholt werden, muss ein cron-Job mit folgendem Inhalt eingerichtet werden (z.B.: als `/etc/cron.hourly/uvr1611-logger`): 

	#!/bin/bash

	wget 'http://localhost/uvr1611/analogChart.php' -O /dev/null
	
Die Einrichtung des cron-Jobs ist abhängig vom Betriebssystem des Servers und muss entsprechend angepasst werden. 
Sollten die Daten auch weiterhin über WinSOL ausgelesen werden, ist zu beachten, dass bei der Löschung des Datenspeichers in der Konfigurationsdatei deaktiviert werden muss (`reset = false`)


Login und Freigabe
------

Standardmäßig sind die Diagramme nur für angemeldete Benutzer sichtbar. Das Standard-Passwort ist `1234` und sollte unbedingt nach dem Setup durch Aufrufen des Scripts `changePassword.php` geändert werden. Die Berechtigungen für einzelne Charts können in der Tabelle `t_menu` angepasst werden. Dazu muss für das jeweilige Diagramm das Feld `view` auf `yes` gesetzt werden. Als angemeldeter Benutzer hat man auch die Möglichkeit die Diagramme direkt zu bearbeiten und Linien per Drag & Drop hinzuzufügen. 

![Diagramme bearbeiten](./doc/edit.png)


Anpassen der Anwendung
------

Zuerst empfield es sich die Schema-Grafiken an die eigenen Verhältnisse anzupassen, indem diese Dateien mit einem entsprechenden Grafik Programm bearbeitet werden. Das Freeware Programm Inkscape für PC und Mac eignet sich sehr gut, weil das Festlegen der ID’s besonders einfach ist. Die ID’s werden später in der Tabelle `t_schema` benötigt und nachstehend wird kurz gezeigt, wie diese festgelegt werden.
Nachdem die Grafik angepasst wurde, setzt man mit dem Texttool an den Stellen im Diagramm, wo später z.B. die Temperatur angezeigt werden soll ein `? ° C`, klickt mit der rechten Maustaste auf dieses Objekt und es öffnet sich das nachstehende erste Fenster. Dort wird *Object Properties* angeklickt und das rechte Fenster erscheint. In diesem Fenster legt man mit einer freien Bezeichnung die *ID* und das *Label* fest. Im Beispiel ist die ID `vl2_temp` und das Label `#vl2_temp`. Der Label Eintrag wird später in der Tabelle `t_schema` unter `path` eingetragen.

![Inkscape](./doc/objectid.png)

Sind alle ID’s festgelegt, wird diese Datei im Ordner unter `images` gespeichert.

#### t_menu ####

In dieser Tabelle wird die Haupteingangsseite festgelegt. `name` und `order` sind frei festlegbar, wobei `order` die Reihenfolge der einzelnen Darstellungen in der Haupt-Menu Anzeige festlegt.
Werden einzelne Bereiche nicht benötigt, können diese Reihen einfach in der Tabelle gelöscht werden.

![Hauptmenü](./doc/t_menu.png)

#### t_names ####

In der Spalte `type` wird die aktuelle Belegung der 16 analogen UVR Sensoren Eingänge eingetragen. Unter `name` kann man angeben, welcher Eingang hier geschaltet ist. Nicht benötigte Eingänge werden wieder einfach reihenweise gelöscht.

![Senornamen](./doc/t_names.png)

#### t_names_of_charts ####

In dieser Tabelle wird festgelegt, welche Diagramme erzeugt werden sollen. Dabei wird mit der `chart_id` festgelegt, in welchem Chart welche Größen angezeigt werden sollen. Mit `order` wird bestimmt an welcher Stelle im Chart und der darunter liegenden Tabelle ein Wert angezeigt werden soll.
Soll ein Wert in mehreren Charts angezeigt werden, so muss dieser Wert entsprechend oft in `t_names_of_charts` aufgeführt werden. Im nachstehenden Beispiel ist das z.B. die Größe `analog12`. Dieser Wert wird sowohl im Chart 2 des Hauptmenues, als auch im Chart 4 angezeigt

![Verknüpfung zwischen Sensoren und Diagrammen](./doc/t_names_of_charts.png)

#### t_schema ####

Hier werden die Label-Einträge - wie vorstehend - erläutert eingetragen. Auch die Pumpen, Mischer und Ventil Einstellungen können hier festgelegt werden. Im nachstehenden Beispiel sind dies die digital-Einträge unter `type`. Im Schema erscheinen diese Einträge dann mit ihren aktuellen Zustand `EIN` oder `AUS`.

Als Formatierung kann die Anzahl der Kommastellen angegeben werden (zB.: #.## für 2 Kommastellen). Für die digitalen Ausgänge kann die Funktion `DIGITAL()` verwendet werden um EIN bzw. AUS im Schema anzuzeigen. Für Ventile und Mischer gibt es die Funktionen `MISCHER_AUF()`, `MISCHER_ZU()` und `VENTIL()`. Mischer zeigen den aktuellen Zustand AUF/ZU an und im Ruhezustand erfolgt keine Anzeige. Ventile zeigen den Zustand OFFEN/ZU an. Im nachstehenden Beispiel ist der Heizkreis-Mischer (Ausgang „digital 8“ ) für den Zustand „AUF“ definiert.

![Anzeige der aktuellen Werte im Schema](./doc/t_schema2.png)

Für die Darstellung der Erträge stehen die Funktionen `MWH()` und `KWH()` zur Verfügung. 

![Anzeige der aktuellen Werte im Schema](./doc/t_schema.png)

Darstellung der Pumpen- und Mischerzustände
------

Ab Version v0.90 lassen sich jetzt auch die Pumpen-, Mischer- und Ventilzustände im Diagramm darstellen, wenn diese entsprechend in den Tabellen `t_schema` und `t_names_of_charts` definiert worden sind. Die grafische Darstellung erfolgt in einem eigenen Diagramm oberhalb der "üblichen" Charts, wenn nicht komplett separat in `t_menu` als Chart definiert.

Screenshots
------

![Hauptmenü](./doc/menu.png)

![Schema](./doc/schema.png)

![Temperaturdiagramme](./doc/linien.png)

![Ertäge](./doc/balken.png)

![Wetter](./doc/weather.png)

Issues
------

First check whether the SQL views and procedures (at the end of [structure.sql](./sql/structure.sql)) where imported correctly and working fine.

If you get the error message "Error Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'subtab.id' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by", you have to disable this setting: `SET GLOBAL sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));`

Development
------

CAN X2 ID | Name       | Resolution | Unit
--------: | ---------- | ---------: | -----
 0 | dimensionless     |          1 |
 1 | Temperature       |        0,1 | °C
 2 | Global radiation  |          1 | W/m²
 3 | Flow rate         |          1 | l/h
 4 | Seconds           |          1 | s
 5 | Minutes           |          1 | m
 8 | Percent           |        0,1 | %
10 | Power             |       0,01 | kW
11 | Energy            |        0,1 | kWh
12 | Energy            |          1 | MWh
13 | Voltage           |       0,01 | V
14 | Current           |        0,1 | mA
15 | Hours             |          1 | h
16 | Days              |          1 | d
17 | Number of pulses  |          1 | Imp
18 | Resistance        |       0,01 | kΩ
19 | Volume            |          1 | l
20 | Speed             |          1 | km/h
21 | Frequency         |       0,01 | Hz
22 | Flow rate         |          1 | l/min
23 | Pressure          |       0,01 | bar
24 | Performance factor|       0,01 |
26 | Length            |        0,1 | m
27 | Length            |        0,1 | mm
28 | Volume            |          1 | m³
35 | Flow rate         |          1 | l/d
36 | Speed             |          1 | m/s
37 | Flow rate         |          1 | m³/min
38 | Flow rate         |          1 | m³/h
39 | Flow rate         |          1 | m³/d
46 | RAS               |        0,1 | °C
50 | Euro              |       0,01 | €
51 | Dollar            |       0,01 | $
52 | Absolute humidity |        0,1 | g/m³
53 | dimensionless,5   |    0,00001 |
54 | Degree            |        0,1 | °
58 | dimensionless,1   |        0,1 |
60 | Time              |          1 | min
63 | Current           |        0,1 | A
65 | Pressure          |        0,1 | mbar
66 | Pressure          |          1 | Pa
67 | CO2 content       |          1 | ppm
69 | Power             |          1 | W
70 | Weight            |       0,01 | t
71 | Weight            |        0,1 | kg
72 | Weight            |        0,1 | g
73 | Length            |        0,1 | cm
