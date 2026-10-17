# ÜK 188: Lernhilfen (Cheatsheet, Lernkarten, 14-Tage-Plan)

Ergänzung zu `UEK-188-Vorbereitung.md`. Markierung bei den Quellen:
- [Q] belegt in einer genannten Quelle
- [W] aus allgemeinem Fachwissen, bitte mit den Kursunterlagen abgleichen





<!-- toc -->
**Inhalt**

- 1. Cheatsheet: Befehle für die Lab-Umgebung
- 2. Lernkarten (Frage, Antwort), 119 Stück
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
- 2p. Muster: Berechtigungskonzept dokumentieren und begründen
- 2q. Muster: IP-Adress- und Namenskonzept (für AD, DNS, DHCP)
- 2r. Muster: Betriebsdokumentation der Muster AG (zum Üben des Nachführens)
- 2s. Ergänzungen aus dem Script von ICT-BZ (Bulk-User, GPO, FSRM, Hashwerte, Zabbix, MIB)
- 2t. Lab-Ablauf: AD DS, DNS und DHCP «gemäss Konzept» (Anleitung der älteren Lernenden)
- 2u. So ist eine echte Betriebsdokumentation aufgebaut (Musterdoku der älteren Lernenden)
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
Weitere AD-Verwaltung, alles aus der Microsoft-Doku belegt [Q: ActiveDirectory-Modul und Add-Computer]:
```powershell
New-ADOrganizationalUnit -Name "UserAccounts" -Path "DC=FABRIKAM,DC=COM"                       # OU anlegen
New-ADOrganizationalUnit -Name "UserAccounts" -Path "DC=FABRIKAM,DC=COM" -ProtectedFromAccidentalDeletion $False
Get-ADUser -Filter * -SearchBase "OU=Finance,OU=UserAccounts,DC=FABRIKAM,DC=COM"                # Benutzer einer OU
Move-ADObject -Identity "OU=ManagedGroups,DC=Fabrikam,DC=Com" -TargetPath "OU=Managed,DC=Fabrikam,DC=Com"   # Objekt verschieben
Unlock-ADAccount -Identity PattiFu                                                             # gesperrtes Konto entsperren
Disable-ADAccount -Identity PattiFul                                                           # Konto deaktivieren
Set-ADAccountPassword -Identity 'CN=Elisa Daugherty,OU=Accounts,DC=Fabrikam,DC=com' -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "p@ssw0rd" -Force)   # Passwort zurücksetzen
Add-Computer -DomainName Domain01 -Restart                                                     # Client in die Domäne aufnehmen
```
Das Passwort im Beispiel ist ein Platzhalter der Microsoft-Doku, in der Praxis nie im Klartext in Scripts schreiben, sondern `Read-Host -AsSecureString` verwenden [W]. Mit `-ProtectedFromAccidentalDeletion` schützt du ein Objekt vor dem Löschen. Ist der Schutz gesetzt, kannst du das Objekt erst nach Ändern der Eigenschaft löschen [Q: Microsoft Learn]. Dass neue OUs standardmässig geschützt sind, steht nicht auf der Doku-Seite, ich kenne es aus der Praxis [W]. Das erklärt Fehler beim Löschen einer OU.

Gruppenmitgliedschaften prüfen und ändern [Q: ActiveDirectory-Modul]:
```powershell
Get-ADGroupMember -Identity GG_Buchhaltung                                   # Mitglieder einer Gruppe
Get-ADPrincipalGroupMembership -Identity skeller                             # Gruppen eines Benutzers
Remove-ADGroupMember -Identity "DocumentReaders" -Members administrator,DavidChew   # Mitglieder entfernen
```
Die ersten zwei Befehle brauchst du für den Nachweis in der Betriebsdokumentation und für den Test eines Berechtigungskonzepts: Ist `skeller` wirklich in `GG_Buchhaltung`? Hinweis: Die Microsoft-Seite zu `Get-ADGroupMember` zeigte in meinem Abruf keine Beispiele, der Aufruf mit `-Identity` ist aus meinem Wissen [W].

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
# Ausschlussbereich: Adressen, die nie vergeben werden [Q: Add-DhcpServerv4ExclusionRange]
Add-DhcpServerv4ExclusionRange -ScopeId 10.1.1.0 -StartRange 10.1.1.1 -EndRange 10.1.1.10
# Aktuelle Leases eines Scopes ansehen [Q: Get-DhcpServerv4Lease]
Get-DhcpServerv4Lease -ScopeId 10.10.10.0
# Aus Lease eine Reservation machen [Q]
Get-DhcpServerv4Lease -IPAddress 10.10.10.11 | Add-DhcpServerv4Reservation
```
**DHCP-Server im Active Directory autorisieren** [Q: Microsoft Learn, Add-DhcpServerInDC]: Ein DHCP-Dienst auf einem domänenverbundenen Computer muss im AD autorisiert sein, bevor er Adressen vergeben darf.
```powershell
Add-DhcpServerInDC -DnsName "dhcpserver.contoso.com" -IPAddress 10.10.10.2   # autorisieren
Add-DhcpServerInDC                                                           # lokalen Server autorisieren
Get-DhcpServerInDC                                                           # autorisierte Server anzeigen
```
Ohne Parameter wird der lokale Server eingetragen, und der Befehl löst auch eine Autorisierungsprüfung im DHCP-Dienst aus. Wenn dein Scope aktiv ist und Clients trotzdem keine Adresse bekommen, ist die fehlende Autorisierung ein typischer Grund [W].

Client testen [Q: Microsoft Learn, ipconfig]: `ipconfig /release` gibt die DHCP-Adresse frei (bei Adaptern mit automatischer Adresse, TCP/IP wird für diesen Adapter vorübergehend deaktiviert), `ipconfig /renew` holt eine neue. Beides wirkt nur bei Adaptern mit automatischer IP-Konfiguration und leert den DNS-Cache nicht. Dafür gibt es `ipconfig /flushdns`, anzeigen mit `ipconfig /displaydns`. Mit `ipconfig /all` kontrollierst du die Konfiguration.

### DNS
```powershell
# A-Record mit PTR [Q: Add-DnsServerResourceRecordA]
Add-DnsServerResourceRecordA -Name "host23" -ZoneName "contoso.com" -IPv4Address "172.18.99.23" -TimeToLive 01:00:00 -CreatePtr
```
Weitere DNS-Befehle [Q: Microsoft Learn, DnsServer-Modul]:
```powershell
Add-DnsServerPrimaryZone -Name "west01.contoso.com" -ReplicationScope "Forest" -PassThru   # AD-integrierte Zone
Add-DnsServerPrimaryZone -Name "west02.contoso.com" -ZoneFile "west02.contoso.com.dns"      # Zone mit Datei
Add-DnsServerForwarder -IPAddress 172.23.90.124 -PassThru                                    # Weiterleitung
Get-DnsServerResourceRecord -ZoneName "contoso.com"                                          # alle Einträge einer Zone
Get-DnsServerResourceRecord -ZoneName "contoso.com" -Name "Admin01"                          # ein Eintrag
Get-DnsServerZone                                                                            # alle Zonen des Servers
Remove-DnsServerResourceRecord -ZoneName "contoso.com" -RRType "A" -Name "Host01"            # Eintrag löschen, fragt nach
Clear-DnsServerCache -Force                                                                  # Cache des DNS-Servers leeren
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
Vorhandene Sicherungen anzeigen [Q: Microsoft Learn, wbadmin get versions]: `wbadmin get versions -backupTarget:H:` oder für eine Netzfreigabe `wbadmin get versions -backupTarget:\\servername\share -machine:server01`. Die Ausgabe nennt Zeit, Speicherort, Versionskennung und die möglichen Wiederherstellungsarten. Die Versionskennung brauchst du für `wbadmin start recovery -version:...`.

