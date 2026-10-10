# ÜK 188: Probeprüfung (Eigenbau) mit Musterlösung

Das ist keine offizielle Prüfung. Ich habe sie aus den Handlungszielen und den Vorgaben der Leistungsbeurteilung (LBV 188-1) im Modulbaukasten abgeleitet: ein praktischer Teil (60 Prozent, Richtzeit 2 Stunden) und ein schriftlicher Teil (40 Prozent, Richtzeit 1 Stunde). Die LBV nennt AD, DNS und DHCP, Berechtigungen, Updates, Monitoring und Umsysteme als Prüfgegenstände, dazu Reports und Maps. Die genauen Aufgaben der Learnfactory kenne ich nicht.

Markierung: [Q] belegt in einer Quelle (siehe `UEK-188-Vorbereitung.md` und `UEK-188-Lernhilfen.md`), [W] aus allgemeinem Fachwissen.

<!-- toc -->
**Inhalt**

- Teil A: Praxis (2 Stunden)
- Teil B: Schriftlich (1 Stunde)
- Hilfreiche Zusatzinfos für die Praxis

<!-- /toc -->

---

## Teil A: Praxis (2 Stunden)

### Ausgangslage
Die Muster AG ist ein kleines Büro mit 12 Mitarbeitenden. Es gibt einen Server `SRV01` (Windows Server, Domäne `muster.local`, IP 192.168.10.10) und drei Test-Clients. Die Rollen AD DS, DNS, DHCP und Dateiserver laufen bereits. Du übernimmst den Betrieb. Ein Monitoring-Server `MON01` (192.168.10.20) mit PRTG ist vorhanden, `SRV01` ist aber noch nicht eingebunden.

Auszug aus der Betriebsdokumentation:

| Punkt | Wert |
| --- | --- |
| DHCP-Scope | 192.168.10.100 bis .200, Maske 255.255.255.0, Gateway 192.168.10.1, DNS 192.168.10.10 |
| Reservation | Drucker 192.168.10.50 |
| Freigabe | `\\SRV01\Buchhaltung` auf `D:\Daten\Buchhaltung` |
| Gruppen | `GG_Buchhaltung` (global), `DL_Buchhaltung_RW` (domänenlokal) |
| Berechtigungskonzept | Zugriff nur über domänenlokale Gruppen. Buchhaltung darf ändern, Geschäftsleitung darf lesen. |

### Aufgaben

**1. Analyse (10 Minuten, Handlungsziel 1)**
Prüfe mit der Dokumentation und dem System, ob der Ist-Zustand stimmt. Ermittle den DHCP-Scope, die DNS-Zone, die Freigabe und die Gruppen. Halte Abweichungen schriftlich fest.

**2. Dienste prüfen (10 Minuten, Handlungsziel 2)**
Stelle fest, welche der Dienste DNS, DHCP, Server und Druckwarteschlange laufen. Starte einen gestoppten Dienst und finde den passenden Eintrag in der Ereignisanzeige.

**3. Performance-Baseline (10 Minuten, Handlungsziel 2)**
Miss CPU, freien Speicherplatz und die Netzwerkauslastung von `SRV01` und notiere die Werte.

**4. Patches (20 Minuten, Handlungsziel 3)**
Ermittle, welche Updates installiert sind. Plane einen Patch-Durchlauf für den Server in 5 Schritten, führe vorher einen Snapshot oder ein Backup durch und halte das Vorgehen in der Betriebsdokumentation fest.

**5. Monitoring einbinden (30 Minuten, Handlungsziel 4)**
Installiere den SNMP-Dienst auf `SRV01`, erlaube nur `MON01`, binde `SRV01` in PRTG ein (Ping, CPU, freier Speicherplatz auf D:, Dienststatus DHCP). Lege begründete Grenzwerte fest, richte eine Benachrichtigung ein und teste sie, indem du einen Dienst stoppst. Erstelle einen automatischen Wochenreport (PDF) für die vier Sensoren und lege eine Map mit `SRV01` an, die den Status anzeigt.

**6. Berechtigungen (20 Minuten, Handlungsziel 5)**
Lege einen neuen Mitarbeiter `Sara Keller` an, nimm ihn in die Gruppe der Buchhaltung auf und prüfe, dass sie auf `\\SRV01\Buchhaltung` ändern darf, aber keinen Zugriff auf `\\SRV01\Geschaeftsleitung` hat. Dokumentiere die Rechte.

