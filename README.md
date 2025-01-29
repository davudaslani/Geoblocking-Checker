# 🌍 Automatisierter Geoblocking-Checker

## 📌 Projektbeschreibung
Dieses Skript nutzt das **Tor-Netzwerk**, um zu überprüfen, ob bestimmte Websites in verschiedenen Ländern blockiert sind. Es sendet HTTP-Anfragen über verschiedene Exit-Nodes (Länder) und analysiert die Antworten.

---

## 🔧 Anforderungen
### **1. Virtuelle Maschine (Ubuntu) vorbereiten**
Falls du noch keine VM hast, erstelle eine **Ubuntu 22.04**-VM mit Internetzugang.

### **2. Notwendige Pakete installieren**
Öffne das Terminal und führe die folgenden Befehle aus:
```bash
sudo apt update && sudo apt install -y tor privoxy python3 python3-pip
pip3 install requests beautifulsoup4 stem
```

**Erklärung:**
- `tor`: Das Tor-Netzwerk zum Routing der Anfragen.
- `privoxy`: Proxy-Server zur Nutzung von Tor.
- `requests`: Python-Bibliothek für HTTP-Anfragen.
- `beautifulsoup4`: HTML-Parsing für Webseiteninhalte.
- `stem`: Python-Schnittstelle für Tor.

### **3. Tor & Privoxy konfigurieren**
Öffne die Privoxy-Konfigurationsdatei:
```bash
sudo nano /etc/privoxy/config
```
Füge folgende Zeile am **Ende** hinzu:
```plaintext
forward-socks5t / 127.0.0.1:9050 .
```
Speichern mit `CTRL + X`, `Y`, `ENTER`.

Jetzt **Tor & Privoxy starten**:
```bash
sudo systemctl enable --now tor privoxy
```
---

## 🖥️ **Skript: Geoblocking-Check**
Erstelle eine Datei `geoblocking_checker.py`:
```bash
nano geoblocking_checker.py
```
Füge den folgenden Python-Code ein:
```python
import requests
import random
import time
from stem.control import Controller

def change_tor_ip():
    with Controller.from_port(port=9051) as controller:
        controller.authenticate(password='mypassword')  # Passwort aus torrc
        controller.signal(2)  # Neustart von Tor

def check_website(url):
    proxies = {
        'http': 'socks5h://127.0.0.1:9050',
        'https': 'socks5h://127.0.0.1:9050'
    }
    try:
        response = requests.get(url, proxies=proxies, timeout=10)
        if response.status_code == 200:
            return "Zugänglich ✅"
        else:
            return f"Blockiert ❌ (Status: {response.status_code})"
    except requests.exceptions.RequestException:
        return "Blockiert oder Verbindung fehlgeschlagen ❌"

if __name__ == "__main__":
    websites = ["https://www.google.com", "https://www.wikipedia.org"]
    for website in websites:
        print(f"Teste Zugriff auf {website}...")
        result = check_website(website)
        print(f"Ergebnis: {result}\n")
        time.sleep(random.randint(5, 10))  # Wartezeit, um nicht gesperrt zu werden
```

**Erläuterung:**
- `change_tor_ip()`: Ändert die IP-Adresse (muss in `torrc` konfiguriert sein).
- `check_website(url)`: Verbindet sich über Tor und überprüft, ob die Seite blockiert ist.
- Websites können in der Liste `websites` hinzugefügt werden.

---

## ▶️ **Skript ausführen**
Speichere die Datei mit `CTRL + X`, `Y`, `ENTER` und führe es aus:
```bash
python3 geoblocking_checker.py
```

Falls Fehler mit `Tor`-Verbindung auftreten, stelle sicher, dass **Tor & Privoxy laufen**:
```bash
sudo systemctl restart tor privoxy
```
---

## 🔥 **Erweiterungen**
1. **Mehr Länder testen** → Konfiguration von Tor mit bestimmten Exit-Nodes.
2. **Ergebnisse in eine Datenbank speichern** → SQLite oder CSV.
3. **Webinterface mit Flask** → Darstellung der Ergebnisse auf einer Weltkarte.

Viel Spaß beim Coden! 🚀