Täglichen Zeitplan anlegen [Q: Microsoft Learn, wbadmin enable backup]:
```cmd
wbadmin enable backup -addtarget:H: -include:T:,D:\documents -exclude:D:\documents\~tmp -vssfull -schedule:01:00
wbadmin enable backup
```
Der erste Befehl legt einen täglichen Zeitplan um 01:00 an (mehrere Zeiten mit Komma, zum Beispiel `-schedule:09:00,18:00`), der zweite zeigt ohne Parameter die aktuellen Einstellungen. Du brauchst dafür Rechte als Mitglied der Gruppe Sicherungsoperatoren oder Administratoren und eine Eingabeaufforderung als Administrator. Die Laufwerks-ID für `-addtarget` findest du mit `wbadmin get disks`. Das Beispiel der Microsoft-Seite übergibt ein Passwort im Klartext, das solltest du in der Praxis nicht tun [W].
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
# Regeln ansehen und ändern [Q: Get-NetFirewallRule und Set-NetFirewallRule]
Get-NetFirewallRule -PolicyStore ActiveStore                                    # wirksame Regeln
Get-NetFirewallProfile -Name Public | Get-NetFirewallRule                       # Regeln eines Profils
Set-NetFirewallRule -DisplayName "AllowWeb80" -RemoteAddress "192.168.0.2"      # Quelle einer Regel ändern
Enable-NetFirewallRule -DisplayGroup "Windows Firewall Remote Management"       # vordefinierte Gruppe aktivieren
```

### Windows Server 2025 beachten [Q: Microsoft Learn, entfernte Features]
WMIC nur noch als Feature on Demand, PowerShell 2.0 nicht mehr enthalten, WSUS nicht mehr aktiv entwickelt. Ältere Anleitungen mit `wmic` oder `wuauclt /detectnow` funktionieren nicht mehr wie früher.

---

## 2. Lernkarten (Frage, Antwort), 119 Stück

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
39. **Was ist das Least-Privilege-Prinzip?** Jeder bekommt nur die Rechte, die er für die Aufgabe braucht. Microsoft begründet das so: Läuft Schadsoftware unter einem privilegierten Konto, kann sie die ganze Domäne erreichen, unter einem normalen Konto nur den lokalen Rechner. Admin-Konten nicht für den Alltag nutzen, Rechte über Delegation und Rollen vergeben. [Q: Microsoft Learn, Verwaltungsmodelle der geringsten Berechtigungen]

### Handlungsziel 6: Umsysteme
40. **Nenne zwei typische Anpassungen auf Umsystemen.** DNS-Einträge und Firewall-Regeln. [Q: Modulbeschreibung 6.1]
41. **Was muss ein Änderungsauftrag an Dritte enthalten?** Quelle, Ziel, Protokoll und Port, Zweck, Befristung, Testverfahren. [W, siehe Beispiel in der Vorbereitungsdatei]
42. **Wie testest du eine Firewall-Anpassung?** Von der Quelle aus verbinden, zum Beispiel `Test-NetConnection -ComputerName SRV01 -Port 443`. Entscheidend ist `TcpTestSucceeded`: True heisst Verbindung steht, False deutet auf ein Verbindungsproblem hin. Das Cmdlet testet TCP, für UDP-Ports wie SNMP (161) taugt es nicht, dort hilft `snmpwalk` oder das Monitoring-Tool. [Q: Microsoft Learn, Test-NetConnection, UDP-Hinweis aus meinem Wissen]
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

### Zusatzkarten: üK-Reglement (75 bis 78)
75. **Wann musst du einen üK wiederholen, und wie meldest du eine Absenz?** Bei mehr als 20 Prozent Absenz, bei 5 Tagen also mehr als einem Tag, ist der üK oder sind die Tage ab dem Fehlen unter Kostenfolge nachzuholen. Unverschuldete Gründe (ärztlich bescheinigte Krankheit oder Unfall, Militärdienst) sofort schriftlich dem Bildungsanbieter melden. Kostenlos nachholen geht nur bei nachgewiesenem Nichtverschulden. [Q: üK-Reglement Art. 7]
76. **Was zählt als Unredlichkeit und was droht?** Fälschen, nicht deklarierte Hilfe Dritter, Übernehmen fremder Texte, Konfigurationen oder Codes ohne Quelle, Selbstplagiat, andere Täuschung. Du hast 5 Arbeitstage für eine schriftliche Stellungnahme, bei Unredlichkeit gilt Note 1 für den betroffenen Teil. [Q: üK-Reglement Art. 12]
77. **Welche Gründe gelten bei Fernbleiben von einer Leistungsbeurteilung?** Ärztlich attestierte Krankheit oder Unfall, Todesfall in der Familie mit Bestätigung, Verspätung im öffentlichen Verkehr mit Bestätigung des Transportanbieters, rechtlich gültige Verfügung (Militär, Zivilschutz, Gericht, Polizei). Ohne schriftlichen Nachweis gilt die LB als nicht besucht. [Q: üK-Reglement Art. 13]
78. **Wie wird die Kompetenz eines Moduls ausgewiesen?** Mit einer halben oder ganzen Note (Kompetenznachweis), geprüft anhand einer Leistungsbeurteilung nach LBV. [Q: üK-Reglement Art. 9 und 10]

### Zusatzkarten: Script ICT-BZ (79 bis 98)
79. **Was ist der Unterschied zwischen SNMP-Polling und SNMP-Traps?** Beim Polling fragt das Monitoring aktiv ab (UDP 161). Traps schickt das Gerät spontan bei einem Ereignis (UDP 162). Traps sind unzuverlässig (keine Empfangsbestätigung), melden keine Recovery und können das Monitoring fluten. [Q: Script ICT-BZ, Seite SNMP]
80. **Welche SNMP-Version ist in der Praxis üblich und warum?** v2c, weil es Bulk-Abfragen und 64-Bit-Counter kann. Sicherheitstechnisch ist es wie v1, die Community geht im Klartext über das Netz. v3 verschlüsselt, ist aber aufwendiger und braucht mehr Rechenleistung. [Q: Script ICT-BZ]
81. **Was sind OID und MIB?** Eine OID ist die Zahlenfolge im Baum, die ein Objekt auf dem Gerät eindeutig benennt. Die MIB übersetzt die Zahlen in Namen, Typen und Beschreibungen, oft herstellerspezifisch. [Q: Script ICT-BZ]
82. **Warum heisst SNMP-Monitoring «agentenlos», obwohl es einen SNMP-Agenten gibt?** Der SNMP-Agent ist vom Hersteller schon eingebaut. Du installierst keine zusätzliche Monitoring-Software auf dem Gerät. [Q: Script ICT-BZ]
83. **Wann nimmst du WMI und wann SNMP?** WMI nur für Windows-Systeme, es ist vorinstalliert und liefert tiefe Betriebssystemdaten. SNMP funktioniert herstellerübergreifend, vor allem für Netzwerkgeräte, Drucker und USVs, liefert bei Servern aber meist nur Hardwaredaten. [Q: Script ICT-BZ, Seiten WMI und SNMP]
84. **Was ist WMI technisch und was ersetzt `Get-CimInstance`?** WMI ist die Microsoft-Umsetzung von WBEM auf Basis von CIM. `Get-CimInstance` löst `Get-WmiObject` ab und nutzt WinRM statt DCOM, das ist firewallfreundlicher. [Q: Script ICT-BZ]
85. **Wann ist ein Agent besser als SNMP?** Wenn du Betriebssystem und Anwendungen tief überwachen willst. Der Agent liefert mehr Details und braucht weniger Ressourcen, muss aber installiert und gepflegt werden. [Q: Script ICT-BZ]
86. **Nenne die fünf Phasen eines Monitoring-Störungsfalls.** Erkennung und Alarmierung, Klassifizierung und Priorisierung, Analyse und Diagnose, Behebung und Wiederherstellung, Abschluss und Nachbereitung. [Q: Script ICT-BZ, Ablauf Monitoring Case]
87. **Was ist der Unterschied zwischen Fix und Workaround?** Ein Fix behebt die Ursache. Ein Workaround entschärft das Problem nur kurzfristig, die Ursache wird später behoben. [Q: Script ICT-BZ]
88. **Wozu dient «Acknowledge» im Monitoring?** Es zeigt anderen, dass jemand das Problem bearbeitet, mit Kommentar. So laufen keine parallelen Analysen. In Zabbix lässt sich der Status auch für Eskalationen nutzen. «Suppress» blendet ein Problem aus. [Q: Script ICT-BZ, Zabbix-Doku]
89. **Wie prüfst du den Hashwert einer heruntergeladenen Datei unter Windows?** `certutil -hashfile Datei SHA256` oder `Get-FileHash Datei -Algorithm SHA256`, dann mit dem Wert der Herstellerseite vergleichen. Bei Abweichung nicht installieren, neu laden und die Quelle prüfen. [Q: Script ICT-BZ für certutil, W für Get-FileHash]
90. **Beweist ein passender Hashwert, dass eine Datei harmlos ist?** Nein. Er zeigt nur, dass die Datei dem Original entspricht und nicht verändert wurde. Die Quelle muss vertrauenswürdig sein. [Q: Script ICT-BZ]
91. **Wie verhinderst du Musik- und Videodateien auf einem Fileserver?** Mit dem Feature File Server Resource Manager (FSRM) eine aktive Dateiprüfung (File Screen) auf den Ordner legen, mit der Dateigruppe «Audio and Video Files». Bereits gespeicherte Dateien bleiben zugänglich. [Q: Microsoft Learn New-FsrmFileScreen, Gruppenname W]
92. **Wie erzeugst du einen HTML-Bericht über die gespeicherten Dateitypen?** FSRM-Speicherbericht vom Typ `FilesByFileGroup` im Format HTML, zum Beispiel mit `New-FsrmStorageReport`. [Q: Microsoft Learn New-FsrmStorageReport]
93. **Was unterscheidet Ressourcenmonitor und Systemmonitor?** Der Ressourcenmonitor (`resmon`) zeigt live, welcher Prozess CPU, RAM, Datenträger und Netzwerk nutzt. Der Systemmonitor (`perfmon`) misst Leistungsindikatoren und zeichnet sie über die Zeit auf (Datensammlersätze, Alarme). [W]
94. **Wie testest du die Aufzeichnung für das Wachstum eines Datenlaufwerks?** Mit Dummy-Dateien: `fsutil file createnew D:\Test\dummy.bin 1073741824` legt 1 GiB an. Dann prüfen, ob der Verlauf in der Aufzeichnung sichtbar ist. [W, das Script nennt fsutil]
95. **Wo stellst du Kennwortalter, Chronik und Mindestlänge für die Domäne ein?** In der Default Domain Policy unter Computerkonfiguration > Richtlinien > Windows-Einstellungen > Sicherheitseinstellungen > Kontorichtlinien > Kennwortrichtlinie. Es gibt pro Domäne nur eine, auf OU-Ebene gilt sie nur für lokale Konten. Für einzelne Gruppen gibt es Fine-Grained Password Policies. [Q: Microsoft Learn]
96. **Wie zeigst du in einer Freigabe nur die Ordner an, auf die der Benutzer Zugriff hat?** Mit der zugriffsbasierten Aufzählung (Access-based Enumeration): `Set-SmbShare -Name Firma -FolderEnumerationMode AccessBased`. [Q: Microsoft Learn Set-SmbShare]
97. **Wie verbindest du ein Netzlaufwerk per GPO?** Benutzerkonfiguration > Einstellungen > Windows-Einstellungen > Laufwerkzuordnungen, Aktion Erstellen oder Aktualisieren mit UNC-Pfad und Laufwerksbuchstabe. [W]
98. **Welche OIDs brauchst du für den Tonerstand eines Druckers und wie rechnest du in Prozent?** `prtMarkerSuppliesLevel` (1.3.6.1.2.1.43.11.1.1.9) geteilt durch `prtMarkerSuppliesMaxCapacity` (1.3.6.1.2.1.43.11.1.1.8) mal 100. Negative Werte sind Sonderwerte (zum Beispiel unbekannt) und kein Füllstand. [Q: PRINTER-MIB bei Observium für die OIDs, Sonderwerte W]

### Zusatzkarten: Kahoot und Anleitung der älteren Lernenden (99 bis 119)
99. **Welcher Port gehört zum RDP-Protokoll?** TCP 3389. [Q: Kahoot «Modul 188 (Theorie)»]
100. **Welcher Dienst nutzt Port 53?** DNS, per UDP und TCP. [Q: Kahoot, W für UDP und TCP]
101. **Welche Ports nutzt DHCP?** UDP 67 (Server) und UDP 68 (Client). [Q: Kahoot, W für das Protokoll]
102. **Was bedeutet eine Adresse 169.254.x.x am Client?** Das ist APIPA: Der Client hat keine DHCP-Antwort bekommen und vergibt sich selbst eine lokale Adresse. Dann Scope, DHCP-Dienst und Netzweg prüfen. [Q: Kahoot und Anleitung der älteren Lernenden]
103. **Was gehört nicht in eine Betriebsdokumentation?** Persönliche Reflexionen zum Projekt. Hinein gehören zum Beispiel Netzwerkeinstellungen, Benutzer- und Gruppentabellen und Checklisten wie die zur Benutzererstellung. [Q: Kahoot]
104. **Wofür steht IGDLA und was hat es mit AGDLP zu tun?** Identities, Global groups, Domain Local groups, Access (Berechtigungen). Es ist dasselbe Prinzip wie AGDLP, nur anders benannt. [Q: Kahoot, W für die Erklärung]
105. **Braucht ein Netzwerk immer einen DHCP-Server?** Nein, statische Adressen gehen auch. DHCP spart Aufwand und vermeidet Fehler bei der Vergabe. [Q: Kahoot, W für die Begründung]
106. **Braucht man zwingend einen internen DNS-Server?** Allgemein nein. Mit Active Directory aber ja, weil AD für Anmeldung und Diensterkennung DNS mit SRV-Records braucht. [Q: Kahoot und Anleitung der älteren Lernenden]
107. **Welches Monitoring braucht mehr Ressourcen und Wartung, dezentral oder zentral?** Das dezentrale. Dass zentrales Monitoring weniger flexibel sei, stimmt laut Quiz nicht. [Q: Kahoot]
108. **Dürfen Microsoft-Updates direkt in der Produktivumgebung eingespielt werden?** Nein, zuerst testen (Testgruppe oder Testumgebung), dann ausrollen. [Q: Kahoot]
109. **Wie werden Microsoft-Updates in einem Windows-Netz verteilt?** Mit einem internen WSUS-Server, mit SCCM (Configuration Manager) oder mit RMM-Software von Drittanbietern. Die Option «Windows Update Proxy Service» war im Quiz die falsche Antwort. [Q: Kahoot]
110. **Wie versteckst du eine Freigabe in der Netzwerkumgebung?** Mit einem Dollarzeichen am Ende des Freigabenamens, zum Beispiel `Marketing$`. Das versteckt sie nur und schützt sie nicht. [Q: Kahoot, W für den Zusatz]
111. **Wie zeigst du alle deaktivierten Benutzer in PowerShell an?** `Get-ADUser -Filter {Enabled -eq $false}`. Das Quiz schreibt `"False"` als Text, das funktioniert auch. Alternative: `Search-ADAccount -AccountDisabled -UsersOnly`. [Q: Kahoot, W für die Schreibweise und die Alternative]
112. **Welche FSMO-Rolle ist für Passwort-Rücksetzungen zuständig?** Der PDC-Emulator. [Q: Kahoot]
113. **Nenne die fünf FSMO-Rollen.** Schema-Master und Domain Naming Master (je einmal pro Forest), RID-Master, PDC-Emulator und Infrastructure Master (je einmal pro Domäne). [W]
114. **Wie wird eine DHCP-Reservierung umgesetzt?** Über die MAC-Adresse: Das Gerät bekommt immer dieselbe Adresse aus dem Scope. [Q: Kahoot]
115. **Was tust du, wenn der Domänenbeitritt wegen eines falschen DNS am Client scheitert?** Den DNS am Client auf den DC stellen. Beim DHCP-Betrieb die Option 006 auf die DC-IP setzen. [Q: Anleitung der älteren Lernenden]
116. **In welcher Reihenfolge richtest du AD DS, DNS und DHCP ein?** Zuerst AD DS installieren und zum DC hochstufen (DNS kommt dabei mit), danach die DHCP-Rolle installieren und im AD autorisieren. [Q: Anleitung der älteren Lernenden]
117. **Wozu dient der AD-Papierkorb und wie schaltest du ihn ein?** Er stellt gelöschte AD-Objekte wieder her. Die Aktivierung gilt forestweit und einmalig, im Active Directory Administrative Center oder mit `Enable-ADOptionalFeature -Identity 'Recycle Bin Feature' -Scope ForestOrConfigurationSet -Target <Forest>`. [Q: Anleitung der älteren Lernenden, Microsoft Learn]
118. **Wozu dient eine Reverse-Zone mit PTR-Einträgen?** Sie löst IP-Adressen in Namen auf. Fehlt sie, ist die Namensauflösung unvollständig. [Q: Anleitung der älteren Lernenden]
119. **Warum schaltest du Zonentransfers aus, wenn es nur einen DNS-Server gibt?** Es gibt keinen zweiten Server, der die Zone braucht, und so kann niemand die Zone komplett abfragen. `Set-DnsServerPrimaryZone -Name <Zone> -SecureSecondaries NoTransfer`. [Q: Anleitung der älteren Lernenden, Microsoft Learn]

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

**DNS-Eintragstypen** [Q: https://learn.microsoft.com/de-de/windows-server/networking/dns/resource-record-types und https://learn.microsoft.com/de-de/windows-server/networking/dns/manage-resource-records]
| Typ | Bedeutung |
| --- | --- |
| A, AAAA | Hostname zu IPv4- bzw. IPv6-Adresse |
| CNAME | Alias auf einen anderen kanonischen Namen |
| MX | Mailserver einer Domain, kleinere Priorität wird bevorzugt |
| NS | autoritative Nameserver einer Zone, ermöglicht Delegation |
| PTR | Reverse-Lookup von der IP-Adresse zum Namen, braucht eine passende Reverse-Lookupzone |
| SOA | steht am Anfang jeder Zone, wird beim Anlegen der Zone automatisch erstellt, enthält primären Server, Kontakt und Refresh-Angaben |
| SRV | Host, Port und Protokoll eines Dienstes, nötig damit Clients Active-Directory-Domänencontroller per DNS finden |
Das passt zu Kenntnis 6.1 (DNS-Einträge als Umsystem-Anforderung): Für die Änderung beschreibst du Typ, Name, Wert und TTL.

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
- «Berechtigungen ändern» und «Besitz übernehmen» sind eigene erweiterte NTFS-Rechte. Der Besitzer einer Datei oder eines Ordners kann die Berechtigungen unabhängig von den bestehenden Rechten ändern. Administratoren können den Besitz übernehmen, auch ohne ausdrückliche Berechtigung. Verweigern hat immer Vorrang vor Zulassen. [Q: Microsoft Support und Sicherheitshandbuch]
- Ob «Vollzugriff» diese beiden erweiterten Rechte einschliesst, ist in den Quellen nicht ausdrücklich belegt. Nach meinem Wissen ja, prüfe es in den erweiterten Sicherheitseinstellungen deiner VM. [W]

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
**Eigene Ansicht in der Ereignisanzeige** [Q: Microsoft Q&A und Server-Manager-Doku, Abfragetext aus meinem Wissen]
- Unter «Benutzerdefinierte Ansichten» mit Rechtsklick «Benutzerdefinierte Ansicht erstellen», dann im Reiter «Filter» Ebene (zum Beispiel Fehler) und Ereignisquelle wählen, Namen und Beschreibung vergeben und speichern.
- Im Reiter «XML» kannst du «Abfrage manuell bearbeiten» aktivieren. Danach ist der Reiter «Filter» nicht mehr nutzbar.
- Beispiel für alle Dienstfehler im Systemlog [W]. Ebene 2 entspricht «Fehler»:
```xml
<QueryList>
  <Query Id="0" Path="System">
    <Select Path="System">*[System[Provider[@Name='Service Control Manager'] and (Level=2)]]</Select>
  </Query>
