
# Guia pràctica: Configuració de DNAT i VPN (Roadwarrior) amb IPFire

## Escenari:
* **Servidor IPFire** en xarxa interna (GREEN) i xarxa externa (RED).
* **Equip local Zorin OS** (client web) a la xarxa GREEN (IP privada, per exemple `192.169.24.1`).
* **Client extern Windows** a la xarxa RED (NAT).

---

## 1. Preparació del servidor web a Zorin OS

Al Zorin instal·lem els serveis SSH i Apache. Obre un terminal i executa:

sudo apt update && sudo apt install apache2 openssh-server

Un cop instal·lat, editem la pàgina web per defecte perquè mostri un missatge personalitzat:

sudo nano /var/www/html/index.html

---

## 2. Destination NAT (DNAT) – Accés a la web des de l’exterior

Volem que el client Windows (xarxa RED) pugui veure la web del Zorin posant la IP de l’adaptador RED de l’IPFire.

### 2.1 Crear la regla DNAT a IPFire

Accedeix a l’entorn gràfic d’IPFire. Ves a **Firewall → Reglas de cortafuegos** i fes clic a **Agregar**. Omple els camps:

* **Origen:**
* Direcció d’origen: Todos
* Xarxes estàndard: RED


* **NAT:** Marca **Usar traducción de direcciones de red (NAT)** i selecciona **NAT de destino (DNAT – reenvío de puertos)**.
* **Destí:**
* Adreça de destí: `192.169.24.1` (IP del Zorin a GREEN)
* Xarxes estàndard: Cualquiera


* **Protocol:** TCP
* Port d’origen: (deixa buit)
* Port de destí: `80`
* Port extern (NAT): `80` (o el que vulguis obrir a la RED)


* **Ajustos addicionals:** Pots posar una observació com “Web Zorin”.

### 2.2 Prova des del client Windows

Des del client Windows, obre un navegador i accedeix a la IP de l’adaptador RED de l’IPFire (per exemple `http://10.0.2.25`). Hauries de veure la pàgina del Zorin.

> ⚠️ **Si no funciona:** Revisa que el servei Apache estigui actiu a Zorin (`systemctl status apache2`) i que la IP RED de l’IPFire sigui accessible des del Windows.

---

## 3. VPN amb OpenVPN (Roadwarrior)

Ara farem el mateixamp però de forma segura, creant un túnel xifrat sense obrir ports directament a Internet.

### 3.1 Generar els certificats de l’IPFire

A l’IPFire, ves a **Servicios → OpenVPN**.

Com que és la primera vegada, apareixerà un botó **Generate Root/Host Certificates**.

**Omple el formulari** (organització, país, etc.) i **genera’ls**. Pot trigar una mica.

### 3.2 Afegir l’usuari Roadwarrior

A la pestanya **Control y estado de conexión** fes clic a **Agregar**.

Selecciona Tipo de conexión: **Red privada virtual VPN Host-to-Net (Roadwarrior)** i torna a clicar **Agregar**. A la configuració, conexió, Activat: marca ✓

### 3.3 Exportar els fitxers del client

Un cop creat l’usuari, l’IPFire generarà dos fitxers:

* **Serveis24.ovpn** (configuració OpenVPN)
* **Serveis24.p12** (certificat + clau privada)

Descarrega’ls, aquests els hauràs de passar a la VM client extern (windows), per tant pots usar serveis com Google Drive o webs que et generin un enllaç de descàrrega a partir de fitxers com FileMail.

### 3.4 Habilitar el servei

Dins de **Servicios, OpenVPN** marca l’opció **Activado**.

### 3.5 Preparar el client Windows

* **a) Instal·lar OpenVPN GUI** Descarrega e instal·la la **OpenVPN GUI** des del web oficial.
* **b) Editar l’arxiu .ovpn (molt important)** Obre l’arxiu `.ovpn` amb el Bloc de notes. Busca la línia que comença per `remote` i canvia la IP per la IP de l’adaptador RED del teu IPFire. Guarda els canvis.
* **c) Crear la carpeta de configuració** **Fes clic dret** sobre el icono de OpenVPN, fes clic a **Import files**, selecciona el fitxer `.ovpn`. Es crearà una nova carpeta al directori amb l’arxiu dins `C:\Users\usuari\OpenVPN\config`. Copia el `.p12` aquí dins també.
* **d) (Opcional però recomanable) Editar el fitxer hosts de Windows** Per permetre la resolució DNS del nom **ipfire.foodlogistic.test** (que apareix dins del `.ovpn`), editem el fitxer `C:\Windows\System32\drivers\etc\hosts` com a **administrador**. Afegim la línia:
`192.169.24.254 ipfire.foodlogistic.test`

### 3.6 Connectar la VPN

Fes clic dret a la icona d’**OpenVPN GUI** i selecciona **Conectar**. Et demanarà la contrasenya **PKCS12** si és que la vas posar en crear l’usuari.

### 3.7 Comprovar des del servidor IPFire

A l’IPFire, a **OpenVPN → Control y estado de conexión** veuràs l’usuari **Serveis24** amb estat **CONECTADO**.

### 3.8 Accedir a la web a través de la VPN

Des del client Windows, ara prova d’accedir a la **IP privada del Zorin** (per exemple `http://192.169.24.1`). Hauries de veure la mateixa pàgina web. Funciona correctament. Com que estàs dins del túnel VPN, la comunicació va xifrada.

```

```
