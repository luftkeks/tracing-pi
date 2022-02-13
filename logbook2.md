# Logbook Part 2
So nach dem ds ganze beim ersten mal so grandios schief gegangen ist nochmal von vorne:
Image kommt [hier](https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2022-01-28/) her und [hier](https://www.raspberry-pi-geek.de/ausgaben/rpg/2017/10/ssh-und-wlan-schon-bei-der-installation-konfigurieren/) steht wie man von anfang an ssh aktivieren kann.
Einfach im root eine Datei mit `touch ssh` erstellen.  Für WLAN von anfang an braucht man eine Datei mit dem Namen `wpa_supplicant.conf` und dem Inhalt:
```
country=DE
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
ssid="WLAN-Name"
psk="Passwort"
}
```

Login danach hat getan, Passwort habe ich geändert.

Nächster Schritt ist nun wieder, Upadaten, `.local` alias und dann Docker + Docker-compose installieren.
```
sudo apt-get update && apt-get upgrade
sudo apt-get install avahi-daemon
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker pi
sudo pip3 install docker-compose
sudo systemctl enable docker
```

Jetzt folgt direkt die installation von portainer. Den ganzen TLS Kram lasse ich aus, ich möchte eh nicht über tcp auf Docker zugreifen.
Für die installation von Portainer folge ich wieder [dieser Anleitung](https://docs.portainer.io/v/ce-2.9/start/install/server/docker/linux).

## Viele Tage später

Ja Folgen wir dieser Anleitung.
```
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    cr.portainer.io/portainer/portainer-ce:2.9.3
```

Jetzt passieren Dinge.

ACHTUNG - beim ersten login erstellt man seinen Admin user - das sollte man also schnell machen.

Aber Step One ist scheinbar done. Portainer läuft!

Weil ich keine Lust auf gitlab habe, fange ich jetzt mit dem Pi-Hole an. Mit [dieser Anleitung](https://smarthome-training.com/de/pi-hole-docker-installation/) scheint das echt einfach zu sein.

Zuerst ein Volume erstellen: ich nenne das jetzt einfach mal wie in der Anleitung `pi-hole`.
Dann einen neuen Container namens `pi-hole` erstellenn der den container von `pihole/pihole:latest` nutzt.
Vier Ports mappen:
```
53 -> 53 TCP
80 -> 80 TCP
443 -> 443 TCP
53 -> 53 UDP
```
wobei ich den Port 80 später eventuell ändern will, obwohl ich keine Ahnung habe wie das geht.
