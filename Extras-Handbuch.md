<br>
----
...
**Das ist der perfekte Text für das Hatching! 🐣** <br>

Dieser Text liefert dem Bot exakt die Informationen, die er laut der BOOTSTRAP.md-Logik benötigt, um seine Identität und Arbeitsweise festzulegen.
Sende den Text jetzt genau so in den Chat.
Was im Hintergrund passiert (Technical Breakdown)
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
3. Self-Destruct: Als letzten Schritt löscht er die Datei BOOTSTRAP.md. Damit ist die "Geburt" abgeschlossen. Beim nächsten Neustart fragt er nicht mehr, wer er ist, sondern lädt direkt seine Persönlichkeit aus den neuen Dateien.
Erwartete Reaktion: Er wird kurz bestätigen (z.B. "Verstanden. ZeroLab online. Warte auf Input, Chris."), da du ihn ja gerade angewiesen hast, "kurz angebunden" zu sein.
Damit ist dein ZeroLab offiziell online! 🦞

<br>
<br>
<br>
...

----
# Dokument Ende 

Weiter geht es mit dem Dokument Anpassung und Erweiterung (...).md





**Hier sollten wir doch nochmal Hand anlegen bevor wir durchstarten.**

Wir machen die offenen perönslichen Daten und die Token, die im Klartext in der openclaw.json stehen erstmal sicher und legen eine DAtei für diese Variablen an.
<br>
<br>
Die Datei hat den Namen .env und wirdd im Ordner .openclaw erstellt.
<br>
Die .env sollte wie folgt aussehen:
```bash
# --- SICHERHEIT & ZUGANG ---
# Das Token ersetzt den Eintrag in der json. So ist es nicht im Code sichtbar.
# OpenClaw Token vom den internen Agent.

OPENCLAW_GATEWAY_TOKEN=Hier-steht-der-Gateway-Token-von-OpenClaw

# --- NETZWERK ---
# Optional, aber sauber getrennt

OPENCLAW_GATEWAY_PORT=18789
OPENCLAW_GATEWAY_BIND=0.0.0.0

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
GOOGLE_API_MODEL=gemini-1.5-flash
GOOGLE_API_DEPLOYMENT=gemini-2.5-flash
GOOGLE_API_LOCATION=us-central1


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
...
```

...
