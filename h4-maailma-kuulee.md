*Tekijä: Aapo Tavio*

# h4 Maailma kuulee

Tehtävässä tarkoituksena oli vuokrata oma virtuaalinen serveri palveluntarjoajalta, tehdä välttämättömät muutokset asetuksiin uudella virtuaalisella serverillä ja asentaa apache demoni virtuaaliselle palvelimelle. Lisäksi vaihdoin vielä oman sivun oletussivuksi apacheen name based virtual host -menetelmällä.

## x) Tiivistelmät
•	Pilvipalveluntarjoajia on olemassa useita  
•	Ensiaskeleet uudella palvelimella on pakettien päivittäminen, palomuurin asentaminen, uuden käyttäjän tekeminen ja käyttäjän asettaminen sudo-ryhmään  
•	Tavoite on päästä kirjautumaan uudella käyttäjällä ja lukita root-käyttäjä tämän jälkeen  
•	Suojaus on todella tärkeää, koska murtautumisia yritetään koko ajan, jopa automatisoidusti  
(Lehto 2022, URL: https://susannalehto.fi/2022/teoriasta-kaytantoon-pilvipalvelimen-avulla-h4/)

•	On tärkeää muistaa tehdä etäpalvelimelle ssh-portti auki palomuuria asentaessa  
•	Julkiseen serveriin on myös konfiguroitava avonainen portti, jotta siihen voi saada yhteyden julkisesta verkosta  
(Karvinen 2017, URL: https://terokarvinen.com/2017/first-steps-on-a-new-virtual-private-server-an-example-on-digitalocean/

## Käytettävän ympäristön ominaisuudet

- Isäntä:
  >- HP Laptop 15s-eq3xxx  
  >- Microsoft Windows 11 Home (versio 24H2)  
  >- AMD Ryzen 7 5825U, Radeon Graphics  
  >- 16 GB RAM (15,3 GB käytettävissä)
  >- x64-pohjainen
  >- Verkkokorttina Realtek WiFi 6

- Virtuaalikone
  >- Debian GNU/Linux 12 (bookworm) xfce
  >- Virtualbox

Verkko yhteytenä minulla oli tehtävässä kotini reitittimen kautta tuleva valokuitu 200Mb/s nopeudella.  
Suoritin virtuaalisen serverin vuokraamisen isäntäkoneella ja isäntäkoneen virtuaalikoneella otin yhteyden ssh:lla maksulliseen virtuaaliserveriin.

## a) 7.2.2025 klo 13.30  
### SSH-avaimen luominen
Aloitan luomalla ssh-avainparin virtuaalikoneellani. Ensimmäiseksi komento: ”sudo apt-get update”, jonka jälkeen ”sudo apt-get -y dist-upgrade”. Huomasin ensin komennolla ”apt search openssh client”, että minulla olikin todennäköisesti jo valmiina asennettu kyseinen ohjelma. Varmistin asian ajamalla komennon: ”apt list –installed|grep openssh”, joka ilmaisi ohjelman olevan jo valmiiksi asennettu.

![]()

### Palvelimen vuokraaminen
Päätin tehdä virtuaalisen palvelimeni DigitalOcean-palveluun, koska sain ilmaisia krediittejä GitHub Education ohjelman kautta. Valitsin Amsterdamista palvelimeni, koska sen lisäksi oli vain toinen vaihtoehto, joka oli Frankfurt. Käyttöjärjestelmäksi luonnollisesti otin Debian 12:a, koska se on kurssilla käytettävä käyttis.

Koneeksi valitsin 6usd/kk maksavan, joka oli 1GB prosessorilla ja 25GB SSD-levyllä varustettu. SSH-avaimen valitsin pääsynhallintametodiksi, koska salasana on todennäköisesti turvattomampi. Muutin samalla host-nimen valmiiksi digitalocean:n sivuilla, luodessani virtuaalikonetta.

## b) 7.2.2025 klo 15.05  
### Uusi käyttäjä ja palomuuri
Aloitin yhteyden muodostamisen komennolla ”ssh root@142.93.132.235”. Pääsin sisään, jonka jälkeen tein uuden käyttäjän komennolla ”sudo adduser aapo”. Lisäsin vielä uuden käyttäjän sudo-ryhmään komennolla ”sudo adduser aapo sudo”.

Siirryin aapo-käyttäjälle ja aloitin palomuurin asennuksen ajamalla ”sudo apt-get update”. Ufw-palomuurin asennus: ”sudo apt-get install ufw”. Minun piti vielä avata portti ssh-yhteydelle ennen kuin laitan palomuurin päälle. Tämä tapahtui komennolla ”sudo ufw allow 22/tcp”. Jonka jälkeen palomuuri päälle, joten ajoin ”sudo ufw enable”. Varmistin vielä palomuurin olevan päällä ja oikean portin olevan auki ”sudo ufw status” komennolla ja kaikki näytti olevan oikein.

### Root-käyttäjän sulkeminen
7.2.2025 klo 15.38  
Seuraavaksi vuorossa oli ”sudo su”, jolla menin root-käyttäjälle takaisin. Kopioin ensin ssh-asetukset komennolla ”sudo cp -rvn /root/.ssh/ /home/aapo/”, jonka jälkeen siirsin omistuksen aapo-käyttäjälle hakemistoon /home/aapo/. Aikaisemmin tämä hakemisto oli rootin omistuksessa.

Koitin tässä vaiheessa ottaa yhteyden aapo-käyttäjään suoraan, joten ajoin komennon ”ssh aapo@142.93.132.235”. Ja sehän onnistui! Varmistin vielä ssh-konfigurointi tiedostosta, että salasanalla kirjautuminen oli estetty ja vain ssh-avaimella saa ottaa etäyhteyden. Kaikki oli kunnossa. Tämän jälkeen suljin root-käyttäjän komennolla ”sudo usermod –lock root”. Kävin vielä tarkistamassa, että root-käyttäjä oli lukittu avaamalla tiedoston polusta /etc/shadow, josta näinkin rootin olevan lukittu, koska salasanan kohdalla ei ole tiivistettä. Sen sijaan siinä on merkit: ”!*”. Lopuksi nimesin vielä Teron ohjeiden mukaan uudestaan root-käyttäjän hakemiston: .ssh, hakemistoksi nimeltä: DISABLED-ssh, joka selkeästi ilmaisee, että ssh-yhteyttä ei saa enää kyseiseen käyttäjään (Karvinen, URL: https://terokarvinen.com/linux-palvelimet/). Tämä tapahtui komennolla ”sudo mv -nv /root/.ssh /root/DISABLED-ssh/”.

### SSH-yhteyden sulkeminen root-käyttäjältä
7.2.2025 klo 16.30  
Huomasinkin Teron materiaaleissa olevan kohta, jossa erikseen estetään ssh-yhteys root-käyttäjälle, joten kävin katsomassa asiaa. Tiedostossa sshd_config, polussa /etc/ssh olikin vielä etäyhteys sallittu rootille. Muutin kohdassa ”PermitRootLogin yes” muotoon ”PermitRootLogin no”, jolloin ssh yhteys pitäisi olla estetty.

Lähdin kokeilemaan, että onhan ssh-yhteys estetty rootille. Siispä komento ”ssh root@142.93.132.235” antoikin kielteisen vastauksen kirjautumiseen, jota toivoinkin. Tämän jälkeen päivitin koneen, joten komennot ”sudo apt-get update” ja ”sudo apt-get upgrade”.

## c) 8.2.2025 klo 9.19  
### Ohjelmien asentaminen ja palomuurin konfigurointia
Asensin ensin helpottaakseni työskentelyäni micro-editorin, joten komennot ”sudo apt-get update” ja ”sudo apt-get install -y micro”. Tämän jälkeen tuli apachen vuoro, joten ajoin ”sudo apt-get -y install apache2”. Tuttuun tapaan edellisen tehtävän tavoin vaihdoin oletussivun komennolla: ’echo "Hey you"|sudo tee /var/www/html/index.html’.

Palomuuriin piti tietenkin tehdä reikä, jos haluaa sivun julkiseksi, joten komento ”sudo ufw allow 80/tcp”. Tarkastin portin 80 olevan auki kuten pitääkin.

## d) 8.2.2025 klo 9.56
### Name based virtual host apacheen
Seuraavaksi ”sudoedit /etc/apache2/sites-available/new.com.conf” ja tarpeelliset name based virtual host -tiedot kyseiseen tiedostoon. Minun piti tehdä vielä uusi kansio, johon viittasin konfigurointi tiedostossa, joten komento ” sudo mkdir -p /home/websites/publicsites/new.com”. Kyseinen new.com-hakemisto oli rootin omistuksessa ja minulla ei ollut omalla käyttäjällä oikeuksia muokata hakemistoa, joten ajoin ”sudo chown -R aapo /home/websites/publicsites/new.com/”. Halusin vielä varmuuden vuoksi muilta käyttäjiltä kaikki oikeudet pois, joten ajoin ”chmod o-rwx new.com/”. En sisällyttänyt polkuna kuin new.com-hakemiston, koska olin valmiiksi polussa: /home/websites/publicsites.

8.2.2025 klo 10.24  
Loin edelleen index.html-tiedoston polkuun: /home/websites/publicsites/new.com, jotta saisin oman sivuni näkyviin. Oletussivu pois päältä komennolla: ”sudo a2dissite 000-default.conf” ja apachen uudelleenkäynnistys ”sudo systemctl restart apache2”.

Uusi sivu päälle, joten komento ”sudo a2ensite new.com.conf” ja uudelleenkäynnistys tietysti ”sudo systemctl restart apache2”. Sivulle ei päässyt, koska sivu antoi 403-koodin, joka tarkoittaa estettyä pääsyä.

### Ongelman ratkaiseminen
8.2.2025 klo 11.00  
Kävin katsomassa ensimmäiseksi virhelokia. Tämän jälkeen katsoin konfigurointitiedoston, jossa ei näkynyt ongelmia mielestäni. Sitten muokkasin index.html-tiedostoa ja ajattelin varsinkin <!doctype html> elementin auttavan, mutta ei auttanut. Tarkistin palomuurin, jonka jälkeen vielä new.com.conf-tiedoston olemassaolon sites-enabled-hakemistosta.

Minulle tuli mieleen, että voisiko asia johtua oikeuksista, koska poistiin kaikki oikeudet muilta käyttäjiltä new.com-hakemistosta. Löysin Better Stackin (URL: https://betterstack.com/community/questions/what-permissions-should-my-website-directory-have-on-linux/) ja serverfault (URL: https://serverfault.com/questions/357108/what-permissions-should-my-website-files-folders-have-on-a-linux-webserver) -sivuilta tietoa apachen vaatimista oikeuksista hakemistoihin ja tiedostoihin.

Aloitin prosessin antamalla lukuoikeuden hakemistoon /home/websites/publicsites/new.com/ komennolla ”chmod o+r new.com/”, olin valmiiksi jo polussa /home/websites/publicsites/. Käynnistin apachen uudelleen. Edelleenkään ei toiminut, joten ajamisen oikeudet new.com/-hakemistoon komennolla ”chmod o+x new.com/” ollessani polussa /home/websites/publicsites/.

8.2.2025 klo 11.42  
Curlilla toimii ainakin localhost! Ja näköjään puhelimeni selaimenkin kautta, kun syötän osoiteriville digitalocean-virtuaalikoneeni IP-osoitteen. Loppu hyvin, kaikki hyvin.

<br>
<br>

## Lähteet

Better Stack Team 2023. What permissions should my website files/folders have on a Linux webserver? Luettavissa: https://betterstack.com/community/questions/what-permissions-should-my-website-directory-have-on-linux/. Luettu: 8.2.2025.

Karvinen, T. 19.9.2017. First Steps on a New Virtual Private Server – an Example on DigitalOcean and Ubuntu 16.04 LTS. Luettavissa: https://terokarvinen.com/2017/first-steps-on-a-new-virtual-private-server-an-example-on-digitalocean/. Luettu: 7.2.2025.

Karvinen, T. Linux Palvelimet 2025 alkukevät. Luettavissa: https://terokarvinen.com/linux-palvelimet/. Luettu: 7.2.2025.

Lehto, S. 14.2.2022. Teoriasta käytäntöön pilvipalvelimen avulla (h4). Luettavissa: https://susannalehto.fi/2022/teoriasta-kaytantoon-pilvipalvelimen-avulla-h4/. Luettu: 7.2.2025.

Server Fault forum. What permissions should my website files/folders have on a Linux webserver? Luettavissa: https://serverfault.com/questions/357108/what-permissions-should-my-website-files-folders-have-on-a-linux-webserver. Luettu: 8.2.2025.
<br>
<br>
<br>
<br>
<br>
<br>
*Tätä dokumenttia saa kopioida ja muokata GNU General Public License (versio 3 tai uudempi) mukaisesti. http://www.gnu.org/licenses/gpl.html*  
*Pohjana Tero Karvinen 2025: Linux Palvelimet 2025 alkukevät, https://terokarvinen.com/linux-palvelimet/*
