# --- SICHERHEIT & ZUGANG ---
# Das Token ersetzt den Eintrag in der json. So ist es nicht im Code sichtbar.
# OpenClaw Token vom den internen Agent.

####################################################################################

# Gateway
# Varaiable für openclaw.json | "${OPENCLAW_GATEWAY_TOKEN}" | "token": "${OPENCLAW_GATEWAY_TOKEN}"

# alter OPENCLAW TOKEN
# OPENCLAW_GATEWAY_TOKEN=18cc164713ce62bcd3ede9afca9c8936e67e4a6901f94dac
OPENCLAW_GATEWAY_TOKEN=hier kommt der Gateway-Token rein (ksjdahfgksghkdsjgfhskdjdfgh)

# aktueller OPENCLAW TOKEN
OPENCLAW_GATEWAY_TOKEN=hier kommt der Gateway-Token rein (ksjdahfgksghkdsjgfhskdjdfgh)

# Mein eigenes Openlcaw Passwort
OPENCLAW_GATEWAY_PASSWORD=hier kommt das Passwort für das Gateway / Web-Ui rein (Geheimes-Passwort!123456)

# --- NETZWERK ---
# Optional, aber sauber getrennt

# OPENCLAW_GATEWAY_PORT=18789
# OPENCLAW_GATEWAY_BIND=lan (0.0.0.0) wird derzeit nihct benutzt und steht direkt in der openclaw.json

####################################################################################

# --- NETZWERK ---
# Optional, aber sauber getrennt

# OPENCLAW_GATEWAY_PORT=18789
# OPENCLAW_GATEWAY_BIND=0.0.0.0

####################################################################################

# --- PROVIDER CONFIG (Referenz) ---
# Konfiguration der einzelnen Provider und der API-Keys.
# Die Keys sind hier nur als Referenz eingetragen.

# API-Key von LM-Studio, Standard Key da es lokal auf meinem Multimedia-Server lÃ¤uft.
# LM_STUDIO_KEY=lmstudio-local
# LM_STUDIO_MODEL=llama2-70b-chat
# LM_STUDIO_URL=http://192.168.178.100
# LM_STUDIO_PORT=11434


# API-Key von Anthropic (Claude), Konto wurde mit 5$ Guthaben erstellt.
ANTHROPIC_API_KEY=hier kommt der API-Token vom Falback Ki-Modell rein (ksjdahfgksghkdsjgfhskdjdfgh)

# API-Key von Google Studio (Gemini) Basis Free Tier.
GOOGLE_API_KEY=hier kommt der API-Token vom ersten Ki-Modell rein (ksjdahfgksghkdsjgfhskdjdfgh)

####################################################################################

# --- HINWEIS ZUR TELEFONNUMMER ---
# OpenClaw benutzt aktuell KEINE direkte Umgebungsvariable von Listen (Arrays) 
# wie "allowFrom" in der JSON.
# STRATEGIE: Da die Nummer in der JSON bleiben muss, setze die Datei 
# 'openclaw.json' auf deine .gitignore Liste, damit sie niemals verï¿½ffentlicht wird!

# Variablen-Werte für die Telefonnummer
# Beispiel Telefonnummer im Channel Block:
# "allowFrom": [
#        "${MY_PHONE_NUMBER}"

# Telefonnummer
MY_PHONE_NUMBER=+49555Schuh


# Whatsapp Gruppen-IDs
# Beispiel Guppen-Chat im Channel Block:
# "groupAllowFrom": [
#        "${WHATSAPP_GROUP_WAMBA}"

# Gruppe WAMBA
WHATSAPP_GROUP_WAMBA=12345678901234567890@g.us

####################################################################################

# Weiter Keys oder Token und Geheimnisse in der gleichen Syntax
