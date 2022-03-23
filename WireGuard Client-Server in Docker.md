# Konfiguracija virtualne privatne mreže alatom WireGuard

[Virtualna privatna mreža](https://en.wikipedia.org/wiki/Virtual_private_network) (engl. virtual private netvork, kraće VPN) pruža usluge privatne mreže korištenjem neke javne mreže kao što je internet. Pomoću VPN-a možemo slati i primati podatke preko javne mreže, a istovremeno koristiti konfiguraciju privatne lokalne mreže. VPN koristi različite sigurnosne mehanizme koji osiguravaju tajnost i autentičnost podaka koje šaljemo. Virtualnu privatnu mrežu moguće je stvoriti brojnim protokolima, npr. [L2TP](https://en.wikipedia.org/wiki/Layer_2_Tunneling_Protocol) na veznom sloju (koji implementira [xl2tpd](https://github.com/xelerance/xl2tpd)), [IPsec](https://en.wikipedia.org/wiki/IPsec) na mrežnom sloju (koji implementira [strongSwan](https://www.strongswan.org/)) i [SSTP](https://en.wikipedia.org/wiki/Secure_Socket_Tunneling_Protocol) na transportnom sloju (koji implementira [SoftEther VPN](https://www.softether.org/)).

WireGuard je vrlo jednostavan, ali moderan VPN poslužitelj i klijent koji koristi najsuvremenije kriptografske tehnologije. Cilja biti brži, jednostavniji, pouzdaniji i korisniji od IPsec-a. Također je ozbiljna konkurencija OpenVPN-u.

Inicijalno je izdan za sustave bazirane na Linux jezgri, no sada je postao višeplatformsko rješenje za uspostavu VPN mreže.
Razvoj WireGuarda još uvijek je u toku, iako se već sada smatra najsigurnijim i najjednostavnijim za korištenje VPN rješenjem u industriji.

WireGuard koristi najmodernije kriptografske tehnologije poput:
* [Noise protocol framework](http://www.noiseprotocol.org/), 
* [Curve25519](https://en.wikipedia.org/wiki/Curve25519), 
* [ChaCha20](https://www.cryptopp.com/wiki/ChaCha20), 
* [Poly1305](https://en.wikipedia.org/wiki/Poly1305), 
* [BLAKE2](https://www.blake2.net/), 
* [SipHash24](https://en.wikipedia.org/wiki/SipHash), 
* [HKDF](https://en.wikipedia.org/wiki/HKDF)

Izlaskom Linux jezgre 5.6, inačica WireGuarda za linux platformu prelazi u stabilno produkcijsko izdanje te se ugrađuje u samu Linux jezgru.
S obzirom da je Linux jezgra, i njezine komponente, otvorenog tipa (GNU General Public Licence (GPL) version 2), spajanjem WireGuarda s Linux jezgrom, izvorni kod postaje dostupan široj javnosti. 

Nakon povezivanja WireGuard klijenta s WireGuard poslužiteljem (koje opisujemo u nastavku), stvara se tunel kroz internet kroz koji se onda šalju podaci. Podaci koji prolaze tunelom su šifirirani kako ih treće strane na internetu ne bi mogle čitati, a dodatno se mogu koristiti i razne vrste kompresije kako bi podataka bilo manje.

# Podešavanje WireGuard VPN arhitekture koristeći Docker

## Instalacija alata Docker
### Ako nemate administratorske ovlasti
Zamolite Vašeg administratora da izvrši instalaciju alata Docker naredbom:
* Za sustave bazirane na Ubuntu distribuciji:
```shell
sudo apt-get install docker docker-compose
```
* Za sustave bazirane na Arch distribuciji:
```shell
sudo pacman -S docker docker-compose
```
Zatim instalirajte sve potrebne zavisnosti paketa za alat Docker:
* Za sustave bazirane na Ubuntu distribuciji:
```shell
sudo apt-get install -y dbus-user-session
sudo apt-get install -y uidmap
```
* Za sustave bazirane na Arch distribuciji:
```shell
sudo pacman -S shadow
sudo pacman -S fuse-overlayfs
```
Zatim ugasite Docker servis naredbom:
```shell
sudo systemctl disable --now docker.service docker.socket
```
Prije instalacije alata Docker potrebno je generirati subuid i subgid.
[Subuid](https://man7.org/linux/man-pages/man5/subuid.5.html) daje ovlasti korisničkom ID-u za mapiranje raspona korisničkih ID-jeva iz svog imenskog prostora u podređene imenske prostore.
[Subgid](https://man7.org/linux/man-pages/man5/subgid.5.html) daje ovlasti ID-ju korisničke grupe za mapiranje raspona ID-jeva grupe iz svojih imenskih prostora u podređene imenske prostore.

Za generiranje subuid-a i subgid-a potrebno je napisati Python skriptu sa slijedećim linijama:
```python
f = open("/etc/subuid", "w")
for uid in range(1000, 65536):
    f.write("%d:%d:65536\n" %(uid,uid*65536))
f.close()

f = open("/etc/subgid", "w")
for uid in range(1000, 65536):
    f.write("%d:%d:65536\n" %(uid,uid*65536))
f.close()
```
Zatim izvršavamo Python skriptu i vršimo instalaciju alata Docker iz repozirorija "[rootless](https://docs.docker.com/engine/security/rootless/)":
```shell
sudo python3 uid.py
curl -fsSL https://get.docker.com/rootless | sh
```

Po instalaciji, ako koristite sustave bazirane na Ubuntu distribuciji, izvršite označavanje varijabli i funkcija koje se prosljeđuju podređenim procesima (eng. "export"):
```shell
export PATH=/home/lljubojevic/bin:$PATH
export DOCKER_HOST=unix:///run/user/1000/docker.sock
```
Postavite odgovarajuće dozvole pristupa za Vašeg korisnika Docker servisima naredbom:
```shell
sudo chmod 666 /var/run/docker.sock
```
Na kraju, za potrebe podešavanja WireGuard klijent-poslužitelja izvršite naredbe za prihvaćanje prometa na vratima koje će WireGuard server koristiti:
```shell
sudo ufw allow 51820
sudo iptables -I INPUT -p tcp -m tcp --dport 51820 -j ACCEPT
```
Zatim, na Vašem korisničkom profilu izvršite pokretanje alata Docker:
```shell
systemctl --user enable docker
systemctl --user start docker
```
### Ako imate administratorske ovlasti
Izvršite instalaciju alata Docker na slijedeći način:
* Za sustave bazirane na Ubuntu distribuciji:
```shell
sudo apt-get install docker docker-compose
```
* Za sustave bazirane na Arch distribuciji:
```shell
sudo pacman -S docker docker-compose
```
Pokrenite servise alata Docker naredom:
```shell
sudo systemctl enable --now docker.service docker.socket
```
Dodatno, ako želite pokretanje alata Docker prilikom pokretanja sustava izvršite naredbu:
* Za sustave bazirane na Ubuntu distribuciji:
```shell
sudo loginctl enable-linger $(whoami)
```
* Za sustave bazirane na Arch distribuciji:
```shell
sudo loginctl enable-linger (whoami)
```

## Podešavanje WireGuard VPN poslužitelja
Pod pretpostavkom da imate administratorske ovlasti, za potrebe podešavanja WireGuard klijent-poslužitelja izvršite naredbe za prihvaćanje prometa na vratima koje će WireGuard server koristiti:
```shell
sudo ufw allow 51820
sudo iptables -I INPUT -p tcp -m tcp --dport 51820 -j ACCEPT
```
Kreirajte direktorij:
```shell
mkdir wireguard-server
```
Unutar direktorija wireguard-server stvorite datoteku "docker-compose.yaml"
```shell
nano docker-compose.yaml
```
Sadržaj datoteke **docker-compose.yaml za poslužitelja**:
```shell
version: "2.1"
services:
  wireguard:
    image: linuxserver/wireguard
    container_name: wireguard-server
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Zagreb
      - SERVERURL=10.13.13.1
      - SERVERPORT=51820
      - PEERS=1
      - PEERDNS=auto
    volumes:
      - /home/lljubojevic/wireguard-server/config:/config
      - /home/lljubojevic/wireguard-server/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv6.conf.all.disable_ipv6=0
    restart: unless-stopped
```
Izvršite generiranje Docker kontejnera naredbom:
```
docker-compose up -d
```
**Provjera ispravnosti postupka:**
Ukoliko naredba:
```shell
docker logs wireguard-server
```
Pruži izlaz oblika:
```shell
[s6-init] making user provided files available at /var/run/s6/etc...exited 0.
[s6-init] ensuring user provided files have correct perms...exited 0.
[fix-attrs.d] applying ownership & permissions fixes...
[fix-attrs.d] done.
[cont-init.d] executing container initialization scripts...
[cont-init.d] 01-envfile: executing... 
[cont-init.d] 01-envfile: exited 0.
[cont-init.d] 01-migrations: executing... 
[migrations] started
[migrations] no migrations found
[cont-init.d] 01-migrations: exited 0.
[cont-init.d] 02-tamper-check: executing... 
[cont-init.d] 02-tamper-check: exited 0.
[cont-init.d] 10-adduser: executing... 

-------------------------------------
_         ()
| |  ___   _    __
| | / __| | |  /  \ 
| | \__ \ | | | () |
|_| |___/ |_|  \__/


Brought to you by linuxserver.io
-------------------------------------

To support the app dev(s) visit:
WireGuard: https://www.wireguard.com/donations/

To support LSIO projects visit:
https://www.linuxserver.io/donate/
-------------------------------------
GID/UID
-------------------------------------

User uid:    1000
User gid:    1000
-------------------------------------

[cont-init.d] 10-adduser: exited 0.
[cont-init.d] 30-module: executing... 
Uname info: Linux 12320232623e 5.16.10-zen1-1-zen #1 ZEN SMP PREEMPT Wed, 16 Feb 2022 19:35:21 +0000 x86_64 x86_64 x86_64 GNU/Linux
**** It seems the wireguard module is already active. Skipping kernel header install and module compilation. ****
[cont-init.d] 30-module: exited 0.
[cont-init.d] 40-confs: executing... 
**** Server mode is selected ****
**** External server address is set to 10.13.13.1 ****
**** External server port is set to 51820. Make sure that port is properly forwarded to port 51820 inside this container ****
**** Internal subnet is set to 10.13.13.0 ****
**** AllowedIPs for peers 0.0.0.0/0, ::/0 ****
**** PEERDNS var is either not set or is set to "auto", setting peer DNS to 10.13.13.1 to use wireguard docker host's DNS. ****
**** Server mode is selected ****
**** Server related environment variables changed, regenerating 1 server and 1 peer/client confs ****
PEER 1 QR code:
█████████████████████████████████████████████████████████████████
█████████████████████████████████████████████████████████████████
████ ▄▄▄▄▄ █▀▀█▄█ ▄▄ ▄██ ▄▀▀▄▄▄ ▄█ ▄▄▄▄▄▀▄ █▄ ▄█▄▀▄ ██ ▄▄▄▄▄ ████
████ █   █ █▄█ ▀▀▄███ ▄ █▄▄█▄▀▄█▄▀ ▄▄█ █▄▀▀▄▄▀▀▄▄▀▄ ██ █   █ ████
████ █▄▄▄█ █ █▄▀▄▄████ ▀██▀▀▄  ▄▄▄ ▀ ▄██▄▀▄ █  ▀▀▄▀▄██ █▄▄▄█ ████
████▄▄▄▄▄▄▄█ ▀ █ █ ▀ ▀ ▀ ▀ ▀▄█ █▄█ ▀▄▀▄█▄█ ▀▄█▄▀ ▀ ▀▄█▄▄▄▄▄▄▄████
████  ▀▄█▀▄ ▀▄█  ▄▀██▀▄▀█  ▄▄▀▄ ▄▄▄▀█▀▄▄ ▀█▀█ █▀▄█▀▄▄█▄  ▀  ▀████
████▀████▀▄  █ ▄█▀█ ▀▀▄█▄  ▀▀   ▀▀█▄█ █  ▀▀▄▀▄█   ▀▄▄ ▄ ▀ █  ████
██████▀   ▄█▄▀▀ ▄▄██▀█▄▄▄ ▀   ▀█ ▀█ ▄ ▀ ▄█▄ ▄ █▄▄█▀▀▄ ▄ ▄█▄█▄████
████ ▄    ▄ █▄▀▀█ █▀▄█▄██▄ ███▄███▄█ ▀▄▀▀  ▀▄ ██ █ █ █ ▀ ▄███████
████  ▄▄▀▀▄▀ ▄ ▀▄ ▄██▄▄▄▀▄▀ ██ █ ▀▄▄ █▄▄▄█  ▀ █  ▄▄██▄  ▄█▀██████
████▄▀▀▀█▀▄ ▄ ▄▀▀▀▄█ ▀▄███ ▄███▄ ▀▀▄ ▄▀▀▄ █▀▀ ▄▀ ███▀▀█▀▄███▄████
█████ █▀█ ▄██▀█ ████ ▀▄ █▄▄▄ ▄▄   ▄ ▀▄█ ▄ █▄▄██▄█ ▄ ██▄ ▄▀▄██████
████▀ ▄ ▀▀▄ █▀ ▄▄  █  ▄▀ █▄▀▀▀ ▀▄ ▀█ ▄█ ▄▀█ ▀▀▄▀▄█▄▄ ▄█  ▄▀█ ████
████▀▄ ▄██▄ ███▀▀▄██▀  ▄█▄ █▄▀  ▄ █▀▀ ▀▀▄▄ ▀▄▀█▀▄▀ ██▀█▄   ▄ ████
████ ▄▄█ ▄▄▄ ▄▄ ▀▄ ▄▀▀▀ ██▄  █ ▄▄▄ ▄▀   █ █▀ ▀█▄▀▀▄  ▄▄▄ █▄██████
████▄ ▄▄ █▄█ ▄ ▀ ▀█▄█▄█▄▄██▄▄  █▄█ ▄ █▄▄ ▄▄▄▀█▄█▄▄█▄ █▄█ ▀▀█▀████
████▄▀ ▄▄▄▄▄ ▄ ▀▀██ ██████ ▀▄▄▄ ▄▄ ▀▀▄ ████▀▀▄█  █▀   ▄▄ ▄█▄ ████
████ ▀██▄ ▄ ▀▄▄▀▀▄ ▄█▀  ▄▀ █▄ ▀▄█ ▄▀█▀▄▀ ▄▄█▄▄▄ ▀▀█ ▀█▀▄█▀██▀████
████ ▀█▄ █▄▄▄ ▀█▀███▀▄██▀▀█▀▀▀ ▄█▀▄▄▄▀▄▄▀█▀▄▄▄▄ █ ▄█▄▀ ▄█▀▄▄ ████
████▀ ▄██ ▄█  ▄ ▀▄▄  ▀▄▀ █▄ ▄▄▀  ▄▄ ▄▄ ▀▄▀▄████▄ █   █  ▀▄█▀▄████
█████▀▄▀▄▄▄▄▀▄█▀▀█  ▄▀▀█  ▄▀▄▀▄▄ ▀▄█▄ ▀█▄ ▄▄ ██▀█▀██▀ ▄▀▄█▄▀█████
████▀▄▄█ █▄██▄ ▄ ▄ █▄▄ █  ▀ ▄  ███   ▀██▄  ▀ ▄▀▄▄▀▄█▀▄ █▄▀ ▄▄████
████ ▄▀███▄▀  ▄█▀ █▀ █▄█  █▀ █▄▀▀█ ▀█  ▀   ▀▄▄▀ ██ ██▄ ▀▀███▄████
████▀█▄▀▀▀▄▄ ▄▀ █  ▄▄▄▄█▄ █ █▀█▀▄ █ ██▄▀ ███ ▀▄ ▀█▄ █▀ █▄▀██▀████
████ ▀ ▀▀▄▄▀  ▀▀▀▀▄▀ ▄▄▄▄█   ▀▄▀█▀  █ ▄█▀▄▀█▀█▀ ▄▀▄  ▀ ▄█ ██ ████
██████████▄█ ▀▄█   █ ▄▄▀█▄ █▄█ ▄▄▄ ▀▄▀ ▄▄▀▄█ █▄▄ ▄▀█ ▄▄▄ ▄█ ▄████
████ ▄▄▄▄▄ █▄   ▄█▀▄▄ ▀█▄▀▄  ▀ █▄█    █▀▄▀▄██▀ ▄▀█▀▀ █▄█ █▄ ▀████
████ █   █ █▀▄█▀ ▀▄█▀█▄▀   ▀ ▄▄    ▀▄▀█▄▀█ ▀█▀█▀ █▄█  ▄  ██ ▀████
████ █▄▄▄█ █▀█▀▄▀▄███ ▄▄▄█ █▀▄▀▄▄ ▄▀▄▄▀█▄ ▀█▄▀ ▄▄  ▀▄ █  ▄▄█▄████
████▄▄▄▄▄▄▄█▄█▄▄▄▄▄▄██▄▄▄▄▄███████▄██▄█▄██▄███████▄█▄█▄▄▄█▄▄█████
█████████████████████████████████████████████████████████████████
█████████████████████████████████████████████████████████████████
[cont-init.d] 40-confs: exited 0.
[cont-init.d] 90-custom-folders: executing...
[cont-init.d] 90-custom-folders: exited 0.
[cont-init.d] 99-custom-scripts: executing...
[custom-init] no custom files found exiting...
[cont-init.d] 99-custom-scripts: exited 0.
[cont-init.d] done.
[services.d] starting services
[services.d] done.
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.13.13.1 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] ip -4 route add 10.13.13.2/32 dev wg0
[#] iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
.:53
CoreDNS-1.9.0
linux/amd64, go1.17.6, ace3dcb
```
Poslužitelj je ispravno generiran.

## Podešavanje WireGuard VPN klijenta
Kreirajte direktorij:
```shell
mkdir wireguard-klijent
```
Unutar direktorija wireguard-klijent stvorite datoteku "docker-compose.yaml"
```shell
nano docker-compose.yaml
```
Sadržaj datoteke **docker-compose.yaml za klijenta**:
```shell
version: '3.7'
services:
  wireguard:
    image: linuxserver/wireguard
    container_name: wireguard-klijent
    restart: unless-stopped
    networks:
      - backbone
    volumes:
      - './wireguard:/config'
      - '/lib/modules:/lib/modules:ro'
    environment:
      - PUID=1000
      - PGID=1000
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv6.conf.all.disable_ipv6=0
       
networks:
  backbone:
    driver: bridge
```
Potrebno je generirati docker kontejner za klijenta, koji trenutno nije funkcionalan, kako bi se stvorio konfiguracijski direktorij:
```shell
docker-compose up -d
```
Zatim, po pokretanju kontejnera, potrebno je stvoriti WireGuard mrežno sučelje pomoću kojeg je klijent komunicirati s poslužiteljem:
```shell
docker exec -it wireguard-klijent ip link add dev wg0 type wireguard
```
Prilikom generiranja poslužitelja, stvoriti će se i konfiguracijska datoteka za čvorove (eng. "peer") pod nazivom "peerX.conf". Sadržaj te datoteke potrebno je prepisati u datoteku wg0.conf koja će se nalaziti u direktoriju wireguard-klijent. 
Postupak teče:
```shell
cp ./wireguard-server/config/peer/peer1.conf ./wireguard-klijent/wireguard/wg0.conf
```
Datoteka wg0.conf je oblika:
```shell
[Interface]
Address = 10.13.13.2
PrivateKey = 8NWt7q+Vg2f5IdULtIZQ06osuEfNZI4aOf3fjp0CTXY=
ListenPort = 51820
DNS = 10.13.13.1

[Peer]
PublicKey = nRWxNsBzC+bre4qzjLsWQp8Y9vo82JYF7N8XNLiTaxQ=
Endpoint = 10.13.13.1:51820
AllowedIPs = 0.0.0.0/0, ::/0
```

Napomene:
* Ukoliko stavite parametar iz kofiguracije poslužitelja u broj veći od jedan (PEERS=n) nastati će n direktorija za čvorove i konfiguracije za njih, upute podrazumijevaju da imate samo jedan klijentski čvor.
* Datoteka wg0.conf može varirati.

## Testiranje klijent-poslužitelj VPN arhitekture:
```shell
docker exec -it wireguard-server curl -w "\n" ifconfig.me
46.188.154.181

docker exec -it wireguard-klijent curl -w "\n" ifconfig.me
46.188.154.181
```
Ukoliko su IP adrese iste, konfiguracija je valjana.
Prikazana IP adresa je Vaša trenutna (ili trajna) IP adresa kojom pristupate internetu.
# Pojašnjenje konfiguracijskih parametara u datotekama docker-compose.yaml
Poslužiteljska strana mora sadržavati parametre:
* version: "2.1" - tražena verzija alata docker-compose
* services: - servisi koje kontejner pokreće
* wireguard: - ime servisa
* image: - tražena slika koju alat Docker povlači iz svog repozitorija, te container_name: - proizvoljni naziv kontejnera.
* cap_add: - pruža kontejneru povišena dopuštenja na domaćinskom računalu i omogućuje mu interakciju s mrežnim sučeljima domaćina.
* environment: - postavke okruženja unutar Docker kontejnera
    * PUID i PGID - varijable koje definiraju korisnika i grupu
    * TZ=Europe/Zagreb - vremenska zona kontejnera, potrebno definirati
    * SERVERURL - naziv domene poslužitelja (može biti prazan, definiran javnom IP adresom poslužitelja ili postavljen na automatsko konfiguriranje)
    * SERVERPORT - vrata na kojima se poslužitelj otvara
    * PEERS - broj čvorova u VPN mreži (može biti broj ili naziv tipa: laptom, tablet, phone)
    * PEERDNS - kad je postavljen na auto, kontejner sam podešava DNS postavke
* volumes: - direktoriji na domaćinu koje kontejner smije koristiti
* ports: - vrata koja kontejner smije koristiti
* sysctls: - parametri koji omogućuju da kontejner komunicira s mrežama prema van 
* restart: - definira u kojim se slučajevima kontejner smije ponovno pokrenuti.

# Izvori
[Pedrolamas](https://www.pedrolamas.com/2020/11/20/how-to-connect-to-a-wireguard-vpn-server-from-a-docker-container/)
[Markontech](https://markontech.com/linux/install-wireguard-vpn-server-with-docker/)
[Spad.uk](https://spad.uk/wireguard-as-a-vpn-client-in-docker-using-pia/)
