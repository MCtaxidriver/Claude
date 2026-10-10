# ÜK 188: Lernhilfen (Cheatsheet, Lernkarten, 14-Tage-Plan)

Ergänzung zu `UEK-188-Vorbereitung.md`. Markierung bei den Quellen:
- [Q] belegt in einer genannten Quelle
- [W] aus allgemeinem Fachwissen, bitte mit den Kursunterlagen abgleichen


<!-- toc -->
**Inhalt**

- 1. Cheatsheet: Befehle für die Lab-Umgebung
- 2. Lernkarten (Frage, Antwort), 74 Stück
- 2b. Falls die Learnfactory Linux nimmt: Gegenstücke (Ubuntu Server)
- 2c. Konzepte in Kürze (für den schriftlichen Teil)
- 2d. SNMP und WMI auf dem Server einrichten (Kenntnis 4.2)
- 2e. Sicherheits-Events für Berechtigungen und Anmeldungen (offiziell)
- 2f. Überwachung per Script (Bildungsplan f5.2 und d1.3)
- 2g. Passwortregeln und Agent-Monitoring (Ergänzungen)
- 2h. Glossar und Fehlersuche-Ablauf
- 2i. Active Directory: Gruppenbereiche und Standardgruppen (offiziell)
- 2j. Gruppenrichtlinien: Verarbeitung
- 2k. Leistungsüberwachung: Alarm mit Aktion (offiziell)
- 2l. Patch-Management Checkliste (Best Practice, nicht Schweizer Vorgabe)
- 2m. PRTG für die eigene Übung
- 2n. Lab zu Hause aufbauen (Tag 4 im Plan)
- 2o. Fernverwaltung und Zeitsynchronisation
- 3. 14-Tage-Plan (je ca. 45 bis 60 Minuten)

<!-- /toc -->

---

## 1. Cheatsheet: Befehle für die Lab-Umgebung

Alle Beispiele in PowerShell, als Administrator. Namen und Adressen sind Platzhalter.

### Verzeichnisdienst, Benutzer, Gruppen
```powershell
# Rolle installieren [W]
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
# Neue Gesamtstruktur mit DNS [Q: Microsoft Learn, Install-ADDSForest]
Install-ADDSForest -DomainName "corp.contoso.com" -InstallDNS
# Gruppe anlegen [Q: New-ADGroup]
New-ADGroup -Name "RODC Admins" -SamAccountName RODCAdmins -GroupCategory Security -GroupScope Global -Path "CN=Users,DC=Fabrikam,DC=Com"
# Mitglieder hinzufügen [Q: Add-ADGroupMember]
Add-ADGroupMember -Identity SvcAccPSOGroup -Members SQL01, SQL02
# Benutzer anlegen [Q: New-ADUser, Parameter Name]
New-ADUser -Name 'ChewDavid'
```
Hinweise [Q]: DNS wird beim Anlegen einer neuen Gesamtstruktur standardmässig mitinstalliert. Der Befehl fragt das DSRM-Passwort ab und startet den Server danach neu, wenn du das nicht abstellst.

### Freigaben und NTFS-Rechte
```powershell
# Freigabe mit Rechten [Q: New-SmbShare]
New-SmbShare -Name 'VMSFiles' -Path 'C:\Daten' -ChangeAccess 'CONTOSO\Finance Users','CONTOSO\HR Users' -FullAccess 'Administrators'
# Verschlüsselte Freigabe [Q]
New-SmbShare -Name "Data" -Path "J:\Data" -EncryptData $True
```
```cmd
:: NTFS-Rechte mit icacls [Q: Microsoft Learn, icacls]
icacls C:\Daten /grant "CONTOSO\Finance Users":(OI)(CI)M
icacls C:\Daten /deny  "CONTOSO\Praktikant":(OI)(CI)W
icacls C:\Daten /inheritancelevel:d
icacls C:\Daten /save acl.txt /t
icacls C:\ /restore acl.txt
```
Vererbungsflags [Q]: `(OI)` Objekte erben, `(CI)` Container erben, `(IO)` nur vererben, `(NP)` nicht weitergeben. `/inheritancelevel` kennt `e` (Vererbung an), `d` (aus, ACEs kopieren) und `r` (aus, ACEs entfernen). Die Rechtekürzel M (Ändern), F (Vollzugriff), R (Lesen) sind [W].

### DHCP
```powershell
# Scope [Q: Add-DhcpServerv4Scope]
Add-DhcpServerv4Scope -Name "Lab" -StartRange 10.10.10.1 -EndRange 10.10.10.254 -SubnetMask 255.255.255.0
# Optionen: Router, DNS, Domain [Q: Set-DhcpServerv4OptionValue]
Set-DhcpServerv4OptionValue -ScopeId 10.10.10.0 -DnsServer 192.168.1.2 -DnsDomain "contoso.com" -Router 192.168.1.1
# Reservation [Q: Add-DhcpServerv4Reservation]
Add-DhcpServerv4Reservation -ScopeId 10.10.10.0 -IPAddress 10.10.10.8 -ClientId "F0-DE-F1-7A-00-5E" -Description "Reservation for Printer"
# Aus Lease eine Reservation machen [Q]
Get-DhcpServerv4Lease -IPAddress 10.10.10.11 | Add-DhcpServerv4Reservation
```
Client testen [W]: `ipconfig /release`, dann `ipconfig /renew`, dann `ipconfig /all`.

### DNS
```powershell
# A-Record mit PTR [Q: Add-DnsServerResourceRecordA]
Add-DnsServerResourceRecordA -Name "host23" -ZoneName "contoso.com" -IPv4Address "172.18.99.23" -TimeToLive 01:00:00 -CreatePtr
```
Testen [W]: `nslookup host23.contoso.com`, `Resolve-DnsName host23.contoso.com`.

### Druckdienst
```powershell
# Reihenfolge: Treiber, Port, Drucker [Q: Microsoft Learn, PrintManagement]
Add-PrinterDriver -Name "Driver Name"
Add-PrinterPort -Name "TCPPort:" -PrinterHostAddress "192.168.100.100"
Add-Printer -Name "mxdw 2" -DriverName "Microsoft XPS Document Writer v4" -PortName "portprompt:"
# Client verbindet sich mit dem Druckserver [Q]
Add-Printer -ConnectionName \\printServer\printerName
# Warteschlange ansehen [Q]
Get-PrintJob -PrinterName "PrinterName"
```
Bei Problemen [W]: Dienst «Druckwarteschlange» (Spooler) neu starten mit `Restart-Service Spooler`.

### Gruppenrichtlinien (GPO)
```powershell
New-GPO -Name TestGPO -Comment "This is a test GPO."                                   # [Q: New-GPO]
New-GPO -Name "MyGPO" | New-GPLink -Target "ou=MyOU,dc=contoso,dc=com" -LinkEnabled Yes  # [Q: New-GPLink]
```
```cmd
gpupdate /force          :: Richtlinien sofort neu anwenden [Q]
gpresult /r              :: angewendete Richtlinien anzeigen [Q: gpresult mit /r und /z]
```

### Backup und Restore (Windows Server Backup)
```cmd
:: Sicherung von Volumes oder Ordnern auf ein Ziel [Q: Microsoft Learn, wbadmin start backup]
wbadmin start backup -backupTarget:d: -include:g:\folder1,h:\folder2 -systemstate -vsscopy
wbadmin start backup -backupTarget:\\backupshare\backup1 -noinheritacl -nonrecurseinclude:d:\folder1
:: Wiederherstellung einer Datei oder eines Ordners [Q: wbadmin start recovery]
wbadmin start recovery -version:03/31/2020-09:00 -itemType:File -items:d:\folder -recursive
```
Das Feature «Windows Server Backup» muss dafür installiert sein: `Install-WindowsFeature -Name Windows-Server-Backup -IncludeManagementTools`, Status mit `Get-WindowsFeature Windows-Server-Backup`. Das Feature stellt auch `wbadmin` bereit. Fehlen Quelldateien (Fehler 0x800f081f), hilft der Parameter `-Source` oder `DISM /Online /Enable-Feature /FeatureName:WindowsServerBackup` [Q: Microsoft Learn und Microsoft Q&A]. Danach immer einen Restore-Test machen, damit du weisst, dass es funktioniert (NCSC-Empfehlung).

### Dienste, Updates, Logs, Performance
```powershell
Get-Service "wmi*"                               # [Q: Get-Service]
Get-Service -DisplayName "*network*"             # [Q]
Get-HotFix                                       # installierte Updates [Q: Get-HotFix]
Get-WinEvent -ListLog *                          # alle Ereignisprotokolle [Q: Get-WinEvent]
Get-CimInstance -ClassName Win32_Process         # WMI/CIM-Abfrage [Q: Get-CimInstance]
Get-Counter -Counter "\Processor(_Total)\% Processor Time" -SampleInterval 2 -MaxSamples 3   # [Q: Get-Counter]
Get-Counter -Counter "\LogicalDisk(_Total)\% Free Space"                                     # [Q]
```
Zählernamen sind lokalisiert (auf deutschem System deutsch). `Get-Counter -ListSet *` zeigt die Namen [Q].

