# ÜK 188: Spickzettel (eine Seite)

Kurzfassung aus `UEK-188-Vorbereitung.md` und `UEK-188-Lernhilfen.md` (Stand 14.10.2026, mit Ergänzungen aus dem Script von ICT-BZ). Alle Details und Quellen stehen dort.

## Modul
Services betreiben, warten und überwachen. Objekt: Server mit File-, Print-, DHCP-, DNS- und Verzeichnis-Services im LAN. 5 Tage. Prüfung: 60 % Praxis (2 h), 40 % schriftlich (1 h). Geprüft werden AD, DNS, DHCP, Monitoring (auch Reports und Maps), Updates und vor allem Berechtigungen (bis 40 % Praxis, bis 30 % schriftlich). Du bekommst eine Betriebsdokumentation vorgegeben und ergänzt sie.

## Die 6 Handlungsziele
1. Mit der Betriebsdoku Systeme analysieren
2. Services mit Bordmitteln überwachen und betreiben
3. Updates und Patches testen und einspielen, Doku nachführen
4. Systeme ins Monitoring einbinden (SNMP, WMI), Schwellwerte und Alarme
5. Berechtigungen nach Konzept verwalten und dokumentieren
6. Anpassungen auf Umsystemen definieren (DNS, Firewall)

## Merksätze
- **Patch:** Planen, Beschaffen (seriöse Quelle, Hash oder Signatur), Testen, Installieren (Wartungsfenster, Backup, Rollback), Dokumentieren.
- **Monitoring:** Was überwachen, welche Methode, Baseline messen, Schwellwerte mit Aktion, Alarm testen, Ergebnis dokumentieren.
- **SNMP:** v1 und v2c nur Community-String im Klartext. v3 mit Benutzer, Authentifizierung und Verschlüsselung. Port UDP 161. Windows-Agent: eigene Community, nur lesen, erlaubte Hosts einschränken.
- **AGDLP:** Accounts, globale Gruppen, domänenlokale Gruppen, Permissions. Nie Benutzer direkt in die ACL.
- **Rechte:** Freigabe gegen NTFS gilt die restriktivere Ebene. Verweigern schlägt Erlauben.
- **Backup:** Regelmässig, extern trennen, Restore testen.
- **Umsystem-Auftrag:** Quelle, Ziel, Protokoll, Port, Zweck, Befristung, Testverfahren.

## Befehle, die du im Schlaf können solltest
```powershell
Get-Service "dhcp*"                          # Dienste
Get-HotFix                                   # installierte Updates
Get-Counter "\Processor(_Total)\% Processor Time" -SampleInterval 2 -MaxSamples 3
Get-WinEvent -ListLog *                      # Ereignisprotokolle
Get-CimInstance Win32_Process                # WMI/CIM
Install-WindowsFeature SNMP-Service,SNMP-WMI-Provider -IncludeManagementTools
Add-DhcpServerv4Scope -Name "Lab" -StartRange 10.10.10.1 -EndRange 10.10.10.254 -SubnetMask 255.255.255.0
Add-DnsServerResourceRecordA -Name "host23" -ZoneName "contoso.com" -IPv4Address "172.18.99.23" -CreatePtr
New-SmbShare -Name 'Data' -Path 'D:\Data' -ChangeAccess 'CONTOSO\Finance'
New-NetFirewallRule -DisplayName "SNMP von MON01" -Direction Inbound -Protocol UDP -LocalPort 161 -RemoteAddress 192.168.10.20 -Action Allow
Checkpoint-VM -Name SRV01 -SnapshotName BeforeUpdates
```
```cmd
icacls D:\Data /grant "CONTOSO\Finance":(OI)(CI)M
gpupdate /force
gpresult /r
ipconfig /release  &  ipconfig /renew
```

## Event-IDs
- 7036 Dienst gestartet oder gestoppt, 7034 Dienst unerwartet beendet
- 4624 Anmeldung ok, 4625 Anmeldung fehlgeschlagen, 4740 Konto gesperrt
- 41 Neustart ohne sauberes Herunterfahren

## Zahlen und Fakten
- Subnetting /29: 8 Adressen, 6 Hosts. 10.89.14.24/29 hat Broadcast .31, Hosts .25 bis .30.
- DHCP-Lease Standard 8 Tage. GPO-Hintergrundaktualisierung alle 90 Minuten plus bis 30.
- Prozessor-Warteschlange dauerhaft über 2 deutet auf Überlast.
- Ports: RDP 3389, DNS 53, DHCP 67 (Server) und 68 (Client), SNMP 161 (Abfrage) und 162 (Traps). Adresse 169.254.x.x bedeutet APIPA, also keine DHCP-Antwort.
- Reihenfolge im Lab: AD DS installieren und hochstufen, dann DHCP installieren und im AD autorisieren. Passwort-Rücksetzung läuft über den PDC-Emulator. Eine Freigabe mit `$` am Namen ist versteckt, aber nicht geschützt.
- Windows Server 2025: WMIC nur noch als Feature on Demand, PowerShell 2.0 entfernt, WSUS nicht mehr aktiv entwickelt.
- Passwörter laut NCSC: mindestens 12 Zeichen, nicht im Wörterbuch, pro Dienst ein eigenes, Passwortmanager.

## Falls dein Kurs wie das Script von ICT-BZ läuft (Details in Lernhilfen 2s)
```powershell
Get-WindowsFeature | Where-Object Installed                                   # Rollen und Features für die Doku
Set-SmbShare -Name Firma -FolderEnumerationMode AccessBased                   # nur erlaubte Ordner anzeigen
Install-WindowsFeature FS-Resource-Manager -IncludeManagementTools            # FSRM
New-FsrmFileScreen -Path "D:\Daten" -IncludeGroup "Audio and Video Files" -Active   # Medien blockieren
Get-FileHash .\datei.iso -Algorithm SHA256                                    # Hash prüfen (oder certutil -hashfile)
```
- SNMP: Polling UDP 161, Traps UDP 162. v2c in der Praxis üblich, Community im Klartext. OID = Zahlenfolge, MIB = Übersetzung.
- Toner in Prozent: prtMarkerSuppliesLevel (1.3.6.1.2.1.43.11.1.1.9) durch MaxCapacity (...1.8) mal 100.
- Zabbix: Host, Template, Item, Trigger. Schweregrade Not classified, Information, Warning, Average, High, Disaster.
- Störungsfall: Erkennung, Klassifizierung, Analyse, Behebung (Fix oder Workaround), Abschluss.
- Kennwortrichtlinie: Default Domain Policy, Kontorichtlinien. Eine pro Domäne.

## Offene Fragen ans Sekretariat
Windows oder Linux? Welche Monitoring-Tools? Dürfen die Kursunterlagen (laut AGB am ersten Tag verteilt) vorab eingesehen werden? Zugang zur Lernplattform vor dem Kurs? Eigenes Notebook? Genaue Leistungsbeurteilung? Ist eine handschriftliche A4-Seite als Hilfsmittel erlaubt? sekretariat@ict-learnfactory.ch, +41 31 521 00 90.

## Muster zum Üben (in `UEK-188-Lernhilfen.md`)
- 2p Berechtigungskonzept mit Matrix und Begründungen (grösster Prüfungsposten)
- 2q IP-Adress- und Namenskonzept für AD, DNS, DHCP
- 2r Betriebsdokumentation mit Änderungsprotokoll zum Nachführen
- 2k, 2m Alarme, Reports und Maps in der Leistungsüberwachung und in PRTG
