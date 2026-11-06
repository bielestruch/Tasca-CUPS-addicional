# AA2 - CUPS: impressora virtual compartida

## 2. Preparar les màquines i comprovar la xarxa

Primer he iniciat les dues màquines virtuals: Ubuntu Server, que farà de servidor, i Zorin OS, que serà el client.

He comprovat que les dues màquines estiguin connectades a la mateixa xarxa Host-Only. Al servidor he executat:


ip -br addr


Amb aquesta ordre he pogut veure les interfícies de xarxa i trobar la IP que faré servir per connectar el client amb el servidor.

Després, des del client, he fet un ping a la IP del servidor:


ping IP_DEL_SERVIDOR


Si arriben les respostes, vol dir que les dues màquines es poden comunicar.

**Dades de la meva configuració**
- IP del servidor: 192.168.56.108
- IP del client: 192.128.58.103

### Captura 1 - Comprovació de la xarxa

En aquesta captura es veu la IP del servidor i la comprovació de la connexió amb el client. Així puc confirmar que les dues màquines es comuniquen.

![Captura 1](images/screenshot1.png)

---

## 3. Instal·lar CUPS al servidor

Al servidor Ubuntu he actualitzat els paquets i he instal·lat CUPS amb aquestes ordres:


sudo apt update
sudo apt install cups cups-client


Després he activat el servei perquè s'iniciï automàticament i l'he posat en marxa:


sudo systemctl enable --now cups


Per comprovar que funciona, he executat:


systemctl status cups


He comprovat que el servei aparegui com a `active (running)`.

### Captura 2 - Instal·lació i estat de CUPS

En aquesta captura es veu la instal·lació de CUPS i que el servei està funcionant correctament.

![Captura 2](images/screenshot2.png)

---

## 4. Instal·lar la impressora virtual PDF

Ara he instal·lat el paquet que permet crear una impressora virtual que desa els documents en PDF:


sudo apt install printer-driver-cups-pdf


Després he comprovat quines impressores detecta CUPS:


lpstat -p -d


Aquí hauria d'aparèixer la impressora PDF. El nom pot variar segons la instal·lació, així que he apuntat el nom que surt al meu servidor.

**Nom de la impressora:** biel-VirtualBox

### Captura 3 - Impressora PDF instal·lada

En aquesta captura es veu que el paquet de la impressora PDF està instal·lat i que CUPS la detecta.

![Captura 3](images/screenshot3.png)

---

## 5. Permetre l'accés a CUPS des de la xarxa

Per poder utilitzar la impressora des del client, he de permetre que CUPS accepti connexions de la xarxa.

Primer he fet una còpia del fitxer de configuració:


sudo cp /etc/cups/cupsd.conf /etc/cups/cupsd.conf.backup


Després l'he obert per editar-lo:


sudo nano /etc/cups/cupsd.conf


He revisat la configuració perquè CUPS escolti pel port 631 i permeti l'accés des de la xarxa local. També he comprovat que les seccions d'accés permetin els equips de la xarxa local amb `@LOCAL`.

A continuació he executat:


sudo cupsctl --remote-admin --share-printers

I he reiniciat el servei perquè s'apliquin els canvis:


sudo systemctl restart cups


Per comprovar que CUPS escolta pel port 631, he utilitzat:


sudo ss -ltnp | grep 631


### Captura 4 - Configuració de CUPS

En aquesta captura es veu la configuració que he modificat per permetre l'accés des de la xarxa.

![Captura 4](images/screenshot4.png)

### Captura 5 - CUPS escoltant pel port 631

En aquesta captura es veu que CUPS està escoltant pel port 631, que és el que utilitza per a les connexions.

![Captura 5](images/screenshot5.png)



## 6. Compartir la impressora

Des del navegador del servidor he entrat a la pàgina d'administració de CUPS:


http://localhost:631


He anat a l'apartat **Administration** i després a **Manage Printers**. He seleccionat la impressora PDF i he comprovat que estigués compartida a la xarxa.

També es pot entrar des del client utilitzant la IP del servidor:


http://IP_DEL_SERVIDOR:631


### Captura 6 - Administració de CUPS

En aquesta captura es veu la pàgina web de CUPS i la impressora que he configurat.

![Captura 6](images/screenshot6.png)



## 7. Afegir la impressora al client Zorin OS

Ara he anat al client Zorin OS. Des de la configuració del sistema he entrat a l'apartat d'impressores i he afegit una impressora de xarxa.

He utilitzat la IP del servidor i el nom de la cua de la impressora. 


He substituït 192.128.58.103  i Biel-VirtualBox pels valors reals de la meva configuració.

Quan l'he afegit, he comprovat que aparegués a la llista d'impressores del client.

### Captura 7 - Impressora afegida al client

En aquesta captura es veu la impressora de xarxa que he afegit a Zorin OS.

![Captura 7](images/screenshot7.png)

---

## 8. Fer una prova d'impressió

Per comprovar que funciona, he creat un document de prova al client i l'he enviat a la impressora virtual.

També es pot fer una prova des del terminal amb un fitxer de text:


echo "Prova d'impressio CUPS" > prova.txt
lp -d NOM_DE_LA_IMPRESSORA prova.txt


He canviat Estruch-VirtualBox pel nom real de la cua que he configurat.

Per veure els treballs d'impressió he utilitzat:


lpstat -o


Després he anat al servidor i he comprovat que s'hagués creat el PDF. En una instal·lació habitual de CUPS-PDF, els documents es guarden dins de:


/var/spool/cups-pdf/


Per buscar el fitxer PDF, he utilitzat:


sudo find /var/spool/cups-pdf -type f -name "*.pdf"


La ubicació exacta pot variar segons la configuració del paquet.

### Captura 8 - Prova d'impressió i PDF creat

En aquesta captura es veu el resultat de la prova i el fitxer PDF que s'ha creat al servidor.

![Captura 8](images/screenshot8.png)

---