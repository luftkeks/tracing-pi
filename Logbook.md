# Tagebuch des Verlorenen auf der Suche nach der richtigen Installation

Ziel vorerst ist es Docker zum laufen zu bringen so dass ich diesen einen Container laden kann
TODO - link einfügen
Außerdem könnte man ZFS mittels [dieser Anleitung](https://lookslikematrix.de/server/2021/03/28/zfs-raspberry-pi.html) zum laufen bringen. Dafür hab ich mir [diesen SATA auf USB Adapter](https://www.amazon.de/gp/product/B07L5DK7C5) bestellt, Festplatte hab ich eh irgendwo noch rum liegen. Eventuell funktioniert das ja.

## Day One - Hour Zero
Ich habe zuerst dieses [Raspberry Pi Image](https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2022-01-28/) mir runter geladen und mit [Balea Etcher](https://www.balena.io/etcher/) auf die micro SD Karte gezogen.

## Day Two - Hour Zero
Dieser eine Container heißt [Portainer](https://www.portainer.io/). Außerdem könnte man ein [Pi-Hole](https://pi-hole.net/) installieren, sowie eine [gitlab](https://docs.gitlab.com/ee/install/docker.html) Instanz.

## Day Two - Hour One
Ziel ist es erstmal ssh zum laufen zu kriegen, dass ich mir den extra Bildschirm sparen kann. Dafür hab ich `sudo touch ssh` in `/` gemacht. Außerdem hab ich mittels `raspi-config` den hostname auf `raspi` geändert. Trotzdem wird die connection refused. Entsprechend dieses [Tutorials](https://dev.to/elalemanyo/raspberry-pi-setup-6jm) kann ich mir eine `.local` adresse machen, also führe ich `sudo apt-get install avahi-daemon` aus. Dieses [Tutorial](https://phoenixnap.com/kb/enable-ssh-raspberry-pi) liefert den richtigen Hinweis. Man kann SSH einfach über das grafische Interface über Preferences aktivieren.
Nächster Punkt wäre Docker zu installieren. Ich verwende folgende [Anleitung](https://dev.to/elalemanyo/how-to-install-docker-and-docker-compose-on-raspberry-pi-1mo). Ergo beginne ich mit `curl -sSL https://get.docker.com | sh`. Anschließend adde ich den `pi` User zur Usergroup `docker` mittels `sudo usermod -aG docker pi`.
Nach kurzer konsultation diverser Experten bin ich zu dem schluss gekommen, dass ich auch docker-compose installieren möchte. Da `pip3` in meinem RaspberryImage offensichtlich bereits installiert ist mache ich dies mittels `sudo pip3 install docker-compose`.
Da ich docker definitiv im autostart haben möchte aktiviere ich dieses mittels `sudo systemctl enable docker`. Anschließend starte ich meinen rapsi neu.

## Day Two - Hour Two
Dank Caps-Lock dachte ich kurz, dass ich mein Passwort vergessen habe. `docker run hello-world` spuckt als Ergebnis aus, dass Docker richtig installiert ist. Der nächste Schritt wäre Portainer. Laut der [Website](https://docs.portainer.io/v/ce-2.9/start/install/agent/docker/linux) muss ich *nur*
```
docker run -d -p 9001:9001 --name portainer_agent --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker/volumes:/var/lib/docker/volumes cr.portainer.io/portainer/agent:2.9.3
```
ausführen.

Das tut soweit auch, allerdings komme ich noch nicht über den Browser auf die Portainer installation. Freundlicherweise verweist die Portainer Website zu diesem Problem auf die *Docker Dokumentation* ~ihr mich auch jungs~. [Docker Dokumentation](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-socket-option) sagt ich muss socket options auf tcp ändern. Aber soll aufpassen und den [access protecten](https://docs.docker.com/engine/security/protect-access/). Also machen wir das mal so wie es dort steht.
```
openssl genrsa -aes256 -out ca-key.pem 4096
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
openssl genrsa -out server-key.pem 4096
openssl req -subj "/CN=raspi.local" -sha256 -new -key server-key.pem -out server.csr
echo subjectAltName = DNS:raspi.local,IP:127.0.0.1 >> extfile.cnf
echo extendedKeyUsage = serverAuth >> extfile.cnf
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out server-cert.pem -extfile extfile.cnf
openssl genrsa -out key.pem 4096
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
echo extendedKeyUsage = clientAuth > extfile-client.cnf
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out cert.pem -extfile extfile-client.cnf
rm -v client.csr server.csr extfile.cnf extfile-client.cnf
chmod -v 0400 ca-key.pem key.pem server-key.pem
chmod -v 0444 ca.pem server-cert.pem cert.pem
dockerd \
    --tlsverify \
    --tlscacert=ca.pem \
    --tlscert=server-cert.pem \
    --tlskey=server-key.pem \
    -H=0.0.0.0:2376
docker --tlsverify \
    --tlscacert=ca.pem \
    --tlscert=cert.pem \
    --tlskey=key.pem \
    -H=raspi.local:2376 version
```
Der letzte Befehl funktioniert nicht - also starte ich meinen Raspi mal wieder neu. Ich hab ehrlich keine Ahnung was das tut oder nicht tut, die Meldung ist:
```
Client: Docker Engine - Community
 Version:           20.10.12
 API version:       1.41
 Go version:        go1.16.12
 Git commit:        e91ed57
 Built:             Mon Dec 13 11:44:32 2021
 OS/Arch:           linux/arm64
 Context:           default
 Experimental:      true
Cannot connect to the Docker daemon at tcp://127.0.0.1:2376. Is the docker daemon running?
```
Ich mache trotzdem noch:
```
mkdir -pv ~/.docker
cp -v {ca,cert,key}.pem ~/.docker
export DOCKER_HOST=tcp://raspi.local:2376 DOCKER_TLS_VERIFY=1
```
und jetzt tut docker gar nicht mehr - offensichtlich ist irgendwas ziemlich kaputt.

## Day Two - Hour Three
Okay scheinbar muss ich tcp auch noch über `dockerd` aktivieren. Hierfür nehme ich den Befehl aus der [Doku](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-socket-option) mit den IPs die ich oben irgendwo gesetzt habe.
```
sudo dockerd -H unix:///var/run/docker.sock -H tcp://raspi.local -H tcp://127.0.0.1
```
okay das war auch nicht ganz so richtig, ich habe nachdem ich über `systemctl stop docker.service` den service gestoppt habe nochmal den dockerd befehl von oben eingegeben und siehe da die letzten beiden Befehle die vorher nicht funktioniert haben, funktionieren jetzt. ABER ich habe jetzt auch eine offene konsole - das ist nervig.
Ich beende Konsole über strg-C. Anschließend starte ich den Befehl
```
sudo dockerd \
  -H unix:///var/run/docker.sock\
  --tlsverify\
  --tlscacert=ca.pem\
  --tlscert=server-cert.pem\
  --tlskey=server-key.pem\
  -H=0.0.0.0:2376
```
ich habe eine grundlegende Ahnung was dieser Befehl tun könnte.

## Day Two - Hour Four

Das mit dem TLS war ne scheiß Idee - ich mach das jetzt alles nicht rückgängig, aber ich änder mal den `DOCKER_HOST` wieder.
```
export DOCKER_HOST=unix:///var/run/docker.sock DOCKER_TLS_VERIFY=0
```


Ich habe rausgefunden, dass der Container den ich gestartet habe, der falsche war `docker stop <name>` beendet den container. Ich probiere jetzt anhand der [Anleitung](https://docs.portainer.io/v/ce-2.9/start/install/server/docker/linux) Portainer zu starten.
```
docker volume create portainer_data
```

### Jetzt tut gar nichts mehr - ich fange von vorne an und installiere das raspi komplett neu - dieses mal mit SSH zugriff per default.
