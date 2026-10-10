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
Das Feature «Windows Server Backup» muss dafür installiert sein [W]. Danach immer einen Restore-Test machen, damit du weisst, dass es funktioniert (NCSC-Empfehlung).

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

## 2. Lernkarten (Frage, Antwort), 56 Stück

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

### Zusatzkarten: Print, GPO, Backup
51. **In welcher Reihenfolge richtest du einen Netzwerkdrucker auf dem Druckserver ein?** Treiber (`Add-PrinterDriver`), Port (`Add-PrinterPort`), Drucker (`Add-Printer`). [Q: Microsoft Learn]
52. **Wie verbindet sich ein Client mit dem freigegebenen Drucker?** `Add-Printer -ConnectionName \\printServer\printerName`. [Q]
53. **Der Drucker druckt nicht mehr, Warteschlange hängt. Erste Schritte?** `Get-PrintJob` ansehen, hängenden Auftrag entfernen, Dienst Druckwarteschlange neu starten. [W]
54. **Wie wendest du eine Gruppenrichtlinie sofort an und wie prüfst du, was gilt?** `gpupdate /force` und `gpresult /r`. [Q]
55. **Wie verknüpfst du ein GPO mit einer OU?** `New-GPO -Name "MyGPO" | New-GPLink -Target "ou=MyOU,dc=contoso,dc=com" -LinkEnabled Yes`. [Q]
56. **Wie sicherst und stellst du Dateien mit Windows Server Backup wieder her?** `wbadmin start backup ...` und `wbadmin start recovery -itemType:File ...`. Danach den Restore testen. [Q: wbadmin-Doku, NCSC]

## 2b. Falls die Learnfactory Linux nimmt: Gegenstücke (Ubuntu Server)

Das Modul legt sich nicht auf ein Betriebssystem fest. Hier die Linux-Entsprechungen, damit du in beiden Fällen vorbereitet bist.

| Aufgabe | Windows | Linux (Ubuntu) |
| --- | --- | --- |
| Dienst prüfen | `Get-Service` | `systemctl status <dienst>`, `systemctl is-active <dienst>`, `systemctl list-units --failed` [Q: Ubuntu man systemctl] |
| Dienst steuern | `Start-Service`, `Restart-Service` | `systemctl start/stop/restart/enable <dienst>` [Q] |
| Logs lesen | Ereignisanzeige, `Get-WinEvent` | `journalctl -u <dienst>`, `--since`, `-b` (seit Boot) [Q: Ubuntu man journalctl] |
| Updates | WSUS, Windows Update | `apt update`, `apt upgrade`, automatisch mit `unattended-upgrades` [Q: Ubuntu Server Docs] |
| Berechtigungen | NTFS, `icacls` | `chmod`, `chown`, ACLs mit `setfacl -m` und `getfacl` [Q: Ubuntu man setfacl, W] |
| Firewall | `New-NetFirewallRule` | `ufw`, z.B. `sudo ufw allow from <IP> to any port <Port>` [Q: Ubuntu Server Docs beschreiben «Allow access from specific hosts», Syntax aus meinem Wissen] |
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
- Konkrete Ports (zum Beispiel 135 und dynamische RPC-Ports bei DCOM) habe ich nicht aus einer Quelle belegt [W]. In der Praxis lieber CIM über WSMan (PowerShell-Remoting, Port 5985) nutzen, wenn das Monitoring-Tool es kann [W].

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
