# 🦞 ZeroLab OpenClaw Edition

![Status](https://img.shields.io/badge/Status-Active-success)
![Hardware](https://img.shields.io/badge/Server-Proxmox%20LXC-blue)
![GPU](https://img.shields.io/badge/GPU-RTX_3070-76B900?logo=nvidia&logoColor=white)
![LLM](https://img.shields.io/badge/LLM-Llama%203.2-orange)
![Messenger](https://img.shields.io/badge/Messenger-WhatsApp-25D366)

Willkommen im **ZeroLab**. Dies ist mein persönliches, lokal gehostetes Setup für einen autonomen KI-Agenten basierend auf [OpenClaw](https://github.com/openclaw/openclaw).
Das Ziel dieses Projekts ist **maximale Privatsphäre bei voller Autonomie**. Der Agent läuft auf eigener Hardware, nutzt lokale LLMs via LM Studio und kommuniziert über WhatsApp.

Später kann er auch auf die freie Wildbahn losgelassen werden, ohne auf einem Rechner mit persönlichen Daten zu laufen, oder zugreifen zu können. Eine VM / LXC ermöglichen einen sicheren und kontrollierten Betrieb.

Das LLM und die Rechenlast der Ki wurden auf einen älteren Rechner mit NVIDIA Karte ausgelagert. Denn nicht jeder möchte zum Spielen und Testen, Tokens auf den Ki-Plattformen bezahlen.

---

## 🏗️ Architektur & Hardware

Das System ist darauf ausgelegt, Cloud-Abhängigkeiten zu minimieren.

| Komponente | Details |
| :--- | :--- |
| **Core** | OpenClaw (Node.js) |
| **LLM Backend** | LM Studio (Lokaler Server) |
| **Modell** | Llama 3.1 8B (via OpenAI-Kompatibilität) |
| **Hardware** | Windows 11 Pro, NVIDIA RTX 3070 |
| **Messenger** | WhatsApp (via Gateway) |
| **Performance** | Latenz ~2s (Lokal) vs. 500ms (Cloud) |

---

## 🔒 Sicherheitshinweise (WICHTIG)

> [!WARNING]
> **Dies ist ein öffentliches Repository eines privaten Setups.**
> Sensible Daten wurden aus diesem Code entfernt.

Dieses Repository nutzt eine **strikte Trennung von Code und Geheimnissen**:
*   🚫 **Niemals enthalten:** API-Keys, echte Telefonnummern, Session-Token.
*   ✅ **Enthalten:** Konfigurations-Struktur, Prompts (`IDENTITY.md`, `SOUL.md`), Tools.

Die Datei `.gitignore` stellt sicher, dass `.env` und `openclaw.json` (mit echten Daten) lokal bleiben.

---

## 🚀 Installation & Start

Willst du das ZeroLab-Setup nachbauen? Folge diesen Schritten:

### 1. Repository klonen

git clone https://github.com/ZeroBerlin/OpenClaw-ZeroLab.git
cd OpenClaw-ZeroLab
2. Konfiguration wiederherstellen
Da die echten Configs nicht im Repo sind, nutze die Vorlagen im Ordner .openclaw/:
1. Config: Benenne openclaw.example.json um zu openclaw.json.
    ◦ Trage deine echte Telefonnummer bei +555Schuh ein, denke an die Vorwal, z.B. +49 für Germay dann den Rest von deiner Handyrufnummer.
2. Secrets: Erstelle eine .env Datei (siehe .env.example).
    ◦ Füge deine API-Keys (falls benötigt) hinzu.
3. Starten
# Startet das Gateway und verbindet sich mit LM Studio
```bash
openclaw gateway start
```

----
🧠 Das "Gehirn" (Prompts)
Der Charakter des Bots wird durch Markdown-Dateien im Root-Verzeichnis gesteuert:
• IDENTITY.md: Definiert die Persona (Nerd, direkt, kein Assistent).
• SOUL.md: Die Prinzipien ("Sei ehrlich und eigenwillig").
• TOOLS.md: Regelwerk für Werkzeuge ("Denke in Bash").

----
📈 Roadmap
• [x] Grundlegendes Setup auf Windows 11
• [x] Anbindung an LM Studio (Llama 3.2)
• [x] GitHub "Clean Slate" Release
• [ ] Fallback-Strategie (Hybrid Cloud/Lokal)
• [ ] Erweiterung der lokalen Tools

----
Project by ZeroBerlin - Tech Thinks Gaming & Tutorials