</QueryList>
```
- Der Bildungsplan verlangt Überwachung per Bordmittel oder Monitoring (f5.2). Eine solche Ansicht ist ein einfaches Bordmittel, um Dienstabstürze (7034) schnell zu sehen.

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
- **Konfliktlösung «last writer wins»** [Q: gleiche Seite]: Legen mehrere GPOs dieselbe Einstellung fest, gewinnt das zuletzt angewendete. Die Reihenfolge ist lokal, Standort, Domäne, OU. Innerhalb desselben Containers entscheidet die Verknüpfungsreihenfolge in der Gruppenrichtlinien-Verwaltungskonsole (GPMC). Für die Prüfung: Bei einem unerwarteten Ergebnis zuerst `gpresult /r` und die Verknüpfungsreihenfolge ansehen.
- **GPO-Filterung** [Q: https://learn.microsoft.com/de-de/windows-server/identity/ad-ds/manage/group-policy/group-policy-scope]: Sie bestimmt, für welche Benutzer und Computer ein GPO gilt. Bei der **Sicherheitsfilterung** brauchen Benutzer oder Computer die Berechtigungen «Lesen» und «Gruppenrichtlinien anwenden» auf das GPO, sonst wird es nicht angewendet. Die **WMI-Filterung** verfeinert das mit Abfragen, die Bedingungen auf dem Zielsystem prüfen, zum Beispiel Betriebssystem oder Systemfunktionen. So richtest du ein GPO gezielt an eine Gruppe, ohne eine neue OU zu bauen [W].
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

**Updates per Gruppenrichtlinie und WSUS steuern** (zum Verteilen mit Softwareverteilungssystemen, Handlungsziel 3) [Q: https://learn.microsoft.com/de-de/windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates und https://learn.microsoft.com/de-de/windows/deployment/update/waas-wu-settings]
- Die Richtlinie «Automatische Updates konfigurieren» liegt unter Computerkonfiguration, Administrative Vorlagen, Windows-Komponenten, Windows Update. Für WSUS empfiehlt Microsoft die Option «Automatisch herunterladen und laut Zeitplan installieren», damit genehmigte Updates ohne Eingriff des Benutzers zeitnah installiert werden.
- Damit die Clients den WSUS-Server nutzen, braucht es die Richtlinie «Internen Pfad für den Microsoft Updatedienst angeben». Die Clients müssen dem WSUS-Server zugeordnet werden, sei es per lokalem oder domänenbasiertem Gruppenrichtlinienobjekt.
- Der Installationszeitplan (Tag und Uhrzeit) wird mit Option 4 festgelegt. Ohne Angabe gilt täglich 03:00 Uhr. In der Registry stehen die Werte `ScheduledInstallDay` und `ScheduledInstallTime` unter `HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU`.
- Änderungen an Gruppenrichtlinien erreichen die Clients etwa alle 90 Minuten, mit `gpupdate /force` sofort. Die Quellen sind teils älter, die Menüs können in neueren Windows-Versionen abweichen.
- Für die Prüfung heisst das: Updates zentral genehmigen, Testgruppe zuerst, Zeitplan im Wartungsfenster, danach Kontrolle (siehe Patch-Zyklus in 2l).

**WSUS: Updates genehmigen und zurücknehmen** [Q: https://learn.microsoft.com/de-de/windows-server/administration/windows-server-update-services/manage/updates-operations]
- Bei einem neuen Update wählst du «Installieren» oder «Ablehnen». Genehmigen kannst du für «Alle Computer» oder für Untergruppen. Ohne Genehmigung bleibt das Update «Nicht genehmigt», die Clients können aber bewerten, ob sie es brauchen.
- Nach der Genehmigung kannst du sie auf untergeordnete Gruppen anwenden, einen **Stichtag** für die automatische Installation festlegen (überschreibt die Clienteinstellungen, ein Datum in der Vergangenheit heisst sofort bei der nächsten Verbindung) und ein bereits installiertes Update **entfernen lassen**, wenn es die Entfernung zulässt.
- Kein Stichtag ist möglich, wenn für das Update Benutzereingaben nötig sind. Das steht in den Updateeigenschaften im Feld «Benutzereingabe möglicherweise erforderlich».
- Im Replikatmodus kannst du auf dem WSUS-Server keine Updates genehmigen.
- Weitere Funktionen laut Seitenstruktur: automatisches Genehmigen, automatisches Ablehnen abgelöster Updates, Bereinigen abgelehnter Updates, Ändern eines genehmigten Updates auf «Nicht genehmigt», Genehmigen zum Entfernen.
- Für das Testen vor dem Rollout bietet sich eine Testgruppe an, der du Updates zuerst genehmigst [W].

**Update zurückrollen und Workarounds** (Thema der schriftlichen Prüfung: «Updates und Workarounds», Quellen sind Microsoft-Foren und Support-Antworten, bitte in der Test-VM prüfen)
- **Installierte Updates ansehen:** `Get-HotFix`. Pakete auf Betriebssystemebene: `Dism /Online /Get-Packages` oder `Get-WindowsPackage -Online -PackageName "*KB<Nummer>*"`.
- **Update entfernen, Variante 1:** `wusa /uninstall /kb:<Nummer>` (Leerzeichen vor weiteren Schaltern nicht vergessen). Das scheitert bei neueren kumulativen Updates oft mit Fehler 87 («Parameter falsch»). Der stille Modus ist bei neueren Servern nicht mehr verfügbar.
- **Variante 2:** `Remove-WindowsPackage -Online -PackageName <Name>` oder `Dism /Online /Remove-Package /PackageName:<Name>`. Wenn der Name fehlt, die `.msu` mit `expand -F:* <MSU-Pfad> <Ordner>` entpacken und die `.cab` mit `/PackagePath:<cab>` entfernen.
- **Grenzen:** Ein kombiniertes Paket aus Servicing Stack Update (SSU) und kumulativem Update lässt sich nur teilweise entfernen, die SSU bleibt. Einzelne Updates sind überhaupt nicht deinstallierbar.
- **Weitere Workarounds, wenn ein Patch Probleme macht** [W]: betroffene Funktion oder Dienst vorübergehend anders betreiben, Hersteller-Hinweis (zum Beispiel Registry-Einstellung) umsetzen, Update für diese Maschinen zurückstellen (in WSUS ablehnen oder Genehmigung verzögern), VM auf den Snapshot zurücksetzen, Problem beim Hersteller melden und dokumentieren.
- **Vorgehen konzeptionell** [W]: Fehler eingrenzen (was hat sich geändert), Auswirkung abschätzen, Workaround festlegen, mit Stakeholdern abstimmen, ausführen, testen, Betriebsdokumentation nachführen, später sauber beheben.

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

**Erste Schritte mit PRTG** [Q: https://www.paessler.com/manuals/prtg/step_1_download_installation_and_first_login]
1. Aktuelle stabile Version als Testversion von der Paessler-Website laden.
2. Setup auf dem Windows-Computer ausführen, der der Core-Server werden soll, und dem Assistenten folgen.
3. Danach im Browser die IP-Adresse oder den DNS-Namen des Core-Servers öffnen und «Log in» klicken. Benutzername und Startpasswort sind beide `prtgadmin`, das Login-Fenster füllt sie meist schon aus.
4. Das Standardpasswort sofort ändern, unter Setup, Account Settings, My Account. Das PRTG Administration Tool auf dem Core-Server kann Login-Namen und Passwort ebenfalls ändern.
5. Danach folgt laut Handbuch der Smart Setup (Schritt 2) zum Einrichten der ersten Geräte.
Das Ändern von Standardpasswörtern gehört auch in die Betriebsdokumentation und passt zum Sicherheitsgedanken von Handlungsziel 5.

**Zugangsdaten und Auto-Discovery in PRTG** [Q: https://www.paessler.com/manuals/prtg/smart_setup/ und https://www.paessler.com/manuals/prtg/auto-discovery]
- **Zugangsdaten** trägst du in den Einstellungen ein. Für WMI-Überwachung von Windows-Servern braucht PRTG Windows-Administratorzugangsdaten, bei Active Directory empfiehlt das Handbuch ein Domänenadministratorkonto. Für SNMP-Geräte gilt als Standard SNMP v2c mit dem Community String `public` auf Port 161. Zugangsdaten werden in der Root-Gruppe gespeichert und an untergeordnete Geräte vererbt, auf jeder Ebene lässt sich die Vererbung abschalten.
- Aus Sicht von Handlungsziel 5 (Least Privilege) ist ein Domänenadministrator für das Monitoring eher zu viel. Ein eigenes Dienstkonto mit den nötigen Rechten wäre sauberer, dazu Abschnitt 2i [W].
- **Auto-Discovery:** PRTG pingt das Subnetz des Core-Servers durch, fügt erreichbare Geräte hinzu, erkennt den Gerätetyp per SNMP, WMI und weiteren Protokollen und legt passende Sensoren aus Gerätevorlagen an. Stufen: Standard, Detailed und eigene Gerätevorlagen. Standardmässig läuft sie nur einmal, häufige Läufe können die Leistung belasten.
- **Voraussetzungen auf Windows-Zielen für WMI:** Die Dienste Remote Registry und RPC laufen und stehen auf «Automatisch», WMI ist aktiv, die Firewall des Ziels erlaubt WMI. SNMP lässt sich unter neueren Windows-Clients über die optionalen Features installieren.

**Passende PRTG-Sensoren** [Q: Sensorverzeichnis im PRTG-Handbuch, https://www.paessler.com/manuals/prtg/]
| Was du überwachst | Sensor (Name laut Handbuch) |
| --- | --- |
| Erreichbarkeit | Ping Sensor, Ping v2 Sensor, Port Sensor |
| DHCP, DNS, AD | DHCP Sensor, DNS v2 Sensor, Active Directory Replication Errors Sensor |
| CPU per SNMP / WMI | SNMP CPU Load Sensor, SNMP CPU Usage Sensor / WMI CPU Load v2 Sensor |
| Speicher (RAM) | SNMP Memory v2 Sensor / WMI Memory v2 Sensor |
| Festplatte | SNMP Disk Free v2 Sensor / WMI Free Disk Space (Multi Disk) Sensor |
| Dienste | SNMP Windows Service v2 Sensor / WMI Service Sensor |
| Laufzeit | SNMP System Uptime Sensor |
| Ereignisse | Event Log (Windows API) Sensor, WMI Event Log Sensor |
| Updates | Windows Updates Status (PowerShell) Sensor |
| Eigene OIDs | SNMP Library Sensor |
**Reports ausführen: Zeitraum wählen** [Q: https://www.paessler.com/manuals/prtg/run_reports]
- **Aktueller Zeitraum:** Daten der laufenden Periode (heute, diese Woche, dieser Monat, dieses Jahr), je nach der im Report eingestellten Periode.
- **Vorheriger Zeitraum:** gestern, letzte Woche, letzter Monat oder letztes Jahr.
- **Vordefinierter Zeitraum:** ein anderer Bereich aus einer Liste.
- **Benutzerdefiniert:** Start- und Enddatum frei wählen, mit Schnellauswahl-Buttons.
- Einen Report kannst du sofort mit den eingestellten Werten ausführen oder nach Zeitplan laufen lassen. In einem Cluster sind Failover-Knoten standardmässig nur lesend. Für die Wochenreport-Aufgabe in der Probeprüfung wählst du die Periode «Woche» und den Zeitraum «vorheriger Zeitraum», dann zeigt der Montagsreport die letzte Woche [W].

**Abhängigkeiten und Benachrichtigungs-Trigger in PRTG** [Q: https://www.paessler.com/manuals/prtg/dependencies und https://www.paessler.com/manuals/prtg/notification_triggers_settings]
- **Abhängigkeiten (Dependencies):** Ein Master-Sensor kann abhängige Sensoren pausieren, solange er Down ist. So vermeidest du Alarmfluten und falsche Ausfallzeiten, wenn zum Beispiel das ganze Gerät nicht erreichbar ist. Bei Auto-Discovery ist der Ping-Sensor eines Geräts standardmässig der Master.
- **Falle bei Wartung:** Pausierst du nur den Master-Sensor, pausieren die abhängigen Sensoren nicht mit und können weiter alarmieren. Pausiere stattdessen das übergeordnete Gerät. Ein pausiertes Objekt erfasst keine Daten, ändert den Status nicht und löst keine Benachrichtigungen aus. Das passt zum Wartungsfenster beim Patchen.
- **Trigger-Typen:** State, Speed, Volume, Threshold und Change. Welche möglich sind, hängt vom Objekttyp ab.
- **State Trigger:** löst bei Statuswechsel aus. Die Latenz (Sekunden) verhindert Fehlalarme bei kurzen Ausfällen und sollte nicht kürzer als das Scanintervall sein. Die Eskalation sendet nach einer Eskalationslatenz eine weitere Benachrichtigung, das Wiederholungsintervall 0 bedeutet keine Wiederholung. Standardwerte laut Helpdesk: 60 Sekunden Latenz, 300 Sekunden Eskalationslatenz (versionsabhängig).
- **Empfehlung des Handbuchs:** mindestens zwei Benachrichtigungen mit unterschiedlichen Methoden pro Trigger, zum Beispiel E-Mail und SMS.
- **Schwellwerte:** Die Warn- und Fehlergrenzen setzt du in den Kanaleinstellungen, ein Threshold Trigger ändert den Sensorstatus nicht (Hinweis aus einem älteren Paessler-KB-Beitrag).

**Status in PRTG lesen (für «Monitoring interpretieren» und «Map deuten»)** [Q: https://www.paessler.com/manuals/prtg/sensor_states]
| Status | Farbe | Bedeutung |
| --- | --- | --- |
| Up | grün | Letzter Scan erfolgreich, alles in Ordnung |
| Warning | gelb | Fehler erkannt, PRTG versucht es erneut. Nach einem zweiten fehlgeschlagenen Versuch wird der Sensor Down. Auch Warnschwellen setzen Warning. |
| Down | rot | Fehler, zum Beispiel Gerät nicht erreichbar oder Fehlergrenze überschritten |
| Down (Acknowledged) | hellrot | Alarm wurde quittiert, es gehen keine weiteren Benachrichtigungen raus |
| Unusual | orange | Werte weichen stark vom historischen Durchschnitt für diesen Wochentag und diese Uhrzeit ab (Unusual Detection) |
| Paused | blau | Sensor pausiert, befristet, unbefristet oder wegen einer Abhängigkeit |
| Unknown | grau | Für diesen Zeitraum liegen keine Daten vor, zum Beispiel nach einem Neustart des Core-Servers |
| Down (Partial) | | nur im Cluster, wenn Knoten uneinig sind |
In zusammengefassten Ansichten wie Gerätebaum oder Map gewinnt der Status mit der höchsten Priorität. Ist ein Sensor Down, zeigt auch das Gerät Down.
Daraus für die Map-Frage: Rot beim Gerät heisst, mindestens ein Sensor ist Down. Im Gerätebaum nach dem roten Sensor suchen, Ursache beheben und dokumentieren. Gelb ist Vorwarnung. Orange ist ungewöhnlich gegenüber der Norm und kann auf eine schleichende Änderung hinweisen. Blau ist keine Störung, sondern Pause oder Wartungsfenster.

Einzelheiten aus den Sensorseiten [Q: PRTG-Handbuch, DHCP Sensor, DNS v2 Sensor, SNMP Windows Service v2 Sensor]:
- **DHCP Sensor:** sendet einen Broadcast und wartet auf die Antwort eines DHCP-Servers, zeigt dessen Adresse und die angebotene IP. Das Probe-Gerät muss eine statische IP haben, sonst droht Datenverlust. Der Sensor zeigt einen Timeout, wenn kein DHCP antwortet oder wenn mehr als zwei DHCP-Sensoren pro Gerät angelegt sind.
- **DNS v2 Sensor:** fragt einen DNS-Server nach einem Namen ab und vergleicht das Ergebnis mit einem Filter. Er schlägt an, wenn der Server den Namen nicht korrekt auflöst. Der Abfragetyp ist wählbar.
- **SNMP Windows Service v2 Sensor:** überwacht den Status eines Windows-Dienstes per SNMP. Er braucht SNMP-Zugangsdaten, die weiter oben im Gerätebaum hinterlegt sind, und liest die Dienste aus der LAN-Manager-MIB.

Das deckt sich mit den geprüften Themen der LBV: AD, DNS, DHCP und Updates haben jeweils einen eigenen Sensor. Welche davon die Learnfactory verwendet, weiss ich nicht.

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
- Microsoft verweist in der Hyper-V-Doku für Windows Server 2025 auf das Evaluation Center: https://www.microsoft.com/evalcenter/evaluate-windows-server-2025 (die Seite war für mich gerade nicht abrufbar, wegen hoher Last). Evaluierungskopien laufen laut Evaluation-Center-Seiten 180 Tage und müssen innerhalb von 10 Tagen online aktiviert werden, sonst fährt das System automatisch herunter. Die ISO enthält die Editionen Standard und Datacenter. Eine Verlängerung per `slmgr /rearm` wird in der Community genannt, ist aber keine verbindliche Microsoft-Angabe [Q: Evaluation Center und Microsoft Q&A].

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

## 2p. Muster: Berechtigungskonzept dokumentieren und begründen

Das Zugriffs- und Berechtigungskonzept ist laut LBV der grösste Prüfungsposten (bis 40 % im Praxisteil, bis 30 % im schriftlichen Teil). Dieses Muster ist mein Entwurf [W], die Grundlagen (AGDLP, Freigabe gegen NTFS, Least Privilege, Matrix) sind in den Abschnitten 2c und 2i belegt.

**1. Namensschema**
- Globale Gruppen (Rollen): `GG_<Abteilung>`, zum Beispiel `GG_Buchhaltung`, `GG_Geschaeftsleitung`, `GG_Einkauf`
- Domänenlokale Gruppen (Zugriff): `DL_<Ressource>_<Recht>`, zum Beispiel `DL_Buchhaltung_RW`, `DL_Buchhaltung_R`
- Recht-Kürzel: `R` lesen, `RW` ändern, `FC` Vollzugriff (nur für Administratoren)

**2. Matrix Gruppen gegen Freigaben** (Beispiel Muster AG)

| Freigabe, Pfad | GG_Buchhaltung | GG_Geschaeftsleitung | GG_Einkauf | Administratoren |
| --- | --- | --- | --- | --- |
| `\\SRV01\Buchhaltung`, `D:\Daten\Buchhaltung` | Ändern (`DL_Buchhaltung_RW`) | Lesen (`DL_Buchhaltung_R`) | kein Zugriff | Vollzugriff |
| `\\SRV01\Einkauf`, `D:\Daten\Einkauf` | kein Zugriff | Lesen (`DL_Einkauf_R`) | Ändern (`DL_Einkauf_RW`) | Vollzugriff |
| `\\SRV01\Allgemein`, `D:\Daten\Allgemein` | Lesen | Lesen | Lesen | Vollzugriff |

**3. Technische Umsetzung**
- Freigabe: keine zu breiten Rechte wie «Jeder: Vollzugriff» vergeben. Das Freigaberecht für die DL-Gruppen grosszügig setzen (zum Beispiel «Ändern») und die Feinsteuerung über NTFS machen. Es gilt die restriktivere Ebene.
- NTFS: Vererbung vom Ordner `D:\Daten` aufbauen, Rechte nur an `DL_*`-Gruppen, nie direkt an Personen. `Administratoren` und `SYSTEM` behalten Vollzugriff.
- Schritte: Ordner anlegen, Freigabe mit `New-SmbShare`, NTFS mit `icacls` setzen, DL-Gruppen anlegen, GG in DL aufnehmen, Benutzer in GG, mit einem Testkonto prüfen.

**4. Begründung (so schreibst du sie in der Prüfung)**
- «Rechte hängen an Gruppen statt an Personen. Dadurch ändere ich bei Wechseln nur die Gruppenmitgliedschaft, die ACLs bleiben unverändert.» (AGDLP, Microsoft empfiehlt Gruppen)
- «Die Geschäftsleitung bekommt nur Lesen, weil sie Daten einsehen, aber nicht ändern muss (Least Privilege).»
- «Freigabe- und NTFS-Rechte sind getrennt, es gilt die restriktivere Ebene. Ich setze die Freigabe breit und steuere die Details über NTFS, damit die Verwaltung an einer Stelle liegt.»
- «Admin-Konten werden nur für Verwaltungsaufgaben genutzt, nicht im Alltag.»

**5. Test und Dokumentation**
- Testkonto aus `GG_Buchhaltung`: Datei in Buchhaltung anlegen geht, in Einkauf kein Zugriff.
- Testkonto aus `GG_Geschaeftsleitung`: Buchhaltung lesen geht, schreiben nicht.
- In die Betriebsdokumentation: Matrix, Namensschema, Datum der Änderung, wer es umgesetzt hat, Testergebnis.

**6. Ergänzen im Prüfungsfall** (LBV: «ergänzen und begründen»): Neue Abteilung oder neue Freigabe heisst: neue GG, neue DL-Gruppen, Matrix erweitern, NTFS und Freigabe setzen, testen, dokumentieren, begründen.

---

## 2q. Muster: IP-Adress- und Namenskonzept (für AD, DNS, DHCP)

Die LBV prüft AD, DNS und DHCP (Praxis 15 bis 25 %). Bei Modul 123 verlangt die Prüfung dafür ein IP-Adresskonzept und ein Namenskonzept, die du planst und umsetzt. Dieses Muster ist mein Entwurf [W], die Befehle stehen im Cheatsheet (Abschnitt 1).

**IP-Adresskonzept** (Netz 192.168.10.0/24, Maske 255.255.255.0, Gateway 192.168.10.1)

| Bereich | Adressen | Verwendung |
| --- | --- | --- |
| Netzwerkgeräte | .1 bis .9 | Router, Switch, Access Point (statisch) |
| Server | .10 bis .29 | `SRV01` (.10), `MON01` (.20), statisch |
| Drucker und Geräte | .30 bis .99 | Reservationen im DHCP (zum Beispiel Drucker .50) |
| DHCP-Pool Clients | .100 bis .200 | dynamisch vergeben |
| Reserve | .201 bis .254 | Erweiterung, nicht verteilt |

DHCP-Scope: Pool .100 bis .200, Optionen Router .1, DNS .10, DNS-Domäne `muster.local`, Lease-Dauer zum Beispiel 8 Tage (Standard) oder kürzer für Gäste-WLAN [W]. Der DHCP-Server wird im Active Directory autorisiert (Abschnitt 1, DHCP).

**Namenskonzept**
- Rechner: `<Typ><Nummer>`, zum Beispiel `SRV01`, `MON01`, `CL001`, `PRN01`
- Domäne: `muster.local` (Active Directory), DNS-Zone gleich benannt
- Benutzer: `<Vorname-Initial><Nachname>`, zum Beispiel `skeller`
- Gruppen: siehe Abschnitt 2p
- OUs: pro Abteilung eine OU unter `muster.local`, darunter Benutzer, Computer, Gruppen
- DNS-Einträge: A-Records für Server und Drucker mit `-CreatePtr`, Alias (CNAME) für Dienste wie `files.muster.local` [W]

**Begründung für die Prüfung**
- Feste Bereiche pro Gerätetyp machen die Adresse lesbar und die Fehlersuche einfach.
- Server, Netzgeräte und Drucker brauchen stabile Adressen, Clients sind dynamisch (Modul 123 Kenntnis 1.5).
- Eine Reserve verhindert, dass der Pool bei Wachstum knapp wird.
- Einheitliche Namen erleichtern Dokumentation, Monitoring und Berechtigungen.

**Testen und dokumentieren**
- Client: `ipconfig /release`, `ipconfig /renew`, `ipconfig /all` (Adresse aus dem Pool, Gateway und DNS stimmen).
- Namen: `nslookup srv01.muster.local`, `Resolve-DnsName`, Gegenprobe über die IP (Reverse).
- Domäne: Client in die Domäne aufnehmen, Anmeldung mit einem Domänenkonto.
- Betriebsdokumentation: IP-Plan, Scope-Einstellungen, Namenskonzept, Datum und Test.

---

## 2r. Muster: Betriebsdokumentation der Muster AG (zum Üben des Nachführens)

In der Prüfung bekommst du eine Betriebsdokumentation vorgegeben, analysierst damit das System und ergänzt sie am Schluss (LBV, Element 1). Dieses Muster ist mein Entwurf [W], die Gliederung folgt der Schweizer Vorlage von Datenschutz.ch (Abschnitt 2e der Vorbereitungsdatei). Übe, es mit dem Lab zu vergleichen und die Änderungen nachzuführen.

**1. Allgemeines.** Zweck: Betriebsdokumentation der IT der Muster AG. Verantwortlich: Systemadministration. Version 1.0.

**2. Systemübersicht**

| System | Typ | Betriebssystem | IP | Aufgabe |
| --- | --- | --- | --- | --- |
| SRV01 | Server (VM) | Windows Server | 192.168.10.10 | AD, DNS, DHCP, Dateiserver |
| MON01 | Server (VM) | Windows Server | 192.168.10.20 | Monitoring (zum Beispiel PRTG) |
| CL001 bis CL003 | Clients | Windows 11 | DHCP | Arbeitsplätze |
| PRN01 | Drucker | | 192.168.10.50 (Reservation) | Netzwerkdrucker |

**3. Netzwerk.** Netz 192.168.10.0/24, Gateway 192.168.10.1, DNS 192.168.10.10, Domäne `muster.local`. Netzplan als Bild beilegen.

**4. Dienste auf SRV01**
- DNS: Zone `muster.local`, Weiterleitung auf den Router
- DHCP: Scope .100 bis .200, Optionen Router und DNS, Reservation für den Drucker, im AD autorisiert
- AD DS: Domäne `muster.local`, OUs pro Abteilung
- Dateiserver: Freigaben `Buchhaltung`, `Einkauf`, `Allgemein` unter `D:\Daten`

**5. Berechtigungen.** Matrix und Namensschema wie in Abschnitt 2p. Passwörter stehen nicht in dieser Doku, sondern im Notfallkonzept.

**6. Monitoring.** `MON01` überwacht `SRV01` mit Ping, CPU, Speicher auf D:, Dienst DHCP-Server. Grenzwerte: CPU Warnung ab 80 Prozent, Speicher Warnung unter 20 Prozent frei. Benachrichtigung per E-Mail an die Systemadministration. Wochenreport als PDF.

**7. Updates und Wartung.** Patchen im Wartungsfenster (zum Beispiel dienstags ab 18 Uhr), vorher Snapshot, danach Kontrolle der Dienste und der Monitoring-Sensoren.

**8. Umsysteme.** Firewall: UDP 161 von `MON01` zu `SRV01` erlaubt (SNMP). Änderungen beim Netzwerkpartner beantragen.

**9. Änderungsprotokoll** (hier trägst du in der Prüfung nach)

| Datum | Änderung | Durch | Test |
| --- | --- | --- | --- |
| | SNMP auf SRV01 installiert, nur MON01 erlaubt | | Sensor grün |
| | Benutzerin S. Keller in GG_Buchhaltung aufgenommen | | Zugriff geprüft |
| | Update KB... installiert | | Dienste laufen, Hotfix-Liste kontrolliert |
| | Firewallregel UDP 161 für MON01 beantragt | | Abfrage von MON01 erfolgreich |

Übung: Mach eine Änderung im Lab und trage sie vollständig ein (Datum, Änderung, wer, Test). So trainierst du das «Nachführen» aus Handlungsziel 3.

## 2s. Ergänzungen aus dem öffentlichen Script von ICT-BZ (Tag 2 bis 5)

Quelle: https://m188.ict-bz.ch (Script zum üK 188 der ICT-Berufsbildung Zentralschweiz). Das ist nicht das Script der Learnfactory, Aufgaben und Tools können bei dir anders sein. Ich habe hier nur ergänzt, was in meinen Unterlagen fehlte, in eigenen Worten. Die Befehle stammen aus Microsoft Learn (mit [Q]) oder aus meinem Fachwissen (mit [W]). Überblick über das Script: `UEK-188-Vorbereitung.md`, Abschnitt 2f.

### Bulk-User aus einer CSV-Datei (Tag 2)
Aufgabe im Script: Benutzerliste aus Excel per PowerShell im AD anlegen. Gerüst: Modul laden, Datei mit `Import-Csv` lesen, mit `foreach` jede Zeile verarbeiten. Spaltennamen und Domäne unten sind Beispiele [W]:
```powershell
Import-Module ActiveDirectory
$pw    = Read-Host -AsSecureString "Startpasswort"
$users = Import-Csv -Path "C:\Temp\Benutzer.csv" -Delimiter ";" -Encoding UTF8
foreach ($u in $users) {
    New-ADUser -Name "$($u.Vorname) $($u.Nachname)" `
        -GivenName $u.Vorname -Surname $u.Nachname `
        -SamAccountName $u.Benutzername `
        -UserPrincipalName "$($u.Benutzername)@contoso.local" `
        -Path "OU=Verkauf,OU=Firma,DC=contoso,DC=local" `
        -AccountPassword $pw -ChangePasswordAtLogon $true -Enabled $true
    Write-Output "Benutzer angelegt: $($u.Vorname) $($u.Nachname)"
}
```
Fallen bei CSV-Dateien (die Aufgabe fragt danach) [W]:
- Trennzeichen: Excel speichert hier meist mit Semikolon, `Import-Csv` erwartet standardmässig Komma. Darum `-Delimiter ";"`.
- Zeichensatz: Beim Speichern «CSV UTF-8» wählen, sonst sind Umlaute kaputt. Beim Import `-Encoding UTF8`.
- Die Spaltennamen der ersten Zeile werden zu Eigenschaften. Leerzeichen oder Tippfehler darin brechen das Skript.
- Benutzernamen ohne Leerzeichen und Umlaute, jeder nur einmal. Die Ziel-OU muss existieren. Das Startpasswort muss zur Kennwortrichtlinie passen.

### Kennwortrichtlinie per GPO (Aufgabe: 60 Tage, Chronik 10, Länge 10)
- Pfad: Default Domain Policy bearbeiten, Computerkonfiguration > Richtlinien > Windows-Einstellungen > Sicherheitseinstellungen > Kontorichtlinien > Kennwortrichtlinie [Q: Microsoft Learn, Modul «Configure a domain password policy»].
- Werte der Aufgabe: Maximales Kennwortalter 60 Tage, Kennwortchronik erzwingen 10, Minimale Kennwortlänge 10, Komplexitätsvoraussetzungen aktiviert.
- «Mit Sonderzeichen»: Die Komplexitätsregel verlangt drei von vier Zeichengruppen (Gross, Klein, Ziffer, Sonderzeichen), nicht zwingend ein Sonderzeichen [W].
- Es gibt pro Domäne nur eine Kontorichtlinie. Auf OU-Ebene verknüpft gilt sie nur für lokale Konten auf Mitgliedsservern. Für einzelne Gruppen nimmt man Fine-Grained Password Policies [Q: Microsoft Learn, ältere Quellen].
- Danach `gpupdate /force`. Mit `net accounts /domain` siehst du die aktiven Werte [W].
- Die Frage «Was macht ein gutes Passwort aus?» beantwortest du mit Abschnitt 2g (NCSC).

### Laufwerke, ausgeblendete Ordner, Hintergrundbild per GPO
- Zwei Laufwerke verbinden (Transfer als `G:`, Firmenlaufwerk als `M:`): Benutzerkonfiguration > Einstellungen > Windows-Einstellungen > Laufwerkzuordnungen, Aktion Erstellen oder Aktualisieren, UNC-Pfad und Buchstabe [W].
- Nur erlaubte Ordner in `M:` anzeigen: zugriffsbasierte Aufzählung auf der Freigabe [Q: Microsoft Learn]:
```powershell
Set-SmbShare -Name Firma -FolderEnumerationMode AccessBased
```
  Das blendet nur aus. Der Schutz kommt weiterhin von den NTFS-Rechten (AGDLP).
- Hintergrundbild: Benutzerkonfiguration > Richtlinien > Administrative Vorlagen > Desktop > Desktop > Desktophintergrund [W]. Die Aufgabe fragt, was du beachten musst. Meine Antwort: Der Pfad muss ein UNC-Pfad sein, auf den alle Benutzer lesen dürfen (Freigabe und NTFS), kein lokaler Serverpfad. Die GPO muss mit der OU der Benutzer verknüpft sein, und die Benutzer brauchen eine neue Anmeldung [W, eigene Annahme, nicht aus dem Script].

### Rollen und Features für die Betriebsdoku auslesen (Server Manager)
```powershell
Get-WindowsFeature | Where-Object Installed | Select-Object DisplayName, Name, InstallState | Out-File C:\Doku\Features.txt
Get-WindowsFeature -ComputerName SRV01 | Where-Object Installed     # Remote-Server
```
Die Aufgabe will das Resultat mit Servername, Administrator, Datum und Unterschrift abgeben [Q: Script ICT-BZ]. Befehle [W].

### Dateidienste: Musik und Videos blockieren, Bericht (FSRM)
Das Szenario im Script: Der Platz auf dem Datenlaufwerk wächst, ein neuer Mitarbeiter legt Musik und Videos ab. Mit dem Feature File Server Resource Manager lässt sich das blockieren und auswerten.
```powershell
Install-WindowsFeature FS-Resource-Manager -IncludeManagementTools
Get-FsrmFileGroup | Select-Object Name                                   # vorhandene Dateigruppen
New-FsrmFileScreen -Path "D:\Daten" -IncludeGroup "Audio and Video Files" -Active
New-FsrmStorageReport -Name "Dateitypen D" -Namespace @("D:\Daten") `
    -ReportType @("FilesByFileGroup") -ReportFormat @("Html") -Interactive
```
- `-Active` blockiert wirklich und ist laut Doku standardmässig an. Eine passive Prüfung, die nur protokolliert, legst du mit `-Active:$false` an. Bereits gespeicherte Dateien bleiben zugänglich [Q: Microsoft Learn, New-FsrmFileScreen].
- Interaktive Berichte erzeugt der Server sofort und legt sie unter `C:\StorageReports\Interactive` ab. Berichtstypen sind unter anderem `LargeFiles`, `FilesByFileGroup`, `FilesByOwner`, `QuotaUsage`. Formate sind `Html`, `DHtml`, `Csv`, `Text`, `XML` [Q: Microsoft Learn, New-FsrmStorageReport].
- Den Namen der Dateigruppe prüfst du mit `Get-FsrmFileGroup`. «Audio and Video Files» ist der Standardname auf englischen Systemen [W]. Mit einer Vorlage geht es auch: `New-FsrmFileScreen -Path "D:\Daten" -Template "Block Image Files"` ist das Beispiel von Microsoft, für Medien gibt es eine ähnliche Standardvorlage [Q und W].

### Systemmonitor und Verlauf des Datenplatzes
- Ressourcenmonitor: `resmon`, live pro Prozess. Systemmonitor: `perfmon`, Indikatoren über die Zeit. `perfmon /res` öffnet ebenfalls den Ressourcenmonitor [W].
- Wachstum von `D:` aufzeichnen mit einem Datensammlersatz, zum Beispiel per Kommandozeile [W]:
```cmd
logman create counter DiskGrowth -c "\LogicalDisk(D:)\Free Megabytes" "\LogicalDisk(D:)\% Free Space" -si 00:15:00 -o C:\PerfLogs\DiskGrowth
logman start DiskGrowth
fsutil file createnew D:\Test\dummy1.bin 1073741824
```
  Die letzte Zeile legt zum Testen eine 1 GiB grosse Datei an (`fsutil` kennt die Länge in Byte). Nach dem Test die Dummy-Dateien wieder löschen.
- Als Alternative zeigt ein Monitoring-Tool (Zabbix, PRTG) den Verlauf automatisch als Graph.

### Hashwerte und Signaturen (Tag 4)
```cmd
certutil -hashfile setup.iso SHA256
```
```powershell
Get-FileHash .\setup.iso -Algorithm SHA256
```
- Häufige Algorithmen: SHA-256 (heute Standard), SHA-1 und MD5 gelten als schwach und taugen nur noch gegen versehentliche Beschädigung [W].
- Stimmt der Hash nicht, nicht installieren: Datei neu laden, Quelle prüfen, anderen Spiegelserver nehmen.
- Ein passender Hash ist keine Virenfreiheit. Er bestätigt nur «unverändert gegenüber dem Original» [Q: Script ICT-BZ].
- Ein Hash schützt nicht, wenn Datei und Hash von derselben manipulierten Seite kommen. Eine Signatur (Herstellerschlüssel) ist stärker. Unter Windows prüft `Get-AuthenticodeSignature` signierte Programme [W].

### Windows Admin Center (Tag 4)
- Installation auf einem der Clients, Aufruf im Browser über HTTPS. Das Zertifikat ist im Lab selbstsigniert, die Browserwarnung ist dort normal [W].
- Ziele hinzufügen: Server oder Client mit Name oder IP und Anmeldedaten. Auf dem Ziel muss WinRM laufen, darum die Aufgabe `Enable-PSRemoting` auf dem Client (siehe 2o).
- Das kannst du damit: Übersicht und Leistung (Systemmonitor im Browser), PowerShell-Sitzung, Remotedesktop, Rollen und Features installieren, Ereignisse lesen, Dienste und Updates [W].
- Aufgabe DHCP: Rolle über das Admin Center installieren, Bereich anlegen und im AD autorisieren (siehe Cheatsheet). Dann im Netz den DHCP des Routers abschalten, sonst gibt es zwei DHCP-Server. Genau das meint «Passen Sie alle Konfigurationen im Netzwerk an» [W, Deutung]. Danach die Betriebsdokumentation nachführen.

### Zabbix, falls ihr es auch einsetzt (Tag 3)
Das Script von ICT-BZ arbeitet mit der Zabbix Appliance (fertige VM). Ob die Learnfactory Zabbix, PRTG oder etwas anderes nimmt, ist unbekannt. Die Konzepte sind in jedem Tool ähnlich.

| Begriff | Bedeutung [W] |
| --- | --- |
| Host und Host Group | überwachtes Gerät und Gruppierung (z.B. «Network», «Windows-Server») |
| Template | Vorlage mit Items, Triggern und Graphen, die mehrere Hosts bekommen |
| Item | ein einzelner Messwert (CPU, freier Platz, Interface-Status) |
| Trigger | Bedingung auf Messwerten, die ein Problem mit einer Schwere auslöst |
| Graph, Map, Dashboard | Verlauf, Netzplan mit Verbindungen, Übersichtsseite (z.B. für den Service Desk) |
| Discovery | automatisches Finden von Geräten oder Objekten wie Interfaces und Laufwerken |

- Schweregrade eines Problems in Zabbix: Not classified, Information, Warning, Average, High, Disaster [Q: Zabbix-Doku].
- Acknowledge meldet «ich bearbeite das» mit Kommentar. Suppress blendet ein Problem aus [Q: Zabbix-Doku]. Zum Pausieren eines Hosts nimmt man eine Wartungsperiode (Maintenance) oder deaktiviert den Host [W].
- SNMP-Gerät einbinden: Host mit SNMP-Interface (IP, Port 161, Version, Community) anlegen und das Template «Network Generic Device by SNMP» verknüpfen [Q: Script ICT-BZ]. Konfigurieren musst du SNMP an zwei Orten: auf dem Gerät (SNMP an, Community, erlaubter Manager) und im Monitoring (Host-Interface). Ob die Verbindung geht, siehst du am Verfügbarkeits-Symbol des Hosts und an Werten unter «Latest data» [W].
- Windows-Server oder Client per Agent: Zabbix-Agent installieren (siehe 2g), Servername und Hostname müssen zusammenpassen, Port 10050 in der Firewall für den Zabbix-Server öffnen, Template «Windows by Zabbix Agent» verknüpfen [Q: Script ICT-BZ für das Template, W für den Rest].
- Ausfälle simulieren: einen Dienst stoppen, den Agenten stoppen, ein Netzwerkkabel ziehen. Das Problem erscheint unter Monitoring > Problems und im Dashboard [W].
- Eine Website ausserhalb des Netzes (www.ict-bz.ch) überwachst du vom Monitoring-Server aus, zum Beispiel mit einem Web-Szenario oder einer einfachen Prüfung auf Port 443 [W].
- Die Zabbix-Standardzugangsdaten ändert man im echten Betrieb sofort. Nach dem ersten Start der Appliance dauert es einige Minuten, bis das Webinterface bereit ist [Q: Script ICT-BZ].

### SNMP-Abfragen mit dem MIB-Browser (Tag 3)
Die Aufgabe: Werte von Router, Switch und Server per SNMP abfragen und die Einstellungen und OIDs dokumentieren. Die OIDs stammen aus Standard-MIBs [W], die Drucker-OIDs sind belegt [Q: PRINTER-MIB bei Observium].

| Gesucht | Objekt und OID | Hinweis |
| --- | --- | --- |
| Name der Interfaces | ifDescr `1.3.6.1.2.1.2.2.1.2` | liefert den Index, zum Beispiel für das WAN-Interface |
| MAC-Adressen | ifPhysAddress `1.3.6.1.2.1.2.2.1.6` | pro Interface-Index |
| Geschwindigkeit | ifSpeed `1.3.6.1.2.1.2.2.1.5` (bit/s), ifHighSpeed `1.3.6.1.2.1.31.1.1.1.15` (Mbit/s) | ifSpeed ist bei schnellen Ports begrenzt, dann ifHighSpeed nehmen |
| Fehler am Switchport | ifInErrors `1.3.6.1.2.1.2.2.1.14`, ifOutErrors `1.3.6.1.2.1.2.2.1.20` | Zähler, besser den Verlauf ansehen |
| Freier Platz auf C: | hrStorage `1.3.6.1.2.1.25.2.3.1` mit Descr .3, AllocationUnits .4, Size .5, Used .6 | frei = (Size - Used) mal AllocationUnits |
| Tonerstand | prtMarkerSuppliesLevel `1.3.6.1.2.1.43.11.1.1.9`, MaxCapacity `...1.8` | Prozent = Level / MaxCapacity mal 100 |

- Auf Windows muss dafür der SNMP-Dienst mit Community laufen (Abschnitt 2d).
- Mit Net-SNMP auf der Kommandozeile: `snmpwalk -v2c -c <Community> <IP> 1.3.6.1.2.1.2.2.1.2` zum Finden des Index, dann `snmpget -v2c -c <Community> <IP> 1.3.6.1.2.1.2.2.1.5.<Index>` [W].
- Sonderwerte beim Toner: negative Zahlen bedeuten kein Füllstand (nach meinem Wissen -2 unbekannt und -3 mindestens noch etwas vorhanden, im RFC 3805 nachlesen) [W]. Farbtoner, die ein Schwarzweiss-Drucker gar nicht hat, filtert man über die Beschreibung der Verbrauchsmaterialien, damit keine erfundenen Werte entstehen.
- Merksatz aus dem Script: «Wer Mist misst, misst Mist.» Prüfe jeden Wert auf Plausibilität.

### Zwei Vorfälle durchdenken (Aufgabe im Script)
Beide Szenarien verlangen eine kurze Dokumentation: Vorgehen, geprüfte Systeme und Sensoren, Begründung. Meine Antworten sind [W]:
1. **Download ist langsam, aber nur bei einem Benutzer.** Eingrenzen: Ist es lokal oder die Infrastruktur? Andere Clients im Vergleich testen. Im Monitoring die Auslastung des WAN-Interface ansehen. Den Switchport des Clients auf Fehler und Duplex prüfen (ifInErrors, Geschwindigkeit). Auf dem Client CPU, RAM, Datenträger und DNS-Auflösung prüfen. Weil nur ein Benutzer betroffen ist, liegt es eher am Client oder am Port als am Internetzugang.
2. **Backup meldet zu wenig freien Platz auf dem AD-Server.** Im Monitoring den Verlauf des freien Platzes ansehen und herausfinden, was gewachsen ist (Logs, Backup-Versionen, Update-Reste). Mit `wbadmin get versions` die alten Sicherungen ansehen, aufräumen oder die Platte erweitern. Den Schwellwert früher alarmieren lassen, zum Beispiel Warnung bei 20 Prozent und kritisch bei 10 Prozent frei. Das Ereignis dokumentieren.

### Trial-Lizenz im Lab
Die Evaluationsversionen laufen ab. Das Script nennt `slmgr.vbs -rearm` zum Verlängern, sonst fährt der Server nach einer Stunde herunter [Q: Script ICT-BZ]. Nach dem Rearm ist ein Neustart nötig, die Anzahl der Rearm-Vorgänge ist begrenzt [W].

## 2t. Lab-Ablauf: AD DS, DNS und DHCP «gemäss Konzept» (Anleitung der älteren Lernenden)

Quelle: PDF «Umsetzung Konzept» (22 Seiten, ohne Autor und Anbieter, Kachel 4 «ADDS, DNS, DHCP Installation und Konfiguration», 8 Stunden, Ergebnis «lauffähige Domäne, Betriebsdoku», Lernziel Handlungsziel 2). Die Anleitung baut ein Lab mit neuem Forest `it-champs.local`, einem DC und einem Windows-11-Client. Ich fasse sie in eigenen Worten zusammen. Die IP-Werte sind Beispielwerte aus dem PDF, in deinem Kurs gelten die Werte deines Konzepts.

**Grundsatz:** «Gemäss Konzept umsetzen» heisst, exakt die Werte des Konzepts zu verwenden (IP-Tabelle, DNS- und DHCP-Vorgaben, OU-Ziele), keine eigenen. Das passt zum LBV-Kriterium «AD, DNS, DHCP konfigurieren» (siehe Vorbereitung 2a).

**Ablauf**
1. Voraussetzungen prüfen: Hostname des künftigen DC gesetzt, statische IPv4, DNS zeigt auf die eigene IP, Konzept vorhanden, keine «wild» vorinstallierten Rollen.
2. Rolle AD DS installieren (Server Manager oder `Install-WindowsFeature AD-Domain-Services`). Zum Domain Controller hochstufen mit neuem Forest, DNS-Server aktiv, DSRM-Passwort setzen, danach Neustart und als Domänen-Admin anmelden.
3. DNS konfigurieren: Zone AD-integriert mit sicheren dynamischen Updates, Host-Einträge (A), Alias (CNAME), eine Sonderzone oder bedingte Weiterleitung, Reverse-Zone mit PTR, Zonentransfers aus, DNS horcht nur auf die DC-IPv4 (in der DNS-Konsole unter Servereigenschaften, Schnittstellen) [W].
4. DHCP-Rolle installieren, im AD autorisieren, Scope mit Optionen und Reservierung anlegen.
5. Client umbenennen, der Domäne beitreten, neu starten, mit einem Domänenbenutzer anmelden. Das Computerobjekt liegt zuerst im Container `Computers` und wird in die Ziel-OU verschoben.
6. AD-Papierkorb aktivieren (empfohlen).
7. Validieren und dokumentieren (5 bis 7 Screenshots, siehe unten).

**Beispielwerte aus dem PDF**

| Thema | Wert |
| --- | --- |
| Forest und Zone | `it-champs.local` (AD-integriert, sichere Updates), dazu `_msdcs.it-champs.local` |
| DC (Name «James») | 192.168.100.10/24, DNS = eigene IP |
| Fileserver (Name «May») | 192.168.100.20, per DHCP-Reservierung (MAC), nicht statisch |
| DNS-Einträge | A für `james` und `may`, CNAME `1st` zeigt auf `may`, Sonderzone `richard.local` mit A-Eintrag `www` auf 192.21.129.101 (oder Conditional Forwarder) |
| Reverse-Zone | `100.168.192.in-addr.arpa` mit PTR für `james` und `may` |
| DHCP-Scope «IT-Champs» | Range .100 bis .200, Exclusion .1 bis .99, Lease 42 Stunden |
| DHCP-Optionen | 003 Router 192.168.100.1, 006 DNS 192.168.100.10, 015 DNS-Suffix `it-champs.local` |

Hinweis zum Beispiel: Eine Exclusion .1 bis .99 liegt ausserhalb eines Scopes, der erst bei .100 beginnt. Sie wäre dann überflüssig oder wird vom Server abgelehnt (nach meinem Wissen, nicht geprüft) [W]. Sinnvoll wäre ein Scope ab .1 mit dieser Exclusion. Prüfe das bei deinem Konzept.

**Befehle** (belegte Befehle stehen bei Microsoft Learn, die übrigen sind [W])
```powershell
# DNS
Add-DnsServerResourceRecordA -Name "james" -ZoneName "it-champs.local" -IPv4Address 192.168.100.10 -CreatePtr
Add-DnsServerResourceRecordCName -Name "1st" -HostNameAlias "may.it-champs.local" -ZoneName "it-champs.local"   # [Q]
Add-DnsServerPrimaryZone -NetworkId "192.168.100.0/24" -ReplicationScope Domain                                  # Reverse-Zone [W]
Add-DnsServerConditionalForwarderZone -Name "richard.local" -MasterServers 192.21.129.101 -ReplicationScope Domain   # [Q]
Set-DnsServerPrimaryZone -Name "it-champs.local" -SecureSecondaries NoTransfer                                   # [Q] keine Zonentransfers

# DHCP
Add-DhcpServerv4Scope -Name "IT-Champs" -StartRange 192.168.100.100 -EndRange 192.168.100.200 -SubnetMask 255.255.255.0
Set-DhcpServerv4Scope -ScopeId 192.168.100.0 -LeaseDuration 1.18:00:00                                            # [Q] 42 Stunden = 1 Tag 18 Stunden
Set-DhcpServerv4OptionValue -ScopeId 192.168.100.0 -Router 192.168.100.1 -DnsServer 192.168.100.10 -DnsDomain "it-champs.local"
Add-DhcpServerv4Reservation -ScopeId 192.168.100.0 -IPAddress 192.168.100.20 -ClientId "00-15-5D-01-02-03" -Name "may"
Add-DhcpServerInDC -DnsName "james.it-champs.local" -IPAddress 192.168.100.10                                     # im AD autorisieren

# AD
Enable-ADOptionalFeature -Identity 'Recycle Bin Feature' -Scope ForestOrConfigurationSet -Target 'it-champs.local'   # [Q] forestweit, einmalig
```

**Validierung**
```cmd
ipconfig /all
nslookup it-champs.local
nslookup james.it-champs.local
ping 192.168.100.10
```
```powershell
Get-ADDomain
Get-DnsServerZone
Get-DhcpServerv4Scope | Format-Table ScopeId, StartRange, EndRange, LeaseDuration
Resolve-DnsName may.it-champs.local
Get-DhcpServerv4Lease -ScopeId 192.168.100.0
```
Erwartet: Lease aus deinem Scope, DNS-Suffix stimmt, Namen lösen auf, Ping zum DC geht, Client liegt in der richtigen OU, die Zone `_msdcs` ist vorhanden.

**Typische Fehler laut Anleitung**
- Falscher DNS am Client, der Domänenbeitritt scheitert. Fix: DHCP-Option 006 auf die DC-IP.
- Keine Reverse-Zone, die Auflösung ist unvollständig. Fix: Reverse-Zone und PTR-Einträge.
- Rollenreihenfolge missachtet. Fix: zuerst AD DS hochstufen, dann DNS und DHCP sauber konfigurieren.
- DHCP nicht autorisiert, es gibt keine Leases. Fix: im AD autorisieren.
- OU-Löschschutz blockiert das Aufräumen. Fix: in den Advanced Features «Protect from accidental deletion» gezielt entfernen.
- Ohne DHCP-Antwort vergibt Windows eine Adresse 169.254.x.x (APIPA), ein klares Zeichen für ein DHCP- oder Netzproblem. Auf dem Server findest du mit `Get-DhcpServerv4Lease -ScopeId <Scope>` das Lease, ohne Treffer prüfst du Scope, Exclusion, DHCP-Dienst und Netzweg.

**Abgabe der Betriebsdokumentation (laut Anleitung)**
Fünf bis sieben Screenshots: Server Manager mit installiertem AD DS, DNS-Manager mit Zonen (inklusive Reverse), DNS-Records, DHCP-Scope-Eigenschaften, DHCP-Reservierung, ADUC mit OU-Gerüst und Client in der Ziel-OU, Client mit `ipconfig /all` und `nslookup`. Selbstcheck vor der Abgabe: Stimmt jeder Wert mit dem Konzept? Sind die Schritte für Aussenstehende nachvollziehbar (Screens und kurze Begründung)? Laufen `nslookup` und `ping`? Zeigt `Get-DhcpServerv4Scope` die richtige Range und Lease-Dauer?

**Internet fürs Lab (Router-Varianten in der Anleitung)**
Die Anleitung nutzt VMware Player mit zwei Netzen: WAN per NAT (VMnet8, bekommt DHCP vom Host) und LAN als Host-only (VMnet1, zum Beispiel 192.168.50.0/24). Alle Lab-VMs hängen nur am LAN-Netz. Drei Varianten:
- Windows-Router-VM mit zwei Netzwerkkarten und Rolle Remote Access (RRAS) mit NAT: Variante A, empfohlen für ein Lab mit GUI.
- Windows 10 oder 11 oder Server als Router mit PowerShell-NAT (`New-NetNat`): Variante B, schnell und einfach.
- pfSense als Firewall-VM: Gateway der Lab-VMs ist die pfSense-LAN-Adresse, den DNS liefert weiterhin der AD-DNS. Die Firewallregeln startet man offen (Pass, LAN net, any) und engt sie nach den Tests ein, zum Beispiel DNS nur zum DC.

Wichtig dabei: Windows-ICS (Internetverbindungsfreigabe) nicht benutzen, es erzwingt ein eigenes Netz und einen eigenen DHCP und kollidiert mit dem AD-DHCP. Die Clients dürfen nur den AD-DNS haben und nicht den Router als DNS. Der AD-DNS bekommt einen Forwarder (zum Beispiel 1.1.1.1 oder 8.8.8.8). Wenn du wie in Abschnitt 2n mit Hyper-V arbeitest, gilt dasselbe Prinzip mit einem internen virtuellen Switch und einer Router-VM.

## 2u. So ist eine echte Betriebsdokumentation aufgebaut (Musterdoku der älteren Lernenden)

Quelle: Word-Datei «Muster Betriebsdokumentation». Sie ist eine fiktive Kundendoku (Dienstleister «ICT Security GmbH», Kunde «Block & Chain GmbH», Version 1.2.0 vom 01.01.2023, mit Titelblatt, Versionstabelle und Inhaltsverzeichnis). Ich fasse Aufbau und Lehren in eigenen Worten zusammen. Zusammen mit meinem Muster in Abschnitt 2r gibt dir das ein gutes Bild davon, was du in der Prüfung zu lesen und zu ergänzen bekommst.

**Gliederung**
1. Quickinfo: Wer ist der Kunde, was betreut der Dienstleister und was nicht, wo liegen die Passwörter und wer gibt sie frei, Geheimhaltung.
2. Physische Infrastruktur: Standorte mit Ansprechpersonen, Kundenkontakte mit Vertrag, Erreichbarkeit und SLA, Rackaufbau, Raumplan.
3. Netzwerke: Remote-Zugriff, physische und virtuelle Server (je Zeile IP, Priorität, Betriebssystem, Disk, Funktion), Drucker, Netzwerkkomponenten, VLANs mit DHCP-Bereichen, Internet, Firewall, Switches, WLAN, Patchpanel.
4. Server: pro Server Hardware, Aufgabe und Besonderheiten. Dazu eine Backup-Tabelle (Häufigkeit, Startzeit, Anzahl behaltener Sicherungen, Kommentar), der Verweis auf den Notfallplan und das Kapitel Wartungsarbeiten.
5. Software: Liste mit Installationsort, Lizenz und Vertrag, pro Software der Konfigurationsstand mit Abweichungen vom Standard, dazu die Sensortabelle des Monitorings (PRTG).
6. Clients.

**Was ich daraus lerne**
- Passwörter stehen nicht in der Doku. Es steht nur, wo sie liegen und wer sie freigibt. Das deckt sich mit Lernkarte 3.
- Abweichungen vom Standard werden ausdrücklich festgehalten. In der Musterdoku nutzt zum Beispiel der CTO den Domänen-Administrator statt eines persönlichen Administrators. Das ist sicherheitstechnisch heikel (gemeinsames Konto, keine Nachvollziehbarkeit) und gehört in der Analyse benannt [W].
- Verantwortungsgrenzen sind klar geregelt: Telefonie- und Threema-Server betreut ein Dritter, die Firma betreut nur die Windows-Updates auf dem Anwendungsserver.
- Die Monitoring-Tabelle nennt je System die Sensoren (zum Beispiel Ping, CPU, RAM, Speicherkapazität, Dienste wie AD, DNS, DHCP, SQL, Backup-Job, Hardware-Health). Schwellwerte stehen dort nicht. Die musst du in der Prüfung selbst festlegen und begründen (LBV, schriftlicher Teil, Schwellwerte 10 bis 20 Prozent) [W].
- Backups: Domänencontroller und Datenbankserver zweimal täglich mit 60 Sicherungen, Anwendungs- und Telefonieserver täglich mit 30, Infrastruktur- und Citrix-Server samstags voll mit 4 Sicherungen. Die Kopie geht auf ein zweites NAS und ins externe Rechenzentrum, alles verschlüsselt. Das passt zum Abschnitt «Backup» in der Vorbereitung.

**Wartungsablauf aus der Musterdoku (gute Vorlage für Handlungsziel 3)**
Wartungsfenster: Samstag um 13:00 im Anschluss an den Patch Tuesday (die Formulierung im Original ist unklar). Kritische Updates ausserhalb des Fensters nur mit Freigabe des CTO. Bei fehlerhaften Microsoft-Updates den CTO informieren und ein neues Fenster vereinbaren. Der Ablauf lässt sich so ordnen:
1. Backup kontrollieren (ob erfolgreich), Fehler beheben, danach die Backup-Jobs pausieren.
2. Zustand prüfen: Hardware (iLO, Speichersystem), USV (Selbsttest), freier Platz auf den Disks, Ereignisprotokolle.
3. Snapshots der VMs erstellen.
4. Windows-Server updaten.
5. Dienste kontrollieren, ob alles wieder online ist (inklusive Anwendungen, die manuell gestartet werden müssen).
6. NAS prüfen und updaten, Firewall-Konfiguration sichern und Firewalls updaten, Site-to-Site-VPNs prüfen.
7. Snapshots löschen, Backup wieder aktivieren und starten.
8. Bericht an den CTO: alles in Ordnung oder Probleme.
Das ist der Patch-Zyklus aus Abschnitt 2l in einer echten Umgebung. Vergleiche: Planen (Fenster, Freigabe), Testen (Snapshot als Rückfallpunkt), Installieren, Kontrollieren, Dokumentieren (Bericht).

**Auffälligkeiten in der Musterdoku (Training für «Betriebsdokumentation analysieren»)** [W]
- Die IP-Adresse 10.0.0.24 ist doppelt vergeben (Citrix Controller und erster Citrix-PVS-Server). Die Serverreihe wäre sonst .24, .25, .26 und so weiter.
- Kleine Fehler und Widersprüche: Überschrift «BAC-NAS01 & BAC-NAS01» (zweites sollte NAS02 sein), «BAK-APK01» statt «BAC-APK01», beim Synology-NAS steht ein QNAP-Modell dabei.
- Es fehlen Schwellwerte, und Wartungs- und Backup-Angaben sind verstreut. Beim Lesen lohnt sich die Frage: Was ist unvollständig, widersprüchlich oder veraltet?

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

Wenn dein Kurs wie das Script von ICT-BZ läuft: Nimm an Tag 10 statt PRTG die Zabbix-Konzepte aus Abschnitt 2s und nutze die freien Tage für Bulk-User, FSRM, Hashwerte, Windows Admin Center und den MIB-Browser (ebenfalls 2s, Lernkarten 79 bis 98).

Wenn du Zeit übrig hast: Abschnitt 2t ist ein fertiger Lab-Ablauf für AD, DNS und DHCP (guter Ersatz für Tag 5 und 6), Abschnitt 2u zeigt, wie eine echte Betriebsdokumentation aufgebaut ist (hilft an Tag 8). Die Lernkarten 99 bis 119 stammen aus dem Quiz und der Anleitung der älteren Lernenden.

Hinweis zur Prüfung: Laut Leistungsbeurteilung zählt der Praxisteil 60 Prozent und der schriftliche Teil 40 Prozent. Übe darum auch das Begründen (warum dieser Schwellwert, warum diese Rechtevergabe).
