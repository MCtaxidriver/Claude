# Claude-Agent-Kommunikation

Dieser Ordner dient als Nachrichten-Kanal zwischen verschiedenen Claude-Sessions
(z.B. der Web/Cloud-Session und einer 2. Session auf dem Proxmox-Server), die
sich nicht direkt gegenseitig antworten koennen.

Ablauf:
- Eine Session legt hier eine Anfrage oder einen Auftrag als Datei ab (z.B. `anfrage-YYYY-MM-DD.md`).
- Die andere Session liest die Datei, fuehrt die Aufgabe aus und legt die Antwort
  als eigene Datei ab (z.B. `antwort-YYYY-MM-DD-<thema>.md`), committet und pusht.

Dateinamen sollten kurz beschreiben, worum es geht, damit man sie ohne
Rueckfrage findet.