### Firewall (Umsysteme)
```powershell
# [Q: New-NetFirewallRule]
New-NetFirewallRule -DisplayName "Allow App" -Direction Inbound -Program "C:\Pfad\app.exe" -RemoteAddress LocalSubnet -Action Allow
New-NetFirewallRule -DisplayName "Block Outbound Port 80" -Direction Outbound -LocalPort 80 -Protocol TCP -Action Block
```

### Windows Server 2025 beachten [Q: Microsoft Learn, entfernte Features]
WMIC nur noch als Feature on Demand, PowerShell 2.0 nicht mehr enthalten, WSUS nicht mehr aktiv entwickelt. Ältere Anleitungen mit `wmic` oder `wuauclt /detectnow` funktionieren nicht mehr wie früher.

---

## 2. Lernkarten (Frage, Antwort), 74 Stück

Reihenfolge nach den 6 Handlungszielen des Moduls. Decke die Antwort ab und sprich sie laut.

### Handlungsziel 1: Betriebsdokumentation
1. **Warum braucht es eine nachgeführte Betriebsdokumentation?** Damit Systeme wartbar bleiben und kritische Situationen schnell gelöst werden können. [Q: Modulbeschreibung 1.2]
2. **Nenne fünf Inhalte einer Betriebsdokumentation.** Zum Beispiel Zweck und Verantwortliche, Systemübersicht, Netzwerkplan, Serverkonfiguration (Betriebssystem, Speicher, Anwendungen), Log-Dateien, Zutritts- und Schutzmassnahmen. [Q: Vorlage Datenschutz.ch]
3. **Gehören Passwörter in die Betriebsdokumentation?** Nein, laut der Vorlage gehören sie ins Notfallkonzept. [Q]
4. **Wann führst du die Doku nach?** Nach jeder Änderung, zum Beispiel Patch, neuer Service oder neue Berechtigung. [Q: Handlungsziel 3]
5. **Welche Services nennt das Modul als verbreitet?** Verzeichnisdienst, File, Print, DHCP, DNS. [Q]

### Handlungsziel 2: Überwachen und betreiben mit Bordmitteln
6. **Welche Windows-Bordmittel überwachen einen Server?** Leistungsüberwachung (perfmon), Ressourcenmonitor, Ereignisanzeige, Task-Manager. [Q: Lerndokumentation, W]
7. **Welcher Zähler zeigt die CPU-Last?** `\Processor(_Total)\% Processor Time`. [Q]
8. **Ab wann deutet die Prozessor-Warteschlange auf Überlast hin?** Dauerhaft über 2 gilt als Richtwert. [Q: ChannelPartner]
9. **Was bedeuten viele Page Reads und Writes pro Sekunde?** Meist zu wenig RAM. [Q]
10. **Was zeigt `Current Disk Queue Length` dauerhaft hoch?** Die Platte ist ausgelastet. [Q]
11. **Wie liest du den Dienststatus per PowerShell?** `Get-Service`, mit Filter zum Beispiel `Get-Service "wmi*"`. [Q]
12. **Event-ID 7036 und 7034, was heisst das?** 7036: Dienst gestartet oder gestoppt. 7034: Dienst unerwartet beendet. [Q: Drittquellen] Sicherheitslog: 4624 Anmeldung ok, 4625 Anmeldung fehlgeschlagen, 4740 Konto gesperrt. [Q: Microsoft Learn]
13. **Was ist eine Baseline und wozu brauchst du sie?** Normalwerte des Systems im Betrieb. Schwellwerte sinnvoll setzen geht nur im Vergleich dazu. [Q, W]

### Handlungsziel 3: Updates und Patches
14. **Beschreibe den Patch-Prozess in Schritten.** Inventar, Priorisierung nach Risiko, Test in Testumgebung, Backup und Rollback-Plan, Pilotgruppe, gestaffelter Rollout im Wartungsfenster, Kontrolle, Doku nachführen. [Q: n-able, TechTarget]
15. **Woher beziehst du Patches?** Von vertrauenswürdigen Quellen, also dem Hersteller, und du prüfst Hashwert oder Signatur. [Q: Modulbeschreibung 3.2]
16. **Welche Gefahren haben Patches für ein Unternehmen?** Inkompatibilität, Neustart nötig, Dienstausfall. [Q: Modulbeschreibung 3.3, Quellen]
17. **Was ist WSUS?** Serverrolle, mit der Microsoft-Updates zentral genehmigt und verteilt werden. Wird nicht mehr aktiv entwickelt. [Q: Microsoft Learn]
18. **Wie prüfst du, welche Updates installiert sind?** `Get-HotFix`. [Q]
19. **Unterschied Hotfix und Upgrade?** Hotfix behebt schnell einen Fehler oder eine Sicherheitslücke. Upgrade wechselt die Version, zum Beispiel Windows 10 zu 11. [Q: Lerndokumentation]

### Handlungsziel 4: Monitoring einbinden
20. **Nenne zwei Techniken zur zentralen Erfassung von Performance-Daten.** SNMP und WMI. [Q: Modulbeschreibung 4.1]
21. **Was ist der Unterschied zwischen SNMP v2c und v3?** v2c: nur Community-String, Klartext. v3: Benutzer, Authentifizierung (MD5 oder SHA), optional Verschlüsselung (DES oder AES). [Q: Paessler, NinjaOne]
22. **Wie heissen die Sicherheitsstufen bei SNMPv3?** noAuthNoPriv, authNoPriv, authPriv. [Q]
23. **Wie installierst du den SNMP-Dienst auf Windows Server?** `Install-WindowsFeature SNMP-Service,SNMP-WMI-Provider -IncludeManagementTools`. [Q: theitbros]
24. **Welche Einstellung ist beim SNMP-Dienst Pflicht?** Community bzw. Zugriff und erlaubte Hosts, mindestens die IP des Monitoring-Servers. [Q: Paessler]
25. **Auf welchen Ports läuft SNMP?** UDP 161 für Abfragen (Manager an Agent), UDP 162 für Traps (Meldungen vom Agenten). Mit TLS: 10161 und 10162. [Q: Fortinet, Zenarmor, RFC 1157 über Drittquellen]
26. **Wie setzt du in PRTG einen Schwellwert?** Grenzwerte am Kanal des Sensors setzen, zum Beispiel Warnung ab 60 und Fehler ab 80. [Q: PRTG-Handbuch]
27. **Wie löst PRTG eine Mail aus?** Mit einem Benachrichtigungs-Trigger auf Warning oder Down und einer Benachrichtigungsvorlage. [Q]
28. **Warum Alarme regelmässig testen?** Damit du weisst, dass das Überwachungssystem zuverlässig funktioniert (Leistungsziel f5.4). [Q: Bildungsplan]
29. **Welche Last erzeugt WMI-Überwachung?** Über DCOM hohe Last, Paessler empfiehlt maximal etwa 200 solcher Sensoren pro Probe. [Q]
30. **Was ersetzt WMIC?** PowerShell, zum Beispiel `Get-CimInstance`. [Q: Microsoft Learn]

### Handlungsziel 5: Berechtigungen
31. **Was heisst AGDLP?** Accounts in globale Gruppen, diese in domänenlokale Gruppen, dort die Permissions auf die Ressource. [Q: Tenfold, ManageEngine]
32. **Warum AGDLP statt Rechte direkt auf Benutzer?** Übersicht, einfacher bei Wechseln, ACLs verweisen nur auf Gruppen. [Q]
33. **Unterschied Freigabe- und NTFS-Rechte?** Freigaberechte gelten nur über das Netz und kennen Lesen, Ändern, Vollzugriff. NTFS-Rechte gelten immer und sind feiner. Es gilt die restriktivere Ebene. [Q: Netwrix, Varonis]
34. **Was gewinnt, Erlauben oder Verweigern?** Verweigern. [Q]
35. **Was bedeutet Vererbung?** Rechte eines übergeordneten Ordners gelten für Unterordner und Dateien. [Q: Lerndokumentation]
36. **Was kopiert `icacls /inheritancelevel:d`?** Es schaltet die Vererbung ab und kopiert die geerbten Einträge als explizite ACEs. [Q]
37. **Authentisierung, Authentifizierung, Autorisierung?** Identität behaupten, Identität prüfen, Zugriff gewähren. [Q: Lerndokumentation]
38. **Wie dokumentierst du Rechte?** Zum Beispiel mit einer Matrix Benutzergruppen gegen Shares. [Q: Modul 117, 5.2]
39. **Was ist das Least-Privilege-Prinzip?** Jeder bekommt nur die Rechte, die er für die Aufgabe braucht. [W]

