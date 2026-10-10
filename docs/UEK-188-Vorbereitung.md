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

## 2. Offizielle Modulidentifikation (Modulbaukasten, Version 1.0, 26.03.2021)

PDF: https://modulbaukasten.ch/Module/188_1_Services%20betreiben%2C%20warten%20und%20%C3%BCberwachen.pdf
Webseite: https://www.modulbaukasten.ch/module/188/1/de-DE?title=Services-betreiben,-warten-und-%C3%BCberwachen

**Kompetenz:** Betreibt und aktualisiert Services in einer bestehenden Umgebung. Überwacht Performance, Verfügbarkeit sowie Systemsicherheit gemäss den Betriebs- und Sicherheitsvorgaben eines Unternehmens.

**Objekt:** Server mit File-, Print-, DHCP-, DNS- und Verzeichnis-Services, betriebsbereites LAN mit Arbeitsstationen (Clients). Kompetenzfeld: System Management.

**Handlungsziele und handlungsnotwendige Kenntnisse**

1. Mit der Betriebsdokumentation die bestehenden Systeme und ihre Umgebung analysieren
   - 1.1 Inhalt, Aufbau und Anwendung einer Betriebsdokumentation
   - 1.2 Bedeutung einer vollständigen, nachgeführten Doku für die Wartbarkeit
   - 1.3 Eigenschaften verbreiteter Services (Verzeichnisdienst, File, Print, DHCP, DNS) und ihr Beitrag im Netzwerk
2. Services überwachen und betreiben mit den verfügbaren Hilfsmitteln
   - 2.1 Im Betriebssystem integrierte Hilfsmittel zur Systemüberwachung
   - 2.2 Wichtigste Performancemesswerte kennen und interpretieren
3. Updates und Patches manuell und mit Softwareverteilungssystemen installieren und testen, Betriebsdoku nachführen
   - 3.1 Vorgehen bei der Installation von Updates und Patches
   - 3.2 Vertrauenswürdige Quellen und Sicherheitsmassnahmen (Hashwerte, Schlüssel)
   - 3.3 Mögliche Auswirkungen und Gefahren von Patches für ein Unternehmen
   - 3.4 Szenarien zum Testen von Patches und Updates
4. Systeme in bestehende Monitoring-Tools einbinden und Messwerte verifizieren
   - 4.1 Techniken zur zentralen Erfassung von Performance-Daten (SNMP, WMI) und deren Sicherheitsmechanismen
   - 4.2 Konfigurationsschritte auf dem Server, um Messungen zu aktivieren
   - 4.3 Schritte, um Server in ein Monitoring-Tool einzubinden
   - 4.4 Sinnvolle Schwellwerte setzen, Alarmierung einrichten
5. Berechtigungen nach bestehendem Berechtigungskonzept administrieren und dokumentieren
   - 5.1 Inhalt, Aufbau, Anwendung eines Berechtigungskonzepts
   - 5.2 Möglichkeiten der Services, Zugriffsrechte auf Ressourcen festzulegen
   - 5.3 Vorgehen bei Berechtigungsanpassungen gemäss Konzept
   - 5.4 Methoden, um Anpassungen zu dokumentieren
6. Erforderliche Anpassungen auf Umsystemen definieren
   - 6.1 Anforderungen an Umsysteme (z.B. DNS-Einträge, Firewall-Regeln)
   - 6.2 Anforderungen so beschreiben, dass Dritte sie umsetzen können
   - 6.3 Anpassungen an Umsystemen testen

Die Kenntnisse sind laut Dokument nicht abschliessend. Die genauen Lernziele und den Lernweg legt der Bildungsanbieter fest (also die ICT-Learnfactory).

## 2b. Noch offen

