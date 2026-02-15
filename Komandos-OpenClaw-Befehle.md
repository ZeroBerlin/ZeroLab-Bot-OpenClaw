### <u> OpenClaw - Komandos </u>
---
<br> <br>

```bash
systemctl restart openclaw
```
Startet den Service für Openclaw neu

---

```bash
openclaw gateway restart
```

Startet das Gateway neu.

---

```bash
systemctl status openclaw
```

Zeigt ob alle Konfiguratioen sauber eingelesen werden können

---

```bash
openclaw tui
```

Startet den Chat im Terminal / in der Console

---

```bash
systemctl stop openclaw
```

Startet den Dienst openclaw

---

```bash
systemctl start openclaw
```

Startet den Service für Openclaw neu

---

```bash
journalctl -u openclaw --follow
```

Fordlaufender Status / Log im Terminal (Staop mit STRG - C)

---

```bash
journalctl -u openclaw -n 50 --no-pager
```

Ein live Log-File welches man im Terminal verfolgen kann wenn im Messner z.B: WhatsApp mit dem Bot kommuniziert wird

---

```bash
openclaw channels login
```

Um den Channal Token (LogIn) neu zu generieren. Es kann sein das die Kopplung z.B. mit WhatsApp irgentwann ihre Gültigkeit verliert, hiermit kann der Messanger neu gekoppelt werden.

---

```bash
openclaw onboard
```

Start des Standard Konfigurations Prozesse von openclaw selbst, man kann so auch seine Konfiguration reparieren, indem man bei der Auswahl auf vorhanden Konfigurations Variablen benutzen klickt.

---

```bash
openclaw sessions prune --all
```

Löschen und bereinigen der in Sessions gespeicherten Daten. Die Gehirnwäsche quasi.

---

```bash
openclaw doctor
```

Befehl für den Selbsttest und als CheckUp.

---

```bash
openclaw agent --message "Dein Text"
```
Mit diesem Befehl kannst du eine KI-Interaktion manuell auslösen, ohne WhatsApp zu benutzen. Gebe deinen Promt direkt als Message an den Agent ein.

---
