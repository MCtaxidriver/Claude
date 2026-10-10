# ÜK 188: Lernhilfen (Cheatsheet, Lernkarten, 14-Tage-Plan)

Ergänzung zu `UEK-188-Vorbereitung.md`. Markierung bei den Quellen:
- [Q] belegt in einer genannten Quelle
- [W] aus allgemeinem Fachwissen, bitte mit den Kursunterlagen abgleichen

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

## 2. Lernkarten (Frage, Antwort)

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
12. **Event-ID 7036 und 7034, was heisst das?** 7036: Dienst gestartet oder gestoppt. 7034: Dienst unerwartet beendet. [Q: Drittquellen]
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
25. **Auf welchem Port läuft SNMP?** UDP 161 (Abfragen). [W]
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
49. **Welche Backup-Arten gibt es?** Voll, differentiell (alles seit dem letzten Voll-Backup), inkrementell (seit dem letzten Backup). [W]
50. **Was rät das NCSC beim Backup?** Regelmässig sichern, externe Medien danach trennen, Wiederherstellung testen. [Q: Netzwoche]

---

## 3. 14-Tage-Plan (je ca. 45 bis 60 Minuten)

Voraussetzung: Ein Laptop mit Hyper-V oder VirtualBox und eine Windows-Server-Testversion (Evaluation). Falls das nicht geht, nur die Theorie-Teile und Befehle lesen.

| Tag | Thema | Konkret |
| --- | --- | --- |
| 1 | Modul verstehen | `UEK-188-Vorbereitung.md` Abschnitte 1, 2 und 2a lesen. Fragen fürs Sekretariat abschicken. Kursprogramm suchen. |
| 2 | Vorwissen 117 | Selbsttest und Subnetting: 10 Aufgaben (z.B. /24 bis /30), Lernkarten 44 bis 46. |
| 3 | Vorwissen 123 | DHCP, DNS, File, Print erklären können. Lernkarten 45 bis 47. |
| 4 | Lab aufbauen | VM mit Windows Server, zweite VM als Client, internes Netz, feste Server-IP. |
| 5 | AD und DNS | `Install-ADDSForest`, Benutzer und Gruppen anlegen, Client in die Domäne. Lernkarten 31 und 32. |
| 6 | DHCP | Scope, Optionen, Reservation, Client mit `ipconfig /release` und `/renew` testen. |
| 7 | Rechte | Freigabe und NTFS: eine Freigabe, zwei Gruppen, Zugriff testen. AGDLP umsetzen. Lernkarten 33 bis 39. |
| 8 | Betriebsdoku | Kurze Doku zum Lab schreiben (Netzplan, Server, Dienste, Rechte). Lernkarten 1 bis 5. |
| 9 | Bordmittel-Monitoring | `Get-Counter`, Ressourcenmonitor, Ereignisanzeige. Eine Baseline notieren. Lernkarten 6 bis 13. |
| 10 | SNMP und PRTG | SNMP-Dienst installieren, PRTG (Gratisversion) einrichten, drei Sensoren, Grenzwerte, Benachrichtigung, Testalarm. Lernkarten 20 bis 30. |
| 11 | Dienst-Ausfall | Einen Dienst stoppen, Alarm prüfen, Ereignis 7036 finden, Dienst wieder starten, alles dokumentieren. |
| 12 | Patches | `Get-HotFix`, Patch-Prozess durchspielen (Snapshot, Update, Kontrolle, Doku). Lernkarten 14 bis 19. |
| 13 | Umsysteme und Backup | Firewall-Regel für SNMP nur von der PRTG-IP, Änderungsauftrag schreiben, Backup und Restore testen. Lernkarten 40 bis 43, 49, 50. |
| 14 | Generalprobe | Alle Lernkarten durchgehen. Ein Mini-Szenario aus dem Kopf: Service aufsetzen, überwachen, patchen, Rechte vergeben, dokumentieren. 1 Stunde Zeit, so wie die Praxisprüfung (2 Stunden) plus schriftlich (1 Stunde). |

Hinweis zur Prüfung: Laut Leistungsbeurteilung zählt der Praxisteil 60 Prozent und der schriftliche Teil 40 Prozent. Übe darum auch das Begründen (warum dieser Schwellwert, warum diese Rechtevergabe).
