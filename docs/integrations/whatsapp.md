# WhatsApp Setup (Baileys)

## 🎯 Übersicht

xatrix nutzt **Baileys** (@whiskeysockets/baileys) für WhatsApp-Integration.
Baileys läuft als Bridge-Service (Node.js) und kommuniziert via WebSocket mit Python.

**Architektur:**
```
Python Agent (xatrix)
    ↓ WebSocket
Baileys Bridge (Node.js)
    ↓ WhatsApp Web Protocol
WhatsApp
```

---

## 📋 Voraussetzungen

### 1. Node.js installieren

**Option A: System-Installation (empfohlen)**
```bash
sudo apt update
sudo apt install -y nodejs npm
```

**Option B: nvm (Node Version Manager)**
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install --lts
```

**Prüfen:**
```bash
node --version  # sollte v18+ sein
npm --version
```

---

## 🚀 Installation

### 1. Bridge Dependencies installieren

```bash
cd bridges/whatsapp
npm install
```

### 2. Python WebSocket Library

```bash
# Bereits in pyproject.toml, aber falls nötig:
uv add websocket-client
```

---

## 🔐 WhatsApp Authentifizierung

### QR-Code Methode (Standard)

```bash
cd bridges/whatsapp
npm run auth
```

**Es erscheint ein QR-Code:**

1. Öffne WhatsApp auf deinem Handy
2. Gehe zu **Einstellungen → Verknüpfte Geräte**
3. Tippe auf **Gerät verknüpfen**
4. Scanne den QR-Code

✅ **Auth-Daten werden gespeichert in:** `data/whatsapp-auth/`

---

## ▶️ Agent mit Baileys starten

### 1. Bridge Server testen (manuell)

```bash
cd bridges/whatsapp
npm run dev
```

Du solltest sehen:
```
WhatsApp Bridge running on port 8743
Connected to WhatsApp
```

### 2. xatrix Agent starten

```bash
xatrix start
```

Der Agent startet automatisch:
- Baileys Bridge Server (Port 8743)
- WebSocket Verbindung
- Job Worker + Scheduler

---

## ✅ Testen

### WhatsApp-Nachricht senden

Sende eine Nachricht an dich selbst (oder registrierte Nummer):

```
Hallo Agent!
```

### Observer starten

```bash
xatrix observe
```

Du solltest sehen:
```
21:XX:XX | user_input       | 👤 User: Hallo Agent!
21:XX:XX | tool_call        | 🔧 ...
21:XX:XX | agent_response   | 🤖 Agent: Hallo!
```

---

## 🔧 Konfiguration

`.env` - **WICHTIG:** Alte Cloud API Config entfernen!

```bash
# WhatsApp Baileys Bridge
WHATSAPP_ENABLED=true
WHATSAPP_WEBHOOK_PORT=8743  # Bridge WebSocket Port
```

**Nicht mehr benötigt:**
- ~~WHATSAPP_ACCESS_TOKEN~~
- ~~WHATSAPP_PHONE_NUMBER_ID~~
- ~~WHATSAPP_APP_SECRET~~

---

## 🐛 Troubleshooting

### Bridge startet nicht

**Fehler:** `Bridge dependencies nicht installiert`

**Lösung:**
```bash
cd bridges/whatsapp
npm install
```

---

### "QR Code required"

**Problem:** Auth-Daten fehlen oder abgelaufen

**Lösung:**
```bash
cd bridges/whatsapp
rm -rf ../../data/whatsapp-auth  # Alte Auth löschen
npm run auth                      # Neu authentifizieren
```

---

### WebSocket Connection Failed

**Prüfen:**
```bash
# Bridge läuft?
ps aux | grep node

# Port belegt?
lsof -i :8743

# Health Check
curl http://localhost:8743/health
```

---

## 📱 Multi-Device vs. Haupt-Account

**⚠️ Sicherheitshinweis:**

Baileys nutzt WhatsApp Web Protocol (Linked Device).
- ✅ **Empfohlen:** Separate WhatsApp-Nummer/Account
- ⚠️ **Nicht empfohlen:** Haupt-Account mit sensiblen Chats

**Warum separate Nummer?**
- Auth-State liegt lokal (könnte geleakt werden)
- Inoffizieller Client (Meta könnte blockieren)
- Mehr Kontrolle über Risiko

---

## 🔄 Von Cloud API migrieren

**Alt (Cloud API):**
- Business Account nötig
- Access Token
- Phone Number ID

**Neu (Baileys):**
- ✅ Keine Business Account nötig
- ✅ QR-Code Auth
- ✅ Normale WhatsApp-Nummer

**Migration:**
1. Alte `.env` Config entfernen (siehe oben)
2. `npm run auth` ausführen
3. Agent neu starten
