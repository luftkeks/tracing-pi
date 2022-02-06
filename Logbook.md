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
