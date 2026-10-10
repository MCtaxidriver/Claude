# ÜK Modul 188: Services betreiben, warten und überwachen

Stand: 10.10.2026. Für Informatiker/in EFZ Plattformentwicklung (BiVo 2021), ICT-Learnfactory.

Lernhilfen mit Cheatsheet, 74 Lernkarten und 14-Tage-Plan: siehe `UEK-188-Lernhilfen.md`. Probeprüfung mit Praxisszenario, 20 schriftlichen Fragen und Musterlösung: siehe `UEK-188-Probepruefung.md`. Einseitiger Spickzettel: `UEK-188-Spickzettel.md`. Lernkarten als CSV zum Import in Anki oder Quizlet (Tab-getrennt: Frage, Antwort, Kategorie): `UEK-188-Lernkarten.csv`.

<!-- toc -->
**Inhalt**

- 1. Gesichert (aus Quellen)
- 2. Offizielle Modulidentifikation (Modulbaukasten, Version 1.0, 26.03.2021)
- 2a. Leistungsbeurteilung (LBV 188-1, Modulbaukasten, publiziert 10.06.2022)
- 2b. Noch offen
- 2c. Infos der ICT-Learnfactory (https://www.ict-learnfactory.ch/informatiker)
- 2d. Was andere Lernende im üK 188 behandelt haben
- 3. Was ich vor dem Kurs klären würde
- 4. Vorbereitung nach Themen (aus meinem Fachwissen, entlang der Handlungsziele)
- 4b. Selbsttest zum Vorwissen (aus den Kompetenzmatrizen ZH, Stufe Beginner)
- 4c. Vorwissen offiziell: Module 123 und 117 (Modulbaukasten)
- 4d. Zusätzliche Lernquellen (nicht offiziell, bitte kritisch lesen)
- 5. Konkrete Übungen für die zwei Wochen
- 6. Freie Unterlagen zu Nachbarmodulen
- 7. Quellen
- 8. Was ich geprüft habe und was nichts brachte (Stand 10.10.2026, 21:25)

<!-- /toc -->

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

**Verwandt: f4 Serversysteme und -dienste warten und verwalten** (im Bildungsplan nicht Modul 188 zugeordnet, passt aber zu «warten»)
- f4.1 Wartungsaufgaben und Wartungszyklen definieren (K4). Der Bildungsplan nennt täglich, wöchentlich, monatlich, bei bestimmten Ereignissen und gemäss SLA.
- f4.2 Wartungsaufgaben zuverlässig nach Vorgabe durchführen (K3). f4.3 Ausgeführte Wartung nachvollziehbar dokumentieren (K3).
- f4.4 Updates testen und in die Produktivumgebung einspielen (K3).
- f4.5 Funktionalität, Performance und Sicherheit regelmässig und systematisch testen (K3).
- f4.6 Eingriffe und Anpassungen nachvollziehbar dokumentieren und die Schweigepflicht einhalten (K3).
Die Kompetenz fordert ausdrücklich, dass Eingriffe jederzeit nachvollziehbar sind und mit Daten vertrauensvoll und sorgfältig umgegangen wird.

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

## 2a. Leistungsbeurteilung (LBV 188-1, Modulbaukasten, publiziert 10.06.2022)

Quelle: https://www.modulbaukasten.ch/module/188/1/de-DE?title=Services-betreiben,-warten-und-%C3%BCberwachen (Reiter LBV, beide Elemente aufgeklappt). Das sind die Vorgaben des Verbands. Die genauen Aufgaben legt die Learnfactory fest.

**Gesamt:** Die Beurteilung deckt alle Handlungskompetenzen ab. Du baust im Modul die Systemumgebung eines kleinen KMU mit mehreren Services auf, die mit Aufträgen erweitert wird. Zwei Elemente, Richtzeit total 3 Stunden. (Die Seite trägt im Titel fälschlich «Dienste in der Public Cloud», das ist Modul 109. Der Inhalt passt aber zu 188.)

### Element 1: Praxis, 60 Prozent, 2 Stunden
- Form: praktisch am Objekt, Einzelarbeit.
- Aufgabe: Mit einer **vorgegebenen Betriebsdokumentation** wird eine **physische Systemumgebung** analysiert. Mit gezielten Aufgaben passt du die Umgebung an und führst die Betriebsdokumentation nach. Hilfsmittel ist die Betriebsdokumentation zum Prüfungssystem.
- Praxisbezug: Eine bestehende Kundenumgebung wird nach Kundenauftrag analysiert, konfiguriert und in der Dokumentation ergänzt.

| Kriterium | Gewicht | Handlungsziel |
| --- | --- | --- |
| Praktische Konfigurationsanpassungen zu **AD, DNS und DHCP** gemäss Vorgabe | 15 bis 25 % | 1 |
| System- und Ressourcenüberwachung mit **Bordmitteln oder zentralem Monitoring** ausführen und ergänzen | 15 bis 25 % | 2 |
| System auf Updates prüfen und gegebenenfalls einspielen | 5 bis 10 % | 3 |
| **Zugriffs- und Berechtigungskonzept** erfassen, ergänzen, dokumentieren | 20 bis 40 % | 5 |
| Anpassungen an **Umsystemen** erfassen und umsetzen | 15 bis 25 % | 6 |

### Element 2: Schriftlich, 40 Prozent, 1 Stunde
- Form: schriftlich, Einzelarbeit, praxisorientierte Fragen. Hilfsmittel nach Vorgabe des üK-Anbieters (zum Beispiel die Betriebsdokumentation zum Prüfungssystem).
- Praxisbezug laut LBV: AD, DNS und DHCP nach Kundenauftrag konfigurieren. Das Zugriffs- und Berechtigungskonzept umsetzen, ergänzen, dokumentieren und **begründen**. Klares konzeptionelles Vorgehen bei Updates und **Workarounds**. Services überwachen und **automatische Reports** generieren. **Monitoring interpretieren** und gegebenenfalls eine **Map** deuten. Benötigte Änderungen an Umsystemen an Dritte weiterleiten.

| Kriterium | Gewicht | Handlungsziel |
| --- | --- | --- |
| Praxisorientierte Fragen zur Konfiguration von AD, DNS, DHCP | 15 bis 25 % | 1 |
| Konzeptionelle Fragen zu Updates und Workarounds, Möglichkeiten bei Problemfällen aufzeigen | 5 bis 20 % | 3 |
| Fragen zum Umgang mit System- und Ressourcenüberwachung | 15 bis 25 % | 4 |
| Fragen zu Implementierung und **Schwellwerten** beim Monitoring | 10 bis 20 % | 4 |
| Zugriffs- und Berechtigungskonzept **ergänzen und begründen** | 20 bis 30 % | 5 |
| Fragen zur Analyse von Umsystemen | 5 bis 10 % | (6) |

### Hilfsmittel (Hinweis)
Für 188 gilt im schriftlichen Teil «nach Vorgabe des üK-Anbieters». Beim Folgemodul 190 erlaubt dieselbe Stelle laut LBV «keine oder 1 Seite A4 handschriftliche Notizen» (https://www.modulbaukasten.ch/module/190/1/de-DE, Reiter LBV). Frag beim Sekretariat, ob bei 188 eine handschriftliche A4-Seite erlaubt ist. Der Spickzettel `UEK-188-Spickzettel.md` eignet sich dann als Vorlage zum Abschreiben.

### Was daraus folgt
- **Berechtigungen sind der grösste Brocken** (bis 40 Prozent im Praxisteil, bis 30 Prozent im schriftlichen Teil). Übe AGDLP, Freigabe- und NTFS-Rechte und vor allem das Begründen.
- **AD, DNS und DHCP** sind die geprüften Services, also sehr wahrscheinlich ein Windows Server. Print, File und Linux-Dienste werden in der LBV nicht genannt.
- **Monitoring:** Bordmittel oder ein zentrales Tool, dazu Reports und Maps lesen und deuten. PRTG hat beides, ob die Learnfactory PRTG nutzt, ist nicht bestätigt.
- **Betriebsdokumentation:** Du bekommst sie vorgegeben, analysierst damit das System und ergänzt sie am Schluss. Übe das Lesen und Nachführen.
- **Updates:** Im Praxisteil nur prüfen und einspielen (kleines Gewicht), im schriftlichen Teil geht es um Konzept und Workarounds.

## 2b. Noch offen

- Die Kompetenzmatrizen ZH (https://kompetenzmatrix.ch/cluster-platform/m188/) führen M188 als «ÜK-Modul, keine Matrix vorhanden». Dort gibt es keine Lernziele dazu.
- Welches Monitoring-Tool die ICT-Learnfactory einsetzt (PRTG, Zabbix, Nagios, nur Bordmittel). Die LBV nennt Reports und Maps, was zu einem Tool wie PRTG passt, bestätigt ist das nicht.
- Windows Server ist wahrscheinlich: Die LBV prüft AD, DNS und DHCP, das Modulobjekt nennt File-, Print-, DHCP-, DNS- und Verzeichnis-Services. Eine Bestätigung der Learnfactory fehlt.

## 2c. Infos der ICT-Learnfactory (https://www.ict-learnfactory.ch/informatiker)

- Ort: ICT-LearnFactory AG, Bernapark 28, Eingang West, 2. Stock, 3066 Stettlen (Deisswil)
- Aufbau: 7 üK-Module zu je 5 Kurstagen. Modul 188 ist im 2. Bildungsjahr (zusammen mit 190 und 109)
- Kosten pro Tag: CHF 160 (Mitglieder) bzw. CHF 240 (Nicht-Mitglieder)
- Lernkonzept: begleitetes, selbstorganisiertes Lernen (BGSOL), handlungskompetenzorientiert. Du arbeitest also viel selbständig mit Aufträgen und nicht nur im Frontalunterricht.
- Tools, Lernplattform und Unterlagen stehen auf der Website nicht. Das Kursprogramm ist im Kursverwaltungstool OdAOrg sichtbar.
- Jeder üK schliesst mit einer Leistungsbeurteilung ab (Prüfung und/oder Projektbewertung nach Vorgaben des Modulbaukastens), laut üK-Broschüre Bern.

- Laut ICT-Berufsbildung Bern (https://www.ict-berufsbildung-bern.ch/grundbildung/informatiker-in-efz/ueberbetriebliche-kurse) erhalten Lernende und Lehrbetriebe zu Beginn jedes Lehrjahres das Kursprogramm von der ICT-LearnFactory. Schau in deinen Mails nach oder frag deinen Berufsbildner, vermutlich hast du es schon. Dort stehen die Daten und eventuell Hinweise zur Vorbereitung. Insgesamt sind es 7 Kursmodule zu je 5 Tagen.
- Pädagogisches Konzept (PDF, 2 Seiten): https://www.ict-learnfactory.ch/_files/ugd/c05327_9d2feafdc11d41f6ba8e7d510e7bd84e.pdf
  - Lerncoaches begleiten, du arbeitest selbstorganisiert und problembasiert an realistischen Aufgaben
  - Lerninhalte liegen für Lernende und Lehrbetriebe transparent auf einer Plattform. Frag das Sekretariat nach dem Zugang, vielleicht kannst du schon vorher hineinschauen.
  - Aufgaben enthalten einen Teil mit freier Wahl des Lösungswegs
  - Du führst ein Lernjournal, in dem du das Gelernte festhältst. Selbstüberprüfungen sind nicht notenwirksam.
  - Lehrbetriebe können firmeneigene Aufgaben einbringen. Frag deinen Berufsbildner, ob er das für den 188er plant.
- Ablauf laut Berichten über die Learnfactory (Kanton Bern: https://www.bkd.be.ch/de/start/news/zeitungsartikel-einsteiger/ict-learn-factory.html):
  - Du bekommst an den Kurstagen eine Reihe von Aufträgen und wählst Reihenfolge und Zeitpunkt selbst.
  - Die Module sind schweizweit standardisiert (Modulbaukasten). Lerncoaches kommen oft direkt aus laufenden Betriebsprojekten.
  - Räume mit Input-Bereichen und Lernnischen, du kannst dich frei zusammensetzen.
  - Insgesamt meist 35 Kurstage in der Lehre, jeweils 5 bis 6 Tage am Stück.
- Regeln laut Suchergebnis zu einem Berner-Zeitung-Artikel (den Artikel selbst konnte ich nicht öffnen, bitte beim Sekretariat bestätigen lassen): Wer mehr als einen Tag des fünftägigen Kurses verpasst, muss ihn wiederholen. Die Aufgaben, die du im Kurs löst, zählen für den Abschluss des Moduls. Passt zur Modul-LBV (siehe 2a).
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
3. Kursprogramm des Lehrjahres in deinen Mails oder beim Berufsbildner suchen (kommt laut ICT-Berufsbildung Bern zu Beginn jedes Lehrjahres).
4. Modulbeschreibung selbst im Browser öffnen: https://www.modulbaukasten.ch, Modul 188 suchen, PDF herunterladen.
5. Eigene gibb-Unterlagen zu 117 und 123 durchgehen.

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

## 4b. Selbsttest zum Vorwissen (aus den Kompetenzmatrizen ZH, Stufe Beginner)

Die üK-Broschüre Bern verlangt, 117 und 123 zu repetieren. Die Kompetenzmatrizen (https://kompetenzmatrix.ch/cluster-platform/m123/ und .../m117/) formulieren das als Ich-Aussagen. Kannst du das erklären oder tun?

M123 Serverdienste:
- Wesentliche Merkmale von Serverhardware aufzählen
- Das Server-Client-Prinzip, die per DHCP zugewiesenen Angaben und die DNS-Namensauflösung erklären
- Netzwerkfreigaben erklären, Vor- und Nachteile eines Druckservers gegenüber Direktdruck nennen
- Serverrollen installieren und prüfen, ob der Dienst korrekt startet
- Auf dem Client die Funktion einrichten
- Zugriffsschutz im Netzwerk und im Dateisystem erklären
- Systeme und Software warten, laufende Dienste erkennen
- Mit einfachen Tests einen Dienst prüfen und ihn dokumentieren

M117 Netz- und Infrastruktur:
- Anforderungen eines Netzwerks beschreiben und dokumentieren, Basisdienste (DHCP, DNS, Fileserver) erläutern
- Netzwerkschemas lesen, IP-Adressierung erläutern
- Switch, Router, Access Point im Schema deuten, OSI-Modell erklären, Protokolle den Schichten zuordnen
- Netzwerk-Ressourcen und Zugriffsberechtigungen erklären, grundlegende Fehlersuche
- Nutzen von Tests und Dokumentation aufzeigen

Wenn du bei einem Punkt ins Stocken kommst, ist das ein guter Repetitionskandidat vor dem Kurs.

### Microsoft-Doku und Befehle (geprüft)

- WSUS (Updates zentral verwalten, mit PowerShell-Cmdlets steuerbar, SHA-256-Hash). Microsoft nennt WSUS veraltet, es wird aber weiter unterstützt: https://learn.microsoft.com/de-de/windows-server/administration/windows-server-update-services/get-started/windows-server-update-services-wsus
- Performance-Werte per PowerShell mit `Get-Counter`: https://learn.microsoft.com/de-de/powershell/module/microsoft.powershell.diagnostics/get-counter
  - `Get-Counter -Counter "\Processor(_Total)\% Processor Time" -SampleInterval 2 -MaxSamples 3`
  - `Get-Counter -Counter "\LogicalDisk(_Total)\% Free Space"`
  - `Get-Counter -ListSet *` listet alle Zählersätze. Die Namen sind lokalisiert (auf deutschem System deutsch).
- Wichtige Zähler und Richtwerte (ChannelPartner): https://www.channelpartner.de/article/3894353/know-how-windows-performance-counter.html
  - CPU: Processor % Processor Time. Warteschlange (System Processor Queue Length) dauerhaft über 2 deutet auf Überlast hin.
  - RAM: viele Page Reads/Writes pro Sekunde heissen meist RAM-Mangel
  - Disk: PhysicalDisk Current Disk Queue Length dauerhaft hoch heisst Platte ausgelastet
  - Netz: Network Interface Bytes Total/sec gegen die Bandbreite der Netzwerkkarte halten
  - Konkrete Schwellwerte hängen von der Serverrolle ab, deshalb zuerst eine Baseline messen.
- Lab-Beispiel zum Nachbauen (Mitschrift einer Schule in Österreich, Windows Server 2025 in einer VM mit zwei Netzwerkadaptern, AD, Benutzer, DHCP, Domänen-Beitritt, Freigabe, Gruppenrichtlinien, Software per GPO): https://lernemit.duckdns.org/it-doku/pdf/WINDOWS_SERVER_WINCLIENT_ACTIVE-DIRECTORY.pdf (30 Seiten, nur als Übungsidee, nicht Kursmaterial)

### Abgleich mit der Gemini-Zusammenfassung

Passt zu den offiziellen Quellen:
- Vier Kernbereiche Analysieren, Betreiben, Updates und Patches, Monitoring
- Berechtigungen nach Firmenkonzept, manuelle Patches und Softwareverteilung
- Bezug zu 117 und 123 und die Folgemodule 190 und 109

Nicht belegt, bitte nicht als Fakt nehmen:
- Webserver (Apache, Nginx, IIS) und Datenbanken (MariaDB, PostgreSQL) als Kursinhalt. Das Modul nennt File-, Print-, DHCP-, DNS- und Verzeichnis-Services.
- Zabbix, Nagios und Prometheus als Kurs-Tools. Das Modul sagt nur «bestehende Monitoring-Tools». Die Learnfactory nennt kein Tool.
- Linux oder Windows. Das Modul legt sich nicht fest. Die LBV prüft aber AD, DNS und DHCP, das spricht klar für Windows Server.
- Die Quellenangaben bei Gemini (ict-sh.ch, bit.admin.ch, wiss.ch, YouTube) enthalten zu Modul 188 nichts, was ich dort finden konnte.

### Fachgrundlagen zu den Handlungszielen (aus Quellen, mit Vorbehalt)

**SNMP-Versionen (Handlungsziel 4, Kenntnis 4.1 «Sicherheitsmechanismen»)**
- v1 und v2c: Authentifizierung nur über den Community-String, Übertragung im Klartext. Mit Wireshark mitlesbar. v2c bringt GetBulk und 64-Bit-Zähler, bleibt aber unsicher.
- v3: Benutzer, Authentifizierung und optional Verschlüsselung. Stufen noAuthNoPriv, authNoPriv, authPriv. Authentifizierung per HMAC mit MD5 oder SHA, Verschlüsselung mit DES oder AES. MD5 und DES gelten als veraltet.
- Quellen: https://blog.paessler.com/snmp-v2-vs-v3-understanding-snmps-evolution-in-network-management und https://www.ninjaone.com/de/blog/unterschiede-zwischen-snmp-versionen/

**Berechtigungskonzept AGDLP (Handlungsziel 5)**
- Accounts kommen in globale Gruppen (Rollen, z.B. Abteilung), diese in domänenlokale Gruppen (Zugriffsgruppen), erst dort werden die Rechte (Permissions) auf Ressourcen gesetzt.
- Vorteil: In den ACLs stehen nur Gruppen, das ist übersichtlicher und bei Wechseln einfacher. Bei mehreren Domänen kommt AGUDLP mit universellen Gruppen dazu.
- Schwäche: Muss konsequent gepflegt werden, sonst driftet es durch Ausnahmen auf Benutzerebene.
- Quellen: https://www.tenfold-security.com/?p=9261 und https://www.manageengine.com/products/ad-manager/blog/what-is-agdlp.html

**Freigabe- gegen NTFS-Rechte (Handlungsziel 5)**
- Freigaberechte gelten nur beim Zugriff über die Freigabe und kennen nur Lesen, Ändern, Vollzugriff. NTFS-Rechte gelten immer und sind feiner.
- Beim Zugriff über das Netz gilt die restriktivere der beiden Ebenen. «Verweigern» hat Vorrang.
- Übliche Praxis: Freigabe grosszügig, Feinsteuerung per NTFS.
- Quellen (keine Microsoft-Seiten, bitte mit Kursunterlagen abgleichen): https://netwrix.com/de/resources/blog/ntfs-vs-share-permissions/ und https://www.varonis.com/de/blog/der-unterschied-zwischen-freigaben-und-ntfs-berechtigungen

**Patch-Prozess (Handlungsziel 3, Kenntnisse 3.1 bis 3.4)**
- Ablauf: Inventar, Priorisierung nach Risiko, Test in Testumgebung, Backup und Rollback-Plan, Pilotgruppe, gestaffelter Rollout im Wartungsfenster, Kontrolle, Dokumentation (Betriebsdoku nachführen).
- Vertrauenswürdige Quellen: Hersteller selbst, Hashwert und Signatur prüfen.
- Gefahren: Inkompatibilität, Neustart nötig, Dienst fällt aus. Deshalb zuerst auf weniger kritischen Systemen testen.
- Quellen: https://www.n-able.com/de/blog/effective-patch-management-a-guide-for-it-security-and-system-stability und https://www.techtarget.com/cybersecurity/tip/Key-software-patch-testing-best-practices
- Offizielle Vorgabe zum Abgleichen: BSI IT-Grundschutz, Baustein OPS.1.1.3 Patch- und Änderungsmanagement (ich habe ihn nicht selbst gelesen).

**Weitere Dateien**
- Modulplan Plattformentwicklung (Kanton SH): https://ict-sh.ch/wp-content/uploads/2023/06/Modulplan_INF-Plattform_BiVo2021_Kt_SH_v1.0.pdf
- Lernzielkatalog als Excel (Leistungsziele des Bildungsplans, ohne zusätzliche Details zu 188): https://ict-sh.ch/wp-content/uploads/2023/06/Lernzielkatalog_Informatiker_BiVo21-Plattformentwickler.xlsm

### Befehlsübersicht für die Services (PowerShell, Windows Server)

Aus der Microsoft-Doku geprüft (Links unten):
- DHCP-Scope anlegen: `Add-DhcpServerv4Scope -Name "Lab" -StartRange 10.10.10.1 -EndRange 10.10.10.254 -SubnetMask 255.255.255.0`. Standard-Lease 8 Tage, Scope ist standardmässig aktiv. Verwandt: Get-/Set-/Remove-DhcpServerv4Scope, Get-DhcpServerv4ScopeStatistics (Auslastung des Scopes).
- DNS A-Record anlegen: `Add-DnsServerResourceRecordA -Name "host23" -ZoneName "contoso.com" -IPv4Address "172.18.99.23" -TimeToLive 01:00:00`. Mit `-CreatePtr` entsteht der passende Reverse-Eintrag. Verwandt: Get-/Set-/Remove-DnsServerResourceRecord.
- Ereignisprotokolle: `Get-WinEvent -ListLog *` zeigt alle Logs mit Grösse und Anzahl Einträge. Filtern geht mit `-FilterHashtable` (z.B. nach LogName, Level, Zeitraum), `-MaxEvents` begrenzt die Anzahl.
- WMI/CIM abfragen: `Get-CimInstance -ClassName Win32_Process` (Standardnamespace root/CIMV2). Remote mit `-ComputerName Server01` oder einer CIM-Sitzung (`New-CimSession`). CIM über WSMan ist der Nachfolger des alten WMI-Zugriffs per DCOM.
- Performance-Zähler: siehe Get-Counter weiter oben.

Aus meinem Wissen, nicht in den Quellen geprüft:
- Dienste: `Get-Service`, `Start-Service`, `Stop-Service`, `Restart-Service`, `Set-Service -StartupType Automatic`
- Updates: `Get-HotFix` (installierte Patches), `wuauclt` ist veraltet, besser Windows Update über die Einstellungen oder WSUS
- Berechtigungen: `Get-Acl`, `icacls <Pfad>`, `New-SmbShare`, `Get-SmbShareAccess`
- Netzwerk: `ipconfig /all`, `ipconfig /release` und `/renew`, `nslookup`, `Test-NetConnection -Port 161` (SNMP-Port, UDP), `Resolve-DnsName`

Quellen:
- https://learn.microsoft.com/de-de/powershell/module/dhcpserver/add-dhcpserverv4scope
- https://learn.microsoft.com/de-de/powershell/module/dnsserver/add-dnsserverresourcerecorda
- https://learn.microsoft.com/de-de/powershell/module/microsoft.powershell.diagnostics/get-winevent
- https://learn.microsoft.com/de-de/powershell/module/cimcmdlets/get-ciminstance

### Betriebsdokumentation (Handlungsziel 1 und 3)

Die Vorlage für Gemeinden von Datenschutz.ch ist eine Schweizer «Maximalvariante» und zeigt gut, was in eine Betriebsdokumentation gehören kann: https://docs.datenschutz.ch/u/d/informationssicherheit/gemeinden/vorlage_betriebsdokumentation.docx
Gliederung (gekürzt):
- Allgemeines: Zweck, Zielgruppen, Verweise auf Richtlinien, Änderungskontrolle
- Organigramm: Verantwortlichkeiten (Beschaffung, Betrieb), Kontaktpersonen
- Lokalitäten: Standort, Zutrittsregelung, Schutzmassnahmen (Strom, Feuer, Wasser, Hitze, Einbruch, Blitz), Alarmierung (Überwachungssystem, Empfänger)
- Systemübersicht: Architekturdiagramm, Mengengerüst, Standortpläne, Hard- und Softwareverzeichnis, Lizenzen
- Netzwerk: Netzpläne, Zonen, Protokolle, Verschlüsselung, Authentisierung, WLAN (Access Points, SSID)
- Server: Einsatzzweck, Betriebssystem, Konfiguration, spezielle Einstellungen, Speicher und Partitionierung, installierte Anwendungen, Referenzinstallation, externe Schnittstellen, Verzeichnisstruktur, Log-Dateien
- Passwörter gehören laut Vorlage ins Notfallkonzept und nicht in die Betriebsdokumentation.
Für den Kurs heisst das: Nach jeder Änderung (Patch, neuer Service, neue Berechtigung) die Doku nachführen, das ist Teil von Handlungsziel 3.

### PRTG: Schwellwerte und Alarme einrichten (Handlungsziel 4, Kenntnis 4.4)

Quelle: PRTG-Handbuch, https://www.paessler.com/manuals/prtg/setting_up_a_notification_example
1. Grenzwerte am Kanal des Sensors setzen. Beispiel im Handbuch: obere Warngrenze 60, obere Fehlergrenze 80. Der Sensor wechselt dann in Warning und bei mehr als 80 in Down. Mehrere gleichartige Sensoren lassen sich mit Multi-Edit zusammen einstellen.
2. Delta-Kanäle (Fehlerzähler) liefern Werte pro Sekunde. Ein Fehler pro 60-Sekunden-Intervall entspricht etwa 0,016 pro Sekunde, deshalb liegt die Warnschwelle für «ein Fehler» im Beispiel bei 0,1.
3. Benachrichtigungs-Trigger anlegen (State Trigger auf Warning oder Down) und eine Benachrichtigungsvorlage wählen. Trigger lassen sich auch auf Gruppen im Gerätebaum setzen.
4. Benachrichtigung mit «Send test notification» testen. Das deckt Handlungsziel f5.4 (Überwachung regelmässig testen).
Menünamen können je nach PRTG-Version abweichen.

### Umsysteme anpassen (Handlungsziel 6)

Beispiele aus der Microsoft-Doku zu `New-NetFirewallRule` (https://learn.microsoft.com/de-de/powershell/module/netsecurity/new-netfirewallrule):
- Eingehende Regel nur aus dem lokalen Subnetz für ein Programm: `New-NetFirewallRule -DisplayName "Allow Messenger" -Direction Inbound -Program "C:\Pfad\programm.exe" -RemoteAddress LocalSubnet -Action Allow`
- Port sperren: `New-NetFirewallRule -DisplayName "Block Outbound Port 80" -Direction Outbound -LocalPort 80 -Protocol TCP -Action Block`
- Nur authentifizierte Verbindungen zulassen: Parameter `-Authentication Required`
Beim Thema SNMP heisst das konkret: Auf der Firewall des Servers UDP 161 nur für die IP des Monitoring-Servers freigeben (RemoteAddress). Dass diese Anforderung für Dritte nachvollziehbar beschrieben sein muss, ist Kenntnis 6.2. Ein Beispiel für einen Änderungsauftrag: «Bitte auf Firewall FW01 eine Regel anlegen: Quelle 10.0.0.5 (Monitoring), Ziel 10.0.0.10 (SRV01), UDP 161, Zweck SNMP-Abfrage, befristet bis ..., Test: snmpwalk von 10.0.0.5».

### Wichtige Event-IDs (System- und Sicherheitslog)

Quellen sind Blogs und SIEM-Listen, keine Microsoft-Seiten. Die Bedeutung kannst du in der Ereignisanzeige selbst prüfen.
- 7036: Dienst wurde gestartet oder gestoppt
- 7034: Dienst unerwartet beendet
- 7040: Starttyp eines Dienstes geändert
- 4625: Anmeldung fehlgeschlagen (viele in kurzer Zeit können auf Brute Force hindeuten)
- 4624: Anmeldung erfolgreich
- 6005: Ereignisprotokolldienst gestartet (entspricht Systemstart)
- 6006: Ereignisprotokolldienst beendet (sauberes Herunterfahren)
- 6008: Vorheriges Herunterfahren war unerwartet
- 41: Kernel-Power, System wurde ohne sauberes Herunterfahren neu gestartet. Offiziell belegt: https://learn.microsoft.com/de-de/troubleshoot/windows-client/performance/event-id-41-restart (die Ursache lässt sich aus dem Ereignis allein oft nicht ableiten, ein Bugcheck-Code deutet auf einen Stoppfehler, Code 0 kann auf ein Stromversorgungsproblem hinweisen)
Quellen: https://graylog.org/?p=18855 und https://manageengine.com/products/active-directory-audit/kb/system-events/event-id-6005.html

### Backup (nach NCSC-Empfehlungen, Schweiz)

- Regelmässig sichern. Externe Festplatte nach dem Sichern vom Gerät trennen, damit Schadsoftware das Backup nicht mitverschlüsselt.
- Cloud ist möglich, aber prüfen, welche Daten dorthin dürfen.
- Sichern ist nur die halbe Miete: Die Wiederherstellung regelmässig testen.
- Ein Schweizer Gewerbeverband empfiehlt mindestens wöchentlich für kleine und mittlere Betriebe.
- Die 3-2-1-Regel (3 Kopien, 2 Medien, 1 extern) habe ich in den Quellen nicht gefunden, sie ist aus meinem Wissen.
Quelle: https://www.netzwoche.ch/news/2021-05-03/tipps-vom-ncsc-woran-man-bei-einem-back-up-immer-denken-sollte

## 4c. Vorwissen offiziell: Module 123 und 117 (Modulbaukasten)

PDFs: https://www.modulbaukasten.ch/Module/123_2_Serverdienste%20in%20Betrieb%20nehmen.pdf und https://www.modulbaukasten.ch/Module/117_3_Informatik-%20und%20Netzinfrastruktur%20f%C3%BCr%20ein%20kleines%20Unternehmen%20realisieren.pdf

**M123 Serverdienste in Betrieb nehmen (Version 2.0).** Du installierst und konfigurierst Serverdienste (Verzeichnisdienst, DHCP, DNS, File, Print) für ein lokales Netz nach Vorgaben und prüfst deren Funktion. Objekt: ein Server in einem einfachen Netz mit einer Domäne und identischen Clients. Handlungsziele in Kurzform:
1. Konfiguration der Dienste nach betrieblichen Vorgaben definieren
2. Installieren und konfigurieren, nicht benötigte Dienste deaktivieren
3. Clients anpassen, damit die Benutzer die Dienste nutzen können
4. Grundlegenden Zugriffsschutz des Server-Betriebssystems umsetzen
5. Verfügbarkeit und Funktion prüfen und protokollieren
6. Dokumentation und Abnahme/Betriebsfreigabe
Wichtige Kenntnisse: Vorteile zentraler Benutzerverwaltung, Konfiguration von File-, DHCP-, DNS- und Print-Server, dynamische gegen statische IP-Zuweisung, dedizierte gegen nicht dedizierte und Cloud-Dienste, Inbetriebnahme-Ablauf (installieren, konfigurieren, starten, auf Client verfügbar machen, testen), Authentifizierung und Autorisierung, Passwortregeln, Log-Dateien lesen, Testen und Protokollieren, Abnahmeprotokoll, Struktur einer Betriebsdokumentation.

**M117 Informatik- und Netzinfrastruktur realisieren (Version 3.0).** Lokales Netzwerk ohne zentrale Benutzerverwaltung (bis 10 Anschlüsse, Internetanschluss). Handlungsziele: Anforderungen aufnehmen, Netzwerkschema und Verkabelungsplan erstellen, Installation planen, aufbauen und konfigurieren, Ressourcenverwaltung (Gruppen, Shares) definieren, testen und dokumentieren. Wichtige Kenntnisse: IP-Format, Subnetmaske, private Adressen, Gateway, DNS, Aufgaben von Switch, Access Point und Router, Internetzugang, Kabel- und Steckertypen, Rechte dokumentieren (Matrix Benutzergruppen gegen Shares), typische Netzwerkfehler, OSI- und TCP/IP-Modell mit Zuordnung von Protokollen und Komponenten.

Zusammenhang zu 188: In 123 richtest du die Dienste ein, in 188 betreibst, patchst und überwachst du sie in einer bestehenden Umgebung. Wenn du die Inbetriebnahme von DHCP, DNS und File-Server aus 123 nicht mehr sicher kannst, hilft die Repetition am meisten.

**Wie M123 geprüft wird (LBV 123-3, Modulbaukasten)** [Q: https://www.modulbaukasten.ch/module/123/2/de-DE, Reiter LBV]
- Dreiteilige Prüfung, jedes Element mit Wissensteil, Planungsteil und praktischer Umsetzung. Richtzeit total 5,5 Stunden, Hilfsmittel keine.
  1. DHCP (25 %): Wissensfragen, ein IP-Adresskonzept anhand eines Szenarios planen und im DHCP-Dienst umsetzen.
  2. DNS (25 %): Wissensfragen, ein Namenskonzept planen und im DNS-Dienst umsetzen.
  3. Verzeichnisdienst und File (50 %): Wissensfragen, ein Benutzer-, Freigabe- und Berechtigungskonzept planen, umsetzen und testen, inklusive Dokumentation und Betriebsfreigabe.
- Der praktische Teil hat in jedem Element das grösste Gewicht (40 bis 50 %).
- Das zeigt, wie du für 188 üben solltest: Konzept zuerst auf Papier (IP-Plan, Namen, Gruppen und Rechte), dann umsetzen, testen und dokumentieren.

## 4d. Zusätzliche Lernquellen (nicht offiziell, bitte kritisch lesen)

- Quizlet-Karten zu Modul 123 (54 Karten, Grundlagen, AD-Installation, DHCP-Reservationen): https://quizlet.com/5789099/it-modul-123-flash-cards/ (liess sich für mich nicht öffnen, 403)
- Schulseite zu Modul 123 mit Kapiteln zu DHCP, DNS, Dateiserver, Berechtigungen und Übungen mit vmMachine und Filius: https://edu.juergarnold.ch/modul123.html (für mich nicht erreichbar, Angabe aus der Suche)
- Blog zu Modul 123 mit den Handlungszielen: https://itbaer.ch/2016/11/17/modul-123-serverdienste-betrieb-nehmen/
- Aktuelle Anleitung DHCP und DNS auf Windows Server 2022/2025: https://s-edv.com/anleitungen/dhcp-dns-rolle-windows-server-konfigurieren
- Musterdokumentation eines Lernenden zu Fileserver, DHCP, DNS, Druckdienst (Windows Server 2012, alt): https://silo.tips/download/dokumentation-modul-123-serverdienste-in-betrieb-nehmen
- Zusammenfassung zum alten Modul M127 «Server betreiben» (2008, CC BY-NC-SA, Vorgänger mit ähnlichem Thema, aber veraltet. Dort geht es auch um RAID, USV, Dateisysteme, nslookup, DFS): https://emanuelduss.ch/downloads/Berufsschule/2_Lehrjahr/M127_Server_betreiben_Zusammenfassung.pdf
  - Das erklärt, warum Lerndokumentationen zum üK 188 oft Hardware und RAID enthalten. Im heutigen Modul 188 stehen diese Themen nicht in den Handlungszielen.
- Subnetting üben mit Lösungen: https://administrator.de/forum/subnetting-net-id-berechnen-1611811666.html und https://coddy.tech/learn/de/courses/ip_addressing_and_subnetting/recap__subnetting_drills

### Windows Server 2025: was ältere Anleitungen nicht mehr stimmt

Quelle: https://learn.microsoft.com/de-de/windows-server/get-started/removed-deprecated-features-windows-server (Microsoft Learn, Seite aktualisiert 2026). Falls ihr im Kurs Server 2025 nutzt, ist das relevant:
- WSUS: wird nicht mehr aktiv entwickelt, steht aber weiter zur Verfügung und wird unterstützt.
- WMIC (Kommandozeile): ab Server 2025 nur noch als Feature on Demand (`DISM /Add-Capability`), wird später entfernt. Ersatz ist PowerShell, also `Get-CimInstance`.
- VBScript: Feature on Demand, wird später entfernt. Ersatz PowerShell.
- Windows PowerShell 2.0: mit dem Update vom September 2025 nicht mehr enthalten.
- NTLMv1: entfernt. NTLMv2 und LANMAN: veraltet. Ersatz ist Kerberos über Negotiate.
- SMTP-Server (IIS-Funktion): in Server 2025 entfernt.
- Interne Windows-Datenbank (WID), die z.B. WSUS nutzt: soll später entfernt werden. Microsoft empfiehlt SQL Server.
- TLS 1.0 und 1.1 sind standardmässig deaktiviert.
- Dienste wie Computerbrowser und WINS sind veraltet. DNS statt WINS.
- Zu SNMP stand auf dieser Seite nichts. Ob das SNMP-Feature in deiner Server-Version installierbar ist, siehst du mit `Get-WindowsFeature *SNMP*`.
- Aus der Server-2016-Zeit: `wuauclt.exe /detectnow` wird nicht mehr unterstützt. Updatesuche stattdessen per PowerShell/COM oder in den Einstellungen auslösen.

### Leistungsüberwachung mit Alarm (Bordmittel, Handlungsziel 2 und 4)

Nur aus Drittquellen, bitte in der eigenen VM nachvollziehen:
- `perfmon` als Administrator starten, unter Datensammlungssätze, Benutzerdefiniert, Neu, Datensammlungssatz, «Manuell erstellen (erweitert)», Typ Leistungsindikator-Warnung.
- Indikator und Grenzwert («über» oder «unter») festlegen, Satz starten. Ausgelöste Warnungen erscheinen laut Microsoft im Protokoll Microsoft-Windows-Diagnostics-PLA/Operational (Details siehe `UEK-188-Lernhilfen.md`, Abschnitt 2k). Die Drittquelle nannte fälschlich das Anwendungsprotokoll.
- Schwellenwerte auf Basis einer Baseline setzen. Eine Quelle nennt 4 bis 6 Wochen Beobachtung, im Kurs reicht eine kurze Messung.
- Quellen: https://www.computerperformance.co.uk/?p=5868 und https://help.tableau.com/current/server/de-de/perf_collect_perfmon.htm

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

- Berner Zeitung und Kanton Bern zur Learnfactory: https://www.bkd.be.ch/de/start/news/zeitungsartikel-einsteiger/ict-learn-factory.html
- Bildungsplan Informatiker/in EFZ (BiVo 2021), SBFI: https://www.becc.admin.ch/becc/public/bvz/beruf/download/10475

- ICT-Berufsbildung Bern, üK-Broschüre: https://www.ict-berufsbildung-bern.ch/resources/uek-Broschuere.pdf
- Bildungsplan Modulübersicht (Ost): https://ict-berufsbildung-ost.ch/wp-content/uploads/2021/03/BiPla_Informatiker_20201119_Modulu%CC%88bersicht.pdf
- Modulübersicht: https://mbk.ict-berufsbildung.ch/assets/module/Moduluebersicht-DE-V3.pdf
- Schullehrplan GB Chur: https://www.gbchur.ch/wp-content/uploads/2024/09/Interner-Schullehrplan-Informatiker_in-EFZ-Plattformentwicklung.pdf
- Klubschule Pro, Parallelkurs: https://www.klubschule-pro.ch/kurs/it-services-betreiben-warten-und-uberwachen--D_101476_1016_313
- Modulbaukasten: https://www.modulbaukasten.ch
- ICT-Learnfactory: https://www.ict-learnfactory.ch

## 8. Was ich geprüft habe und was nichts brachte (Stand 10.10.2026, 21:25)

Damit du nichts doppelt suchst:
- Nicht öffentlich oder nur mit Login: Kursseiten und Moodle von ICT-BZ (Zentralschweiz), Kursunterlagen und Lernplattform der ICT-Learnfactory, das Kursprogramm 2026/27 (kommt zu Lehrjahresbeginn per Mail).
- Keine Treffer zu Modul 188: Kompetenzmatrizen ZH («ÜK-Modul, keine Matrix vorhanden»), tbz-it auf GitHub und GitLab (nur M122 und M169 gefunden), Quizlet (nur Modul 123), ict-sh.ch und ict-berufsbildung-ost.ch (nur Modulübersichten), Lernjournale und Portfolios (ausser der einen Lerndokumentation von 2023).
- Zeitweise nicht erreichbar oder blockiert (403/503, Weiterleitungsschleifen): Quizlet, Berner Zeitung, Tages-Anzeiger, edu.juergarnold.ch.
- Aus Sicherheitsgründen nicht genutzt: Ein Endpunkt des Modulbaukastens liefert öffentlich ein Zugriffstoken für das CRM des Verbands. Das habe ich weder verwendet noch gespeichert. Dem Verband zu melden wäre sinnvoll.
- Nicht belegt, weil keine Quelle: Welche Tools und welche Plattform (Windows oder Linux) die Learnfactory im 188er einsetzt. Das kann nur das Sekretariat beantworten.

