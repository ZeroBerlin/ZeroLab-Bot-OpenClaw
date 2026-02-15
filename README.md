# 🦞 ZeroLab OpenClaw Edition

<br>

![Status](https://img.shields.io/badge/Status-Active-success)
![Hardware](https://img.shields.io/badge/Server-Proxmox%20LXC-blue)
![GPU](https://img.shields.io/badge/GPU-RTX_3070-76B900?logo=nvidia&logoColor=white)
![LLM](https://img.shields.io/badge/LLM-Llama%203.2-orange)
![Gemini](https://img.shields.io/badge/LLM-Gemini_2.5-4285F4?logo=google&logoColor=white)
![Claude](https://img.shields.io/badge/LLM-Claude-AE8C7E?logo=anthropic&logoColor=white)
![Messenger](https://img.shields.io/badge/Messenger-WhatsApp-25D366)

<br>

Willkommen im **ZeroLab**. Dies ist mein persönliches, lokal gehostetes Setup für einen autonomen KI-Agenten basierend auf [OpenClaw](https://github.com/openclaw/openclaw).
Das Ziel dieses Projekts ist **maximale Privatsphäre bei voller Autonomie**. Der Agent läuft auf eigener Hardware, nutzt lokale LLMs via LM Studio und kommuniziert über WhatsApp.

Später kann er auch auf die freie Wildbahn losgelassen werden, ohne auf einem Rechner mit persönlichen Daten zu laufen, oder zugreifen zu können. Eine VM / LXC ermöglichen einen sicheren und kontrollierten Betrieb.

Das LLM und die Rechenlast der Ki wurden auf einen älteren Rechner mit NVIDIA Karte ausgelagert. Denn nicht jeder möchte zum Spielen und Testen, Tokens auf den Ki-Plattformen bezahlen.

**Weiter Ausbaustufe ist der Wechsel auf das 1. LLM ![Gemini](https://img.shields.io/badge/LLM-Gemini_2.5-4285F4?logo=google&logoColor=white) in der FreeTier Variante (kostenlas), als Fallback ![Claude](https://img.shields.io/badge/LLM-Claude-AE8C7E?logo=anthropic&logoColor=white) hier wird je Token abgerechnet. Ich benutze die PrePaid Variante, Verbrauche nur das was du vorher eingezahlt hast. Un wenn alles nihcts mehr hilft dann gehts zurück auf die langsame Hardware -offlibe- ![LLM](https://img.shields.io/badge/LLM-Llama%203.2-orange) .**

---

### Was mir wichtig ist:

Da es sich hier um mein erstes richtiges kleines Projekt handelt möchte ich ein paar Spielregeln festlegen.
<br>

1.	Konstruktives Feedback ist erwünscht, solange,
  - Es um das Projekt geht
  - Gerne auch die Handhabung hier auf GitHub
  - Rat und Tipps

2.	Was ich nicht möchte:
  - Belehrungen
  - Beschimpfungen
  - Beiträge die nicht zum Thema gehören
  - Spam!!!

  <br>

Das sollte alles selbstverständlich sein, aber ich sage es trotzdem.

Ich weiß das OpenClaw nicht perfekt und sicher ist. Der Enwickler sieht es genauso.
<br>
Ich versuche alles lokal in VM´s aufzubauen. Die Rechen-Modelle LLM werden ebenfalls Lokal betrieben. Ich versuche wegen der begrenzten lokalen Rechenleistung auch externe Modelle LLM einzubinden. Diese sollen nach Möglichkeit Free-Tokens, PrePaid Varianten für die Token Bezahlung haben und vor allem jederzeit stopp bar, oder abschaltbar sei.

Es geht hier um ein rein privates Projekt. Zum Lernen über OpenClaw selbst, denn es ist unglaublich spannend und faszinierend zugleich. Und zum lernen und begreifen der LLM und wie alles in einander greift.
Wie es weiter geht wir werden sehen,

#### hier ist der Weg das Ziel.

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

#### Hinweis: <br>
Ich habe versucht den Token-Verbrauch in openclaw zu minimieren. Was sich natürlich auch auf die Antworten auswirkt. Ich habe versucht relativ minimalistische IDENTITY.md und SOUL.md zu erstellen. Derzeit liege ich bei der Konfiguration für das lokale LLM bei knapp unter 15.000 Toke und bei der Bezahl-Version auf Anthropic bei rund 18.000 Token.

---

## 🔒 Sicherheitshinweise (WICHTIG)

> [!WARNING]
> **Dies ist ein öffentliches Repository eines privaten Setups.**
> Sensible Daten wurden aus diesem Code entfernt.

Dieses Repository nutzt eine **strikte Trennung von Code und Geheimnissen**:
*   🚫 **Niemals enthalten:** API-Keys, echte Telefonnummern, Session-Token.
*   ✅ **Enthalten:** Konfigurations-Struktur, Prompts (`IDENTITY.md`, `SOUL.md`), Tools.

Die Datei `.gitignore` stellt sicher, dass `.env` und `openclaw.json` (mit echten Daten) lokal bleiben.
<br>
Wie? Die sensible Daten werden in Variablen geschrieben die sich in der .env befinden. <br>
In der eigentlichen Konfigurationsdatei (openclaw.json) werden diese Varibeln aus der .evn abgerufen. <br>
Somit kann man später die Konfiguration veröffentlichen, oder auch online gehen ohne das die: <br>
API-Key´s | Tokens | Telefonnummer | oder sontige Daten, <br>
die man nicht öffentlich teilen sollte, veröffentlicht werden.<br>
Zum Uplaod dient hier die .gitignore damit z.B. die .env nicht mit hochgeladen wird.

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
