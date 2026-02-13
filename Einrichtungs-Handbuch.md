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

#### 6. Installation der OpenClaw Software

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

#### 7 Konfiguration und erste Einstellungen OpenClaw

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

#### 7.1 Der Onbording Prozess von OpenClaw

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


## 🔒 Sicherheitshinweise (WICHTIG)

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
