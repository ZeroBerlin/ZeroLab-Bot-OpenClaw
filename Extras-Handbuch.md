
![Version 0.3](https://img.shields.io/badge/Version_0.3-Status_InArbeit-orange?logo=git&logoColor=white)

# <u> Extras - Funktionserweiterung - mehr Macht </u>

### Geheimnisse in die .env schreiben

<br>

### Wir bauen LLM Redundanz oder auch Fallback

<br>

### Gehe auf die "Dunkle Seite der Macht" Zugriff auf voll Root für den Bot

<br> <br>

...

Hier wird noch wild und unformatiert alles Reingeworfen.

---

## *** Die Datenschleuder ***

---


**Hier sollten wir doch nochmal Hand anlegen bevor wir durchstarten.**

Wir machen die offenen perönslichen Daten und die Token, die im Klartext in der openclaw.json stehen erstmal sicher und legen eine DAtei für diese Variablen an.

<br> <br>

Die Datei hat den Namen .env und wirdd im Ordner .openclaw erstellt.
<br>
Die .env sollte wie folgt aussehen:

<br> 

**Beispiel .env** <br>
Komentare erwünscht ... <br>
Leerzeilen sind erlaubt und bringen Struktur ...

```bash
# --- SICHERHEIT & ZUGANG ---
# Das Token ersetzt den Eintrag in der json. So ist es nicht im Code sichtbar.
# OpenClaw Token vom den internen Agent.

OPENCLAW_GATEWAY_TOKEN=Hier-steht-der-Gateway-Token-von-OpenClaw

# --- NETZWERK ---
# Optional, aber sauber getrennt

# OPENCLAW_GATEWAY_PORT=18789
# OPENCLAW_GATEWAY_BIND=0.0.0.0

# --- PROVIDER CONFIG (Referenz) ---
# Konfiguration der einzelnen Provider und der API-Keys.
# Die Keys sind hier nur als Referenz eingetragen.

# API-Key von LM-Studio, Standard Key da es lokal auf meinem Multimedia-Server läuft.
# für eine spätere Integration als Offline-Modell
LM_STUDIO_KEY=lmstudio-local
LM_STUDIO_MODEL=llama2-70b-chat
LM_STUDIO_URL=http://192.168.178.xxx
LM_STUDIO_PORT=11434


# API-Key von Anthropic (Claude), Konto wurde mit 5$ Guthaben erstellt.
ANTHROPIC_API_KEY=Hier-steht-der-Anthropic-API-Key

# API-Key von Google Studio (Gemini) Basis Free Tier.
GOOGLE_API_KEY=Hier-steht-der-Google-API-Key


# --- HINWEIS ZUR TELEFONNUMMER ---
# OpenClaw benutzt aktuell KEINE direkte Umgebungsvariable von Listen (Arrays) 
# wie "allowFrom" in der JSON.
# STRATEGIE: Da die Nummer in der JSON bleiben muss, setze die Datei 
# 'openclaw.json' auf deine .gitignore Liste, damit sie niemals verï¿½ffentlicht wird!

# Variablen-Werte für die Telefonnummer
MY_PHONE_NUMBER=+49555Schuh


# Whatsapp Gruppen-IDs
# Gruppe WAMBA
WHATSAPP_GROUP_xxxxx=12345678901234567890@g.us

# Weiter Keys oder Token und Geheimnisse in der gleichen Syntax

```

---
<br> <br>


## <u> Angepasste openclaw.json </u>
<br>


**Beispiel openclaw.json**<br>
Komentare erwünscht ..., geht auch hier mit // ... <br>
Leerzeilen sind erlaubt und bringen Struktur ...

```bash
{
  "messages": {
    "ackReactionScope": "group-mentions"
  },
  "models": {
    "providers": {
      "google": {
        "baseUrl": "https://generativelanguage.googleapis.com",
        "apiKey": "${GOOGLE_API_KEY}",
        "models": [
          {
            "id": "gemini-2.5-flash",
            "name": "Gemini 2.5 Flash",
            "contextWindow": 200000
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "maxConcurrent": 4,
      "subagents": {
        "maxConcurrent": 8
      },
      "compaction": {
        "mode": "safeguard",
        "memoryFlush": { "enabled": true }
      },
      "workspace": "/root/.openclaw/workspace",
      "model": {
        // 1. Primäres LLM Google Gemini (kostenlos)
        "primary": "google/gemini-2.5-flash",
        // 2. Fallback auf Anthropic Claude, wenn Google das Limit erreicht (429)
        "fallbacks": [
          "anthropic/claude-sonnet-4-5"
        ]
      }
    }
  },
  "gateway": {
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "${OPENCLAW_GATEWAY_TOKEN}"
    },
    "port": 18789,
    "bind": "loopback",
    "tailscale": {
      "mode": "off",
      "resetOnExit": false
    },
    "nodes": {
      "denyCommands": [
        "camera.snap",
        "camera.clip",
        "screen.record",
        "calendar.add",
        "contacts.add",
        "reminders.add"
      ]
    }
  },
  "auth": {
    "profiles": {
      "google:default": {
        "provider": "google",
        "mode": "api_key"
      }
    }
  },
  "plugins": {
    "entries": {
      "whatsapp": {
        "enabled": true
      }
    }
  },
  "channels": {
    "whatsapp": {
      "selfChatMode": true,
      "dmPolicy": "allowlist",
      "allowFrom": [
        "${MY_PHONE_NUMBER}"
      ]
    }
  },
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "boot-md": {
          "enabled": true
        },
        "command-logger": {
          "enabled": true
        },
        "session-memory": {
          "enabled": true
        }
      }
    }
  },
  "wizard": {
    "lastRunAt": "2026-02-13T17:24:12.522Z",
    "lastRunVersion": "2026.2.12",
    "lastRunCommand": "onboard",
    "lastRunMode": "local"
  },
  "meta": {
    "lastTouchedVersion": "2026.2.12",
    "lastTouchedAt": "2026-02-13T17:24:12.539Z"
  }
}
```

---

...