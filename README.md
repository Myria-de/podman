# Container mit Podman einrichten

Wir zeigen die Einrichtung der aktuellen Podman-Version unter Ubuntu 24.04 oder Linux Mint 22.1 und wie Sie das Tool für interessante Webanwendungen nutzen können. Der Schwerpunkt liegt bei der Konfiguration für den eigenen Rechner oder das lokale Netzwerk. Wer darüber hinaus Dienste auch aus dem Internet erreichen möchte, findet Anleitungen dafür über https://m6u.de/HOMSER.

## Podman unter Linux installieren

Nutzer von Ubuntu und Linux Mint können jedoch die Binärdateien von https://github.com/mgoltzsche/podman-static verwenden. Laden Sie unter „Releases“ die Datei „podman-linux-amd64.tar.gz“ herunter. Öffnen Sie ein Terminal (Strg-Alt-T), entpacken Sie die Datei im Downloadverzeichnis und kopieren Sie den Inhalt nach „/usr/local“ (drei Zeilen):
```
cd ~/Downloads
tar -xzf podman-linux-amd64.tar.gz
sudo cp -r podman-linux-amd64/usr podman-linux-amd64/etc /
```

Da wir Podman aus Sicherheitsgründen ohne erhöhte Rechte („rootless“) verwenden wollen, öffnen Sie die Konfiguration mit
```
sudo nano /etc/containers/storage.conf
```
im Editor und entfernen das Kommentarzeichen („#“) vor der Zeile 
```
rootless_storage_path = "$HOME/.local/share/containers/storage"
```

Podman speichert alle Daten im angegebenen Ordner in Ihrem Home-Verzeichnis.

Einige Anwendungen benötigen für die Kommunikation mit Podman einen Linux-Socket. Damit er nach der Anmeldung bereitsteht, aktivieren und starten Sie den zugehörigen Dienst mit (zwei Zeilen)
```
systemctl --user enable podman.service
systemctl --user start podman.service
```
Wenn Container automatisch beim Linux-Start ohne vorherige Benutzeranmeldung starten sollen, verwenden Sie die folgenden drei Befehlszeilen:
```
systemctl --user enable podman-restart.service
systemctl --user start podman-restart.service
sudo loginctl enable-linger [User]
```
Den Platzhalter „[User]“ ersetzen Sie durch Ihren Benutzernamen. Lassen Sie die letzte Zeile weg, wenn der automatische Start nach der Anmeldung ausreicht.

**Optionale Ergänzung:** Webanwendungen in Containern benötigen einen Port, damit sie sich im Browser aufrufen lassen. Webserver verwenden standardmäßig den Port 80, wenn Sie eine URL wie http://localhost oder im lokalen Netzwerk mit http://[MeinRechner] aufrufen. Sollen mehrere Webanwendungen laufen, verwenden Sie jeweils einen anderen Port, beispielsweise „8080“, „8081“ und so weiter. Den Port hängen Sie an die URL an: http://localhost:8080. Wer grundsätzlich mit den höheren Ports auskommt, muss nichts ändern. Ports kleiner als 1024 darf Podman nicht verwenden, weil dafür höhere Rechte erforderlich sind. Sie können das ändern, indem Sie die Zeile 
```
net.ipv4.ip_unprivileged_port_start=80
```
in die Datei „/etc/sysctl.conf“ eintragen und die Konfiguration mit 
```
sudo sysctl -p
```
aktualisieren.

## Erste Schritte mit Podman

Für einen ersten Test erstellen und starten Sie einen Container mit Ubuntu:
```
podman run -t -i ubuntu bash
```
Die Befehlszeile 
```
apt update && apt install mc
```
richtet den Terminal-Dateimanager Midnight Commander ein. Mit
```
exit
```
beenden Sie die Shell und schließen den Container.

**Wichtige Befehle:**
```
podman ps -a
podman start -i -a [Container-ID]
podman system df -v
podman stop [Container-ID]
podman rm [Container-ID]
podman images
podman rmi [Image-ID]
podman run --help
```
**Tipp:** Podman-Desktop (https://podman-desktop.io) ist eine grafische Oberfläche für Podman.

## Neue Anwendungen bequem einrichten

Sie installieren Dokuwiki mit
```
podman run -p 8080:8080 --name dokuwiki --restart "always" -v dokuwiki:/storage dokuwiki/dokuwiki:stable
```


