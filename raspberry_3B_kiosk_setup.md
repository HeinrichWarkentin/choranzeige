## 🖥️ Raspberry Pi 3B – Einrichtung einer Choranzeige (Kiosk-Modus)

### 📋 Voraussetzungen:

- Raspberry Pi 3B
- microSD-Karte (mind. 16 GB, empfohlen: 32 GB)
- Bildschirm mit HDMI-Anschluss (In diesem Fall war es ein 21:9 Bildschirm (3560x1080))
- Tastatur/Maus oder SSH-Zugang
- WLAN-Zugangsdaten

---

## 🔧 1. Raspberry Pi OS mit Raspberry Pi Imager installieren

### ➔ Im Raspberry Pi Imager folgende Einstellungen setzen:

1. **Raspberry Pi Modell:** Raspberry Pi 3B
2. **Betriebssystem:** Raspberry Pi OS 64 Bit mit Desktop
3. **SD-Karte wählen**
4. **Einstellungen bearbeiten:**
   - Hostname: `choranzeige`
   - Benutzername: `choranzeige` + Passwort festlegen
   - WLAN SSID: `WLAN-TT` + Passwort
   - Sprache: Deutsch
   - Dienste: **SSH aktivieren**, Passwort zur Authentifizierung verwenden

> 🔄 SD-Karte schreiben, einlegen, Raspberry Pi starten.

---

## 🔄 2. Nach dem ersten Start: System vorbereiten

### ➔ Verbinden per SSH oder direkt am Gerät:

```bash
ssh choranzeige@choranzeige
```

### ➔ System aktualisieren:

```bash
sudo apt-get update -y
sudo apt-get upgrade -y
```

### ➔ Raspberry Pi Konfiguration öffnen:

```bash
sudo raspi-config
```

Dann in folgendem Menü konfigurieren:

- **1 System Options** → `S6 Auto Login` → **Terminal and Desktop Autologin**
- **3 Interface Options** → `I3 VNC` → **VNC aktivieren**
- **6 Advanced Options** → `A6 Wayland` → **W1 X11 aktivieren**

> Dann: `Finish` → Neustart:

```bash
sudo reboot now
```

---

## 🌐 3. Choranzeige HTML-Datei herunterladen

```bash
mkdir -p /home/choranzeige/Documents
cd /home/choranzeige/Documents
wget https://raw.githubusercontent.com/HeinrichWarkentin/choranzeige/refs/heads/main/choranzeige.html
```

---

## 💠 4. Hilfsprogramme installieren

```bash
sudo apt-get install -y x11-xserver-utils chromium-browser xcvt
```

---

## 📜 5. Startskript für Kiosk-Modus anlegen

### ➔ Skript erstellen:

```bash
nano /home/choranzeige/start-choranzeige.sh
```

### ➔ Inhalt einfügen:

```bash
#!/bin/bash

export DISPLAY=:0
export XAUTHORITY=/home/choranzeige/.Xauthority

# Warte, bis Desktop geladen ist
sleep 10

# Auflösung angepasst für 21:9, unter 2048x2048 Grenze, da das die obergenze für den Raspberry 3B ist. Mit dem Befehl "xrandr" lässt sie diese ermitteln.
# Für Andere Auflösungen den Befehl cvt x_pixel y_pixel Hz (z.B cvt 2048 878 60) ausführen und Ausgabe hier eintragen.
xrandr --newmode "2048x878_60.00" 160.00 2048 2184 2400 2752 878 881 891 908 -hsync +vsync
xrandr --addmode HDMI-1 2048x878_60.00
xrandr --output HDMI-1 --mode 2048x878_60.00

# Chromium starten
chromium-browser \
  --kiosk \
  --start-fullscreen \
  --incognito \
  --noerrdialogs \
  --no-first-run \
  --disk-cache-dir=/dev/null \
  file:///home/choranzeige/Documents/choranzeige.html
```

### ➔ Skript ausführbar machen:

```bash
chmod +x /home/choranzeige/start-choranzeige.sh
```

---

## ⚙️ 6. Autostart einrichten

### ➔ Autostart-Ordner erstellen:

```bash
mkdir -p ~/.config/autostart
```

### ➔ Autostart-Datei anlegen:

```bash
nano ~/.config/autostart/kiosk.desktop
```

### ➔ Inhalt einfügen:

```ini
[Desktop Entry]
Type=Application
Name=Choranzeige
Exec=/home/choranzeige/start-choranzeige.sh
X-GNOME-Autostart-enabled=true
```

---

## 🔄 7. Neustarten und Testen

```bash
reboot
```

---

## ✅ Ergebnis:

Nach dem Neustart sollte automatisch:

- Die Bildschirmauflösung auf 2048×878 gesetzt werden
- Chromium im Kiosk-Modus starten
- Die `choranzeige.html` im Vollbild angezeigt werden

---

> ❗ Wenn Fehler auftreten:
>
> - Sicherstellen, dass X11 aktiviert ist (`sudo raspi-config` → `A6` → `W1 X11`)
> - Die Auflösung darf max. 2048x2048 betragen (GPU-Limit)
> - Datei-/Pfadnamen exakt prüfen
