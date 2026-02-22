![Version](https://img.shields.io/badge/Version_1.0-Status_final-success?logo=github&logoColor=white)

# 📘 ZeroLab: OpenClaw Einrichtungs-Handbuch

### Version: 1.0 (ZeroLab Edition 2026) Fokus: Schritt-für-Schritt Installation & Konfiguration
----

Die Example Dateien befinden sich im Unterordner .openclaw-adjusted/ , die Konfig openlaw.json und die example-env verlinke ich hier driekt.
<br>
[📝 Demo-Code ansehen: example-env.md (inkl. Kommentare)](./.openclaw-adjusted/example-env.txt)
<br>
[⚙️ Demo-Code ansehen: openclaw.example.json (inkl. Kommentare)](./.openclaw-adjusted/openclaw.example.json)

---

<br>

### 1. Einleitung und Zielsetzung

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
<br>

### 2. Erstellung des LXC Containers (Proxmox)

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
<br>

### 3. Basis-Konfiguration des LXC Containers

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
----
<br>

### 4 Einrichtung des externen Zugriffs (WinSCP / SSH)
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
----
<br>


### 5. Meilenstein: Backup des Basis-Systems
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
<br>

### 6. Installation der OpenClaw Software

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


**Ansicht im Terminal**
```bash
root@pve-openclaw:~# curl -fsSL https://openclaw.ai/install.sh | bash
```
Ausgabe:
```bash
╭───────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                       │
│  🦞 OpenClaw Installer                                                                │
│  I'm not saying your workflow is chaotic... I'm just bringing a linter and a helmet.  │
│  modern installer mode                                                                │
│                                                                                       │
╰───────────────────────────────────────────────────────────────────────────────────────╯

✓ gum bootstrapped (temp, verified, v0.17.0)
✓ Detected: linux
            
Install plan
            
OS                  linux
Install method      npm
Requested version   latest
                           
[1/3] Preparing environment
                           
INFO Node.js not found, installing it now
INFO Installing Node.js via NodeSource
INFO Installing Linux build tools (make/g++/cmake/python3)
⣯  Installing build tools  
```
```bash
[2/3] Installing OpenClaw
                         
INFO Git not found, installing it now
✓ Git installed
INFO Installing OpenClaw v2026.2.12
✓ OpenClaw npm package installed
✓ OpenClaw installed
                      
[3/3] Finalizing setup
                      

🦞 OpenClaw installed successfully (2026.2.12)!
I'm in. Let's cause some responsible chaos.

INFO Starting setup
```

----
<br>

#### 6.1 Konfiguration und erste Einstellungen OpenClaw

```bash
🦞 OpenClaw installed successfully (2026.2.12)!
I'm in. Let's cause some responsible chaos.

INFO Starting setup


🦞 OpenClaw 2026.2.12 (f9e444d) — Your task has been queued; your dignity has been deprecated.

▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
██░▄▄▄░██░▄▄░██░▄▄▄██░▀██░██░▄▄▀██░████░▄▄▀██░███░██
██░███░██░▀▀░██░▄▄▄██░█░█░██░█████░████░▀▀░██░█░█░██
██░▀▀▀░██░█████░▀▀▀██░██▄░██░▀▀▄██░▀▀░█░██░██▄▀▄▀▄██
▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀
                  🦞 OPENCLAW 🦞 

```

#### 6.2 Der Onbording Prozess von OpenClaw

```bash
T  OpenClaw onboarding
|
o  Security ------------------------------------------------------------------------------+
|                                                                                         |
|  Security warning — please read.                                                        |
|                                                                                         |
|  OpenClaw is a hobby project and still in beta. Expect sharp edges.                     |
|  This bot can read files and run actions if tools are enabled.                          |
|  A bad prompt can trick it into doing unsafe things.                                    |
|                                                                                         |
|  If you’re not comfortable with basic security and access control, don’t run OpenClaw.  |
|  Ask someone experienced to help before enabling tools or exposing it to the internet.  |
|                                                                                         |
|  Recommended baseline:                                                                  |
|  - Pairing/allowlists + mention gating.                                                 |
|  - Sandbox + least-privilege tools.                                                     |
|  - Keep secrets out of the agent’s reachable filesystem.                                |
|  - Use the strongest available model for any bot with tools or untrusted inboxes.       |
|                                                                                         |
|  Run regularly:                                                                         |
|  openclaw security audit --deep                                                         |
|  openclaw security audit --fix                                                          |
|                                                                                         |
|  Must read: https://docs.openclaw.ai/gateway/security                                   |
|                                                                                         |
+-----------------------------------------------------------------------------------------+
|
*  I understand this is powerful and inherently risky. Continue?
|    Yes / > No
—
```
1. Drücke die Pfeiltaste nach oben (oder unten), bis Yes farbig markiert/ausgewählt ist.
2. Drücke Enter.

```bash
*  I understand this is powerful and inherently risky. Continue?
|  > Yes /   No
```

Vorschau auf den nächsten Schritt: <br>
Das System wird dich gleich nach dem "Onboarding mode" fragen (z.B. Quickstart oder Manual/Advanced).

Ich werde den offiziellen "Happy Path" des Installers, für die Ordnerstruktur, die Session-Datenbanken und vor allem die BOOTSTRAP.md (das "Schlüpf-Ritual") sauber anlegen zu lassen.

Später werden die einzelnen Dateien und Konfigurationen angepasst und überarbeitet.


### 🔒 Sicherheitshinweise (WICHTIG)

> [!WARNING]
> **Bis nicht alle Dateien für die Konfiguration angepasst und überprüft wurden, sollten keine Tools installiert oder Zugriffe auf externe persönliche Daten konfiguriert werden!!!**

Ich möchte Openclaw nicht produktiv nutzen, oder gar Zugriff auf meine persönlichen Daten gewähren. **Daher auch die strikte Trennung in einer virtuellen Umgebung**:

*   🚫 **Keine echten Daten:** Passwörter, Telefonnummern, Session-Token, eMail-Adressen, etc.
*   ✅ **LXC Container** Das System und die Software OpenClaw läuft in dieser Umgebung.

```bash
*  I understand this is powerful and inherently risky. Continue?
|  > Yes /   No
```

Jetzt zum Nächsten Schritt. <br>
**(Yes)** asuwählen und **(Enter)** drücken.

```bash
*  Onboarding mode
|  > QuickStart (Configure details later via openclaw configure.)
|    Manual
```

**(QuickStart)** auswählen und **(Enter)** drücken.

```bash
*  Model/auth provider
|  > OpenAI (Codex OAuth + API key)
|    Anthropic
|    MiniMax
|    Moonshot AI (Kimi K2.5)
|    Google
|    xAI (Grok)
|    OpenRouter
|    Qwen
|    Z.AI
|    Qianfan
|    Copilot
|    Vercel AI Gateway
|    OpenCode Zen
|    Xiaomi
|    Synthetic
|    Together AI
|    Venice AI
|    LiteLLM
|    Cloudflare AI Gateway
|    Custom Provider
|    Skip for now
```
Ich werde die Basis-Konfiguration mit Google Gemini beginnen. **Hack: KEINE KOSTEN** <br>

Auf Google Studio kann man sich mit seinem Google-Account anmelden und einen API-Key generiren lassen, der als Free-Tier genutzt werden kann. Unter sehr engen Grenzen muss man unter Google-Studio nichts bezahlen.<br>

**Achtung: die Token-Grenze ist sehr eng gefasst.** <br>
Man darf hier nicht zuviel erwarten und man muss OpenClaw dazu bringen nicht mehr, als rund 14.000 Toke für eine Anfrage zu verwenden.

Ich wähle Google:

```bash
|  > Google (Gemini API key + OAuth)
```
und **(Enter)**

```bash
*  Google auth method
|  > Google Gemini API key
|    Google Antigravity OAuth
|    Google Gemini CLI OAuth
|    Back
```
Da ich den Free-Tier API-Key in Google-Studio erstellt habe wähle ich **(Google Gemini API key)** und drücke **(Enter)**.

```bash
*  Enter Gemini API key
|  [GOOGLE_API_KEY]
```
und **(Enter)** drücken, dann geht es weiter zur Auswahl der LLM (Sprach-Modelle) in meiner Variante die von Google Gemini.

```bash
*  Default model
|  > Keep current (google/gemini-3-pro-preview)
|    Enter model manually
|    google/gemini-1.5-flash
|    google/gemini-1.5-flash-8b
|    google/gemini-1.5-pro
|    google/gemini-2.0-flash
|    google/gemini-2.0-flash-lite
|    google/gemini-2.5-flash
|    google/gemini-2.5-flash-lite
|    google/gemini-2.5-flash-lite-preview-06-17
|    google/gemini-2.5-flash-lite-preview-09-2025
|    google/gemini-2.5-flash-preview-04-17
|    google/gemini-2.5-flash-preview-05-20
|    google/gemini-2.5-flash-preview-09-2025
|    google/gemini-2.5-pro
|    google/gemini-2.5-pro-preview-05-06
|    google/gemini-2.5-pro-preview-06-05
|    google/gemini-3-flash-preview
|    google/gemini-3-pro-preview
|    google/gemini-flash-latest
|    google/gemini-flash-lite-latest
|    google/gemini-live-2.5-flash
|    google/gemini-live-2.5-flash-preview-native-audio
—
```
Jetzt wird es interessant, ich zeige hier die Konfiguration die bei mir funktioniert hat und die ich auch getestet habe. <br>

Ich wähle das Modell: **google/gemini-2.5-flash** aus.

```bash
> google/gemini-2.5-flash (Gemini 2.5 Flash · ctx 1024k · reasoning)
```
und wieder **(Enter)** drücken.

Jetzt kommen wir zur "Channel" Konfiguration. Hier wird der Messenger für die spätere Benutzung auf dem Handy eingestellt. <br>

Ich benutze **WhatsApp** die Besonderheit hier ist: OpenClaw spricht in WhatsApp quasi in deinem Namen. Weil wir uns die Telefonnummer teilen.

```bash
*  Select channel (QuickStart)
|  > Telegram (Bot API) (not configured)
|    WhatsApp (QR link)
|    Discord (Bot API)
|    IRC (Server + Nick)
|    Google Chat (Chat API)
|    Slack (Socket Mode)
|    Signal (signal-cli)
|    iMessage (imsg)
|    Feishu/Lark (飞书)
|    Nostr (NIP-04 DMs)
|    Microsoft Teams (Bot Framework)
|    Mattermost (plugin)
|    Nextcloud Talk (self-hosted)
|    Matrix (plugin)
|    BlueBubbles (macOS app)
|    LINE (Messaging API)
|    Zalo (Bot API)
|    Zalo (Personal Account)
|    Tlon (Urbit)
|    Skip for now
```

Also WhatsApp auswählen ...

```bash
*  Select channel (QuickStart)
|    Telegram (Bot API)
|  > WhatsApp (QR link) (not configured)
|    Discord (Bot API)
|    IRC (Server + Nick)
|    Google Chat (Chat API)
|    Slack (Socket Mode)
|    Signal (signal-cli)
|    iMessage (imsg)
|    Feishu/Lark (飞书)
|    Nostr (NIP-04 DMs)
|    Microsoft Teams (Bot Framework)
|    Mattermost (plugin)
|    Nextcloud Talk (self-hosted)
|    Matrix (plugin)
|    BlueBubbles (macOS app)
|    LINE (Messaging API)
|    Zalo (Bot API)
|    Zalo (Personal Account)
|    Tlon (Urbit)
|    Skip for now
—
```
... und erneut mit **(Enter)** bestätigen.

Jetzt muss das Handy, wie für die Benutzung von WhatsApp Web gekoppelt werden. Dies erfolgt über einen QR-Code der jetzt angezeigt wird.

```bash
o  WhatsApp linking ----------------------------------------------------------------------+
|                                                                                         |
|  Scan the QR with WhatsApp on your phone.                                               |
|  Credentials are stored under /root/.openclaw/credentials/whatsapp/default/ for future  |
|  runs.                                                                                  |
|  Docs: whatsapp                            |
|                                                                                         |
+-----------------------------------------------------------------------------------------+
|
*  Link WhatsApp now (QR)?
|  > Yes /   No
```
Bereite dein Handy vor, wähle WhatsApp gehe unten rechts auf dein Profilbild. (unter iOS für iPhone) und dann oben rechts in der Ecke auf das QR-Code Symbol.

Jetzt bist du bereit ...

**(Yes)** bestättigen

```bash
o  Link WhatsApp now (QR)?
|  Yes
Waiting for WhatsApp connection...
Scan this QR in WhatsApp (Linked Devices):
▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
█ ▄▄▄▄▄ ██▀██▀▄▀▀▄█▄▄▄█ ▄▄▄▄ ▀▄▄▀▀█  ▄▄ ▀█▄ ▄██▄ ██ ▄▄▄▄▄ █
█ █   █ █▄▄   ▄███▄▄▀▄▄██▄█▄█▄█ ██▄▄▄██▄▄█▄ ██▀▄ ██ █   █ █
█ █▄▄▄█ ██ ██ ███▄ ▀ ▀▀▄█▄▀ ▄▄▄ ▀ █      ▄▄  ▀ ▀▄██ █▄▄▄█ █
█▄▄▄▄▄▄▄█ █ █ ▀▄▀▄█▄█ █▄▀▄▀ █▄█ █▄█ █ ▀ ▀▄█▄█ ▀▄█ █▄▄▄▄▄▄▄█
```
```bash
█ █▄▄▄█ █ ▄▄█  ██▀▄▄ ▄ ██▀█ ▄▄█▀ ▄█▀▀▀▄▀▀▀ ▀▄  ▀▄ ██ ▄██▀▄█
█▄▄▄▄▄▄▄█▄▄▄████▄██▄██▄▄▄▄▄▄█▄█▄▄███▄▄██▄▄█▄█▄█▄▄█▄▄▄▄██▄▄█

WhatsApp asked for a restart after pairing (code 515); creds are saved. Restarting connection once…
✅ Linked after restart; web session ready.
|
o  WhatsApp DM access ------------------------------------------------------+
|                                                                           |
|  WhatsApp direct chats are gated by `channels.whatsapp.dmPolicy` +        |
|  `channels.whatsapp.allowFrom`.                                           |
|  - pairing (default): unknown senders get a pairing code; owner approves  |
|  - allowlist: unknown senders are blocked                                 |
|  - open: public inbound DMs (requires allowFrom to include "*")           |
|  - disabled: ignore WhatsApp DMs                                          |
|                                                                           |
|  Current: dmPolicy=pairing, allowFrom=unset                               |
|  Docs: whatsapp              |
|                                                                           |
+---------------------------------------------------------------------------+
|
*  WhatsApp phone setup
|  > This is my personal phone number
|    Separate phone just for OpenClaw
—
```
Wenn du es bis hier geschaft hast **"Herzlichen Glückwunsch"** OpenClaw kann jetzt Messenger.

Wie du hier siehst muss noch deine Handynummer eingetragen werden.
```bash
+---------------------------------------------------------------------------+
|
*  WhatsApp phone setup
|  > This is my personal phone number
|    Separate phone just for OpenClaw
—
```
Machen wir das, wähle ...
```bash
|  > This is my personal phone number
```
aus und drücke erneut **(Enter)** <br>
Trage jetzt in dem unteren Format deine Handynummer ein.
```bash
*  Your personal WhatsApp number (the phone you will message from)
|  +15555550123
—
```
In meinem Fall eine Handynummer aus Deutschland +49 ohne die 0 in der Vorwahl und die Nummer. Keine Leerzeichen oder Bindestrichen, Bsp.: +49151987654321
```bash
*  Your personal WhatsApp number (the phone you will message from)
|  +49151555Schuh
—
```
**-Fertig-**
Siehe Ausgabe:
```bash
o  WhatsApp personal phone -----------------------+
|                                                 |
|  Personal phone mode enabled.                   |
|  - dmPolicy set to allowlist (pairing skipped)  |
|  - allowFrom includes +49151555Schuh            |
|                                                 |
+-------------------------------------------------+
|
o  Selected channels ---------------------------------------------------------------+
|                                                                                   |
|  WhatsApp — works with your own number; recommend a separate phone + eSIM. Docs:  |
|  whatsapp                   |
|                                                                                   |
+-----------------------------------------------------------------------------------+
Updated ~/.openclaw/openclaw.json
Workspace OK: ~/.openclaw/workspace
Sessions OK: ~/.openclaw/agents/main/sessions
|
o  Skills status -------------+
|                             |
|  Eligible: 3                |
|  Missing requirements: 39   |
|  Unsupported on this OS: 7  |
|  Blocked by allowlist: 0    |
|                             |
+-----------------------------+
|
*  Configure skills now? (recommended)
|  > Yes /   No
—
```

Skills möchte ich noch nicht installieren, das kommt später. Aslo **(No)** auswählen ...

```bash
*  Configure skills now? (recommended)
|    Yes / > No
```
und **(Enter)**

Es ist fast geschaft ... der nächste Konfig-Bereich:

```bash
o  Configure skills now? (recommended)
|  No
|
o  Hooks ----------------------------------------------------------+
|                                                                  |
|  Hooks let you automate actions when agent commands are issued.  |
|  Example: Save session context to memory when you issue /new.    |
|                                                                  |
|  Learn more: https://docs.openclaw.ai/hooks                      |
|                                                                  |
+------------------------------------------------------------------+
|
*  Enable hooks?
|  [•] Skip for now
|  [ ] 🚀 boot-md
|  [ ] 📝 command-logger
|  [ ] 💾 session-memory
—
```
Wir / Ich werde hier alles auswählen. **Warum?**
<br>
<br>

#### 6.3 Wie funktioniert das mit den .md Dateien
<br>

```bash
[ ] 🚀 boot-md
```
Der boot-md Hook ist ein Autostart-Mechanismus.

* **Funktion:** Er feuert jedes Mal, wenn der OpenClaw-Dienst (Gateway) startet (z. B. nach einem Server-Reboot oder systemctl restart)

* **Ziel:** Er sucht nach einer Datei namens BOOT.md in deinem Workspace und führt deren Anweisungen aus

* **Zweck:** Das wird genutzt, um dem Bot zu sagen: "Wenn du aufwachst, prüfe sofort E-Mails" oder "Schreibe mir auf WhatsApp: System neu gestartet, bin online."

<br>
**Hinweis:** Unterschied zu BOOTSTRAP.md

* BOOTSTRAP.md (Das Schlüpfen): Passiert nur ein einziges Mal ganz am Anfang, wenn der Workspace neu ist. Es dient dazu, die Identität (IDENTITY.md) zu erstellen. Das wollen wir gleich nutzen. Das passiert jetzt wenn OpenClaw nach dem Onboarding Prozess zum aller ersten Mal gestartet wird. Dann nie wieder.

* BOOT.md (Das Aufwachen): Passiert jedes Mal.

<br>
<br>

```bash
[ ] 📝 command-logger
```
**Der Flugschreiber** <br>
Er protokolliert jeden administrativen Befehl (/new, /reset, /stop, /status, etc.), der an den Bot gesendet wird. Er dient dem Debugging und Auditing. Wenn der Bot plötzlich sein Verhalten ändert, kannst du hier prüfen: "Habe ich versehentlich /reset getippt?" oder "Hat jemand anderes einen Befehl ausgeführt?".
<br>

**Wer beschreibt ihn?** <br>
Der Hook selbst (ein internes Skript von OpenClaw) schreibt die Daten. Er läuft "silent" (geräuschlos) im Hintergrund, ohne dass du im Chat eine Bestätigung erhältst.
<br>

**Wo steht das?** <br>
Die Daten landen in einer simplen Textdatei auf deinem Server: ~/.openclaw/logs/commands.log
<br>

**Was steht drin?** <br>
Er schreibt im JSONL-Format (JSON Lines), was ideal für Maschinen lesbar ist. Jeder Eintrag enthält: <br>
* ts: Zeitstempel (Wann?)
* action: Der Befehl (z.B. "new")
* session: Die Session-ID (In welchem Chat?)
* sender: Die ID des Absenders (z.B. deine Telefonnummer +49...)
* source: Der Kanal (z.B. "whatsapp")
* Beispiel: {"ts":"2026-02-09T16:00:00Z","action":"new","session":"agent:main...","sender":"+49162...","source":"whatsapp"}
<br>

**Wissenswertes für ZeroLab:** <br>
 Der Hook besitzt **keine automatische Log-Rotation.** <br>
  Das bedeutet, die Datei wächst unendlich. Für ein privates Setup ist das unkritisch, aber als Admin solltest du wissen: Wenn die Datei in 2 Jahren 500 MB groß ist, musst du sie manuell löschen oder rotieren (logrotate).
  <br>

<br>

```bash
[ ] 💾 session-memory
```
**Das episodische Gedächtnis** <br>


> ## Der Banger ... <br>
> **Dieser Hook löst eines der größten Probleme von LLM-Agenten: den Gedächtnisverlust beim Reset oder nach einem Neustart.**

...




* **Wozu wird er benutzt?** <br>
 Normalerweise, wenn du den Befehl /new eingibst, wird der aktuelle Kontext ("Context Window") gelöscht, um Platz zu sparen oder Fehler zu beheben. Der Bot vergisst alles, worüber ihr gerade geredet habt. Der session-memory Hook greift genau in diesem Moment ein: Bevor alles gelöscht wird, speichert er eine Zusammenfassung der letzten Unterhaltung ab.
 <br>

* **Wer beschreibt ihn?** <br>
Der Hook steuert den Prozess, nutzt aber dein konfiguriertes LLM (z.B. Gemini oder Llama), um einen intelligenten Dateinamen ("Slug") zu generieren, der den Inhalt beschreibt.
<br>

* **Wo steht das?** <br>
Er erstellt neue Markdown-Dateien in deinem Workspace: ~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md.
<br>

* **Was steht drin?** <br>
Er extrahiert die letzten N Nachrichten (Standard: 15, konfigurierbar) der alten Session, bevor sie gelöscht wird. Beispiel-Dateinamen: 2026-02-13-nginx-config-fix.md oder 2026-02-13-backup-strategy.md.
<br>

**Wissenswertes für ZeroLab:** <br>
Dies ist nicht die MEMORY.md. Die MEMORY.md ist für ewige Fakten ("Ich bin ZeroLab"). session-memory erzeugt Tagebucheinträge vergangener Diskussionen.
* Vorteil: Wenn du später fragst "Wie haben wir letzte Woche den Server gefixt?", kann die Vektor-Suche diese alten Dateien finden, auch wenn der Chat längst gelöscht ist.
* Der Hook funktioniert automatisch nur beim /new Befehl.
<br>
<br>

Da wir uns entschieden haben, das System voll auszureizen (Schlüpfen und Aufwachen), aktiviere alle wir alle drei Optionen mit der Leertaste, sodass überall ein [x] steht: <br>

• [x] 🚀 boot-md (Für die Begrüßung nach dem Hochfahren) ybr>

• [x] 📝 command-logger (Für dein Admin-Logbuch) <br>

• [x] 💾 session-memory (Für das automatische Tagebuch) <br>

**Hinweis:** Dass die Datei BOOT.md für den ersten Punkt noch nicht existiert, ist kein Problem. Der Hook prüft einfach, ob sie da ist – wenn nicht, tut er nichts (kein Absturz). Wir erstellen sie im Anschluss einfach manuell.
<br>

```bash
*  Enable hooks?
|  [ ] Skip for now
|  [+] 🚀 boot-md (Run BOOT.md on gateway startup)
|  [+] 📝 command-logger (Log all command events to a centralized audit file)
|  [+] 💾 session-memory (Save session context to memory when /new command is issued)
—
```
und wie schon davor und davor ... bestättige mit **(Enter)**
<br>
<br>

### 7 Wir sind am Ziel ;-)
<br>

```bash
o  Enable hooks?
|  🚀 boot-md, 📝 command-logger, 💾 session-memory
|
o  Hooks Configured -----------------------------------------+
|                                                            |
|  Enabled 3 hooks: boot-md, command-logger, session-memory  |
|                                                            |
|  You can manage hooks later with:                          |
|    openclaw hooks list                                     |
|    openclaw hooks enable <name>                            |
|    openclaw hooks disable <name>                           |
|                                                            |
+------------------------------------------------------------+
|
o  Systemd -------------------------------------------------------------------------------+
|                                                                                         |
|  Systemd user services are unavailable. Skipping lingering checks and service install.  |
|                                                                                         |
+-----------------------------------------------------------------------------------------+
|
o  
Health check failed: gateway closed (1006 abnormal closure (no close frame)): no close reason
  Gateway target: ws://127.0.0.1:18789
  Source: local loopback
  Config: /root/.openclaw/openclaw.json
  Bind: loopback
|
o  Health check help --------------------------------+
|                                                    |
|  Docs:                                             |
|  https://docs.openclaw.ai/gateway/health           |
|  https://docs.openclaw.ai/gateway/troubleshooting  |
|                                                    |
+----------------------------------------------------+
|
o  Optional apps ------------------------+
|                                        |
|  Add nodes for extra features:         |
|  - macOS app (system + notifications)  |
|  - iOS app (camera/canvas)             |
|  - Android app (camera/canvas)         |
|                                        |
+----------------------------------------+
|
o  Control UI -------------------------------------------------------------------------------+
|                                                                                            |
|  Web UI: http://127.0.0.1:18789/                                                           |
|  Web UI (with token):                                                                      |
|  http://127.0.0.1:18789/#token=[GATEWAY-TOKEN-KEY]            |
|  Gateway WS: ws://127.0.0.1:18789                                                          |
|  Gateway: not detected (gateway closed (1006 abnormal closure (no close frame)): no close  |
|  reason)                                                                                   |
|  Docs: https://docs.openclaw.ai/web/control-ui                                             |
|                                                                                            |
+--------------------------------------------------------------------------------------------+
|
o  Workspace backup ----------------------------------------+
|                                                           |
|  Back up your agent workspace.                            |
|  Docs: https://docs.openclaw.ai/concepts/agent-workspace  |
|                                                           |
+-----------------------------------------------------------+
|
o  Security ------------------------------------------------------+
|                                                                 |
|  Running agents on your computer is risky — harden your setup:  |
|  https://docs.openclaw.ai/security                              |
|                                                                 |
+-----------------------------------------------------------------+
|
o  Shell completion --------------------------------------------------------+
|                                                                           |
|  Shell completion installed. Restart your shell or run: source ~/.bashrc  |
|                                                                           |
+---------------------------------------------------------------------------+
|
o  Dashboard ready ----------------------------------------------------------------+
|                                                                                  |
|  Dashboard link (with token):                                                    |
|  http://127.0.0.1:18789/#token=[GATEWAY-TOKEN-KEY]  |
|  Copy/paste this URL in a browser on this machine to control OpenClaw.           |
|  No GUI detected. Open from your computer:                                       |
|  ssh -N -L 18789:127.0.0.1:18789 root@<host>                                     |
|  Then open:                                                                      |
|  http://localhost:18789/                                                         |
|  http://localhost:18789/#token=[GATEWAY-TOKEN-KEY]  |
|  Docs:                                                                           |
|  https://docs.openclaw.ai/gateway/remote                                         |
|  https://docs.openclaw.ai/web/control-ui                                         |
|                                                                                  |
+----------------------------------------------------------------------------------+
|
o  Web search (optional) -----------------------------------------------------------------+
|                                                                                         |
|  If you want your agent to be able to search the web, you’ll need an API key.           |
|                                                                                         |
|  OpenClaw uses Brave Search for the `web_search` tool. Without a Brave Search API key,  |
|  web search won’t work.                                                                 |
|                                                                                         |
|  Set it up interactively:                                                               |
|  - Run: openclaw configure --section web                                                |
|  - Enable web_search and paste your Brave Search API key                                |
|                                                                                         |
|  Alternative: set BRAVE_API_KEY in the Gateway environment (no config changes).         |
|  Docs: https://docs.openclaw.ai/tools/web                                               |
|                                                                                         |
+-----------------------------------------------------------------------------------------+
|
o  What now -------------------------------------------------------------+
|                                                                        |
|  What now: https://openclaw.ai/showcase ("What People Are Building").  |
|                                                                        |
+------------------------------------------------------------------------+
|
—  Onboarding complete. Use the dashboard link above to control OpenClaw.
```
<br>
<br>

### 8 BackUp - Meilen-Stein Sicherung

Das ist der perfekte Zeitpunkt für ein Backup! 🎉
Dieser **"Golden-State"** ist enorm wertvoll. Du hast jetzt ein funktionierendes System:
1. OS & Tools: Sauber installiert und gehärtet.

2. OpenClaw: Erfolgreich installiert und Abhängigkeiten (Node.js) gelöst.

3. Identität: Der Bots steht kurz bevor (Hatching). ER bekommt seinen Namen nach dem Golden-State BackUp.
Meine Namenswahl wird : "ZeroLab" sein.

4. Verbindung: WhatsApp ist gekoppelt.

#### 8.1 Das Golden.State BackUp
Wenn wir jetzt beim Umbau auf ein anders Modell (LM Studio) oder beim Tuning etwas "kaputtkonfigurieren", kannst du immer exakt hierhin zurückkehren, ohne QR-Codes scannen oder Pakete installieren zu müssen.
<br>

1. Zum Backup des LXC-Containers unter PRoxmox:
2. Container 999 (pve-openclaw) auswählen.
3. Menü Backup -> Backup now.
4. Mode: Stop (für Konsistenz).
5. Storage: a11-hdd.
6. Notes: {{guestname}} Golden-State-OpenClaw


Protokoll des Backup-Vorgangs:
```bash
INFO: starting new backup job: vzdump 999 --notification-mode auto --notes-template '{{guestname}} Golden-State-OpenClaw' --compress zstd --mode stop --remove 0 --storage a11-hdd --node pve
INFO: Starting Backup of VM 999 (lxc)
INFO: Backup started at 2026-02-13 18:53:23
INFO: status = running
INFO: backup mode: stop
INFO: ionice priority: 7
INFO: CT Name: pve-openclaw
INFO: including mount point rootfs ('/') in backup
INFO: stopping virtual guest
INFO: creating vzdump archive '/mnt/pve/a11-hdd/dump/vzdump-lxc-999-2026_02_13-18_53_23.tar.zst'
INFO: Total bytes written: 3541831680 (3.3GiB, 20MiB/s)
INFO: archive file size: 1.38GB
INFO: adding notes to backup
INFO: restarting vm
INFO: guest is online again after 178 seconds
INFO: Finished Backup of VM 999 (00:02:58)
INFO: Backup finished at 2026-02-13 18:56:21
INFO: Backup job finished successfully
INFO: notified via target `mail-to-root`
TASK OK
```
<br>
<br>

### 9 Letzter Schliff am LXC - Container

#### 9.1 Noch haben wir keinen Autostart als Systemdienst im LXC
Nach dem Golden-State BackUp sollte der LXC Container neu gestartet sein. Wir erwarten das Openclaw nihct selbständig gestartet ist. Da der Installations-Wizard vorhin den Fehler Systemd user services are unavailable gemeldet hat.
<br>

In einem LXC-Container (als Root) funktionieren die Standard-"Benutzer-Dienste" oft nicht. Wir haben das System zwar gesichert, aber den Autostart-Dienst (Daemon) noch nicht manuell repariert.

**Wir prüfen das** <br>
Gebe in der Konsole des LXC folgenden Befehl ein.

```bash
systemctl status openclaw
```
Jetzt sollte folgende Ausgabe zu sehen sein:
```bash
root@pve-openclaw:~# systemctl status openclaw
Unit openclaw.service could not be found.
root@pve-openclaw:~# 
```
Der Autostart ist noch nicht richtig eingerichtet.
Der automatische Installer konnte den Dienst nicht anlegen, weil in LXC-Containern (als root) die Standard-"Benutzer-Dienste" nicht funktionieren. Das System weiß also noch gar nicht, dass es einen "OpenClaw"-Dienst geben soll.
<br>
Wir müssen die Service-Datei jetzt manuell erstellen. Das ist ohnehin besser, da wir so volle Kontrolle über den Autostart haben.
<br>
<br>

Wir erstellen eine Konfigurationsdatei für systemd, die Linux sagt, wie OpenClaw zu starten ist.
<br>
<br>
Öffne den internen Linux Editor "Nano" mit folgendem Befehl:

```bash
nano /etc/systemd/system/openclaw.service
```
Kopiere den folgenden Block exakt so in den Editor. Er ist auf deine Pfade (/root/.openclaw) und den Root-User angepasst.
<br>
```bash
[Unit]
Description=OpenClaw Gateway Service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/.openclaw
ExecStart=/usr/bin/openclaw gateway
Restart=always
RestartSec=5
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```
Jetzt kann die Datei mit STRG + O geschrieben werden und der Editor kann anschließend mit STRG + X wieder geschlossen werden.
<br>
<br>
Jetzt müssen wir Linux sagen, dass es die neue Datei einlesen und den Dienst beim Booten starten soll. Der Dienst muss aktiviert und gestartet werden.
<br>
<br>
Das machen wir mit den folgenden Befehlen:
```bash
systemctl daemon-reload
```
```bash
systemctl enable openclaw
```
Rückmeldung, der Dienst wurde erstellt.
```bash
root@pve-openclaw:~# systemctl enable openclaw
Created symlink /etc/systemd/system/multi-user.target.wants/openclaw.service -> /etc/systemd/system/openclaw.service.
```

Letzter Befehl:
```bash
systemctl start openclaw
```
<br>
Wenn es bei der Befehlseingabe keinen Fehler als Rückmeldung gegeben hat, sollten wir jetzt einen Status bei der Abfrage erhalten.
<br>
<br>
Probiren wir es aus ..., geb foldenden Befehl nochmal ein:
<br>

```bash
systemctl status openclaw
```
<br>

Die Ausgabe sollte jetzt genau so aussehen:

```bash
root@pve-openclaw:~# systemctl status openclaw
* openclaw.service - OpenClaw Gateway Service
     Loaded: loaded (/etc/systemd/system/openclaw.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-02-13 18:43:09 UTC; 13s ago
   Main PID: 599 (openclaw)
      Tasks: 14 (limit: 18846)
     Memory: 402.3M (peak: 402.5M)
        CPU: 8.058s
     CGroup: /system.slice/openclaw.service
             |-599 openclaw
             `-607 openclaw

Feb 13 18:43:09 pve-openclaw systemd[1]: Started openclaw.service - OpenClaw Gateway Service.
root@pve-openclaw:~# 
```
<br>

**Das sieht hervorragend aus!** 🟢 <br>

Der Status Active: active (running) bestätigt, dass deine manuelle Service-Installation erfolgreich war. Der Dienst läuft jetzt als **System-Service (root)**, startet automatisch beim Booten und ist stabil.
<br>

### 9.2 Golden - State final (BackUp)

Der "Golden-State" ist damit technisch perfekt abgeschlossen.
<br>

Du musst das nicht machen, aber ich machen nochmal ein <br>
**Meilen-Stein BackUp,** <br>
 das tut nicht weh. Aber wir haben jetzt einen funktionierenden Zustand vor dem Hatching / der Geburt wo Openclaw nach seinem Namen und seine Identität fragen sollte. Ich nutze diesen Zustand gerne als Wipe BackUp, falls ich mal neu anfangen will, oder muss.

----
also führe das LXC - Container wie im Punkt **8.1 Das Golden.State BackUp** erneut aus. <br>
<br>
Ich vergebe den Namen: **Hatching-Moment (gewipet)** <br>

----
<br>

**Ein letzter Check ob alles richtig konfiguriert ist.**
<br>
Der Status-Check (Pre-Flight) 🛠️ <br>
Prüfen der Gemini-Anbindung (Das Gehirn).
<br>

1. CheckUp

```bash
openclaw models status
```

Ausgabe:

```bash
root@pve-openclaw:~# openclaw models status

🦞 OpenClaw 2026.2.12 (f9e444d) — If you can describe it, I can probably automate it—or at least make it funnier.

Config        : ~/.openclaw/openclaw.json
Agent dir     : ~/.openclaw/agents/main/agent
Default       : google/gemini-2.5-flash
Fallbacks (0) : -
Image model   : -
Image fallbacks (0): -
Aliases (0)   : -
Configured models (1): google/gemini-2.5-flash

Auth overview
Auth store    : ~/.openclaw/agents/main/agent/auth-profiles.json
Shell env     : off
Providers w/ OAuth/tokens (0): -
- google effective=profiles:~/.openclaw/agents/main/agent/auth-profiles.json | profiles=1 (oauth=0, token=0, api_key=1) | google:default=[GOOGLE-API-KEY]

OAuth/token status
- none
root@pve-openclaw:~# 
```
<br>

2. Checkup

```bash
openclaw status --deep
```
Ausgabe:

```bash
root@pve-openclaw:~# openclaw status --deep

🦞 OpenClaw 2026.2.12 (f9e444d) — I read logs so you can keep pretending you don't have to.

|
gateway connect failed: Error: unauthorized: gateway token mismatch (set gateway.remote.token to match gateway.auth.token)
  
OpenClaw status

Overview
┌─────────────────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ Item            │ Value                                                                                                                                                       │
├─────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Dashboard       │ http://127.0.0.1:18789/                                                                                                                                     │
│ OS              │ linux 6.8.12-15-pve (x64) · node 22.22.0                                                                                                                    │
│ Tailscale       │ off                                                                                                                                                         │
│ Channel         │ stable (default)                                                                                                                                            │
│ Update          │ pnpm · npm latest 2026.2.12                                                                                                                                 │
│ Gateway         │ local · ws://127.0.0.1:18789 (local loopback) · reachable 128ms · auth token · pve-openclaw (192.168.178.239) app unknown linux 6.8.12-15-pve               │
│ Gateway service │ systemd not installed                                                                                                                                       │
│ Node service    │ systemd not installed                                                                                                                                       │
│ Agents          │ 1 · 1 bootstrapping · sessions 0 · default main active unknown                                                                                              │
│ Memory          │ 0 files · 0 chunks · dirty · sources memory · plugin memory-core · vector ready · fts ready · cache on (0)                                                  │
│ Probes          │ enabled                                                                                                                                                     │
│ Events          │ none                                                                                                                                                        │
│ Heartbeat       │ 30m (main)                                                                                                                                                  │
│ Last heartbeat  │ none                                                                                                                                                        │
│ Sessions        │ 0 active · default gemini-2.5-flash (1049k ctx) · ~/.openclaw/agents/main/sessions/sessions.json                                                            │
└─────────────────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

Security audit
Summary: 0 critical · 2 warn · 1 info
  WARN Reverse proxy headers are not trusted
    gateway.bind is loopback and gateway.trustedProxies is empty. If you expose the Control UI through a reverse proxy, configure trusted proxies so local-client c…
    Fix: Set gateway.trustedProxies to your proxy IPs or keep the Control UI local-only.
  WARN Credentials dir is readable by others
    /root/.openclaw/credentials mode=755; credentials and allowlists can be sensitive.
    Fix: chmod 700 /root/.openclaw/credentials
Full report: openclaw security audit
Deep probe: openclaw security audit --deep

Channels
┌──────────┬─────────┬────────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ Channel  │ Enabled │ State  │ Detail                                                                                                                                          │
├──────────┼─────────┼────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ WhatsApp │ ON      │ OK     │ linked · +49555Schuh · auth 5m ago · accounts 1                                                                                               │
└──────────┴─────────┴────────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

Sessions
┌──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┬──────┬─────────┬──────────────┬────────────────┐
│ Key                                                                                                                          │ Kind │ Age     │ Model        │ Tokens         │
├──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼──────┼─────────┼──────────────┼────────────────┤
│ no sessions yet                                                                                                              │      │         │              │                │
└──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┴──────┴─────────┴──────────────┴────────────────┘

Health
┌──────────┬───────────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ Item     │ Status    │ Detail                                                                                                                                                 │
├──────────┼───────────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Gateway  │ reachable │ 6ms                                                                                                                                                    │
│ WhatsApp │ LINKED    │ linked (auth age 5m)                                                                                                                                   │
└──────────┴───────────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

FAQ: https://docs.openclaw.ai/faq
Troubleshooting: https://docs.openclaw.ai/troubleshooting

Next steps:
  Need to share?      openclaw status --all
  Need to debug live? openclaw logs --follow
  Need to test channels? openclaw status --deep
root@pve-openclaw:~# 
```
<br>

Wir starten OpenClaw direkt im Terminal und sehen die live Log-Daten. Ich benutze WhatsApp in meinem Web-Browser. Hier kann ich besser schreiben und Texte kopieren und einfügen. Das geht definitiv besser als auf dem kleinen Display vom Handy.
<br>
<br>

### 10 Wir starten die Geburt unseres eigenen Bot´s
<br>

```bash
openclaw gateway --verbose
```



```bash
root@pve-openclaw:~# openclaw gateway --verbose

🦞 OpenClaw 2026.2.12 (f9e444d) — If you're lost, run doctor; if you're brave, run prod; if you're wise, run tests.

15:31:37 [canvas] host mounted at http://127.0.0.1:18789/__openclaw__/canvas/ (root /root/.openclaw/canvas)
15:31:37 bonjour: starting (hostname=openclaw, instance="pve-openclaw", gatewayPort=18789, minimal=true)
15:31:38 [heartbeat] started
15:31:38 [gateway] agent model: google/gemini-2.5-flash
15:31:38 [gateway] listening on ws://127.0.0.1:18789 (PID 770)
15:31:38 [gateway] listening on ws://[::1]:18789
15:31:38 [gateway] log file: /tmp/openclaw/openclaw-2026-02-14.log
15:31:38 [browser/service] Browser control service ready (profiles=2)
15:31:39 Registered hook: boot-md -> gateway:startup
15:31:39 Registered hook: command-logger -> command
15:31:39 Registered hook: session-memory -> command:new
15:31:39 [hooks] loaded 3 internal hook handlers
15:31:39 [whatsapp] [default] starting provider (+49555Schuh)
15:31:39 [ws] → event health seq=1 clients=0 presenceVersion=1 healthVersion=2
15:31:41 WhatsApp Web connected.
15:31:41 Sent global 'available' presence on connect
15:31:41 [whatsapp] Listening for personal WhatsApp inbound messages.
Ctrl+C to stop.
```
<br>
Das ist ein perfekter Start! 🟢 <br>
1. Das Gehirn ist da: [gateway] agent model: google/gemini-2.5-flash OpenClaw hat erfolgreich Verbindung zum Google-Modell aufgenommen,.

2. Die Ohren sind offen: WhatsApp Web connected. und Listening for personal WhatsApp inbound messages. Die Verbindung zu deinem Handy steht stabil. Der Bot wartet jetzt nur noch auf ein Signal,.

3. Hooks sind geladen: boot-md, command-logger, session-memory sind aktiv,. Das System ist bereit für die Automatisierungen.

**Der nächste Schritt: Das Hatching (Die Geburt) 🐣** <br>
Da das Terminal jetzt "Listening" anzeigt und blockiert (also nicht zur Eingabe zurückkehrt), läuft der Bot live. <br>
...
### 10.1 Erster Kontakt - WhatsApp
Schreibe folgenden Text, oder vergleichbar, in WhatsApp (oder dem konfigurierten Messenger): <br>
```bash
System Start, hallo.
```
Ich bekam folgende Antwort:
```bash
[openclaw] Hey. I just came online. Who am I? Who are you?
```
<br>


**Glückwunsch! 🥳 Das ist der perfekte „Hatch“-Moment (das Schlüpfen).** <br>

Der Bot hat exakt das Skript aus der BOOTSTRAP.md ausgeführt und wartet nun auf seine „Seele“ (SOUL.md). <br>

**Er weiß noch nichts –** weder seinen Namen noch seine Funktion. Das ist deine Chance, ihn final zu prägen. <br>

**Deine Antwort jetzt an ihn:** <br>

Schreibe ihm direkt im Chat folgenden Block (basierend auf deiner geplanten IDENTITY.md):
**Das ist mein Geburtstext an den Agent:** <br>
```bash
"Du bist ZeroLab.
Ein technischer Assistent für alles, aber speziell für Proxmox, Code und Administration. Deine Art: Kurz angebunden, nerdig, direkt und effizient, gerne auch etwas weird. Kein „Dienstleister-Geschwafel“, sondern Lösungen.  Ich bin Chris, dein Admin."
```
<br>

**Was dann passiert:**
Dieser Text liefert dem Bot exakt die Informationen, die er laut der BOOTSTRAP.md-Logik benötigt, um seine Identität und Arbeitsweise festzulegen. <br>

Sende den Text jetzt genau so in den Chat. <br>

**Was im Hintergrund passiert (Technical Breakdown)** <br>
Sobald du die Nachricht abschickst, führt OpenClaw basierend auf dem BOOTSTRAP.md-Skript folgende Schritte aus:
1. Parsing: Er analysiert deinen Text und extrahiert die Kern-Parameter:
    ◦ Name: ZeroLab
    ◦ Vibe: Nerdig, kurz angebunden, weird, direkt.
    ◦ User: Chris (Admin).
    ◦ Mission: Technischer Assistent für Proxmox/Code.

2. Dateisystem-Operationen: Er überschreibt die Platzhalter-Dateien im Workspace (/root/.openclaw/workspace/) mit den konkreten Werten:
    ◦ IDENTITY.md: Wird deinen Namen "ZeroLab" und den Vibe enthalten.
    ◦ SOUL.md: Wird die Verhaltensregel "Kein Dienstleister-Geschwafel" speichern.
    ◦ USER.md: Wird "Chris" und "Admin" eintragen.

3. Self-Destruct: Als letzten Schritt löscht er die Datei BOOTSTRAP.md. Damit ist die "Geburt" abgeschlossen. <br>

Beim nächsten Neustart fragt er nicht mehr, wer er ist, sondern lädt direkt seine Persönlichkeit aus den neuen Dateien. <br>

**Erwartete Reaktion:** <br>
Er wird kurz bestätigen (z.B. "Verstanden. ZeroLab online. Warte auf Input, Chris."), da du ihn ja gerade angewiesen hast, "kurz angebunden" zu sein.
<br>
<br>

**Die Antwort aus dem WhatsApp - Chat:** <br>
```bash
[openclaw] Updated my core files. You're Chris, I'm ZeroLab 💾. Short, nerdy, direct, efficient. Solutions, not fluff.

Deleting BOOTSTRAP.md. Done.
```
```bash
[openclaw] Okay, BOOTSTRAP.md is deleted. Ready.
```
<br>

----

# Es ist vollbracht, der Bot mit dem Namen "ZeroLab" ist damit offiziell online! 🦞
<br>

**Wenn du auch bis hierhin gekommen bist? <br>
 Herzlichen Glückwunsch. Du hast es geschafft den persönlicher OpenClaw Bot funktioniert.**

Du hast jetzt deinen eigenen Ai Agenten geschaffen.

### Nutze ihn mit bedacht, denn Macht geht mit Verantwortung einher.
<br>
<br>

## 11 Nachwort Schlusswort zum Einrichtungs-Handbuch
In diesem Handbuch wurde versucht eine detaillierte und schrittweise aufgebaute Anleitung zu erstellen. Diese soll es auch Laien ermöglichen mit Hilfe von „OpenClaw“ einen persönlichen Ai – Agent (Bot) selbst zu erstellen und zu konfigurieren. <br>

Die OpenClaw Instanz sollte so nicht auf einem persönlichen PC oder gar in der Firma auf einem Büro PC laufen. Auch wenn es verlockend klingt diesen Agent idealer Weise auf einem MacBook zu installieren, rate ich davon ab. <br>

Wir haben dem Agent ROOT – Rechte gegeben und mit den Richtigen Eingaben (Prompt) wird er sie auch benutzen. Der Agent kann alle Dateien auf dem PC / Mac lesen und kann die Enthaltenen Daten wie: eMail-Adressen, Kontakte, persönliche Memos etc. <br>

In einer Messenger Gruppe ist die sogenante Prompt-Injektion (böswillige Prompts, Befehle) an eueren eigenen Bot eine reale Gefahr. **Und er wird den Prompt ausführen. Es ist der Maschine egal wer ihn eingegeben hat.** <br>

Es ist möglich das der Agent im Hintergrunde bei der Bearbeitung analysiert, zusammenfasst und wie in dem hier angewendeten Fall, diese Daten als Prompt an ein Ki-Modell (LLM) zum Bsp. Gemini 2.5 flash sendet.

# Regel:
## Gebe der Ki keine Daten, die du nicht bereit bist anderen zu geben!

## Stelle dir folgende Frage immer zuerst?

## Würde ich wollen das z.B. mein Bankberater, mein Steuerberater, mein Chef, mein Arzt …, meine bei ihm gespeicherten Daten, ungefragt für ein Zusammenfassung oder eine Arbeitserleichterung in das Internet sendet?
<br>

**Wenn die Antwort NEIN lautet, so solltest du es auch nicht tun!**
<br>

**Bei Allen anderen kann ich nur sagen, viel Glück dabei … und müsst ihr wissen!**
<br>

**Im nächsten Handbuch werden wir API-Keys, Telefonnummern aus der Konfiguration in eine .env verbannen. Wir werden ein Fallback LLM- offline, wie auch als Bezahl-Variante bauen. Wir werden die Seele von OpenClaw bearbeiten und fein tunen.**
<br>

Und wir werden uns des Teufels Seele ansehen, OpenClaw hat einen Doktor Jekyll und Mister Hyde Modus, aber keine Angst der ist noch off und klingt schlimmer als er ist …

Seit gespannt wie es weiter geht.
<br>
<br>

Euer Chris alias ZeroLab

----
