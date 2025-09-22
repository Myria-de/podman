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
Ein mit „podman run“ gestarteter Container bleibt im Terminal aktiv, bis Sie ihn mit Strg-C abbrechen und den Container damit stoppen. Um den Container zu starten, verwenden Sie für dieses Beispiel
```
podman start dokuwiki
```
Allgemein können Sie auch die Container-ID verwenden, die Sie mit „podman ps -a“ ermitteln.

## Anwendung mit Podman-Compose erstellen

Damit sich Compose-Dateien nutzen lassen, installieren Sie zuerst 
```
sudo apt install --no-install-recommends podman-compose
```
Podman-compose ist ein Python-Script, das Anweisungen aus einer Compose-Datei mit Podman-Befehlen umsetzt.

Als erstes Beispiel verwenden wir Dokuwiki. Laden Sie die Beispieldatei „dokuwiki-podman-local.yaml“ herunter und speichern Sie sie im Ordner „~/Podman“. Öffnen Sie die Datei in einem Texteditor und passen Sie alle IDs an. „1000“ bezieht sich auf den ersten Benutzer, den Sie bei der Linux-Installation angelegt haben. Wenn Sie ein anderes Konto verwenden, ermitteln Sie mit dem Befehl
```
id
```
die zugehörigen Werte und setzen diese überall statt „1000“ ein. Sie erhalten damit volle Zugriffsrechte auf den Ordner „~/dokuwiki“. Achten Sie bei Änderungen in der YAML-Datei auf die korrekten Einzüge mit Leerzeichen.

In diesem Download-Ordner starten Sie dann
```
podman compose -f dokuwiki-podman-local.yaml up
```
Das Ergebnis ist identisch mit dem von „podman run“. Sie sehen die Meldungen im Terminal und stoppen den Container mit Strg-C. Da in der Compose-Datei „restart: always“ angegeben ist, können Sie den Container alternativ mit
```
systemctl --user restart podman-restart.service
```
aktivieren.

## Weitere Beispiele für Compose-Dateien

**Audiobookshelf** (https://www.audiobookshelf.org) stellt einen Server für gesprochene Audioinhalte im eigenen Netzwerk bereit. Außerdem kann die Software E-Books ausliefern, die sich auch auf dem Smartphone lesen lassen.

**Wordpress** (https://wordpress.org) ist die mit Abstand beliebteste Blog-Software im Internet. Für das lokale Netzwerk ist das CMS (Content-Management-System) vielleicht etwas zu mächtig, eine Installation kann aber sinnvoll sein, wenn man Erfahrungen damit sammeln möchte.

**Paperless-ngx** (https://docs.paperless-ngx.com) ist eine Software zur Ablage und Verwaltung digitalisierter Dokumente. Die Webanwendung unterstützt gängige Dokumenttypen wie PDFs und Word, Excel sowie Libre-Office-Dateien, die Sie mit Tags versehen und damit organisieren können. Der Inhalt der Dokumente lässt sich durchsuchen oder Sie finden Dokumente über einen Filter.

Mit **Immich** (https://immich.app) verwalten Sie Ihre Fotos und Video bequem über eine Weboberfläche. In Kombination mit einer Smartphone-App eignet sich Immich auf dem eigenen PC hervorragend als Alternative zu den Cloud-Speichern von Google und anderen. Starten Sie cen Container mit
```
podman compose -f immich-podman-local.yaml --env-file immich.env up
```
Ein Teil der Konfiguration befindet sich in der Datei "immich.env".

## Windows über einen Container starten
Podman kann mehr als nur Webanwendungen. Die eher ungewöhnliche Idee, Windows 10 oder 11 über einen Container zu starten, stammt von https://github.com/dockur/windows und wir haben die Konfiguration für den deutschsprachigen Raum angepasst. Über http://m6u.de/WDOCK finden Sie die nötigen Dateien. Die Voraussetzung ist ein PC, der KVM/Qemu für die Virtualisierung nutzen kann. Der Podman-Container stellt nur die Software bereit, über die sich Windows starten lässt.

Sie erzeugen den Container mit
```
podman compose -f windows-docker.yaml up
```
Die ISO-Datei für die Installation wird heruntergeladen und Windows automatisch installiert. Den Fortschritt können Sie im Terminal verfolgen. Der Zugriff auf die Windows-Oberfläche erfolgt im Browser über http://127.0.0.1:8006. Welche weiteren Optionen für die Installation verfügbar sind, erfahren Sie über http://m6u.de/WDOCK.

