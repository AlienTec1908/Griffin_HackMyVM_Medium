# Griffin - HackMyVM (Medium)
 
![Griffin.png](Griffin.png)

## Übersicht

*   **VM:** Griffin
*   **Plattform:** HackMyVM (https://hackmyvm.eu/machines/machine.php?vm=Griffin)
*   **Schwierigkeit:** Medium
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 4. August 2025
*   **Original-Writeup:** https://alientec1908.github.io/Griffin_HackMyVM_Medium/
*   **Autor:** Ben C.

## Kurzbeschreibung

Dieses Writeup beschreibt den vielschichtigen Lösungsweg für die Maschine "Griffin". Der initiale Zugriff wurde durch die Entdeckung eines versteckten Debug-Endpunkts in einer Python-Webanwendung erreicht. Ein an anderer Stelle geleakter Token ermöglichte die Ausnutzung einer Remote Code Execution (RCE)-Schwachstelle, um eine Shell als Benutzer `lois` zu erhalten. Die Privilegienerweiterung war ein mehrstufiger Prozess, der mit Port-Forwarding begann, um einen internen Webdienst freizulegen. Dessen CAPTCHA-geschütztes Login wurde mit einem benutzerdefinierten Python-Skript per Brute-Force angegriffen, was zu Anmeldeinformationen für den Benutzer `meg` führte. Eine Kette von `sudo`-Fehlkonfigurationen und das Wechseln zwischen Benutzern (`lois` -> `meg` -> `peter`) führte schließlich dazu, dass ein Benutzer den Texteditor `mg` als `root` ausführen durfte. Dies wurde missbraucht, um die `/etc/sudoers`-Datei zu bearbeiten und volle `root`-Rechte zu erlangen.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `nikto`
*   `curl`
*   `wfuzz`
*   `netcat`
*   `ssh`
*   `socat`
*   `feroxbuster`
*   `git`
*   `python3`
*   `sudo`
*   Standard Linux-Befehle (`ls`, `cat`, `id`, etc.)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Griffin" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Enumeration:**
    *   Identifizierung der Ziel-IP (`192.168.2.169`) mit `arp-scan`.
    *   Ein `nmap`-Scan offenbarte Port 22 (SSH) und einen Python/Werkzeug-Webserver auf Port 8080.

2.  **Web Enumeration:**
    *   Die manuelle Untersuchung und ein `wfuzz`-Scan der Web-App auf Port 8080 deckten zwei versteckte Endpunkte auf: `/debug` (benötigt ein Token) und `/info`.
    *   Der Aufruf von `/info` enthüllte ein gültiges "Diagnostic token".

3.  **Initial Access (Remote Code Execution):**
    *   Der gefundene Token wurde zusammen mit einem `run`-Parameter an den `/debug`-Endpunkt übergeben, was eine RCE-Schwachstelle bestätigte.
    *   Ein `busybox nc`-Payload wurde über die RCE ausgeführt, um eine Reverse Shell als Benutzer `lois` zu erhalten.

4.  **Privilege Escalation (von `lois` zu `meg`):**
    *   Als `lois` wurde ein interner Webdienst auf `localhost:80` entdeckt. Mittels `socat` wurde dieser Port nach außen weitergeleitet.
    *   Der interne Dienst hatte ein CAPTCHA-geschütztes Login. Ein benutzerdefiniertes Python-Skript mit `ddddocr` (OCR-Bibliothek) wurde verwendet, um das CAPTCHA zu umgehen und einen Brute-Force-Angriff mit `rockyou.txt` durchzuführen.
    *   Ein erfolgreicher Login (Benutzer `brian`, Passwort `savannah`) setzte ein `auth_token`-Cookie. Die Base58/Base64-Dekodierung dieses Tokens enthüllte die Anmeldedaten `meg:lovelyfamily`.
    *   Anschließend erfolgte ein SSH-Login als `meg`.

5.  **Privilege Escalation (von `meg` zu `peter` zu `root`):**
    *   Informationen, die zuvor als `lois` aus einer Log-Datei (`/root/startup.log`) gelesen wurden, enthielten das Passwort für den Benutzer `peter`.
    *   Mit `su` wurde zum Benutzer `peter` gewechselt.
    *   `sudo -l` für `peter` zeigte, dass er den Texteditor `mg` als `root` auf jede beliebige Datei ausführen durfte.
    *   Diese Berechtigung wurde genutzt, um `/etc/sudoers` zu bearbeiten (`sudo mg /etc/sudoers`) und dem Benutzer `peter` volle `sudo`-Rechte zu geben.
    *   Mit `sudo su` wurde schließlich eine `root`-Shell erlangt.

## Wichtige Schwachstellen und Konzepte

*   **Remote Code Execution (RCE):** Ein schlecht gesicherter Debug-Endpunkt, dessen Schutzmechanismus (Token) an anderer Stelle offengelegt wurde, führte zu direkter Befehlsausführung.
*   **CAPTCHA Bypass & Brute-Force:** Ein schwaches CAPTCHA-System, das durch OCR automatisch gelöst werden konnte, ermöglichte einen erfolgreichen Wörterbuchangriff.
*   **Insecure Credential Storage:** Klartext-Anmeldeinformationen wurden (nach einfacher Dekodierung) in einem Cookie auf der Client-Seite gespeichert.
*   **Sudo Misconfiguration (Editor Abuse):** Der finale Eskalationsvektor war eine unsichere `sudo`-Regel, die es einem Benutzer erlaubte, einen einfachen Texteditor als `root` auszuführen, was die Bearbeitung kritischer Systemdateien wie `/etc/sudoers` ermöglichte.
*   **Port Forwarding:** Die Nutzung von `socat`, um einen nur lokal erreichbaren Dienst von außen zugänglich zu machen und die Angriffsfläche zu erweitern.

## Flags

*   **User Flag (`/home/lois/user.txt`):** `flag{user-f6b63474e7cc20b0893a82beb9e3b3fd}`
*   **Root Flag (`/root/root.txt`):** `flag{root-be93b7d7f0a30d5159c0460874e6e015}`

## Tags

`HackMyVM`, `Griffin`, `Medium`, `RCE`, `Sudo`, `Privilege Escalation`, `CAPTCHA`, `Port Forwarding`, `Python`, `Linux`, `Web`
