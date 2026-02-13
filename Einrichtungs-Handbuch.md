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

...