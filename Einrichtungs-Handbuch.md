# 📘 ZeroLab: OpenClaw Einrichtungs-Handbuch

### Version: 1.0 (ZeroLab Edition 2026) Fokus: Schritt-für-Schritt Installation & Konfiguration
----

#### 1. Einleitung und Zielsetzung

Dieses Dokument dient als technisches Handbuch zum vollständigen Neuaufbau des ZeroLab Agenten. Im Gegensatz zur allgemeinen Projektübersicht (README.md) liegt der Fokus hier auf der Reproduzierbarkeit. Es ermöglicht Administratoren und technisch versierten Anwendern, das System Zeile für Zeile nachzubauen – von der leeren Linux-Konsole bis zum ersten "System Check" auf WhatsApp.
Das Ziel ist ein autonomer, privater KI-Assistent, der lokal gehostet wird, aber bei Bedarf auf Cloud-Ressourcen zurückgreifen kann (Hybrid-Architektur).
<br>
🏗️ Die Architektur
Das Setup basiert auf einer strikten Trennung von "Körper" (Gateway) und "Gehirn" (LLM), um Datenschutz und Performance zu optimieren.
Komponente
Spezifikation
Rolle
Host System
Proxmox VE (LXC Container)
Die Heimat des Agenten ("Körper").
Betriebssystem
Ubuntu 24.04 LTS
Basis für die Software (Node.js Runtime).
Core Software
OpenClaw (v2026.2.x)
Gateway, Logik und WhatsApp-Anbindung.
Primäres Gehirn
LM Studio (Windows/NVIDIA)
Lokale KI (Llama 3 / Qwen) für Datenschutz.
Sekundäres Gehirn
Anthropic / Google Gemini
Cloud-Fallback für komplexe Logik (optional).
Schnittstelle
WhatsApp Web
Kommunikation mit dem Nutzer.
<br>
⚠️ Sicherheits-Prämisse (Security First)
Dieses Handbuch folgt dem Zero-Trust-Ansatz, um private Daten zu schützen:

1. Trennung von Code und Geheimnissen: API-Keys und Telefonnummern werden niemals direkt in Konfigurationsdateien geschrieben, sondern über Umgebungsvariablen (.env) injiziert.

2. Zugriffskontrolle: Der Agent reagiert ausschließlich auf verifizierte Nummern (Allowlist) und erfordert ein initiales Pairing.

3. Isolation: Gruppen-Chats und unsichere Tools laufen in isolierten Umgebungen (Sandboxing/Non-Main Session).

----

#### 2. Erstellung des LXC Containers (Proxmox)

Wir erstellen einen ressourcenschonenden Container ("Unprivileged LXC"), der als Heimat für das OpenClaw-Gateway dient. Da die KI-Berechnung extern erfolgt, sind die Hardware-Anforderungen moderat.

1. Klicke in der Proxmox-Oberfläche oben rechts auf **"Create CT"**.
2. Folge dem Assistenten mit diesen Einstellungen:

| Reiter | Einstellung | Wert / Hinweis |
| :--- | :--- | :--- |
| **General** | CT ID | `999` |
| | Hostname | `pve-openclaw` |
| | Password | `******` *(Wähle ein sicheres Root-Passwort!)* |
| | Nesting | *(Optional: Haken setzen, falls Docker später gewünscht)* |
| **Template** | Storage | `a11-hdd` |
| | Template | `ubuntu-24.04-standard_24.04-amd64.tar.zst` |
| **Disks** | Storage | `a10-ssd` |
| | Disk size (GiB) | `25` *(Ausreichend für System, Logs & Datenbank)* |
| **CPU** | Cores | `1` *(Reicht für das Gateway völlig aus)* |
| **Memory** | Memory (MiB) | `2048` *(2 GB für stabilen Node.js Betrieb)* |
| | Swap (MiB) | `512` |
| **Network** | IPv4 | `Static` |
| | IPv4/CIDR | `192.168.178.239/24` |
| | Gateway | `192.168.178.1` *(Deine FritzBox/Router IP)* |
| **DNS** | Domain | *(leer lassen)* |
| | Server | *(leer lassen)* |
| **Confirm** | Start after created | **[x] Ja** (Haken setzen) |

3. Klicke auf **Finish**. Der Container wird erstellt und startet automatisch.

----

#### 3. Basis-Konfiguration des LXC Containers

Nachdem der Container gestartet ist, loggen wir uns über die Konsole (Proxmox GUI oder SSH) als `root` ein. Die folgenden Schritte bringen das System auf den aktuellen Stand und installieren notwendige Werkzeuge.

##### 3.1 System-Update & Upgrade
Zuerst aktualisieren wir die Paketquellen und installieren alle verfügbaren Updates für Ubuntu 24.04.

**Befehl:**
```bash
apt update && apt upgrade -y
```

Ausgabe (Auszug):

```bash
Processing triggers for install-info (7.1-3build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.7) ...
Processing triggers for ufw (0.36.2-6) ...
Processing triggers for man-db (2.12.0-4build2) ...
root@pve-openclaw:~#
```

#### 3.2 Installation von System-Tools (htop)
Wir installieren htop zur besseren Überwachung der Systemressourcen (CPU/RAM).

**Befehl:**
```bash
apt install -y htop
```

Ausgabe (Auszug):

```bash
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.7) ...
root@pve-openclaw:~#
```

#### 3.3 Installation von Curl
curl wird zwingend benötigt, um das OpenClaw-Installationsskript und andere Inhalte aus dem Web zu laden.

**Befehl:**
```bash
apt install -y curl
```

