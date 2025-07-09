# Atom - HackMyVM Writeup

![Atom Icon](Atom.png)

## Übersicht

*   **VM:** Atom
*   **Plattform:** [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Atom)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 23. Juni 2025
*   **Original-Writeup:** https://alientec1908.github.io/Atom_HackMyVM_Easy/
*   **Autor:** Ben C.

---

**Disclaimer:**

Dieser Writeup dient ausschließlich zu Bildungszwecken und dokumentiert Techniken, die in einer kontrollierten Testumgebung (HackTheBox/HackMyVM) angewendet wurden. Die Anwendung dieser Techniken auf Systeme, für die keine ausdrückliche Genehmigung vorliegt, ist illegal und ethisch nicht vertretbar. Der Autor und der Ersteller dieses README übernehmen keine Verantwortung für jeglichen Missbrauch der hier beschriebenen Informationen.

---

## Zusammenfassung

Die Box "Atom" war eine interessante Herausforderung, bei der der initiale Zugriff nicht über die typischen, bekannten Ports erfolgte. Die Enumeration zeigte nur SSH auf Port 22 als offenen TCP-Port. Entscheidend war hier der UDP-Scan, der den Port 623 (IPMI) als offen identifizierte. Die Ausnutzung einer IPMI-Schwachstelle ermöglichte das Auslesen und Knacken von Benutzer-Hashes, darunter auch SSH-Zugangsdaten.

Der Login via SSH als der gefundene Benutzer war der nächste Schritt. Auf dem System als dieser Benutzer wurde nach Möglichkeiten zur Privilegieneskalation gesucht. Dabei wurde der Root-Passwort-Hash gefunden und geknackt, was den direkten Wechsel zum Root-Benutzer ermöglichte.

## Technische Details

*   **Betriebssystem:** Debian (basierend auf Nmap-Erkennung und SSH-Banner)
*   **Offene Ports:**
    *   `22/tcp`: SSH (OpenSSH 9.2p1)
    *   `623/udp`: IPMI / asf-rmcp

## Enumeration

1.  **ARP-Scan:** Identifizierung der Ziel-IP (192.168.2.58) im Netzwerk.
2.  **Nmap Scan (TCP):** Bestätigung, dass nur Port 22 (SSH) auf TCP offen ist.
3.  **Nmap Scan (UDP):** Identifizierung des offenen/gefilterten UDP-Ports 623 (IPMI/asf-rmcp). Dies war der kritische Port für den initialen Zugriff.

## Initialer Zugriff (IPMI & SSH)

1.  **IPMI Schwachstelle:** Port 623 läuft der IPMI-Dienst. Eine bekannte Schwachstelle (u.a. CVE-2013-4786) in älteren IPMI-Implementierungen erlaubt das Auslesen von Benutzerinformationen und Passwort-Hashes ohne Authentifizierung.
2.  **Hash-Dump und Cracking:** Das Metasploit-Modul `auxiliary/scanner/ipmi/ipmi_dumphashes` wurde verwendet, um Hashes vom IPMI-Dienst zu extrahieren. Mit einer großen Wordlist (`rockyou.txt`) wurden zahlreiche Hashes geknackt, die verschiedenen Benutzern zugeordnet waren (z.B. `admin`, `analiese`, `briella`, etc.).
3.  **SSH Credential Fund:** Unter den geknackten Paaren wurden Anmeldedaten für SSH gefunden. Nach einem Versuch mit `admin:cukorborso` (geknackt, aber nicht für SSH nutzbar) wurde eine Brute-Force-Attacke mit Hydra und der Liste aller geknackten IPMI-Credentials gegen den SSH-Dienst (Port 22) durchgeführt.
4.  **Ergebnis:** Die Anmeldedaten `onida:jiggaman` waren für den SSH-Login erfolgreich.

## Privilegieneskalation (onida -> root)

1.  **Systemerkundung als `onida`:** Nach dem SSH-Login als `onida` wurde das System erkundet, insbesondere nach bekannten SUID-Binaries oder anderen Auffälligkeiten.
2.  **Root-Hash Fund und Cracking:** Ein Passwort-Hash der Form `$2y$10$...` wurde gefunden. Dies ist typischerweise ein bcrypt-Hash aus der `/etc/shadow`-Datei. Mit John the Ripper und der `rockyou.txt` Wordlist wurde dieser Hash geknackt, was das Root-Passwort `madison` ergab. (Die Methode, wie onida an diesen Hash kam, ist im Protokoll nicht explizit gezeigt, aber das Ergebnis impliziert, dass onida Lesezugriff auf `/etc/shadow` oder eine vergleichbare Datei hatte).
3.  **Wechsel zu Root:** Mit dem geknackten Passwort `madison` konnte mittels des Befehls `su root` zum Root-Benutzer gewechselt werden.
4.  **Ergebnis:** Erfolgreiche Privilegieneskalation zum Root-Benutzer.

## Flags

*   **root.txt:** `d3a4fd660f1af5a7e3c2f17314f4a962` (Gefunden unter `/root/root.txt`)

---
