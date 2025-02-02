*Tekijä: Aapo Tavio*

# h3 Hello Web Server

Tiivistelmät:

•	Nimeen perustuva virtuaalinen palvelu (Name-based virtual host) mahdollistaa useiden palvelujen käyttämisen samalla IP-osoitteella  
•	Nimeen perustuvan virtuaalisen palvelun etsiminen perustuu IP-osoitteen selvittämiseen, jonka jälkeen http-pyynnössä oleva palvelun nimi selvitetään  
•	*-merkki IP-osoitteen kohdalla kaikissa virtuaalisten palvelujen asetuksissa, ohjaa liikenteen nimeen perustuvaan selvitykseen kyseisen yhden IP-osoitteen sisällä  
(The Apache Software Foundation, URL: https://httpd.apache.org/docs/2.4/vhosts/name-based.html) 

•	“sudo apt-get -y install apache2”-komennolla voi ladata apache web-palvelun  
•	/etc/apache2/sites-available/ -polkuun lisätään käyttäjän haluamat sivut  
(Karvinen 2018, URL: https://terokarvinen.com/2018/04/10/name-based-virtual-hosts-on-apache-multiple-websites-to-single-ip-address/) 

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

Verkko yhteytenä minulla on tehtävässä kotini reitittimen kautta tuleva valokuitu 200Mb/s nopeudella.

## Apache web-palvelimen asennus

28.1.2025 klo 19.45  
Aloitin apachen asentamisen komennolla ”sudo apt-get -y install apache2”. Tämän jälkeen vaihdoin oletussivun komennolla: ’echo "Default"|sudo tee /var/www/html/index.html’, josta sain vastauksena "Default".

Yritin tarkastella lokeja harjoituksen vuoksi ollessani polussa: /var/log. Annoin komennon ”ls -a apache2”, mutta sain vastaukseksi: ”Permission denied”. Kokeilen kaikkia komentoja ensin ilman sudoa, koska on turvallisempaa antaa komentoja ilman korotettuja oikeuksia. Yritin uudestaan korotetuilla oikeuksilla samassa polussa: /var/log. Tällä kertaa komento oli siis “sudo ls apache2/”, josta sain vastaukseksi kolme hakemistoa: access.log, error.log ja other_vhosts_access.log.  

Polussa: /var/log annoin edelleen komennon: ”tail apache2/error.log”, josta vastauksena sain: Permission denied. Joten seuraavaksi oli vuorossa komento: ”sudo tail apache2/error.log”, polussa: /var/log. Sain joitain ilmoituksia, mutta niissä ei ollut toistaiseksi mitään ei toivottua.  

1.2.2025 klo 18.15  
Ohjeista poiketen haluan mennä katsomaan mitä polussa /etc/apache2/sites-available sisältää, joten navigoin itseni kyiseiseen kansioon ja siellä olivat kyseisenlaiset tiedostot. Loin polkuun (jossa olen jo valmiiksi) /etc/apache2/sites-available tiedoston komennolla ”sudo nano hattu.example.com.conf ”. Yritin ensin komennolla ”sudoedit hattu.example.com.conf”, mutta tiedosto olisi mennyt väärään hakemistoon.  

Laitoin sivun hattu.example.com päälle komennolla ”sudo a2ensite hattu.example.com”, jonka jälkeen käynnistin uudelleen apachen komennolla ”sudo systemctl restart apache2”. Testasin localhost-osoitteen toimivuuden selaimella.  

## Lokit onnistuneesta sivun lataamisesta

Kuvassa näkyvä ”127.0.0.1” on apache-palvelimen IP-osoite. Seuraavana on kaksi väliviivaa. Ensimmäinen vasemmalta on asiakkaan identiteettitieto, jota ei minulla näytetty. Toinen on käyttäjän tunniste, joka pyytää resursseja.  

Seuraavana näkyy päivämäärä ja kellonaika. Olettaisin ”+0200” olevan ilmoitus aikavyöhykkeestäni. En löytänyt asiasta varmaa tietoa, mutta usealla sivulla kohdassa oli merkintä GMT, joten olettaisin asian olevan aikavyöhykkeeni. (Esim. GeeksforGeeks:n sivulla: https://www.geeksforgeeks.org/state-the-core-components-of-an-http-response/)  

”GET / HTTP/1.1” tarkoittaa get-metodia, jolla pyydetään juuresta ”/” tietoa http/1.1 protokollalla. ”200” on tilakoodi, joka tässä yhteydessä kertoo onnistuneesta suorituksesta. Seuraavana oleva ”-” ilmaisee asiakkaalle siirretyn tiedon määrän, jota tässä tapauksessa ei ole ollenkaan. "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0" on asiakkaan käyttämä selain. (W3schools. URL: https://www.w3schools.com/Tags/ref_httpmethods.asp, Fitzpatrick, S. URL: https://www.sumologic.com/blog/apache-access-log/, Wikipedia. URL: https://en.wikipedia.org/wiki/HTTP.)  

Alla oleva pyyntö eroaa vain muutamalla tavalla yllä olevasta. Polku tiedon hakemiseen on ”/favicon.ico”. Tilakoodi on myös eri: 404, joka merkitsee resurssien hakemisen epäonnistumista. Asiakkaalle siirretyn tiedon määrä on kuitenkin 487B. ”http://localhost/” viittaa osoitteeseen, josta tietoja pyydettiin. (Wikipedia. URL: https://en.wikipedia.org/wiki/List_of_HTTP_status_codes, https://en.wikipedia.org/wiki/Favicon.)  

## Localhost-sivun muuttaminen

1.2.2025 klo 20.30  
Yritin luoda hakemistoja ja tiedoston komennolla ” mkdir -p /home/xubuntu/publicsites/hattu.example.com/index.html”, mutta minulla ei ollut oikeuksia tehdä hakemistoja ja tiedostoa. Tämän jälkeen ajoin komennon ” sudo mkdir -p /home/xubuntu/publicsites/hattu.example.com”, koska ajattelin antaa oikeudet ilman sudoa ainoastaan kansioon hattu.example.com.  

Etsin netistä ohjeet oikeuksien muuttamiseen, koska en muistanut prosessia. Löysinkin freeCodeCampin sivuilta (Hira, Z. URL: https://www.freecodecamp.org/news/linux-chmod-chown-change-file-permissions/) ohjeet, miten muuttaa oikeuksia? Annoin komennon ”sudo chmod o+w /home/xubuntu/publicsites/hattu.example.com”, joka antaa kirjoitusoikeudet kaikille muille käyttäjille omistajan lisäksi. Tämän lisäksi loin tiedoston ”index.html” hakemistoon (jonne olin jo navigoinut) /home/xubuntu/publicsites/hattu.example.com komennolla ”nano index.html”. Samalla lisäsin tarvittavan sisällön tiedostooni.  

2.2.2025 klo 10.00  
Kävin vielä laittamassa oletussivun pois päältä komennolla ”sudo a2dissite 000-default.conf”, ollessani polussa: /etc/apache2/sites-available. Tämän jälkeen käynnistin apachen uudelleen komennolla: ”sudo systemctl reload apache2”.  

2.2.2025 klo 10.30  
Yritin mennä firefox-selaimen kautta localhost-sivulleni, mutta minulle tuli ilmoitus, että pääsy on estetty. Kävin ensin katsomassa apachen virhe lokin, joka ilmoitti DocumentRoot-polun puuttuvan. Huomasin polussa lukevan virheellisesti ”pyora.example.com”, joten menin katsomaan konfigurointi tiedostoa. Siellä olikin virheelliset tiedot poluissa. Kävin korjaamassa ne oikeiksi. Käynnistin apachen vielä uudestaan, jonka jälkeen sivu toimikin kuten pitääkin!  

Tarkistin vielä sivun täyttävän HTML5 kriteerit osoitteessa: https://validator.w3.org/. Lisäsin vielä lang-attribuutin ja merkkien koodauksen (character encoding) utf-8.  

2.2.2025 klo 11.00  
Annoin komennoksi curl -I, joka palautti minulle vastauksena otsakkeita. ”HTTP/1.1 200 OK” kertoo käytetyn protokollan ja tilakoodin (joka on onnistunut). Etag-numerot kertovat resurssin versiohistorian. Jos Etag ei ole muuttunut, voidaan säästää resursseja lataamalla sama sivu muistista. (https://everything.curl.dev/http/modify/conditionals.html.)  

<br>
<br>

## Lähteet

Curl 18.3.2024. everything curl. Conditionals. Luettavissa: https://everything.curl.dev/http/modify/conditionals.html. Luettu: 2.2.2025.  

Fitzpatrick, S. 14.1.2020. Understanding the Apache Access Log: View, Locate and Analyze. Luettavissa: https://www.sumologic.com/blog/apache-access-log/. Luettu: 1.2.2025.  

GeeksforGeeks 20.9.2021. Luettavissa: https://www.geeksforgeeks.org/state-the-core-components-of-an-http-response/. Luettu: 1.2.2025.  

Hira, Z. 27.4.2022. Linux chmod and chown – How to Change File Permissions and Ownership in Linux. Luettavissa: https://www.freecodecamp.org/news/linux-chmod-chown-change-file-permissions/. Luettu: 1.2.2025.  

The Apache Software Foundation. Luettavissa: https://httpd.apache.org/docs/2.4/vhosts/name-based.html. Luettu: 1.2.2025.  

W3schools. HTTP Request Methods. Luettavissa: https://www.w3schools.com/Tags/ref_httpmethods.asp. Luettu: 1.2.2025.  

Wikipedia 12.12.2024. http. Luettavissa: https://en.wikipedia.org/wiki/HTTP. Luettu: 1.2.2025.  

Wikipedia 18.1.2025. List of HTTP status codes. Luettavissa: https://en.wikipedia.org/wiki/List_of_HTTP_status_codes. Luettu: 1.2.2025.  

Wikipedia 21.1.2025. Favicon. Luettavissa: https://en.wikipedia.org/wiki/Favicon. Luettu: 1.2.2025.
<br>
<br>
<br>
<br>
<br>
<br>
*Tätä dokumenttia saa kopioida ja muokata GNU General Public License (versio 3 tai uudempi) mukaisesti. http://www.gnu.org/licenses/gpl.html*  
*Pohjana Tero Karvinen 2025: Linux Palvelimet 2025 alkukevät, https://terokarvinen.com/linux-palvelimet/*