Ausgabe (Auszug):

```bash
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.7) ...
root@pve-openclaw:~#
```

#### 4 Einrichtung des externen Zugriffs (WinSCP / SSH)
Um Dateien (wie Konfigurationen oder Logs) bequem von einem Windows-PC mit WinSCP bearbeiten zu können, muss der direkte Root-Login via SSH aktiviert werden. Standardmäßig ist dieser bei Ubuntu aus Sicherheitsgründen oft deaktiviert oder auf "nur Key-Files" beschränkt.

#### 4.1 SSH-Konfiguration anpassen
Wir bearbeiten die Datei /etc/ssh/sshd_config.

**Befehl:**
```bash
nano /etc/ssh/sshd_config
```

Wir suchen den Abschnitt # Authentication. Änderung: Die Zeile #PermitRootLogin prohibit-password wird geändert zu PermitRootLogin yes.
Konfiguration vorher:

# Authentication:
```bash
#LoginGraceTime 2m
#PermitRootLogin prohibit-password
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

Konfiguration nachher:
# Authentication:
```bash
#LoginGraceTime 2m
PermitRootLogin yes
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

Speichern mit STRG+O, Beenden mit STRG+X.

#### 4.2 Dienst neustarten & Passwort setzen
Damit der Login funktioniert, muss der SSH-Dienst die neue Config laden und der Benutzer root benötigt ein explizites Passwort (da LXC-Container oft initial keines haben).

**Befehle:**
```bash
systemctl restart ssh
passwd
```

Ausgabe:
```bash
root@pve-openclaw:~# systemctl restart ssh
root@pve-openclaw:~# passwd
New password: 
Retype new password: 
passwd: password updated successfully
root@pve-openclaw:~#
```

Test: Der Zugriff via WinSCP auf die IP 192.168.178.239 mit User root funktioniert nun erfolgreich.

#### 5. Meilenstein: Backup des Basis-Systems
Bevor die eigentliche OpenClaw-Software installiert wird, wurde ein vollständiges Backup des sauberen, vorbereiteten Containers erstellt. Dies ermöglicht jederzeit einen schnellen Rollback auf ein frisches System ohne Neuinstallation des OS.
Durchführung in Proxmox:

1. Container 999 (pve-openclaw) auswählen.
2. Menü Backup -> Backup now.
3. Mode: Stop (für Konsistenz).
4. Storage: a11-hdd.
5. Notes: {{guestname}} LXC-komplett.

Protokoll des Backup-Vorgangs:
```bash
INFO: starting new backup job: vzdump 999 --notification-mode auto --notes-template '{{guestname}} LXC-komplett' --compress zstd --mode stop --remove 0 --storage a11-hdd --node pve
INFO: Starting Backup of VM 999 (lxc)
INFO: Backup started at 2026-02-13 15:33:59
INFO: status = running
INFO: backup mode: stop
INFO: ionice priority: 7
INFO: CT Name: pve-openclaw
INFO: including mount point rootfs ('/') in backup
INFO: stopping virtual guest
INFO: creating vzdump archive '/mnt/pve/a11-hdd/dump/vzdump-lxc-999-2026_02_13-15_33_59.tar.zst'
INFO: Total bytes written: 878131200 (838MiB, 82MiB/s)
INFO: archive file size: 245MB
INFO: adding notes to backup
INFO: restarting vm
INFO: guest is online again after 17 seconds
INFO: Finished Backup of VM 999 (00:00:17)
INFO: Backup finished at 2026-02-13 15:34:16
INFO: Backup job finished successfully
INFO: notified via target `mail-to-root`
TASK OK
```

Das System ist nun gesichert, wieder online und bereit für die Installation der Applikation.

----

#### 5. Installation der OpenClaw Software

Wir verwenden das offizielle Installationsskript, um die notwendige Laufzeitumgebung (Node.js 22) und die OpenClaw-Binärdateien sauber zu installieren.

Start der Installation über die Konsole:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Der Installer führt folgende Schritte automatisch aus:
1. Prüfung der Systemvoraussetzungen.
2. Installation von Node.js 22 (LTS), falls noch nicht vorhanden.
3. Installation von Git.
4. Download und Einrichtung der OpenClaw CLI und des Gateway Services.
Nach dem Download startet das Skript automatisch den Einrichtungs-Assistenten ("Onboarding Wizard").
Ausgabe und Interaktion im Terminal:


Processing triggers for libc-bin ... <br>
✓ Git installed <br>
→ Installing OpenClaw 2026.2.6-3... <br>
✓ OpenClaw installed <br>
🦞 OpenClaw installed successfully (2026.2.6-3)! <br>

Finally unpacked. Now point me at your problems.
Starting setup...

| Security warning — please read. |

* I understand this is powerful and inherently risky. Continue?

| Yes / > No

Wichtige Handlungsanweisung für das "ZeroLab"-Setup: Da wir eine maßgeschneiderte, sichere Konfiguration (mit ausgelagerten Secrets und Hybrid-Modell) verwenden wollen, nutzen wir den automatischen Wizard nicht bis zum Ende.

1. Wähle bei der Sicherheitswarnung Yes und bestätige mit Enter.

2. Sobald der Wizard nach dem Modus fragt (oder "Onboarding mode"), brich den Vorgang sofort mit der Tastenkombination STRG + C ab.
Ergebnis: Die Software und alle Abhängigkeiten sind installiert, aber es wurden noch keine Standard-Konfigurationsdateien angelegt. Dies gibt uns die Freiheit, unsere eigene Struktur im nächsten Schritt sauber aufzubauen.

----