### Handlungsziel 6: Umsysteme
40. **Nenne zwei typische Anpassungen auf Umsystemen.** DNS-Einträge und Firewall-Regeln. [Q: Modulbeschreibung 6.1]
41. **Was muss ein Änderungsauftrag an Dritte enthalten?** Quelle, Ziel, Protokoll und Port, Zweck, Befristung, Testverfahren. [W, siehe Beispiel in der Vorbereitungsdatei]
42. **Wie testest du eine Firewall-Anpassung?** Von der Quelle aus verbinden, zum Beispiel `Test-NetConnection -ComputerName SRV01 -Port 443`. [W]
43. **Wozu `-CreatePtr` beim DNS-A-Record?** Es legt automatisch den Reverse-Eintrag an. [Q]

### Vorwissen aus 117 und 123
44. **Wie berechnest du bei 10.89.14.24/29 Netz-ID, Broadcast, erste und letzte Host-IP?** Netz-ID 10.89.14.24, Broadcast 10.89.14.31, Hosts .25 bis .30, Maske 255.255.255.248. [Q: Lerndokumentation]
45. **Was weist DHCP einem Client zu?** IP-Adresse, Subnetmaske, Gateway, DNS-Server, Lease-Dauer. [Q]
46. **Wann statische, wann dynamische IP?** Server, Drucker, Netzkomponenten statisch oder per Reservation. Clients dynamisch. [Q: Modul 123 1.5, W]
47. **Welche Schritte hat die Inbetriebnahme eines Serverdienstes?** Installieren, konfigurieren, starten, auf Client verfügbar machen, testen. [Q: Modul 123 2.1]
48. **Was ist RAID 5, wie viele Platten, wie viel Kapazität?** Striping mit Parität, mindestens 3 Platten, eine Platte Kapazität geht für Parität weg, eine Platte darf ausfallen. [Q: Lerndokumentation]
49. **Welche Backup-Arten gibt es?** Voll, differentiell (alles, was sich seit dem letzten Vollbackup geändert hat), inkrementell (was sich seit dem letzten Backup irgendeiner Art geändert hat). Differentielle Backups wachsen mit der Zeit, für die Wiederherstellung aus inkrementellen Backups braucht man die ganze Kette. Beide setzen auf einem Vollbackup auf. [Q: ComputerWeekly, Netzwelt, EaseUS]
50. **Was rät das NCSC beim Backup?** Regelmässig sichern, externe Medien danach trennen, Wiederherstellung testen. [Q: Netzwoche]

### Zusatzkarten: Print, GPO, Backup (51 bis 56)
51. **In welcher Reihenfolge richtest du einen Netzwerkdrucker auf dem Druckserver ein?** Treiber (`Add-PrinterDriver`), Port (`Add-PrinterPort`), Drucker (`Add-Printer`). [Q: Microsoft Learn]
52. **Wie verbindet sich ein Client mit dem freigegebenen Drucker?** `Add-Printer -ConnectionName \\printServer\printerName`. [Q]
53. **Der Drucker druckt nicht mehr, Warteschlange hängt. Erste Schritte?** `Get-PrintJob` ansehen, hängenden Auftrag entfernen, Dienst Druckwarteschlange neu starten. [W]
54. **Wie wendest du eine Gruppenrichtlinie sofort an und wie prüfst du, was gilt?** `gpupdate /force` und `gpresult /r`. [Q]
55. **Wie verknüpfst du ein GPO mit einer OU?** `New-GPO -Name "MyGPO" | New-GPLink -Target "ou=MyOU,dc=contoso,dc=com" -LinkEnabled Yes`. [Q]
56. **Wie sicherst und stellst du Dateien mit Windows Server Backup wieder her?** `wbadmin start backup ...` und `wbadmin start recovery -itemType:File ...`. Danach den Restore testen. [Q: wbadmin-Doku, NCSC]

### Zusatzkarten: Active Directory, Gruppenrichtlinien, Delegation (57 bis 63)
57. **Welche Gruppenbereiche kennt Active Directory und wer darf Mitglied sein?** Global: Konten und globale Gruppen der eigenen Domäne. Domänenlokal: Konten und Gruppen aus beliebigen (auch vertrauten) Domänen. Universell: Konten und globale/universelle Gruppen aus der ganzen Gesamtstruktur. [Q: Microsoft Learn, Sicherheitsgruppen]
58. **Wo kann eine domänenlokale Gruppe Berechtigungen erteilen?** Nur innerhalb derselben Domäne. Globale Gruppen können in jeder Domäne der Gesamtstruktur Rechte erhalten. [Q]
59. **In welcher Reihenfolge werden Gruppenrichtlinien angewendet und was gewinnt?** Lokal, dann Standort, Domäne, OU. Die tiefste OU wird zuletzt angewendet und gewinnt bei Konflikten. Erzwungene Links setzen sich durch und gehen vor «Vererbung blockieren». [Q: Microsoft Learn, Gruppenrichtlinienverarbeitung (Details via Suchergebnis)]
60. **Wie oft aktualisiert ein Client die Gruppenrichtlinien im Hintergrund?** Standard alle 90 Minuten plus zufällig bis zu 30 Minuten. Sofort geht es mit `gpupdate /force`. [Q]
61. **Welche Standardgruppe gibt Lesezugriff auf Ereignisprotokolle ohne Adminrechte?** Ereignisprotokollleser (Event Log Readers). [Q]
62. **Welche Gruppe darf Leistungszähler, Protokolle und Warnungen verwalten, ohne Administrator zu sein?** Leistungsprotokollbenutzer. Systemmonitorbenutzer dürfen die Leistungsindikatoren nur ansehen. [Q]
63. **Was dürfen DHCP-Administratoren und was DHCP-Benutzer?** Administratoren erstellen, löschen und verwalten Bereiche und können die DHCP-Datenbank sichern und wiederherstellen. Benutzer haben nur lesenden Zugriff. [Q]

### Zusatzkarten: Patch-Management (64 bis 67)
64. **Aus welchen Schritten besteht der Patch-Zyklus?** Planen, Beschaffen, Testen, Installieren, Dokumentieren. [Q: Checkliste Patch Management, Bayerisches Landesamt für Datenschutzaufsicht]
65. **Worauf achtest du bei der Quelle eines Patches?** Nur seriöse Quellen, nicht auf Fake-Patches auf fremden Websites hereinfallen. Integrität und Authentizität per Prüfsumme oder digitaler Signatur sicherstellen. [Q: gleiche Checkliste]
66. **Wie priorisierst du Patches?** Nach dem Risiko bei Nichteinspielung, zum Beispiel nach der Einstufung des Herstellers oder von CERT-Bund. Hilfsmittel sind CVE und CVSS. [Q: gleiche Checkliste]
67. **Was gehört zu einem Patch-Konzept?** Inventarliste der Hard- und Software, Netzpläne, Update-Plan mit Versionsständen, Zuständigkeiten, Regeln zur Priorisierung. [Q]

---

### Zusatzkarten: Prüfung laut LBV, Reports, Maps (68 bis 72)
68. **Welche Services werden laut LBV geprüft?** AD, DNS und DHCP. [Q: Modulbaukasten, LBV 188-1]
69. **Welches Kriterium hat im Praxisteil das grösste Gewicht?** Zugriffs- und Berechtigungskonzept erfassen, ergänzen, dokumentieren, mit 20 bis 40 Prozent. [Q: LBV 188-1, Element 1]
70. **Was bekommst du in der Prüfung vorgegeben?** Eine Betriebsdokumentation zum Prüfungssystem. Du analysierst damit das System und führst die Doku am Schluss nach. [Q: LBV 188-1]
71. **Was kann PRTG automatisch erzeugen und in welchen Formaten?** Berichte nach Zeitplan oder auf Abruf als HTML, PDF, CSV oder XML, auch per E-Mail. Für PDF muss der Spooler-Dienst auf dem Core-Server laufen. [Q: PRTG-Handbuch, Reports]
72. **Was zeigt eine Map im Monitoring?** Objekte aus dem Gerätebaum mit ihrem Status auf einer Karte oder einem Dashboard. [Q: PRTG-Handbuch, Map Designer]

### Zusatzkarten: Wartung (73 bis 74)
73. **Welche Wartungszyklen nennt der Bildungsplan?** Täglich, wöchentlich, monatlich, bei bestimmten Ereignissen und gemäss SLA. [Q: Bildungsplan f4]
74. **Was gehört zur Schweigepflicht bei der Wartung?** Mit Daten vertrauensvoll und sorgfältig umgehen und Eingriffe nachvollziehbar dokumentieren. [Q: Bildungsplan f4]

## 2b. Falls die Learnfactory Linux nimmt: Gegenstücke (Ubuntu Server)

Das Modul legt sich nicht auf ein Betriebssystem fest. Hier die Linux-Entsprechungen, damit du in beiden Fällen vorbereitet bist.

