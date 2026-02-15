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
# 'openclaw.json' auf deine .gitignore Liste, damit sie niemals veröffentlicht wird!

# Variablen-Werte für die Telefonnummer
MY_PHONE_NUMBER=+49555Schuh


# Whatsapp Gruppen-IDs
# Gruppe -Name-
WHATSAPP_GROUP_xxxxx=12345678901234567890@g.us

# Weiter Keys oder Token und Geheimnisse in der gleichen Syntax