**7. Umsystem (10 Minuten, Handlungsziel 6)**
Die Firewall auf `SRV01` blockiert die SNMP-Abfrage. Schreibe einen Änderungsauftrag für den externen Netzwerkpartner, mit dem du eine Regel für `MON01` verlangst, und gib an, wie du die Änderung testest.

### Bewertungsraster (an die LBV angelehnt, 100 Punkte)

Die LBV gibt Spannen vor. Ich nehme die Mitte, damit es 100 ergibt.

| Aufgabe | Punkte | LBV-Kriterium und Spanne |
| --- | --- | --- |
| 1 Analyse und Konfiguration AD, DNS, DHCP | 20 | Konfigurationsanpassungen zu AD, DNS und DHCP (15 bis 25 %) |
| 2 und 3 Dienste und Baseline | 10 | Teil der System- und Ressourcenüberwachung |
| 5 Monitoring einbinden, Report | 20 | Überwachung mit Bordmitteln oder zentralem Monitoring (15 bis 25 %) |
| 4 Patches | 8 | Updates prüfen und einspielen (5 bis 10 %) |
| 6 Berechtigungen | 27 | Zugriffs- und Berechtigungskonzept (20 bis 40 %) |
| 7 Umsystem | 15 | Anpassungen an Umsystemen (15 bis 25 %) |

### Musterlösung (Kurzfassung)

**1.** `Get-DhcpServerv4Scope`, `Get-DhcpServerv4OptionValue -ScopeId 192.168.10.0`, `Get-DnsServerZone`, `Get-SmbShare`, `Get-ADGroup -Filter "Name -like '*Buchhaltung*'"`. Vergleichen mit der Tabelle. Abweichungen (zum Beispiel falsches Gateway im Scope) als Liste notieren. [W, Befehle zum Teil aus Lernhilfen]

**2.** `Get-Service DNS, DHCPServer, LanmanServer, Spooler`. Gestoppten Dienst mit `Start-Service` starten. In der Ereignisanzeige (System) steht dazu Event 7036 (gestartet oder gestoppt), bei einem Absturz 7034. [Q: Get-Service-Doku, Event-ID-Quellen]

**3.** Zum Beispiel:
```powershell
Get-Counter -Counter "\Processor(_Total)\% Processor Time" -SampleInterval 5 -MaxSamples 12
Get-Counter -Counter "\LogicalDisk(D:)\% Free Space"
Get-Counter -Counter "\Network Interface(*)\Bytes Total/sec"
```
Werte mit Uhrzeit und Intervall notieren. Zählernamen sind auf deutschem System lokalisiert, sie lassen sich mit `Get-Counter -ListSet *` finden. [Q: Get-Counter-Doku]

**4.** `Get-HotFix` listet installierte Updates. Ablauf [Q: n-able, TechTarget]:
1. Verfügbare Updates sichten und nach Risiko einstufen
2. Snapshot oder Backup von `SRV01`, Rollback-Plan festhalten
3. Test auf einer Testmaschine oder einem Klon
4. Rollout im Wartungsfenster, danach Neustart
5. Kontrolle (Dienste laufen, Monitoring grün, Hotfix-Liste) und Betriebsdokumentation nachführen
Quelle des Patches: Hersteller, bei manuellen Paketen Hashwert prüfen. [Q: Modul 188, Kenntnis 3.2]

