# ÜK Modul 188: Services betreiben, warten und überwachen

Stand: 10.10.2026. Für Informatiker/in EFZ Plattformentwicklung (BiVo 2021), ICT-Learnfactory.

## 1. Gesichert (aus Quellen)

- Titel: Services betreiben, warten und überwachen
- Ort im Bildungsplan: üK, 2. Lehrjahr, 3. Semester (Plattformentwicklung)
- Dauer: 5 Tage (Angabe Kanton Graubünden / vfei.ch)
- Reihenfolge der üK im Standardpfad (Bern): 187, 216, 106, dann 188, 190, 109, 184
- Vorwissen laut üK-Broschüre Bern: gibb-Module 117 (Informatik- und Netzinfrastruktur) und 123 (Serverdienste in Betrieb nehmen) repetieren
- Folge-üK bauen darauf auf: 190 (Virtualisierung) und 109 (Public Cloud) setzen M188 voraus
- Parallele Schulmodule im 3. Semester: 123, 129, 346, 141

Vergleichskurs Klubschule Pro (Windows-Server-Variante, 54 Lektionen), Themen:
- Active Directory Domain Services installieren und konfigurieren
- Serverinterne Tools
- PowerShell Grundlagen, AD-Verwaltung mit PowerShell
- Updates und Patches verwalten
- SNMP und WMI zum Abfragen von Netzwerkkomponenten
- Schwellwerte für Monitoring definieren, Monitoring-Tool PRTG installieren
- Berechtigungskonzept erstellen und umsetzen
- Gruppenrichtlinien (GPO)
- Anbindung externer Systeme

### Offizielle Leistungsziele (Bildungsplan Informatiker/in EFZ, BiVo 2021)