| Aufgabe | Windows | Linux (Ubuntu) |
| --- | --- | --- |
| Dienst prüfen | `Get-Service` | `systemctl status <dienst>`, `systemctl is-active <dienst>`, `systemctl list-units --failed` [Q: Ubuntu man systemctl] |
| Dienst steuern | `Start-Service`, `Restart-Service` | `systemctl start/stop/restart/enable <dienst>` [Q] |
| Logs lesen | Ereignisanzeige, `Get-WinEvent` | `journalctl -u <dienst>`, `--since`, `-b` (seit Boot) [Q: Ubuntu man journalctl] |
| Updates | WSUS, Windows Update | `apt update`, `apt upgrade`, automatisch mit `unattended-upgrades` [Q: Ubuntu Server Docs] |
| Berechtigungen | NTFS, `icacls` | `chmod`, `chown`, ACLs mit `setfacl -m` und `getfacl` [Q: Ubuntu man setfacl, W] |
| Firewall | `New-NetFirewallRule` | `ufw`, z.B. `sudo ufw allow proto tcp from 192.168.0.2 to any port 22` (SSH nur von einem Host). Für SNMP: `sudo ufw allow proto udp from <Monitoring-IP> to any port 161`. Statt einer IP geht auch ein Netz wie 192.168.0.0/24. [Q: Ubuntu Server Docs, Firewall]. Regeln werden der Reihe nach geprüft, ein früheres «deny» kann eine spätere Erlaubnis aushebeln. |
| SNMP-Agent | Feature SNMP-Service | Paket `snmpd`, Konfiguration in `/etc/snmp/snmpd.conf` [W] |
| Performance | `Get-Counter` | `top`, `vmstat`, `iostat`, `free -h`, `df -h` [W] |