**5.** SNMP installieren und absichern:
```powershell
Install-WindowsFeature SNMP-Service,SNMP-WMI-Provider -IncludeManagementTools
```
In den Eigenschaften des SNMP-Dienstes eine Community eintragen (nur lesen) und als erlaubten Host `192.168.10.20` setzen, nicht «alle Hosts». [Q: Paessler] Besser wäre SNMPv3, aber der Windows-Agent bietet nur v1 und v2c [Q: Microsoft Learn, Supported Versions], deshalb Zugriff auf eine IP und die Firewall begrenzen.
In PRTG Gerät `SRV01` anlegen, SNMP-Zugangsdaten hinterlegen, Sensoren hinzufügen: Ping Sensor, SNMP CPU Load Sensor, SNMP Disk Free v2 Sensor für D:, SNMP Windows Service v2 Sensor für den DHCP-Server, optional DHCP Sensor und DNS v2 Sensor [Q: PRTG-Sensorverzeichnis]. Beispiel für begründete Grenzwerte [W]:
- CPU: Warnung ab 80 Prozent über 5 Minuten, Fehler ab 95 Prozent. Begründung: kurze Spitzen sind normal, dauerhaft hohe Last zeigt echten Engpass.
- Speicherplatz D:: Warnung bei weniger als 20 Prozent frei, Fehler bei weniger als 10 Prozent. Begründung: Reserve für Wachstum und Backup.
- Dienststatus: sofort Fehler, wenn gestoppt.
Grenzwerte am Kanal setzen, State Trigger auf Warning und Down mit E-Mail-Vorlage, mit «Send test notification» testen und dann `Stop-Service DHCPServer` für den echten Test (danach wieder starten). [Q: PRTG-Handbuch]
Report und Map in PRTG [Q: https://www.paessler.com/manuals/prtg/reports und https://www.paessler.com/manuals/prtg/map_designer]: Berichte lassen sich auf Abruf oder nach Zeitplan als HTML, PDF, CSV oder XML erzeugen und per E-Mail versenden, die Vorlage bestimmt Aufbau und Detailtiefe. Für PDF muss der Druckwarteschlangen-Dienst (Spooler) auf dem PRTG-Core-Server laufen. Mit dem Map Designer ziehst du Objekte aus dem Gerätebaum auf eine Karte, die den Status zeigt und auch als Dashboard im Home-Menü abgelegt werden kann.

**6.**
```powershell
New-ADUser -Name "Sara Keller" -SamAccountName skeller -UserPrincipalName skeller@muster.local -AccountPassword (Read-Host -AsSecureString) -Enabled $true
Add-ADGroupMember -Identity GG_Buchhaltung -Members skeller
```
AGDLP: `GG_Buchhaltung` ist Mitglied in `DL_Buchhaltung_RW`, die Gruppe `DL_Buchhaltung_RW` hat auf Freigabe und NTFS das Recht «Ändern». Benutzer nie direkt in die ACL. [Q: Tenfold, ManageEngine]
Test: Anmeldung als `skeller` am Client, Datei in `\\SRV01\Buchhaltung` anlegen (muss gehen), Zugriff auf `\\SRV01\Geschaeftsleitung` (muss abgelehnt werden). Rechtetabelle in die Doku (Matrix Gruppen gegen Shares). [Q: Modul 117, 5.2]

**7.** Beispielauftrag [W]:
> Bitte auf der Firewall von SRV01 (192.168.10.10) eine eingehende Regel anlegen: Quelle 192.168.10.20 (MON01), Protokoll UDP, Port 161, Zweck SNMP-Überwachung. Alle anderen Quellen bleiben gesperrt. Umsetzung bis Freitag, Testverfahren: Abfrage von MON01 aus und Kontrolle der Sensoren in PRTG. Rückmeldung an den Betrieb.
Auf `SRV01` selbst: `New-NetFirewallRule -DisplayName "SNMP von MON01" -Direction Inbound -Protocol UDP -LocalPort 161 -RemoteAddress 192.168.10.20 -Action Allow` [Q: New-NetFirewallRule-Doku, Parameter]. Test von `MON01` aus: Sensor wird grün, zusätzlich `Test-NetConnection` für TCP-Ports und `snmpwalk` für UDP 161. [W]

---

## Teil B: Schriftlich (1 Stunde)

1. Nenne drei Inhalte einer Betriebsdokumentation und erkläre, warum sie nachgeführt werden muss.
2. Erkläre den Unterschied zwischen SNMP v2c und v3.
3. Warum setzt man Schwellwerte nicht einfach bei 50 Prozent? Wie bestimmst du sinnvolle Werte?
4. Beschreibe den Patch-Prozess für einen Produktivserver in fünf Schritten.
5. Welche Gefahren hat ein Patch für ein Unternehmen?
6. Woran erkennst du auf Windows einen Dienst, der unerwartet beendet wurde?
7. Erkläre AGDLP an einem Beispiel mit Abteilung Einkauf.
8. Eine Freigabe hat «Lesen», der NTFS-Ordner «Ändern». Was darf der Benutzer über das Netz?
9. Was passiert, wenn bei NTFS ein Recht auf «Verweigern» steht und der Benutzer in einer Gruppe mit «Erlauben» ist?
10. Wozu dienen DHCP-Reservationen? Nenne ein Beispiel.
11. Warum braucht Active Directory einen funktionierenden DNS-Dienst?
12. Was bedeutet «Baseline» im Monitoring?
13. Wie testest du, ob dein Alarmsystem zuverlässig funktioniert?
14. Nenne zwei Anpassungen auf Umsystemen und wie du sie dem Betreiber beschreibst.
15. Nenne drei Empfehlungen zu Backups und warum der Restore-Test wichtig ist.
16. Du siehst in der Monitoring-Map `SRV01` rot und `SRV02` gelb. Wie gehst du vor?
17. Wozu erstellst du automatische Reports und welche Inhalte gehören hinein?
18. Nach einem Patch läuft der DHCP-Dienst nicht mehr. Nenne einen Workaround und einen langfristigen Schritt.
19. Begründe das Berechtigungskonzept: Die Buchhaltung darf ändern, die Geschäftsleitung nur lesen. Wie setzt du es um und warum so?
20. Du erhältst eine Betriebsdokumentation. Wie gehst du vor, um das System zu analysieren, und was ergänzt du am Schluss?
21. Du patchst `SRV01` im Wartungsfenster und willst keine Alarme. Wie gehst du in PRTG vor und welche Falle gibt es?

### Musterlösung Teil B

1. Zum Beispiel Netzplan, Serverkonfiguration (Betriebssystem, Speicher, Anwendungen), Verantwortliche und Kontakte, Berechtigungskonzept. Sie muss aktuell sein, weil sonst Wartbarkeit und schnelle Hilfe im Störfall leiden. [Q: Modul 188 1.2, Vorlage Datenschutz.ch]
2. v2c: Authentifizierung nur über Community-String, Übertragung im Klartext, mit Sniffer mitlesbar. v3: Benutzer, Authentifizierung (MD5 oder SHA), optional Verschlüsselung (DES oder AES), Stufen noAuthNoPriv, authNoPriv, authPriv. [Q: Paessler, NinjaOne]
3. Weil der Normalwert je nach Serverrolle verschieden ist. Man misst zuerst eine Baseline und leitet Warn- und Fehlergrenzen davon ab, mit Blick auf Risiko und SLA. Kurzzeitige Spitzen sollen keinen Alarm auslösen. [Q: Bildungsplan f5, ChannelPartner]
4. Inventar und Risikoeinstufung, Test in Testumgebung, Backup und Rollback-Plan, Rollout im Wartungsfenster mit Pilotgruppe, Kontrolle und Doku. [Q: n-able, TechTarget]
5. Inkompatibilität mit Software, Neustart nötig, Dienstausfall, Fehlverhalten nach dem Update. [Q: Modul 188 3.3, Quellen]
6. Ereignisanzeige, System-Log, Event 7034. Zusätzlich zeigt `Get-Service` den Status Stopped, und das Monitoring meldet Down. [Q: Event-ID-Quellen, Get-Service-Doku]
7. Konten der Mitarbeitenden im Einkauf kommen in die globale Gruppe `GG_Einkauf`. Diese ist Mitglied in der domänenlokalen Gruppe `DL_Einkauf_RW`. Auf den Ordner bekommt nur `DL_Einkauf_RW` das Recht «Ändern». [Q: Tenfold, ManageEngine]
8. Nur Lesen, denn bei Zugriff über die Freigabe gilt die restriktivere Ebene. [Q: Netwrix, Varonis]
9. «Verweigern» hat Vorrang, der Benutzer hat keinen Zugriff. [Q]
10. Ein Gerät bekommt immer dieselbe Adresse, bleibt aber zentral verwaltet. Beispiel: Netzwerkdrucker 192.168.10.50. [Q: Microsoft Learn DHCP]
11. Clients und Domänencontroller finden sich über DNS. Ohne DNS funktionieren Anmeldung und Gruppenrichtlinien nicht richtig. [Q: Microsoft Learn DNS]
12. Die Normalwerte des Systems im Betrieb. Sie sind Referenz für Schwellwerte und für das Erkennen von Abweichungen. [W, Q: Quellen zu Baseline]
13. Mit regelmässigen Tests: Testbenachrichtigung auslösen und einen Dienst absichtlich stoppen. Ergebnis dokumentieren. [Q: Bildungsplan f5.4, PRTG-Handbuch]
14. DNS-Eintrag (Name, Zone, IP, TTL) und Firewall-Regel (Quelle, Ziel, Protokoll, Port, Zweck, Befristung, Testverfahren). Beschreibung so, dass Dritte sie umsetzen können. [Q: Modul 188 6.1 und 6.2]
15. Regelmässig sichern, externe Medien danach vom System trennen, Cloud nur mit geprüften Daten, Wiederherstellung testen, denn ein Backup ohne getesteten Restore ist nicht sicher. [Q: NCSC über Netzwoche]
16. Rot bedeutet Down (mindestens ein Sensor hat einen Fehler), gelb Warning (Fehler erkannt, PRTG versucht es erneut, nach einem zweiten Fehlversuch wird der Sensor Down). Orange (Unusual) wäre eine starke Abweichung vom üblichen Verlauf, blau pausiert. [Q: PRTG-Handbuch, Sensor States] Zuerst `SRV01` prüfen: welcher Sensor ist rot (Ping, Dienst, Speicher), seit wann, was steht im Ereignisprotokoll (zum Beispiel 7034). Dienst oder Gerät erreichbar? Dann beheben, Alarm quittieren, Ursache dokumentieren. `SRV02` beobachten und mit der Baseline vergleichen, ob der Wert dauerhaft steigt. [W, Q: PRTG-Handbuch zu Status und Schwellwerten]
17. Reports zeigen Verfügbarkeit und Auslastung über die Zeit, belegen SLA und helfen bei Kapazitätsplanung. Hinein gehören Verfügbarkeit pro Dienst, CPU, Speicher, freier Platz, Ausfälle und Alarme im Zeitraum. Sie lassen sich nach Zeitplan als PDF erzeugen und per E-Mail versenden. [Q: PRTG-Handbuch, Reports; W]
18. Workaround: Patch deinstallieren oder VM auf den Snapshot zurücksetzen, Dienst manuell starten und Ereignisprotokoll prüfen. Langfristig: Patch zuerst in der Testumgebung prüfen, Rollback-Plan und Wartungsfenster festlegen, Ursache mit dem Hersteller klären und das Vorgehen dokumentieren. [Q: Patch-Checkliste; W]
19. Gruppen `GG_Buchhaltung` und `GG_Geschaeftsleitung` (global) kommen in `DL_Buchhaltung_RW` (Ändern) und `DL_Buchhaltung_R` (Lesen) und erhalten nur dort Rechte auf Freigabe und NTFS. Begründung: Rechte hängen an Gruppen statt an Personen, das ist übersichtlich, bei Wechseln einfach zu pflegen und nachvollziehbar. Zudem gilt Least Privilege. Die Rechte gehören in eine Matrix in der Betriebsdokumentation. [Q: Tenfold, ManageEngine, Microsoft Learn; W]
20. Erst die Dokumentation lesen (Netzplan, Server, Dienste, Berechtigungen), dann am System prüfen, ob der Ist-Zustand zur Doku passt (Dienste, DHCP-Scope, DNS-Zone, Freigaben, Gruppen), Abweichungen notieren, dann die verlangten Anpassungen ausführen. Am Schluss die Doku nachführen: geänderte Konfiguration, neue Rechte, installierte Updates, Datum und Name. [Q: LBV und Modul 188 1.1 und 3; W]
21. Das übergeordnete Gerät `SRV01` pausieren (nicht nur den Ping-Master-Sensor), denn abhängige Sensoren pausieren sonst nicht mit und alarmieren weiter. Ein pausiertes Objekt erfasst keine Daten und löst keine Benachrichtigungen aus. Nach dem Patch fortsetzen, Sensoren kontrollieren und Ergebnis dokumentieren. Alternativ ein Wartungsfenster im Zeitplan hinterlegen [W]. [Q: PRTG-Handbuch, Dependencies]

---

## Hilfreiche Zusatzinfos für die Praxis

- **SNMP-OIDs** [Q: HOST-RESOURCES-MIB, RFC 2790 laut Mailing-Listen]: CPU-Last pro Prozessor unter `1.3.6.1.2.1.25.3.3.1.2` (hrProcessorLoad, Prozent der Zeit ohne Leerlauf, Mittelwert über die letzte Minute). Mit `snmpwalk` die Instanzen ermitteln, `snmpget` auf die Basis-OID liefert einen Fehler. Speicher und Platten liegen unter `1.3.6.1.2.1.25.2` (hrStorage). Die hrStorageTable hat die Spalten hrStorageIndex, hrStorageType, hrStorageDescr, hrStorageAllocationUnits, hrStorageSize, hrStorageUsed und hrStorageAllocationFailures [Q: MIB-Quelle und Herstellerdokus]. Belegung in Prozent rechnest du selbst: hrStorageUsed geteilt durch hrStorageSize [W]. Die Laufzeit (sysUpTime) ist `1.3.6.1.2.1.1.3.0` aus MIB-II, ein schreibgeschützter TimeTicks-Wert seit dem letzten Neustart des Managementteils [Q: OID-Referenz].
- **IPERKA** [Q: Stadt Bern bezeichnet es als 6-Schritte-Methode]: Informieren, Planen, Entscheiden, Realisieren, Kontrollieren, Auswerten (Buchstaben aus meinem Wissen). Passt gut für Aufträge im üK und für das Lernjournal.
