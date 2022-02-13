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
8081 -> 80 TCP
40443 -> 443 TCP
53 -> 53 UDP
```
Port 80 und 443 hab ich entgegen der Anleitung direkt geändert (steht in einem Kommentar drunter).

Dann das **Binden** an Volumes:
```
/dnsmasq.d
-> /var/lib/docker/volumes/pihole/_data
/pihole
-> /var/lib/docker/volumes/pihole/_data
```

Bei *Network* soll man irgendeinen Hostname eingeben. Keine Ahnung was das macht und anschließend noch bei *Env* folgenden Umgebungsvariablen setzen:
| Variable | Wert |
| -------- | ------|
| TZ       | Europe/Berlin | 
| DNS1     | `1.1.1.1` | 
| DNS2     | `<IP des Rounters>` |
| WEBPASSWORD | `<passwort>` |
Auch hier hab ich als DNS1 direkt mal die IP geändert und hoffe, dass das mit DNS zwei funktioniert. Anschließend klickte ich auf *Deploy Container* und schaute was passiert.

Kurz es funktioniert. Allerdings hab ich jetzt die Angst, dass der Router die IP ständig ändert, weshalb ich jetzt die IP fix auf `...57` gesetzt habe indem ich `/etc/network/interfaces` so geändert habe, dass folgendes drinnen steht, weil das in einer [Anleitung](https://devconnected.com/how-to-change-ip-address-on-linux/) stand.

```
iface wlan0 inet static
address x.x.x.57
netmask 255.255.255.0
gateway x.x.x.1
```

Anschließen über `sudo systemctl restart networking.service` das ganze neu starten. Jetzt muss ich nur noch raus finden, wie ich den DNS Server meines Rechners ändern kann.