Im Bildungsplan (https://www.becc.admin.ch/becc/public/bvz/beruf/download/10475) ist Modul 188 drei Handlungskompetenzen zugeordnet. K3 bis K5 sind die Taxonomiestufen.

**d1 ICT-Prozesse aufnehmen, standardisieren und automatisieren**
- d1.1 Kundenbedürfnisse zu ICT-Prozessen als technische Anforderungen lösungsneutral festhalten (K4)
- Bezug zu Schulmodul 122 (Scriptsprache), Beispiele im Text: Benutzermanagement, Service-Provisionierung, Loganalysen

**f5 Serversysteme und -dienste überwachen**
- f5.1 Mit Stakeholdern definieren, welche Serversysteme und -dienste überwacht werden (K3)
- f5.2 Geeignete Überwachungsmethoden bestimmen (K4)
- f5.3 Unter Risikoeinschätzung sinnvolle Schwellenwerte und Aktionen festlegen (K4)
- f5.4 Regelmässig per Tests die Funktion des Überwachungssystems prüfen und dokumentieren (K4)
- f5.5 Bei Fehler- oder Alarmmeldungen rasch geeignete Massnahmen ergreifen (K4)
- Methoden laut Text: Script, Software, Benachrichtigungen. Risiken auch anhand von SLA abschätzen.

**f6 Sicherheit von Serversystemen und -diensten implementieren, dokumentieren, überprüfen**
- f6.1 Im Team nach Best Practices ein Sicherheitskonzept für Serversysteme und -dienste erstellen (K5)
- Themen im Text: IT-Grundschutz, Firewall, Antivirus, Berechtigungen, SSO
- Weitere Ziele f6.2 bis f6.4: Sicherheitselemente konfigurieren (K3), regelmässig testen und dokumentieren (K4), Konzept auf Aktualität prüfen (K4)

Zugehörige Schulmodule laut Bildungsplan bei f5: 123, 182, 346. Bei f6: 141, 143 (Backup und Restore), 169 (Container), 182.

Das heisst für die Praxis: Im Kurs geht es sehr wahrscheinlich um Monitoring (was, wie, Schwellwerte, Alarm, Test), um Automatisierung per Script und um ein Sicherheitskonzept für Server.

## 2. Nicht gesichert

- Der genaue Text der Modulidentifikation (Handlungsziele, Kenntnisse im Modulbaukasten) liegt nur in der Web-App modulbaukasten.ch und nicht als öffentliche PDF vor. Ich konnte ihn nicht lesen. Die Leistungsziele oben stammen aus dem Bildungsplan, nicht aus der Modulidentifikation.
- Welche Tools die ICT-Learnfactory einsetzt (Windows Server oder Linux, PRTG, Zabbix, Nagios, Prometheus), ist nicht bestätigt. Die ÜK-Unterlagen kommen meist erst am Kurs oder über die Lernplattform.

## 3. Was ich vor dem Kurs klären würde

1. Sekretariat fragen: sekretariat@ict-learnfactory.ch, +41 31 521 00 90. Fragen: Welche Plattform (Windows oder Linux), welche Monitoring-Tools, gibt es Vorabunterlagen oder Zugang zur Lernplattform?
2. Kursprogramm im Kursverwaltungstool OdAOrg ansehen (laut ict-learnfactory.ch ab Juli sichtbar).
3. Modulbeschreibung selbst im Browser öffnen: https://www.modulbaukasten.ch, Modul 188 suchen, PDF herunterladen.
4. Eigene gibb-Unterlagen zu 117 und 123 durchgehen.

## 4. Vorbereitung nach Themen (aus meinem Fachwissen, mit Vorbehalt)

Der Titel deckt typischerweise vier Bereiche ab. Welche davon im Kurs wirklich vorkommen, hängt vom Anbieter ab.

### Betreiben
- Dienste: DNS, DHCP, Fileserver, Verzeichnisdienst (AD oder LDAP), Webserver
- Systemdienste verwalten: Windows `services.msc`, `Get-Service`, Linux `systemctl status/start/stop/enable`, `journalctl -u`
- Benutzer, Gruppen, Berechtigungen (NTFS/Share bzw. chmod/chown/ACL)

### Warten
- Patch- und Update-Management (Windows Update, WSUS, `apt`/`dnf`)
- Backup und Restore (3-2-1-Regel, Voll/Inkrementell/Differentiell, Restore-Test)
- Logs rotieren, Speicherplatz prüfen, Zertifikate erneuern
- Automatisieren mit PowerShell oder Bash

### Überwachen
- Metriken: CPU, RAM, Disk, Netzwerk, Dienststatus, Antwortzeit
- Protokolle: SNMP (OIDs, MIB, Community, v3), WMI, Ping/ICMP, Agenten
- Schwellwerte, Alarmierung (Warning/Critical), Benachrichtigung per Mail
- Tools: PRTG, Zabbix, Nagios/Icinga, Prometheus + Grafana
- Logs zentral sammeln (Syslog, Windows Event Log)

### Dokumentation und Betrieb
- Betriebshandbuch, Netzwerkplan, Passwort- und Berechtigungskonzept
- Incident-Ablauf: erkennen, eingrenzen, beheben, dokumentieren
- Begriffe: SLA, Verfügbarkeit, RTO/RPO, Change-Management, ITIL-Grundidee

## 5. Konkrete Übungen für die zwei Wochen

- Zwei VMs aufsetzen (VirtualBox oder Hyper-V), eine als Server, eine als Client
- Auf dem Server DNS, DHCP und einen Fileserver einrichten, Client in die Domäne oder ins Netz aufnehmen
- PRTG (Gratisversion bis 100 Sensoren) oder Zabbix installieren und drei Sensoren einrichten (Ping, Disk, Dienst)
- Einen Dienst absichtlich stoppen und prüfen, ob der Alarm kommt
- Ein PowerShell- oder Bash-Skript schreiben, das Dienststatus und freien Speicher prüft
- Backup erstellen, Datei löschen, wiederherstellen

## 6. Freie Unterlagen zu Nachbarmodulen

- tbz-it auf GitHub, M122 (Abläufe mit Scriptsprache automatisieren): https://github.com/tbz-it/M122
- tbz-it auf GitHub, M169 (Dienste mit Containern bereitstellen): https://github.com/tbz-it/M169
- Ein Repository speziell zu M188 habe ich dort nicht gefunden. Die Organisation hat mehr Repos, als die Übersichtsseite zeigt: https://github.com/orgs/tbz-it/repositories?type=all
- Ausbildungskonzept Plattformentwicklung (RAU, ab 2026): https://www.r-au.ch/wp-content/uploads/Ausbildungskonzept_Informatik_Plattformentwicklung_ab-2026-2027.pdf

## 7. Quellen

- Bildungsplan Informatiker/in EFZ (BiVo 2021), SBFI: https://www.becc.admin.ch/becc/public/bvz/beruf/download/10475

- ICT-Berufsbildung Bern, üK-Broschüre: https://www.ict-berufsbildung-bern.ch/resources/uek-Broschuere.pdf
- Bildungsplan Modulübersicht (Ost): https://ict-berufsbildung-ost.ch/wp-content/uploads/2021/03/BiPla_Informatiker_20201119_Modulu%CC%88bersicht.pdf
- Modulübersicht: https://mbk.ict-berufsbildung.ch/assets/module/Moduluebersicht-DE-V3.pdf
- Schullehrplan GB Chur: https://www.gbchur.ch/wp-content/uploads/2024/09/Interner-Schullehrplan-Informatiker_in-EFZ-Plattformentwicklung.pdf
- Klubschule Pro, Parallelkurs: https://www.klubschule-pro.ch/kurs/it-services-betreiben-warten-und-uberwachen--D_101476_1016_313
- Modulbaukasten: https://www.modulbaukasten.ch
- ICT-Learnfactory: https://www.ict-learnfactory.ch