- Die Kompetenzmatrizen ZH (https://kompetenzmatrix.ch/cluster-platform/m188/) führen M188 als «ÜK-Modul, keine Matrix vorhanden». Dort gibt es keine Lernziele dazu.
- Welche Tools die ICT-Learnfactory einsetzt (Windows Server oder Linux, PRTG, Zabbix, Nagios). Das Objekt «File-, Print-, DHCP-, DNS-, Verzeichnis-Services» und der Vergleichskurs sprechen für Windows Server und Active Directory, bestätigt ist es nicht.

## 2c. Infos der ICT-Learnfactory (https://www.ict-learnfactory.ch/informatiker)

- Ort: ICT-LearnFactory AG, Bernapark 28, Eingang West, 2. Stock, 3066 Stettlen (Deisswil)
- Aufbau: 7 üK-Module zu je 5 Kurstagen. Modul 188 ist im 2. Bildungsjahr (zusammen mit 190 und 109)
- Kosten pro Tag: CHF 160 (Mitglieder) bzw. CHF 240 (Nicht-Mitglieder)
- Lernkonzept: begleitetes, selbstorganisiertes Lernen (BGSOL), handlungskompetenzorientiert. Du arbeitest also viel selbständig mit Aufträgen und nicht nur im Frontalunterricht.
- Tools, Lernplattform und Unterlagen stehen auf der Website nicht. Das Kursprogramm ist im Kursverwaltungstool OdAOrg sichtbar.
- Jeder üK schliesst mit einer Leistungsbeurteilung ab (Prüfung und/oder Projektbewertung nach Vorgaben des Modulbaukastens), laut üK-Broschüre Bern.

- Pädagogisches Konzept (PDF, 2 Seiten): https://www.ict-learnfactory.ch/_files/ugd/c05327_9d2feafdc11d41f6ba8e7d510e7bd84e.pdf
  - Lerncoaches begleiten, du arbeitest selbstorganisiert und problembasiert an realistischen Aufgaben
  - Lerninhalte liegen für Lernende und Lehrbetriebe transparent auf einer Plattform. Frag das Sekretariat nach dem Zugang, vielleicht kannst du schon vorher hineinschauen.
  - Aufgaben enthalten einen Teil mit freier Wahl des Lösungswegs
  - Du führst ein Lernjournal, in dem du das Gelernte festhältst. Selbstüberprüfungen sind nicht notenwirksam.
  - Lehrbetriebe können firmeneigene Aufgaben einbringen. Frag deinen Berufsbildner, ob er das für den 188er plant.
- Zum Vergleich ICT-BZ (anderer Anbieter): Kurs 08:00 bis 16:45, eigenes Notebook mitbringen, Unterlagen früherer Kurse mitnehmen, mindestens 80 Prozent Anwesenheit. Bei der Learnfactory würde ich das beim Sekretariat bestätigen lassen.

## 2d. Was andere Lernende im üK 188 behandelt haben

Quelle: öffentliche Lerndokumentation eines Lernenden (Robin Rüegg, 2023, anderer Kursanbieter): https://robinrueegg.ch/my-it-skills/1-year-2/uek-188/ mit PDFs unter http://robinrueegg.ch/wp-content/uploads/2023/03/Dokumentation_Robin.pdf und http://robinrueegg.ch/wp-content/uploads/2023/03/UeK-188.pdf. Das zeigt, wie ein 188er in der Praxis aussehen kann, ist aber kein Beleg für den Ablauf bei der ICT-Learnfactory. Seine Doku ist 28 Seiten lang. Die Themen in meinen Worten:

- Einstieg: IPv4-Subnetting (z.B. Netz-ID, Broadcast, erste und letzte nutzbare Adresse bei einem /29), MAC-Adresse, IPv6-Aufbau, OSI-Modell
- Vorgehensmodelle: IPERKA und ITIL
- Server-Grundlagen: Definition, Serverraum (Kühlung, USV, Zutrittskontrolle, getrennte Stromkreise), Baseline, Dauerbetrieb, Bauformen (Tower, Rack, Blade), Höheneinheiten
- Redundanz und RAID: Level 0, 1, 5, 10, JBOD, Hardware- gegen Software-RAID, Hot Spare, nutzbare Kapazität pro Level
- Dateisystem NTFS
- Fernwartung: iLO, iDRAC, iRMC, SSH, RDP einrichten (Windows Server und Ubuntu)
- Netzdienste: DNS, DHCP (Scope, Reservation, Scope-Optionen, ipconfig /release und /renew), Active Directory, Domäne, OU
- Monitoring: Durchschnittswerte deuten (z.B. CPU oder RAM um 80 Prozent, was prüfen und was tun), Windows-Bordmittel (Leistungsüberwachung, Ressourcenmonitor, Ereignisanzeige), SNMP
- Berechtigungen: Authentisierung, Authentifizierung, Autorisierung. Freigabe- gegen NTFS-Rechte, Vererbung, Vollzugriff gegen Ändern, Verhalten beim Kopieren und Verschieben
- Updates: Hotfix, Upgrade, Fehlerstufen
- Backup: Wert von Daten, Aufbewahrungsfristen, Voll, differentiell, inkrementell, Medien (Cloud, Disk, Tape, NAS)
- Hardware: SSD gegen HDD, SATA gegen SAS, RAM-Erweiterung (Typ, Slot, Kompatibilität, Erdung)

Daraus leite ich ab, dass Subnetting, RAID, DHCP-Scope, NTFS-Rechte und Backup-Arten gute Kandidaten für Prüfungsfragen sind. Bei der Learnfactory kann es anders sein.

## 3. Was ich vor dem Kurs klären würde

1. Sekretariat fragen: sekretariat@ict-learnfactory.ch, +41 31 521 00 90. Fragen: Welche Plattform (Windows oder Linux), welche Monitoring-Tools, Zugang zur Lernplattform schon vor dem Kurs, eigenes Notebook nötig, Zeiten, Art der Leistungsbeurteilung?
2. Kursprogramm im Kursverwaltungstool OdAOrg ansehen (laut ict-learnfactory.ch ab Juli sichtbar).
3. Modulbeschreibung selbst im Browser öffnen: https://www.modulbaukasten.ch, Modul 188 suchen, PDF herunterladen.
4. Eigene gibb-Unterlagen zu 117 und 123 durchgehen.

## 4. Vorbereitung nach Themen (aus meinem Fachwissen, entlang der Handlungsziele)

Die Gliederung folgt den offiziellen Handlungszielen. Die konkreten Beispiele und Tools sind meine Ergänzung.

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

### Nützliche Anleitungen für die Praxis (Monitoring)

- SNMP-Dienst auf Windows Server installieren: `Install-WindowsFeature SNMP-Service,SNMP-WMI-Provider -IncludeManagementTools`. Danach in den Diensteigenschaften Community und erlaubte Hosts setzen (mindestens die IP des Monitoring-Servers zulassen). Anleitung: https://theitbros.com/snmp-service-on-windows/ und https://woshub.com/install-configure-snmp-service-windows/
- PRTG: SNMP auf Windows einrichten: https://helpdesk.paessler.com/en/support/solutions/articles/76000042365-how-do-i-install-the-snmp-service-on-windows-systems-
- PRTG: WMI-Dienst-Sensor: https://www.paessler.com/manuals/prtg/wmi_service_sensor (WMI belastet stärker, Paessler empfiehlt höchstens etwa 200 solcher Sensoren pro Probe)
- Hinweis: SNMP v1 und v2c übertragen die Community im Klartext. Für die Handlungsziel-Kenntnis 4.1 (Sicherheitsmechanismen) gehört SNMPv3 mit Authentifizierung und Verschlüsselung dazu.

## 5. Konkrete Übungen für die zwei Wochen

- Zwei VMs aufsetzen (VirtualBox oder Hyper-V), eine als Server, eine als Client
- Auf dem Server DNS, DHCP und einen Fileserver einrichten, Client in die Domäne oder ins Netz aufnehmen
- PRTG (Gratisversion bis 100 Sensoren) oder Zabbix installieren und drei Sensoren einrichten (Ping, Disk, Dienst)
- Einen Dienst absichtlich stoppen und prüfen, ob der Alarm kommt
- Ein PowerShell- oder Bash-Skript schreiben, das Dienststatus und freien Speicher prüft
- Backup erstellen, Datei löschen, wiederherstellen

## 6. Freie Unterlagen zu Nachbarmodulen

Zu den Nachbarmodulen 123, 117 und 143 gibt es Kompetenzmatrizen mit Lernzielen: https://kompetenzmatrix.ch/cluster-platform/m123/ (Pfad für M117, M143 analog).

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