Belegte Details zu Updates [Q: https://documentation.ubuntu.com/server/how-to/software/automatic-updates/]:
- Das Paket `unattended-upgrades` ist bei Ubuntu standardmässig installiert, Sicherheitsupdates werden automatisch eingespielt, der Lauf erfolgt einmal pro Tag.
- Konfiguration: `/etc/apt/apt.conf.d/50unattended-upgrades` (Verhalten, Neustart, gesperrte Pakete) und `/etc/apt/apt.conf.d/20auto-upgrades` (an oder aus, wie oft).
- Logs: `/var/log/unattended-upgrades`.
- Mit der Option `--dry-run` testest du, ohne wirklich zu installieren. Das passt zu Kenntnis 3.4 (Testszenarien für Patches).

Belegte Details zu SNMP [Q: https://manpages.ubuntu.com/manpages/noble/en/man5/snmpd.conf.5.html]:
- `agentaddress` legt fest, auf welchen Adressen der Agent lauscht.
- SNMPv3-Benutzer legst du mit `createUser <name> SHA <authpass> AES <privpass>` an. Unterstützt sind MD5 und SHA-Varianten für die Authentifizierung sowie DES und AES für die Verschlüsselung.
- Zugriff gibst du mit `rouser <name> priv` (nur lesen, mit Verschlüsselung) oder `rwuser` frei. Die Stufen heissen `noauth`, `auth`, `priv` und entsprechen noAuthNoPriv, authNoPriv, authPriv.
- Prüfen von einem anderen Rechner aus [W]: `snmpwalk -v3 -l authPriv -u <name> -a SHA -A <authpass> -x AES -X <privpass> <ip>`

Weitere Linux-Quellen:
- https://manpages.ubuntu.com/manpages/noble/en/man1/journalctl.1.html
- https://manpages.ubuntu.com/manpages/noble/en/man1/systemctl.1.html
- https://documentation.ubuntu.com/server/how-to/security/firewalls/
- https://manpages.ubuntu.com/manpages/noble/en/man1/setfacl.1.html

---

## 2c. Konzepte in Kürze (für den schriftlichen Teil)

In meinen Worten, nach den Übersichtsseiten von Microsoft Learn.

**DHCP** [Q: https://learn.microsoft.com/de-de/windows-server/networking/technologies/dhcp/dhcp-top]
- Der Server verwaltet einen Adresspool und «least» Adressen an Clients. Nicht mehr genutzte Adressen gehen automatisch an den Pool zurück.
- Der Server speichert gültige TCP/IP-Parameter, den Pool inklusive ausgeschlossener Adressen, Reservationen (eine feste Adresse für einen bestimmten Client) und die Leasedauer.
- Der Client bekommt Adresse und Optionen, zum Beispiel Router (Gateway), DNS-Server und DNS-Domänenname.
- Vorteile: weniger Tippfehler und Adresskonflikte, weniger Verwaltungsaufwand, zentrale Konfiguration.

**DNS** [Q: https://learn.microsoft.com/de-de/windows-server/networking/dns/dns-top]
- Ordnet Namen IP-Adressen zu und ist in Windows-Netzen der Standard-Namensauflösungsdienst.
- Bei einer neuen AD-Gesamtstruktur wird DNS automatisch mit installiert. AD DS braucht DNS, damit Clients Domänencontroller finden und die Domänencontroller sich untereinander finden.

**Active Directory Domain Services** [Q: https://learn.microsoft.com/de-de/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview]
- Ein Verzeichnisdienst speichert Objekte (Benutzer, Computer, Drucker, Volumes) hierarchisch. Sicherheit ist über Anmeldung und Zugriffssteuerung eingebaut. Mit einer einzigen Anmeldung im Netz erreichst du berechtigte Ressourcen.

**SMB-Dateifreigabe** [Q: https://learn.microsoft.com/de-de/windows-server/storage/file-server/file-server-smb-overview]
- SMB ist das Netzwerkprotokoll zum Lesen und Schreiben von Dateien auf einem Remoteserver über TCP/IP.

**Zugriffssteuerung** [Q: https://learn.microsoft.com/de-de/windows/security/identity-protection/access-control/access-control]
- Bausteine: Berechtigungen, Besitz von Objekten, Vererbung, Benutzerrechte, Objektüberwachung.
- Benutzer und Gruppen sind Sicherheitsprinzipale mit eindeutiger SID.
- Berechtigungen legen die Art des Zugriffs auf ein Objekt fest, zum Beispiel Lesen, Ändern, Besitzer ändern, Löschen.
- Microsoft empfiehlt, Berechtigungen Gruppen zuzuweisen und nicht einzelnen Benutzern. Das verbessert auch die Leistung beim Prüfen des Zugriffs.
- Authentifizierung kommt zuerst (wer bist du), danach die Autorisierung (was darfst du).

**Backup-Begriffe** [Q: Sekundärquellen, keine Behördenquelle gefunden]
- **3-2-1-Regel:** drei Kopien der Daten, auf zwei verschiedenen Medientypen, eine davon extern. Eine moderne Erweiterung ist 3-2-1-1-0 mit einer zusätzlichen unveränderlichen oder vom Netz getrennten Kopie und null Fehlern beim Wiederherstellungstest (Veeam, ComputerWeekly, SentinelOne). Die CISA soll die Regel in einem Leitfaden nennen, den Originaltext habe ich nicht gelesen.
- **RPO** (Recovery Point Objective, verkraftbarer Datenverlust): maximale Zeit an Daten, die bei einem Ausfall verloren gehen darf, zum Beispiel 30 Minuten. Das Backup-Intervall muss dazu passen. **RTO** (Recovery Time Objective, Wiederanlaufzeit): wie lange die Wiederherstellung maximal dauern darf. Kürzere Werte kosten meist mehr. [Q: Microsoft Azure Well-Architected Framework und Azure Business Continuity]

**Überwachung allgemein** [Q: Bildungsplan f5, PRTG-Handbuch]
- Erst festlegen, was überwacht wird und mit welcher Methode (Script, Software, Benachrichtigung). Dann Schwellwerte und Aktionen definieren und dabei Risiko und SLA beachten. Danach das Überwachungssystem regelmässig testen und die Ergebnisse dokumentieren. Bei Alarm rasch handeln.

---

## 2d. SNMP und WMI auf dem Server einrichten (Kenntnis 4.2)

**SNMP-Dienst unter Windows** [Q: Paessler, https://helpdesk.paessler.com/en/support/solutions/articles/76000042365-how-do-i-install-the-snmp-service-on-windows-systems-]
1. Dienst installieren: Server-Manager (Features, SNMP-Dienst) oder per PowerShell `Install-WindowsFeature SNMP-Service,SNMP-WMI-Provider -IncludeManagementTools`. Vor Windows 8 und Server 2012 war er standardmässig vorhanden.
2. `services.msc` als Administrator öffnen, «SNMP Service» doppelklicken, Starttyp «Automatisch».
3. Reiter «Sicherheit»: eigenen Community-Namen eintragen (sicherer als der Standard «public»), Recht «Nur Lesen», den Eintrag «public» entfernen.
4. Im selben Reiter unten festlegen, von welchen IP-Adressen SNMP-Pakete angenommen werden. Mindestens die IP des Monitoring-Servers muss drin sein.
5. Reiter «Agent»: Kontakt, Standort und die Dienstarten ankreuzen, damit alle Werte verfügbar sind.
6. Speichern und neu starten. Hinweis von Paessler: Manche Anwendungen (zum Beispiel IIS, MS SQL Server) bringen ihre SNMP-Unterstützung nur mit, wenn der SNMP-Dienst vorher installiert war.
7. Windows Server 2012 zeigte teils fehlende Reiter «Sicherheit», «Agent» und «Traps». Paessler verweist dafür auf einen Workaround.

**WMI-Fernzugriff** [Q: Microsoft Learn, https://learn.microsoft.com/en-us/windows/win32/wmisdk/connecting-to-wmi-on-a-remote-computer]
- Fernverbindungen mit WMI hängen von der Windows-Firewall und den DCOM-Einstellungen ab. Auch die Benutzerkontensteuerung (UAC) kann Anpassungen erfordern.
- Die DCOM-Einstellungen müssen für eine Fernverbindung korrekt sein. Wer DCOM ändert, kann Benutzern mit tiefen Rechten Zugriff geben. Darum Rechte sparsam vergeben.
- Zuerst Berechtigung und Verbindungskonfiguration klären, danach ist der Aufruf selbst einfach.
- Konkrete Ports (zum Beispiel 135 und dynamische RPC-Ports bei DCOM) habe ich nicht aus einer Quelle belegt [W]. In der Praxis lieber CIM über WSMan (PowerShell-Remoting) nutzen, wenn das Monitoring-Tool es kann [W]. WinRM lauscht ab Windows 7 standardmässig auf TCP 5985 (HTTP) und 5986 (HTTPS), ältere Systeme nutzten 80 und 443 [Q: Microsoft Learn, https://learn.microsoft.com/en-us/powershell/scripting/security/remoting/winrm-security].

**SNMP-Versionen unter Windows** [Q: https://learn.microsoft.com/en-us/windows/win32/snmp/supported-versions]
- Der Windows-SNMP-Dienst unterstützt SNMPv1 und SNMPv2c, nicht SNMPv3 (laut Microsoft für Vista und Server 2008 dokumentiert, Hersteller wie Genesys und anynode bestätigen es für neuere Windows-Versionen). Microsoft weist ausserdem darauf hin, dass SNMP in späteren Versionen verändert oder nicht mehr verfügbar sein kann, und empfiehlt Windows-Remoteverwaltung (WS-Management).
- Für SNMPv3 unter Windows braucht es Zusatzsoftware wie Net-SNMP. [Q: Drittquelle]
- Folge für die Sicherheit: Community-String im Klartext. Darum Zugriff auf die IP des Monitoring-Servers beschränken, Firewall nutzen und nur lesen zulassen.

**Wahl SNMP gegen WMI** [W, zusammengefasst aus den Quellen]
- SNMP: schlank, plattformübergreifend (auch Switches, Drucker), v1 und v2c unsicher, v3 sicherer.
- WMI: Windows-spezifisch, sehr detailliert (Dienste, Event-Logs), braucht Domänen- oder lokales Konto mit Rechten, belastet bei vielen Sensoren stärker (Paessler: höchstens etwa 200 WMI-Sensoren pro Probe).

---

## 2e. Sicherheits-Events für Berechtigungen und Anmeldungen (offiziell)

Diese Event-IDs stehen im Sicherheitsprotokoll und helfen beim Nachweisen und Prüfen von Rechten. Definition laut Microsoft Learn (https://learn.microsoft.com/de-de/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4624 und analog /event-4625, /event-4740, /event-4720, /event-4732):

| Event | Bedeutung [Q] | Wofür im Betrieb [W] |
| --- | --- | --- |
| 4624 | Konto erfolgreich angemeldet. Entsteht auf dem Computer, auf den zugegriffen wurde. | Prüfen, wer sich wann angemeldet hat |
| 4625 | Anmeldung fehlgeschlagen (jeder Fehlversuch) | Viele in kurzer Zeit können auf Angriffe hindeuten |
| 4740 | Benutzerkonto gesperrt | Support-Fall «Ich komme nicht mehr rein» |
| 4720 | Benutzerkonto erstellt | Kontrolle nach dem Anlegen eines Mitarbeiters |
| 4732 | Mitglied zu einer sicherheitsrelevanten lokalen Gruppe hinzugefügt | Nachweis von Rechteänderungen |

Zur Abfrage dient `Get-WinEvent` mit `-FilterHashtable`, zum Beispiel für Anmeldefehler der letzten Stunde [W]:
```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625; StartTime=(Get-Date).AddHours(-1)}
```
**Dienst- und Neustart-Events im Systemlog:**
- **7036:** Statusänderung eines Dienstes (läuft, gestoppt). Viele Einträge sind normal und unkritisch. [Q: Microsoft Q&A, Community]
- **7034:** Dienst unerwartet beendet. Der Text nennt, wie oft das schon vorkam. Ursachenanalyse nötig (Berechtigungen, Systemdateien, Fehler des Dienstes), zum Beispiel `sfc /scannow` und `DISM /Online /Cleanup-Image /RestoreHealth`, Wiederherstellungsoptionen des Dienstes auf «Neu starten» setzen. [Q: Microsoft Learn, OpenSSH-Artikel, Microsoft Q&A]
- **41 (Kernel-Power):** System ohne sauberes Herunterfahren neu gestartet. Die Ursache lässt sich aus dem Ereignis allein oft nicht ableiten, man schaut, was zum Zeitpunkt passiert ist (Stromausfall, Absturz, Netzschalter). Bugcheck-Code ungleich 0 deutet auf einen Stoppfehler, 0 kann auf Stromversorgung hinweisen. [Q: https://learn.microsoft.com/de-de/troubleshoot/windows-client/performance/event-id-41-restart]

Hinweis: Diese Auditing-Events müssen aktiviert sein (Richtlinie «Überwachung»). In einer frischen Lab-VM sind nicht alle standardmässig an. [W]

---

## 2f. Überwachung per Script (Bildungsplan f5.2 und d1.3)

Der Bildungsplan nennt bei der Überwachung ausdrücklich «Script, Software, Benachrichtigungen». Ein kleines Script zeigt dir, wie das praktisch aussieht. Alle verwendeten Cmdlets sind bei Microsoft Learn dokumentiert (Get-Service, Get-Volume, New-EventLog, Write-EventLog, Register-ScheduledTask, New-ScheduledTaskAction, New-ScheduledTaskTrigger). Das Script selbst ist mein Entwurf und nicht aus einer Kursquelle [W]. Teste es in deiner VM.

```powershell
# C:\Scripts\Check-Server.ps1
$logName   = 'Application'
$source    = 'ServerCheck'
$services  = 'DHCPServer','DNS','LanmanServer','Spooler'
$minFreePc = 20      # Warnung unter 20 Prozent frei

# Ereignisquelle einmalig anlegen (als Administrator)
if (-not [System.Diagnostics.EventLog]::SourceExists($source)) {
    New-EventLog -LogName $logName -Source $source
}

# 1. Dienste prüfen
foreach ($name in $services) {
    $svc = Get-Service -Name $name -ErrorAction SilentlyContinue
    if (-not $svc -or $svc.Status -ne 'Running') {
        Write-EventLog -LogName $logName -Source $source -EventId 2001 -EntryType Error `
            -Message "Dienst $name läuft nicht (Status: $($svc.Status))."
    }
}

# 2. Freien Speicherplatz prüfen
Get-Volume | Where-Object { $_.DriveLetter -and $_.Size -gt 0 } | ForEach-Object {
    $freePc = [math]::Round(($_.SizeRemaining / $_.Size) * 100, 1)
    if ($freePc -lt $minFreePc) {
        Write-EventLog -LogName $logName -Source $source -EventId 2002 -EntryType Warning `
            -Message "Laufwerk $($_.DriveLetter): nur $freePc Prozent frei."
    }
}
```

Als Aufgabe alle 15 Minuten ausführen:
```powershell
$action  = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-NoProfile -File C:\Scripts\Check-Server.ps1"
$trigger = New-ScheduledTaskTrigger -Once -At (Get-Date).AddMinutes(1) -RepetitionInterval (New-TimeSpan -Minutes 15)
Register-ScheduledTask -TaskName "ServerCheck" -Action $action -Trigger $trigger -User "SYSTEM" -RunLevel Highest
```
Hinweis: Die Parameter `-Execute`, `-Once`, `-At` und `-Daily` sind in der Microsoft-Doku belegt. Das Wiederholungsintervall mit `-RepetitionInterval` und `-RunLevel` ist aus meinem Wissen und in dieser Form nicht in den abgerufenen Beispielen. Prüfe es mit `Get-Help New-ScheduledTaskTrigger -Full`.

Wie das Monitoring davon profitiert [W]: Ein Monitoring-Tool wie PRTG kann das Ereignisprotokoll überwachen und bei Event 2001 oder 2002 alarmieren. Das Script ist also eine einfache zweite Methode neben SNMP.

Erklärung für den schriftlichen Teil:
- Das Script prüft Dienste und freien Platz, schreibt Fehler und Warnungen mit eigener Event-ID ins Anwendungsprotokoll und läuft zeitgesteuert.
- Schwellwert für Speicherplatz sind 20 Prozent frei, damit Reserve für Wachstum und Backup bleibt.
- Das Script ersetzt keine Baseline und kein getestetes Alarmsystem.

---

## 2g. Passwortregeln und Agent-Monitoring (Ergänzungen)

**Passwortregeln** (Modul 123, Kenntnis 4.2 «Regeln für Passwörter, die das Hacken erschweren») [Q: NCSC über Netzwoche, https://www.netzwoche.ch/news/2021-05-06/tipps-vom-ncsc-worauf-es-bei-einem-starken-passwort-ankommt und https://www.netzwoche.ch/news/2023-05-02/behoerden-starten-nationale-sensibilisierungskampagne-zu-passwortsicherheit]
- Mindestens 12 Zeichen, mit Gross- und Kleinbuchstaben, Zahlen und Sonderzeichen.
- Das Passwort darf in keinem Wörterbuch vorkommen.
- Pro Dienst und Gerät ein eigenes Passwort, damit bei einem Leak der Schaden klein bleibt.
- Ein Passwortmanager hilft, dann musst du dir nur ein starkes Passwort merken.
- Bildschirm sperren, wenn du das Gerät nicht aktiv nutzt.
Ob Passphrasen und das Ende erzwungener Passwortwechsel auch vom Schweizer NCSC empfohlen werden, habe ich nicht belegt gefunden (die Quellen dazu stammen vom britischen NCSC). In Windows setzt du Passwortrichtlinien per GPO oder in den lokalen Sicherheitsrichtlinien (`secpol.msc`). [Q: Lab-Mitschrift BBRZ]

**Agent statt SNMP: Zabbix-Agent unter Windows** [Q: https://www.zabbix.com/documentation/8.0/en/manual/installation/install/win_msi]
- Installation per 32- oder 64-Bit-MSI, über den Assistenten oder die Kommandozeile.
- Standardpfad `C:\Program Files\Zabbix Agent`. Der Dienst startet verzögert automatisch.
- Im Setup gibst du Hostnamen, Zabbix-Server-IP oder -DNS und den Listen-Port des Agenten an. Standardport 10050.
- Prometheus und Nagios habe ich nicht belegt gefunden, sie sind für diesen Kurs vermutlich nicht nötig. Gemini nennt sie, das Modul tut es nicht.

**Agentenlos (SNMP/WMI) gegen mit Agent** [W]
- Agentenlos: kein Zusatzprogramm, Standardmittel des Systems. Dafür weniger Details und mehr Last auf dem Monitoring-Server.
- Mit Agent: mehr Details, eigener Dienst und eigene Pflege (Updates, Port, Rechte).

---

## 2h. Glossar und Fehlersuche-Ablauf

Alles [W], aus Fachwissen und den Quellen oben, zum schnellen Nachschlagen.

| Begriff | Kurz erklärt |
| --- | --- |
| ACL / ACE | Access Control List, Liste von Zugriffsregeln. Ein Eintrag darin heisst ACE. |
| AD DS | Active Directory Domain Services, Verzeichnisdienst von Microsoft |
| AGDLP | Accounts, Global groups, Domain local groups, Permissions |
| Baseline | Normalwerte eines Systems im regulären Betrieb |
| Betriebsdokumentation | Beschreibung von System, Konfiguration, Verantwortlichen und Abläufen |
| CIM / WMI | Standard und Windows-Schnittstelle, um Systeminformationen abzufragen |
| DCOM | Verteiltes COM, Grundlage der klassischen WMI-Fernverbindung |
| DHCP-Lease | Zeitlich begrenzte Adresszuteilung |
| DHCP-Reservation | Feste Adresse für einen bestimmten Client (über die MAC-Adresse) |
| DNS-Zone | Verwaltungsbereich eines DNS-Servers mit den Einträgen einer Domäne |
| DSRM | Directory Services Restore Mode, Wiederherstellungsmodus eines Domänencontrollers |
| GPO | Group Policy Object, Gruppenrichtlinie |
| Hotfix | Schnelle Korrektur für einen bestimmten Fehler oder eine Lücke |
| Least Privilege | Nur so viele Rechte wie nötig |
| MIB / OID | Struktur und Bezeichner der SNMP-Werte |
| NTFS | Dateisystem von Windows mit feinen Zugriffsrechten |
| OU | Organisationseinheit in Active Directory zum Gliedern und Delegieren |
| Patch | Software-Korrektur, oft Sicherheitslücken |
| RPO / RTO | Verkraftbarer Datenverlust / verkraftbare Ausfallzeit |
| Rollback | Rückkehr zum Zustand vor einer Änderung |
| SLA | Service Level Agreement, vereinbarte Dienstgüte |
| SNMP | Simple Network Management Protocol zur Überwachung von Geräten |
| Schwellwert | Grenzwert, ab dem Warnung oder Fehler gemeldet wird |
| SID | Eindeutige Sicherheitskennung eines Benutzers oder einer Gruppe |
| Snapshot | Zustand einer VM zu einem Zeitpunkt, für schnellen Rückgang |
| Wartungsfenster | Vereinbarte Zeit für Änderungen, die Dienste stören können |
| WSUS | Windows Server Update Services |

**Fehlersuche bei «Dienst nicht erreichbar»** (vom Einfachen zum Aufwendigen) [W]
1. Läuft der Dienst auf dem Server? `Get-Service`, bei Bedarf starten.
2. Gibt es Fehler im Ereignisprotokoll? Event 7034 oder 7036 und das Anwendungsprotokoll prüfen.
3. Löst der Name auf? `nslookup` oder `Resolve-DnsName`.
4. Ist der Server im Netz erreichbar? `ping`, dann `Test-NetConnection -ComputerName <Server> -Port <Port>`.
5. Blockiert eine Firewall? Regeln auf Server und dazwischen prüfen.
6. Stimmen Rechte? Anmeldung des Benutzers, Gruppenzugehörigkeit, Freigabe- und NTFS-Rechte.
7. Was wurde zuletzt geändert? Patch, Konfiguration, Rechte. Betriebsdokumentation und Change-Verlauf ansehen.
8. Lösung dokumentieren und, wenn möglich, ein Monitoring oder einen Alarm dafür einrichten.

---

## 2i. Active Directory: Gruppenbereiche und Standardgruppen (offiziell)

Quelle: https://learn.microsoft.com/de-de/windows-server/identity/ad-ds/manage/understand-security-groups

| Bereich | Mögliche Mitglieder | Kann Berechtigungen erteilen | Typische Rolle in AGDLP |
| --- | --- | --- | --- |
| Global | Konten und globale Gruppen der eigenen Domäne | In jeder Domäne der Gesamtstruktur und in vertrauten Domänen | Rollengruppe (z.B. Abteilung) |
| Domänenlokal | Konten, globale Gruppen und universelle Gruppen aus beliebigen oder vertrauten Domänen, andere domänenlokale Gruppen derselben Domäne | Nur innerhalb derselben Domäne | Zugriffsgruppe auf Ressourcen |
| Universell | Konten und globale/universelle Gruppen aus der Gesamtstruktur | In der ganzen Gesamtstruktur und in vertrauten | Nur bei mehreren Domänen (AGUDLP) |

Hilfreiche Standardgruppen für den Betrieb, damit nicht jeder Domänen-Admin sein muss [Q: gleiche Seite]:
- **DHCP-Administratoren:** Bereiche erstellen, löschen, verwalten, DHCP-Datenbank sichern und wiederherstellen. **DHCP-Benutzer:** nur lesen.
- **DnsAdmins:** Zugriff auf DNS-Informationen mit Lesen, Schreiben, Erstellen und Löschen untergeordneter Objekte.
- **Ereignisprotokollleser:** Ereignisprotokolle lesen.
- **Leistungsprotokollbenutzer:** Leistungszähler, Protokolle und Warnungen verwalten (lokal und remote), ohne Administrator zu sein. **Systemmonitorbenutzer:** Leistungsindikatoren ansehen.
- **Sicherungsoperatoren:** alle Dateien sichern und wiederherstellen, unabhängig von den Dateirechten.
- **Druckoperatoren:** Drucker auf Domänencontrollern verwalten.
- **Remoteverwaltungsbenutzer:** über WS-Management auf WMI zugreifen.
- **Gruppenrichtlinienersteller-Besitzer:** GPOs in der Domäne erstellen, bearbeiten, löschen.

Für das Monitoring [W]: Ein Dienstkonto für das Monitoring sollte nur die nötigen Rechte bekommen, zum Beispiel Mitglied in Ereignisprotokollleser und Leistungsprotokollbenutzer, statt Domänen-Admin (Least Privilege). Das passt zu Handlungsziel 5.

---

## 2j. Gruppenrichtlinien: Verarbeitung

Quelle: https://learn.microsoft.com/de-de/windows-server/identity/ad-ds/manage/group-policy/group-policy-processing und .../group-policy-scope

- Computerrichtlinien werden beim Start angewendet, Benutzerrichtlinien bei der Anmeldung (Vordergrundverarbeitung, synchron oder asynchron). Alle Verarbeitungen müssen innerhalb von 60 Minuten abgeschlossen sein.
- Danach Hintergrundaktualisierung: Standard alle 90 Minuten, plus zufällig bis zu 30 Minuten. Einstellbar per Gruppenrichtlinie. Manches wird nur bei Anmeldung oder Start verarbeitet (zum Beispiel Ordnerumleitung und Softwareinstallation).
- Reihenfolge laut Dokumentation und Suchergebnis: lokales GPO, Standort, Domäne, OU (tiefste zuletzt, gewinnt bei Konflikten). Bei mehreren Links in einem Container hat die niedrigste Zahl die höchste Priorität.
- **Erzwungen** ist eine Eigenschaft des Links. Erzwungene GPOs setzen sich gegen widersprüchliche nicht erzwungene durch. **Vererbung blockieren** ist eine Eigenschaft des Containers und wirkt nicht gegen erzwungene Links. Das lokale GPO wird immer verarbeitet.
- Sofort anwenden: `gpupdate /force`. Prüfen: `gpresult /r`. Modellierung und Ergebnisse gibt es auch in der Gruppenrichtlinien-Verwaltungskonsole.

---

## 2k. Leistungsüberwachung: Alarm mit Aktion (offiziell)

Quelle: https://learn.microsoft.com/de-de/troubleshoot/windows-server/performance/create-performance-counter-alert-send-email

1. Leistungsüberwachung öffnen, Datensammlersätze, Benutzerdefiniert, den Satz mit den Leistungsindikatorwarnungen wählen.
2. Rechtsklick auf den Datensammler vom Typ «Warnung», Eigenschaften, Reiter «Warnungen» (Indikator und Grenzwert).
3. Reiter «Warnungsaktion»: Eintrag ins Ereignisprotokoll schreiben (Anwendungs- und Dienstprotokolle, Microsoft, Windows, Diagnostics-PLA, Operational) und/oder einen Datensammlersatz starten.
4. Reiter «Warnungsaufgabe»: Name einer Aufgabe im Aufgabenplaner angeben, die bei Auslösung läuft (laut Anleitung zum Beispiel mit der Aktion «E-Mail senden»).
5. Aufgabe im Aufgabenplaner anlegen, «Mit höchsten Berechtigungen ausführen» setzen.

Korrektur zu meiner früheren Notiz: Ausgelöste Warnungen stehen laut Microsoft im Protokoll `Microsoft-Windows-Diagnostics-PLA/Operational`. Die Angabe «Anwendungsprotokoll» stammte aus einer Drittquelle. Die Aktion «E-Mail senden» im Aufgabenplaner ist in neueren Windows-Versionen eingeschränkt oder veraltet [W], prüfe das in deiner VM. Als Alternative eignet sich ein PowerShell-Script (Abschnitt 2f) oder PRTG.

---

## 2l. Patch-Management Checkliste (Best Practice, nicht Schweizer Vorgabe)

Quelle: Bayerisches Landesamt für Datenschutzaufsicht, «Patch Management im Alltag», Checkliste, Stand 22.07.2020, https://lda.bayern.de/checkliste_patch_mgmt (PDF, 3 Seiten). Sie bezieht sich auf die EU-DSGVO und ist für KMU gedacht. In der Schweiz gilt das Datenschutzgesetz der Schweiz, der fachliche Ablauf ist aber derselbe [W].

**Zyklus:** Planen, Beschaffen, Testen, Installieren, Dokumentieren.
- **Planen:** Ein Plan ist vorhanden (welche Hard- und Software, wer ist zuständig, welche Prozesse).
- **Beschaffen:** Informationen zu Updates gezielt suchen, Sicherheitshinweise ernst nehmen, nur seriöse Quellen nutzen.
- **Testen:** Patches für kritische Anwendungen nicht ungetestet produktiv einspielen. Zuerst Testumgebung, nach dem Patch den Betrieb auf Funktionalität prüfen.
- **Installieren:** kontrolliert, um unnötige Ausfallzeiten zu vermeiden.
- **Dokumentieren:** Änderungen bleiben nachvollziehbar und können bei Bedarf nachjustiert werden.

**Organisation:** Zuständigkeiten definieren (wer patcht was, wer recherchiert Schwachstellen), Zeitaufwand und Ressourcen einplanen, auch für Notfall-Patches.

**Voraussetzungen:** Systemlandschaft dokumentiert (Netzpläne), Inventarliste aller Hard- und Software, Konzept mit Update-Plan und Versionsständen, Regeln zur Priorisierung nach Risiko (zum Beispiel Einstufung durch Hersteller oder CERT-Bund), Strukturen für fehlerhafte Updates (Ausfallsicherheit, Redundanz).

**Bezugsquellen:** Nicht auf Fake-Patches hereinfallen. Integrität und Authentizität von Paketen mit Prüfsumme oder digitaler Signatur prüfen. Verträge mit Anbietern ansehen (Zyklus der Updates, auslaufende Supportverträge).

**Hilfsmittel zur Bewertung** (in der Checkliste genannt): CERT-Bund (cert-bund.de), CVSS (nvd.nist.gov/vuln-metrics/cvss), CVE-Details (cvedetails.com). Der Bildungsplan nennt für Server ausdrücklich MELANI und CVE als Quellen zur Information über Sicherheitsentwicklungen (f6). [Q: Bildungsplan, Modulbaukasten Kompetenzbeschreibung f6]

**Verweis:** BSI IT-Grundschutz-Kompendium, Baustein OPS.1.1.3 Patch- und Änderungsmanagement (den Baustein selbst habe ich nicht gelesen). Der IKT-Minimalstandard des Bundesamts für wirtschaftliche Landesversorgung (August 2018, Empfehlung) ist ein Schweizer Massnahmenkatalog [Q: PwC Schweiz]. Den Katalog habe ich nicht im Original gefunden.

---

## 2m. PRTG für die eigene Übung

Quellen: https://www.paessler.com/manuals/prtg/system_requirements, https://www.paessler.com/de/pricing, https://www.paessler.com/monitoring/network/how-to-free-network-monitoring (Herstellerangaben, vor dem Download die aktuellen Bedingungen prüfen).
- **Lizenz:** Die Freeware-Edition gilt für bis zu 100 Sensoren kostenlos. Die Testversion ist 30 Tage voll nutzbar und wechselt danach in die Freeware-Edition.
- **Betriebssystem:** unterstützt sind Windows Server 2016 bis 2025 sowie Windows 10 und 11. Server Core und Minimal Server Interface werden nicht offiziell unterstützt.
- **Voraussetzungen:** .NET Framework 4.7.2 oder neuer (für neue Installationen 4.8 empfohlen), FIPS-Modus muss deaktiviert sein.
- **Hardware:** richtet sich nach der Sensoranzahl. Bis 500 Sensoren nennt das Handbuch 4 CPU-Kerne, 4 GB RAM und 100 GB Speicher. Für die Übung in einer VM reicht deutlich weniger.
- **SNMP:** Zielgeräte müssen SNMP v1, v2c oder v3 unterstützen, SNMP muss auf dem Gerät aktiviert sein und der PRTG-Server braucht Zugriff. SNMP v3 ist wegen der Verschlüsselung langsamer, bei sehr vielen Sensoren empfiehlt Paessler v1 oder v2c.
- **WMI:** siehe https://www.paessler.com/manuals/prtg/monitoring_via_wmi (Windows-Zugangsdaten und Firewall müssen stimmen).

Hinweis für die Praxis [W]: Installiere PRTG nicht auf dem Server, den du überwachst, sondern auf einer eigenen VM (`MON01` in der Probeprüfung). So testest du auch den Netzwerkzugriff und die Firewall-Regel.

---

## 2n. Lab zu Hause aufbauen (Tag 4 im Plan)

Quellen: https://learn.microsoft.com/de-de/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v und die Cmdlet-Dokus New-VM und Checkpoint-VM.

**Hyper-V aktivieren** [Q]
- Windows 10 oder 11 in der Edition **Pro oder Enterprise** wird gebraucht. Unter Windows 10/11 **Home** lässt sich die Hyper-V-Rolle nicht installieren.
- Per PowerShell: `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All`. Alternativ per DISM: `DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V`.
- Achtung: Andere Virtualisierungs-Apps wie VMware Workstation oder VirtualBox nutzen dieselben Prozessorfunktionen. Laufen sie zusammen mit Hyper-V, können VMs nicht starten oder unzuverlässig laufen. Wähle also eine Lösung.
- Auf einem Windows Server installierst du die Rolle mit dem Server-Manager oder `Install-WindowsFeature`.

**Windows Server zum Testen** [Q]
- Microsoft verweist in der Hyper-V-Doku für Windows Server 2025 auf das Evaluation Center: https://www.microsoft.com/evalcenter/evaluate-windows-server-2025 (die Seite war für mich gerade nicht abrufbar, wegen hoher Last). Eine Evaluierungskopie ist zeitlich begrenzt gültig [W].

**VM per PowerShell** [Q: New-VM-Doku]
```powershell
New-VM -Name "SRV01" -MemoryStartupBytes 2GB -NewVHDPath D:\vhd\srv01.vhdx -NewVHDSizeBytes 40GB
```
Mehr Arbeitsspeicher nach Bedarf. Zwei VMs (Server und Client) in ein internes virtuelles Netz hängen und dem Server eine feste IP geben [W].

**Snapshot vor jeder Änderung** [Q: Checkpoint-VM-Doku]
```powershell
Checkpoint-VM -Name SRV01 -SnapshotName BeforeInstallingUpdates
```
Das passt zum Patch-Prozess (Backup und Rollback-Plan) und lässt dich Fehler gefahrlos ausprobieren.

**Empfohlene Mindestgrösse** [W]: Host mit 16 GB RAM, damit Server (2 bis 4 GB), Client (4 GB) und Monitoring-VM (2 bis 4 GB) parallel laufen. Mit 8 GB geht es, wenn du die VMs nacheinander startest.

---

## 2o. Fernverwaltung und Zeitsynchronisation

**PowerShell-Remoting** [Q: Microsoft Learn, Enable-PSRemoting, Invoke-Command, Enter-PSSession]
- Remoting nutzt WS-Management. Auf Windows-Server-Plattformen ist es standardmässig aktiviert. Auf Clients aktivierst du es einmalig auf dem Computer, der Befehle empfangen soll, mit `Enable-PSRemoting`. Der Befehl startet Listener, darum nur bei Bedarf ausführen. Auf Computern, die nur Befehle senden, ist das nicht nötig.
- Auf Clients ist die Aktivierung in einem öffentlichen Netzwerk normalerweise nicht erlaubt. `-SkipNetworkProfileCheck` umgeht das, ist aber ein Sicherheitsrisiko [W].
- Interaktive Sitzung: `Enter-PSSession -ComputerName Server01 -Credential Domain01\User01`. Danach steht im Prompt `[Server01]: PS>`.
- Einzelner Befehl auf einem oder mehreren Servern: `Invoke-Command -ComputerName Server01 -ScriptBlock { Get-Service DHCPServer }`.
- Sitzung wiederverwenden: `$s = New-PSSession -ComputerName Server02`, dann `Invoke-Command -Session $s -ScriptBlock { ... }`.
- Ports und Firewall [Q: Microsoft Learn]: TCP 5985 (HTTP) und 5986 (HTTPS). In privaten Netzen akzeptiert die Standard-Firewallregel für PowerShell-Remoting alle Verbindungen, in öffentlichen Netzen nur aus demselben Subnetz. WinRM verschlüsselt die Remoting-Kommunikation nach der Authentifizierung immer, auch über HTTP. HTTPS braucht ein Serverzertifikat, dessen CN zum Hostnamen passt.
- Für Monitoring ist das praktisch: ein Script prüft mehrere Server hintereinander [W].

**Zeitsynchronisation** [Q: https://learn.microsoft.com/de-de/windows-server/networking/windows-time-service/windows-time-service-top]
- Der Windows-Zeitdienst (W32Time) synchronisiert Datum und Uhrzeit für alle Computer in AD DS per Netzwerkzeitprotokoll (NTP). Genaue Zeit ist für viele Windows-Dienste kritisch.
- Ab Windows Server 2016 ist eine Genauigkeit von 1 ms zur UTC möglich, unter geeigneten Bedingungen für Domänenmitglieder von Windows Server 2016 und Windows 10 Anniversary Update.
- Kerberos toleriert nur eine begrenzte Zeitabweichung zwischen Client und Domänencontroller. Die Richtlinie «Maximale Toleranz für Computeruhrsynchronisierung» steht laut Microsoft standardmässig auf 5 Minuten (Registry SkewTime). Kerberos nutzt Zeitstempel gegen Replay-Angriffe, bei grösserer Abweichung scheitert die Authentifizierung (Fehler KRB_AP_ERR_SKEW). [Q: https://learn.microsoft.com/de-de/windows/security/threat-protection/security-policy-settings/maximum-tolerance-for-computer-clock-synchronization] Bei Anmeldeproblemen in einer frischen Lab-Domäne zuerst die Uhrzeit prüfen, besonders bei VMs, die aus einem Snapshot zurückkehren [W].
- Neu synchronisieren: `w32tm /resync /computer:<Ziel> /rediscover` [Q: Microsoft, Kerberos-Fehlerbehebung]. Status prüfen mit `w32tm /query /status` [W].

---

## 3. 14-Tage-Plan (je ca. 45 bis 60 Minuten)

Voraussetzung: Ein Laptop mit Hyper-V oder VirtualBox und eine Windows-Server-Testversion (Evaluation). Falls das nicht geht, nur die Theorie-Teile und Befehle lesen.

| Tag | Thema | Konkret |
| --- | --- | --- |
| 1 | Modul verstehen | `UEK-188-Vorbereitung.md` Abschnitte 1, 2 und 2a lesen. Fragen fürs Sekretariat abschicken. Kursprogramm suchen. |
| 2 | Vorwissen 117 | Selbsttest und Subnetting: 10 Aufgaben (z.B. /24 bis /30), Lernkarten 44 bis 46. |
| 3 | Vorwissen 123 | DHCP, DNS, File, Print erklären können. Lernkarten 45 bis 47. |
| 4 | Lab aufbauen | VM mit Windows Server, zweite VM als Client, internes Netz, feste Server-IP. Siehe Abschnitt 2n. |
| 5 | AD und DNS | `Install-ADDSForest`, Benutzer und Gruppen anlegen, Client in die Domäne. Lernkarten 31, 32, 57 und 58, Abschnitt 2i. |
| 6 | DHCP | Scope, Optionen, Reservation, Client mit `ipconfig /release` und `/renew` testen. |
| 7 | Rechte | Freigabe und NTFS: eine Freigabe, zwei Gruppen, Zugriff testen. AGDLP umsetzen. Lernkarten 33 bis 39 und 61 bis 63, Abschnitt 2i. |
| 8 | Betriebsdoku | Kurze Doku zum Lab schreiben (Netzplan, Server, Dienste, Rechte). Lernkarten 1 bis 5. |
| 9 | Bordmittel-Monitoring | `Get-Counter`, Ressourcenmonitor, Ereignisanzeige. Eine Baseline notieren. Lernkarten 6 bis 13, Abschnitte 2e und 2k. |
| 10 | SNMP und PRTG | SNMP-Dienst installieren, PRTG (Gratisversion) einrichten, drei Sensoren, Grenzwerte, Benachrichtigung, Testalarm, einen Report und eine Map anlegen. Lernkarten 20 bis 30 und 71 bis 72, Abschnitte 2d und 2m. |
| 11 | Dienst-Ausfall | Einen Dienst stoppen, Alarm prüfen, Ereignis 7036 finden, Dienst wieder starten, alles dokumentieren. Optional das Script aus Abschnitt 2f einrichten. |
| 12 | Patches | `Get-HotFix`, Patch-Prozess durchspielen (Snapshot, Update, Kontrolle, Doku). Lernkarten 14 bis 19 und 64 bis 67, Abschnitt 2l. |
| 13 | Umsysteme und Backup | Firewall-Regel für SNMP nur von der PRTG-IP, Änderungsauftrag schreiben, Backup und Restore testen. Lernkarten 40 bis 43, 49, 50. |
| 14 | Generalprobe | Alle Lernkarten durchgehen, besonders die Berechtigungen (68 bis 70). Ein Mini-Szenario aus dem Kopf: Service aufsetzen, überwachen, patchen, Rechte vergeben, dokumentieren. 1 Stunde Zeit, so wie die Praxisprüfung (2 Stunden) plus schriftlich (1 Stunde). |

Hinweis zur Prüfung: Laut Leistungsbeurteilung zählt der Praxisteil 60 Prozent und der schriftliche Teil 40 Prozent. Übe darum auch das Begründen (warum dieser Schwellwert, warum diese Rechtevergabe).
