---
date: "2023-04-02"
title: Prova d'Avaluació Continuada 1 - PAC1 - Plataformes de distribució de continguts
summary: 
  Resolució de l'exercici proposat per M1.206 Plataformes de distribució de continguts. 30 de Març de 2023
author:
- Luca Rullo
abstract:
  Resolució de l'exercici proposat per M1.206 Plataformes de distribució de continguts. 30 de Març de 2023
toc: true
toc-depth: 2
colorlinks: true
fontsize: 10pt
geometry:
- margin=0.8in
draft: true
---

# 0. Instal·lació d'eines per a la gestió de serveis al núvol.

En aquesta primera pràctica he decidit treballar amb els serveis al núvol de Google, dins de l'entorn que Google anomena Google Cloud [^gcloud]. 

[^gcloud]: Google Cloud Platform (GCP), ofert per Google, és un conjunt de serveis de computació en núvol que s'executa a la mateixa infraestructura que Google utilitza internament per als seus productes d'usuari final, com ara la Cerca de Google, Gmail, Google Drive i YouTube. - <https://cloud.google.com/>

La raó principal es poder descobrir aquest servei de Google, ja que els serveis d'Amazon AWS ja els he estat utilitzant en altres projectes i tinc curiositat per comparar les dues opcions. Alhora, crec que a nivell de documentació les seves webs són molt més clares que les d'Amazon. 

Com a sysdev treballo molt utilitzant les eines via comandes o CLI i tots els proveïdora de cloud normalment ofereixen un client per facilitar les tasques de desplegament de serveis al seu núvol. Google Cloud, en aquest cas no és diferent, i ofereix l'eina GCloud i GUtils. 

Com a prerequisit, haurem de tenir un compte a Google i configurar un compte de facturació a la [Consola de Google](https://console.cloud.google.com/billing).

Per tant, la primera tasca que farem serà instal·lar el client a un sistema basat en GNU/Linux Debian en versió estable, el sistema que utilitzarem per desenvolupar aquesta pràctica, encara que hi ha versions per a sistemes basats en Windows, macOS o altres distribucions de GNU/Linux. Pots revisar la compatibilitat del teu sistema a la [web de documentació](https://cloud.google.com/sdk/docs/install?hl=es-419#deb).

No explicarem aquí com aixecar un sistema GNU/Linux Debian base per executar les comandes, però un mètode molt còmode per fer _testing_ de les aplicacions sense afectar al nostre sistema base és treballant amb Vagrant [^vagrant].

[^vagrant]: Vagrant està dissenyat per a tothom com la forma més senzilla i ràpida de crear un entorn virtualitzat - <https://www.vagrantup.com/>

~~~
$ vagrant init debian/bullseye64
$ vagrant up
$ vagrant ssh
~~~

Iniciem instal·lant les dependències per configurar el nou repositori.

~~~
$ sudo apt update && \ 
  sudo apt-get install apt-transport-https ca-certificates gnupg curl
~~~

I posteriorment afegim el nou repositori de Google al nostre llistat de fonts de paquets:

~~~
$ echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] \ 
	https://packages.cloud.google.com/apt cloud-sdk main" \
	| sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
~~~

Com els paquets estan signats via GPG, descarreguem la clau pública i la instal·larem al nostre sistema.

~~~
$ curl https://packages.cloud.google.com/apt/doc/apt-key.gpg \
  | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
~~~

Finalment, descarreguem i instal.lem els paquets:

~~~
$ sudo apt-get update && \
  sudo apt-get install google-cloud-cli
~~~

I executem el inicialitzador de GCloud:

~~~
$ gcloud init

Welcome! This command will take you through the configuration of gcloud.
...

You must log in to continue. Would you like to log in (Y/n)?  Y

Go to the following link in your browser:

[URL]

~~~

Aquí configurarem el nostre accés a partir d'un enllaç que ens redirigeix a la consola d'administració de _Google Cloud_. Obrirem el navegador, enganxarem la URL generada i acceptarem els permisos per a que _Google Cloud SDK_ [^gcloudsdk] pugui accedir al nostre compte. Després d'això, ens generarà un codi per l'activació i a continuació, si hem estat validats, ens indicarà que s'ha fet login correctament.

[^gcloudsdk]:Cloud SDK és un conjunt d'eines, que inclouen eines de línia d'ordres gcloud, gsutil i bq, biblioteques de clients i emuladors locals per desenvolupar amb Google Cloud. - <https://cloud.google.com/sdk/docs?hl=es-419>

Com és el primer cop que entrem, ens convidarà a crear un nou projecte.

~~~
This account has no projects.

Would you like to create one? (Y/n)?  
~~~

Per poder tenir un entorn de treball constant durant tota la pràctica, crearem un projecte (en el nostre cas _static001_) i el mantindrem durant tota la sessió.

# 1. Com crear un website estàtic en CDN
## 1.1 Identifica quin servei o serveis ofereix el CDN per tal fi.

Com hem explicat anteriorment, utilitzarem _Google Cloud_ per realitzar aquest primera pràctica i si hem realitzat correctament els passos de la introducció, tenim ja disponible l'entorn de desenvolupament.

El producte que seleccionarem per crear un web estàtic del conjunt de serveis de Google Cloud és __Cloud Storage__.

## 1.2 Descriu pas a pas el procés de creació del website en el CDN a partir dels tutorials didàctics que s'ofereixen al seu web.

Començarem creant el web estàtic al nostre ordinador local. Podem crear manualment els fitxers d'un web estàtic o podem utilitzar un generador de webs estàtiques com Hugo, Jekyll, ... Nosaltres utilitzarem Hugo [^hugo], per la seva facilitat i rapidesa en generar el nou web.

[^hugo]: Hugo és un dels generadors de llocs estàtics de codi obert més populars. Amb la seva increïble velocitat i flexibilitat, Hugo torna a crear llocs web divertits de nou - <https://gohugo.io/>

~~~
hugo new site uoc
cd uoc
git init
git submodule add \ 
 https://github.com/theNewDynamic/gohugo-theme-ananke themes/ananke
echo "theme = 'ananke'" >> config.toml
hugo
~~~

I a la carpeta _public_ tenim un petit projecte web per publicar.

Per iniciar amb _Google Cloud_, crearem un nou projecte si no l'hem creat anteriorment:

~~~
$ gcloud projects create static001
~~~

Revisem que el resultat hagi estat satisfactori.

Abans de continuar, necessitarem el domini al que apuntarem aquest nou web, encara que Google Cloud ens pot generar un enllaç sota el seu propi domini. En el meu cas, utilitzarem un domini ja adquirit per altres proves i afegirem un nou registre que apunti al servei de Google: uoc.domain.cc. A més d'afegir el nou registre CNAME, Google necessita tenir verificat el domini utilitzat per les proves amb el compte de Google que utilitzem, per tant, hem de fer la verificació de domini també. 

Aquesta modificació de registres es pot fer via web o si tens accés al servidor, pots fer els canvis oportuns via consola. En el meu cas, he fet els canvis al servidor: [^bind]

[^bind]: Utilizo Bind9 per gestionar el servidor de DNS a un VPS propi

~~~
# rndc freeze domain.cc
# vim /etc/bind/db.domain.cc
...
	uoc			CNAME	c.storage.googleapis.com.
	oqulmwcpcxpk.uoc	CNAME	gv-3o3grdjwmtsbjh.dv.googlehosted.com.
	
...
# rndc sign domain.cc
# rndc thaw domain.cc
~~~

Si les DNS s'han replicat correctament, podrem fer un test de resolució del nou subdomini des de la màquina on tenim instal·lat el gcloud

~~~
$ sudo apt install bind9-utils
$ dig uoc.domain.cc
...
;; ANSWER SECTION:
uoc.piperrak.cc.	600	IN	CNAME	c.storage.googleapis.com.
c.storage.googleapis.com. 184	IN	A	142.250.179.112
...
~~~

Ara, crearem un bucket, com anomena Google els recursos _storage_ al cloud:

~~~
$ gcloud storage buckets create gs://uoc.domain.cc
Creating gs://uoc.domain.cc/...
~~~

Ara publiquem el nostre _static site_ que hem generat amb Hugo i comprovem amb un llistat que s'hagin copiat correctament tots els fitxers.

~~~
$ gcloud storage cp -r public/* gs://uoc.domain.cc
$ gcloud storage ls gs://uoc.domain.cc
~~~

## 1.3 Com demostraries que funciona i que és visitada?

Si mirem d'accedir al web, veurem que no tenim accés encara [Figure 1]. 

Hem de donar permisos de lectura a nivell públic per a que els arxius estiguin disponibles per a tots els públics.

~~~
$ gcloud storage buckets add-iam-policy-binding gs://uoc.domain.cc \ 
   --member=allUsers --role=roles/storage.objectViewer
bindings:
- members:
  - projectEditor:static001
  - projectOwner:static001
  role: roles/storage.legacyBucketOwner
- members:
  - projectViewer:static001
  role: roles/storage.legacyBucketReader
- members:
  - allUsers
  role: roles/storage.objectViewer
etag: CAI=
kind: storage#policy
resourceId: projects/_/buckets/uoc.piperrak.cc
version: 1
~~~

I si tornem a accedir al nostre web _http://uoc.domain.cc_ veurem que es accessible. [Figure 2]

## 1.4 Quines eina/es ofereix per mesurar la latència? I altres paràmetres?

Des del panell d'administració del nostre projecte a _Google Cloud_ podem revisar l'estat del nostre servei. Aquí, trobem totes les eines per revisar els accessos, les actualitzacions, afegir nous usuaris a la gestió del nostre projecte, ... 

Per exemple, nosaltres en aquest cas, només hem utilitzat un _bucket_. Podem revisar l'estat del _bucket_ a la secció [Cloud Storage -> Bucket](https://console.cloud.google.com/storage/browser?project=static001&supportedpurview=project&prefix=)

A part de poder accedir al contingut del _bucket_, podem decidir on allotjar-lo als servidors de Google arreu del món, la seva replicació, el tipus de emmagatzemament segons la seva activitat (això canvia els costos) i podem revisar les consultes i els accessos a la pestanya _Observabilidad_. Aquí trobarem si el nostre web estàtic és accessible per altres usuàries, les visites correctes i els errors. [Figure 6]

A nivell general, podem revisar l'estat del nostre servei a _Google Cloud Storage_ a la secció [Cloud Storage -> Supervisión](https://console.cloud.google.com/storage/monitoring?project=static001&supportedpurview=project) [Figure 3]

Aquí trobarem les estadístiques d'accés al nostre web, latències, ample de banda utilitzat, ... I totes aquestes dades poden ser filtrades per la ubicació, que en el nostre cas, només en una, però que si féssim repliques del nostre _bucket_, podríem ampliar-ho a altres ubicacions.


## 1.5 Finalitzar el projecte

Comprovat que funciona, podem esborrar el _bucket_ amb tots els seus fitxers. El projecte el podem esborrar o mantenir per al següent exercici.

~~~
$ gcloud storage rm -r gs://uoc.domain.cc
$ gcloud projects delete static001
~~~
# 2. Com crear un website CMS Wordpress en un CDN

## 2.1 Identifica quin servei o serveis ofereix el CDN per tal fi

En aquest cas, tenim moltes opcions diferents.

Com a primera opció, simple i senzilla, __Google Cloud__ ens ofereix un paquet específic al seu _market_ per publicar directament un [Wordpress](https://console.cloud.google.com/marketplace/product/click-to-deploy-images/wordpress?tutorial=marketplace_wordpress&hl=es&project=static001).

Altres opcions, són el que cataloga com a _"web hosting"_ i per exemple, al servei __Compute Engine__ hi ha una opció per aixecar una infraestructura [LAMP](https://console.cloud.google.com/marketplace/details/click-to-deploy-images/lamp?hl=es&project=static001).

Una opció interessant és el que anomenen _serverless_ dins de __App Engine__. Aquesta opció permet aixecar una aplicació directament amb el seu codi sense necessitat de tenir un servidor. El codi es limitat a alguns llenguatges, però per a PHP està disponible.

També, hi ha l'opció d'aixecar el servei d'allotjar la base de dades MySQL fora del servei de web hosting i utilitzar [Cloud SQL](https://console.cloud.google.com/sql/instances?hl=es&tutorial=marketplace_wordpress&project=static001)

O, si el que volem es una imatge molt específica del nostre _Wordpress_ on tenim controlat el seu codi i contingut, podríem dissenyar un _Dockerfile_[^dockerfile] amb el nostre codi, construir la imatge amb [Cloud Build](https://cloud.google.com/build?hl=es) i llançar-lo a [Container-Optimized OS](https://cloud.google.com/container-optimized-os/docs?hl=es-419). Aquesta opció és també possible utilitzant una instancia de [VM](https://cloud.google.com/compute/docs/instances?hl=es-419) (Virtual Machine) i a la configuració escollir la imatge que volem del nostre registrador de Docker [^docker] preferit (Compute Engine -> Instancia VM -> Nova instancia -> Container)

[^dockerfile]: Un Dockerfile és un document de text que conté totes les ordres que un usuari podria cridar a la línia d'ordres per muntar una imatge. - <https://docs.docker.com/engine/reference/builder/>
[^docker] : Docker és un projecte de codi obert que automatitza el desplegament d'aplicacions dins de contenidors de programari, proporcionant una capa addicional d'abstracció i automatització de virtualització d'aplicacions en múltiples sistemes operatius.1 Docker utilitza característiques d'aïllament de recursos del nucli Linux, tals com a cgroups i espais de noms (namespaces) per permetre que "contenidors" independents s'executin dins d'una sola instància de Linux, evitant la sobrecàrrega d'iniciar i mantenir màquines virtuals. - <https://www.docker.com/>

Per aquest exercici utilitzarem __App Engine__ i __Cloud SQL__, per poder testejar el que anomena Google Cloud sistema __serverless__ encara que l'opció més senzilla seria utilitzar el paquet de __Wordpress__ al market. En aquest exercici utilitzarem __PHP Composer__[^phpcomposer] i els scripts de desplegament que ens ofereixen al [tutorial de Google Cloud](https://cloud.google.com/php/tutorials/wordpress-app-engine-flexible?hl=es-419).

[^phpcomposer] : Composer és una eina per a la gestió de dependències en PHP. Us permet declarar les biblioteques de les quals depèn el vostre projecte i les gestionarà (instal·larà/actualitzarà).

## 2.2 Descriu pas a pas el procés de creació del CMS en el CDN a partir dels tutorials didàctics que s'ofereixen al seu web

### 2.2.1 Prerequisits

Primer, instal·larem alguns requisits de _PHP_ y _Composer_:

~~~
$ sudo apt install composer php-curl php-zip
~~~

També instal.larem un client de MySQL:

~~~
$ sudo apt install mariadb-client
~~~

Per poder fer el deploy, _Google Cloud_ ens ofereix un software per fer de proxy amb el servidor _Cloud SQL_. Així que instal·larem també el paquet que ens ajudarà a fer de proxy:

~~~
$ curl -o cloud-sql-proxy \
  https://storage.googleapis.com/cloud-sql-connectors/cloud-sql-proxy/v2.0.0/
  \cloud-sql-proxy.linux.amd64
$ chmod +x cloud-sql-proxy
~~~
### 2.2.2 Creació de les credencials i accesos per les APIs

Ara, generarem les credencials per accedir al sistema des de la consola de Google Cloud al nostre projecte _static001_:

1. Accedim a API y servicios -> Credenciales -> Crear Credenciales -> Clave de cuenta y servicio
2. Nom del compte UOC Seleccionem App Engine y Crear

També, hem d'accedir a la activació de les APIs per a que el nostre usuari pugui gestionar els accessos, crear la base de dades a _Cloud SQL_, crear el _bucket_ amb els fitxers del projecte i fer el _deploy_ a App Engine. Per fer-ho, [accedim a l'enllaç](https://console.cloud.google.com/flows/enableapi?apiid=sqladmin%2Ccompute_component&hl=es-419) que ens ofereixen al tutorial o ho activem des de la secció de _API y servicios de la consola_.

### 2.2.3 Configuració de Cloud SQL

Ara, creem la instancia	de _Cloud SQL_ al nostre projecte:

~~~
$ gcloud sql instances create uoc-sql-instance \
    --activation-policy=ALWAYS \
    --tier=db-n1-standard-1 \
    --region=us-central1

API [sqladmin.googleapis.com] not enabled on project [259632542999]. Would you like 
to enable and retry (this will take a few minutes)? (y/N)?  y
...
~~~

Actualitzem la contrasenya de _root_ de la base de dades:

~~~
$ gcloud sql users set-password root --instance uoc-sql-instance \     
    --password BcCm6mcUUtwUA3FaOHs0 \    
    --host %
Updating Cloud SQL user...done.   
~~~

Executem el proxy:

~~~
$ ./cloud-sql-proxy \
	static001:us-central1:uoc-sql-instance -c static001-a3c5dc92b760.json
~~~

I obrin un nou terminal per realitzar la configuració de la base de dades:

~~~
$ mysql -h 127.0.0.1 -uroot -pBcCm6mcUUtwUA3FaOHs0
 create database tutorialdb;
 create user 'uoc-user'@'%' identified by 'mcUUtwU';
 grant all privileges on tutorialdb.* to 'uoc-user'@'%';
 flush privileges;
 exit
~~~

### 2.2.4 Configuració de Wordpress i desplegament

Descarreguem el codi de desplegament que ens ofereixen des de Google Cloud per fer treballar amb App Engine i Wordpress:

~~~
$ git clone https://github.com/GoogleCloudPlatform/php-docs-samples.git
$ cd php-docs-samples/appengine/flexible/wordpress
$ composer install
~~~

Per a realitzar la configuració bàsica del Wordpress (configurar el fitxer wp-config.php) i adaptar-lo per poder fer el desplegament a App Engine, executem la comanda de configuració programada amb Composer amb les dades de la base de dades.

~~~
$ php wordpress.php setup -n     --dir=./wordpress-project     --db_instance=uoc-sql-instance 
    --db_name=tutorialdb     --db_user=uoc-user     --project_id=static001     --db_password=mcUUtwU
...
23 package suggestions were added by new dependencies, use `composer suggest` to see details.
...
~~~

Finalment, despleguem el codi contra la instancia d'_App Engine_.

~~~
$ cd wordpress-project
$ gcloud app deploy \
    --promote --stop-previous-version app.yaml
You are creating an app for project [static001].
....
Creating App Engine application in project [static001] and region [europe-central2]....done.                                                                             
Services to deploy:
...
~~~

I quan finalitzi el desplegament, podem accedir al nostre Wordpress a https://static001.lm.r.appspot.com/ [Figure 4]

### 2.2.5 Gestió d'errors i debugging

Encara que la documentació al web de Google Cloud sembli molt senzilla, algunes parts no estan actualitzades i he hagut de canviar alguns processos per poder desplegar finalment el projecte. Primer de tot, la generació de claus ja no es realitza des de les credencials, si no des de el menu _IAM y administración_. Allà hem de donar els permisos oportuns al nostre usuari generat, permisos de edició a Cloud SQL, i descarregar les claus.

La versió que corre l'script de la imatge del desplegament de Wordpress a App Engine, té la versió de PHP 7.3 i la versió estable actualment es la 7.4. Al fer el desplegament, com llancem el Composer des d'un PHP 7.4 dona error. S'ha de forçar a utilitzar Composer amb la versió 7.3 per a que funcioni correctament.

~~~
Step #1: There is no PHP runtime version specified in composer.json, or
Step #1: we don't support the version you specified. Google App Engine
Step #1: uses the latest 7.3.x version.
~~~

La part interessant referent al debugging és que si hi ha algun error al desplegament, podem accedir a una consola virtual per fer una revisió de l'error. La URL es mostra al executar el desplegament en local. [Figure 5]

## 2.3 Com demostres que funciona i que es visitada?

Si accedim a la URL generada, podem veure el _Wordpress_ preparat per a finalitzar la instal·lació.
Ara, hauríem de prepar el servei de HTTPS balancejat per afegir el nostre domini i configurar els accessos. No és necessari per demostrar com es podria instal·lar un Wordpress a Google Cloud.

## 2.4 Quines eines ofereix per mesurar la latència? altres paràmetres?

Com a l'anterior exercici, podem accedir al panell d'administració de Google Cloud i veure les estadístiques d'accés tant del codi PHP desenvolupat a App Engine com de Cloud SQL. Aquí també podem gestionar on es troba el nostre bucket, canviar de servidors, afegir repliques, ... Segons el tràfic o el consum de dades que tinguem, podem afegir més recursos o marcar límits per evitar sobrecostos al nostre sistema. Realment, té un sistema molt intuïtiu i si has estat en contacte amb altres eines de Google com Google Analytics, es força senzill fer-se una idea de com adaptar la infraestructura. [Figure 7]

# 3. Pros i contres

## 3.1 Quins beneficis ofereix poder publicar els continguts via CDN respecte al cas d'un servidor propi? Per contra, quins inconvenients poden aparèixer tant en la seva gestió com en la seva explotació?

Realment, si el projecte a desenvolupar es un projecte a gran escala, que ha de tenir accessos des de diversos continents i volem gestionar correctament la escalabilitat i la baixa latència, els serveis al núvol ens ofereixen una gran estabilitat i facilitat per millorar l'experiència dels nostres usuaris. 

Fins ara, l'opció de tenir un servidor propi (sigui un baremetal en housing o un VPS) ha estat la solució per externalitzar el manteniment d'una infraestructura que per a la pròpia empresa generaria massa costos (electricitat, xarxa de gran ample de banda, tècnics de sistemes, ...). Així i tot, si el nostre projecte creix i la màquina es fa petita (sigui per espai, memòria o ample de banda) hem de realitzar contínuament migracions a noves màquines, i aquest procés és costos a nivell de temps i de dedicació personal.

Un altre gran benefici dels serveis al cloud és que pagues el que consumeixes, ni més ni menys. Si no hi ha usuaris connectats al servei, no hi ha consum de recursos. Si tens un VPS o una màquina física, el consum sempre es el mateix i normalment es sobre-equipen les màquines per tenir marge en cas de tenir grans consums, encara que durant gran part de la seva vida no els utilitzin.

També és cert que per certes empreses petites, sense massa dependència dels seus sistemes a la web i amb un públic orientat a certa part del món, poden continuar utilitzant servidors virtuals sense fer migracions al cloud. La estratègia de manteniment i actualització de les aplicacions desenvolupades per al cloud és totalment diferent a mantenir els serveis a un servidor propi. Els/les sysadmins han de fer un aprenentatge que molts cops no és senzill, ja que han de començar a pensar també com a programadors i no com purament administradors/es de sistema. El concepte __serverless__ t'obliga a pensar els recursos de maquinari com si fos part d'un codi, i les configuracions o orquestracions s'escriuen als frameworks com _Kubernetes_[^kubernetes] o _Terraform_[^terraform] com codi per a desenvolupar una aplicació. Moltes empreses petites, aquest procés no el faran fins que no facin un canvi de l'equip IT.

[^kubernetes]: Kubernetes, també conegut com a K8s, és un sistema de codi obert per automatitzar el desplegament, l'escala i la gestió d'aplicacions en contenidors. - <https://kubernetes.io/>
[^terraform]: Terraform és una infraestructura de codi obert com a eina de programari de codi que us permet crear, canviar i millorar la infraestructura de manera segura i previsible. - <https://www.terraform.io/>

Per altra banda, també ens trobem amb un problema vinculat a la privacitat i seguretat de les dades allotjades al núvol. Moltes empreses mouen dades molt sensibles que en cas de possibles fallades de seguretat al núvol o d'una mala gestió d'accessos per part de l'equip d'IT, pot generar problemes molt greus. El saber mantenir aquestes dades en privat i externalitzar al núvol el material menys sensible, obre la possibilitat a la hibridació entre les dues infraestructures. Per altra banda, mantenir grans quantitats de dades al núvol accessibles segueix sent costós i per tant, moltes empreses segueixen optant per tenir els repositori de dades més grans a la pròpia empresa i externalitzant només les webs o serveis dedicats al públic. La implantació de infraestructures internes que treballen sobre un núvol privat com OpenStack[^openstack] o Apache CloudStack[^cloudstack] ofereixen aquest tipus d'hibridacions amb força facilitat d'implantació. 

[^openstack]: OpenStack és una plataforma de cloud computing estàndard oberta i gratuïta. Es desplega principalment com a infraestructura com a servei (IaaS) tant en núvols públics com privats on els servidors virtuals i altres recursos estan disponibles per als usuaris. - <https://www.openstack.org/>
[^cloudstack]: CloudStack és un programari de computació en núvol de codi obert per crear, gestionar i desplegar serveis d'infraestructura en núvol. - <https://cloudstack.apache.org/>

Personalment, després de treballar alguns anys a diversos equips IT, crec que hi ha un procés intermedi entre els serveis allotjats al servidor físic i el núvol. Durant aquest procés, quan s'externalitza el servei al VPS o sistemes similars, també es treballa en la migració dels serveis per a ser adaptats als nous sistemes de containers (Docker, Kubernetes...) que s'utilitzen al Cloud i és aquí on si aquest procés es realitza correctament, la migració al núvol será molt senzilla.

Per tant, els beneficis són molts i necessaris, però hi ha d'haver un procès d'aprenentatge per part de l'equip de sistemes i una transformació de les infraestructures abans de passar a utilitzar el Cloud.

# 4. i ja que estem ... Com funciona l'emissió de video per HLS a Google Cloud?

Farem un exercici per provar la implementació d'un _live streaming_ amb protocol HLS via el servei de __Google Cloud Live Streaming__. Utilitzarem el [tutorial d'introducció per emissions HLS de Google Cloud](https://cloud.google.com/livestream/docs/quickstarts/quickstart-hls?hl=es-419)

## 4.1 Activació de les APIs

Aprofitarem el projecte generat anteriorment, habilitem l'API de Live Streaming per poder treballar, donem permisos per a que es pugui utilitzar el nostre compte de Google i afegim els rols oportuns al nostre usuari.

~~~
$ gcloud services enable livestream.googleapis.com
$ gcloud auth application-default login
$ gcloud projects add-iam-policy-binding static001 \
	--member="user:mail@gmail.com" --role=roles/livestream.editor
$ gcloud projects add-iam-policy-binding static001 \
	--member="user:mail@gmail.com" --role=roles/storage.admin
~~~

## 4.2 Configuraició del punt de muntatge i el canal d'emissió

Creem un _bucket_ per gestionar el livestreaming:

~~~
$ gsutil mb gs://static001streaming
~~~

Al nostre client d'emissió, utilitzarem _ffmpeg_ per fer la emissió i conversió:

~~~
$ sudo apt install ffmpeg
~~~

Per poder fer la emissió, necessiten un punt de muntatge d'entrada on el canal (pipe) d'emissió emetrà el contingut. Si estem familiaritzats amb el llenguatge de Gstreamer es un SINK d'una pipe. Crearem un fitxer JSON per la petició i utilitzarem CURL per fer la petició:

~~~
cat > request.json << 'EOF'
{
  "type": "RTMP_PUSH"
}
EOF
curl -X POST \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/json; charset=utf-8" \
    -d @request.json \
    "https://livestream.googleapis.com/v1/projects/259632542999/locations/
    europe-west1/inputs?inputId=uoc-pac1-static001"
~~~

Com a resultat tindrem la informació del nostre punt de muntatge on copiarem el _operationID_ per la següent secció [operation-1680275595137-5f833a5f91017-14a21659-0b4784fb]:

~~~
vagrant@bullseye:~$ curl -X POST \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/json; charset=utf-8" \
    -d @request.json \
    "https://livestream.googleapis.com/v1/projects/259632542999/locations/
    europe-west1/inputs?inputId=uoc-pac1-static001"
{
  "name": "projects/259632542999/locations/europe-west1/operations/
  operation-1680275595137-5f833a5f91017-14a21659-0b4784fb",
  "metadata": {
    "@type": "type.googleapis.com/google.cloud.video.livestream.v1.OperationMetadata",
    "createTime": "2023-03-31T15:13:16.510151406Z",
    "target": "projects/259632542999/locations/europe-west1/inputs/uoc-pac1-static001",
    "verb": "create",
    "requestedCancellation": false,
    "apiVersion": "v1"
  },
  "done": false
}
~~~

Ara verificarem que el punt de muntatge està creat correctament i actiu. Es possible que trigui algun temps en activar-se.

~~~
$ curl -X GET \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    "https://livestream.googleapis.com/v1/projects/259632542999/
    locations/europe-west1/operations/
    operation-1680275595137-5f833a5f91017-14a21659-0b4784fb"
~~~

Rebrem una resposta de la API on podrem adquirir URI d'emissió [rtmp://35.205.210.48/live/79f6a679-7ac6-4b16-ae87-02a938c9a418]:

~~~
vagrant@bullseye:~$ curl -X GET \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    "https://livestream.googleapis.com/v1/projects/259632542999/
    locations/europe-west1/operations/operation-1680275595137-5f833a5f91017-14a21659-0b4784fb"
{
  "name": "projects/259632542999/locations/europe-west1/operations/
  operation-1680275595137-5f833a5f91017-14a21659-0b4784fb",
  "metadata": {
    "@type": "type.googleapis.com/google.cloud.video.livestream.v1.OperationMetadata",
    "createTime": "2023-03-31T15:13:16.510151406Z",
    "endTime": "2023-03-31T15:16:16.449268930Z",
    "target": "projects/259632542999/locations/europe-west1/inputs/uoc-pac1-static001",
    "verb": "create",
    "requestedCancellation": false,
    "apiVersion": "v1"
  },
  "done": true,
  "response": {
    "@type": "type.googleapis.com/google.cloud.video.livestream.v1.Input",
    "name": "projects/static001/locations/europe-west1/inputs/uoc-pac1-static001",
    "createTime": "2023-03-31T15:13:16.507240387Z",
    "updateTime": "2023-03-31T15:14:59.109978412Z",
    "type": "RTMP_PUSH",
    "uri": "rtmp://35.205.210.48/live/79f6a679-7ac6-4b16-ae87-02a938c9a418",
    "tier": "HD"
  }
}
~~~

Ara, crearem el canal [uoc-pac1-static001] on indiquem la qualitat d'emissió i algunes característiques sobre l'streaming de vídeo:

~~~
$ cat > request.json << 'EOF'
{
  "inputAttachments": [
    {
      "key": "my-input",
      "input": "projects/259632542999/locations/europe-west1/inputs/uoc-pac1-static001"
    }
  ],
  "output": {
    "uri": "gs://static001streaming"
  },
  "elementaryStreams": [
    {
      "key": "es_video",
      "videoStream": {
        "h264": {
          "profile": "high",
          "widthPixels": 1280,
          "heightPixels": 720,
          "bitrateBps": 3000000,
          "frameRate": 30
        }
      }
    },
    {
      "key": "es_audio",
      "audioStream": {
        "codec": "aac",
        "channelCount": 2,
        "bitrateBps": 160000
      }
    }
  ],
  "muxStreams": [
    {
      "key": "mux_video_ts",
      "container": "ts",
      "elementaryStreams": ["es_video", "es_audio"],
      "segmentSettings": { "segmentDuration": "2s" }
    }
  ],
  "manifests": [
    {
      "fileName": "main.m3u8",
      "type": "HLS",
      "muxStreams": [
        "mux_video_ts"
      ],
      "maxSegmentCount": 5
    }
  ]
}
EOF

$ curl -X POST \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/json; charset=utf-8" \
    -d @request.json \
    "https://livestream.googleapis.com/v1/projects/259632542999/locations/
    europe-west1/channels?channelId=uoc-pac1-static001"

{
  "name": "projects/259632542999/locations/europe-west1/operations/
  operation-1680276364797-5f833d3d9255f-882dd6a4-e9cdc8ea",
  "metadata": {
    "@type": "type.googleapis.com/google.cloud.video.livestream.v1.OperationMetadata",
    "createTime": "2023-03-31T15:26:04.833011517Z",
    "target": "projects/259632542999/locations/
    europe-west1/channels/uoc-pac1-static001",
    "verb": "create",
    "requestedCancellation": false,
    "apiVersion": "v1"
  },
  "done": false
}
~~~

Podem fer un test per veure l'estat del canal i veurem que l'estat de la emissió es _STOPPEP_

~~~
$ curl -X GET     -H "Authorization: Bearer $(gcloud auth print-access-token)"\
 "https://livestream.googleapis.com/v1/projects/259632542999/locations/
 europe-west1/channels/uoc-pac1-static001" \
 | grep StreamingState

"streamingState": "STOPPED",
~~~

Activem ara l'emissió i revisem l'estat de l'streaming:

~~~
$ curl -X POST \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/json; charset=utf-8" \
    -d "" \
    "https://livestream.googleapis.com/v1/projects/259632542999/locations/
    europe-west1/channels/uoc-pac1-static001:start"

$ curl -X GET -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  "https://livestream.googleapis.com/v1/projects/259632542999/locations/
  europe-west1/channels/uoc-pac1-static001"\
   | grep streamingState

"streamingState": "STARTING",
~~~

Encara no podem enviar fins que l'estat sigui AWAITING_INPUT, esperem uns 10 minuts i tornem a provar:

~~~
$ curl -X GET     -H "Authorization: Bearer $(gcloud auth print-access-token)" \
"https://livestream.googleapis.com/v1/projects/259632542999/locations/
	europe-west1/channels/uoc-pac1-static001" \
 | grep streamingState
  "streamingState": "AWAITING_INPUT",
~~~

## 4.3 Emissió HLS a Stream API de Google Cloud

Ara podem fer l'emissió donant accés públic per a poder fer la visualització:

~~~

$ gsutil cors set a.json "gs://static001streaming"
$ gsutil defacl ch -u AllUsers:R gs://static001streaming
$ ffmpeg -re -f lavfi -i "testsrc=size=1280x720 [out0]; sine=frequency=500 [out1]"\  
    -acodec aac -vcodec h264 -f flv \
    rtmp://35.205.210.48/live/79f6a679-7ac6-4b16-ae87-02a938c9a418
~~~

I si usem un player que proposen des de la documentació de Google Cloud, [Shaka](https://shaka-player-demo.appspot.com/demo/), podem visualitzar el resultat de la nostra emissió amb el nostre arxiu d'emissió [https://storage.googleapis.com/static001streaming/main.m3u8] [Figure 8]

Per revisar l'activitat, accedim al panell de Google Cloud i anem a la secció _APIS y servicios habilitados_. Aquí, podem accedir a les mètriques d'ús de l'Stream API i valorar els consums i accessos.  [Figure 9]

## 4.4 Finalitzem l'emissió i tanquem el canal i el punt de muntatge

Per finalitzar, enviem la petició de finalització de la emissió i desactivem tot el que hem activitat durant l'activitat (canal, bucket, ..) i tanquem:

~~~
# transmissio
$ curl -X POST \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/json; charset=utf-8" \
    -d "" \
    "https://livestream.googleapis.com/v1/projects/259632542999/locations/
    europe-west1/channels/uoc-pac1-static001:start"
# Canal
$ curl -X DELETE \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    "https://livestream.googleapis.com/v1/projects/259632542999/\
    locations/europe-west1/channels/uoc-pac1-static001"
# Pipe d'entrada
$ curl -X DELETE     -H "Authorization: Bearer $(gcloud auth print-access-token)"\ 
	"https://livestream.googleapis.com/v1/projects/259632542999/locations/
	europe-west1/inputs/uoc-pac1-static001"
# Bucket
$ gcloud storage rm gs://static001streaming
# Les credencials
$ gcloud auth application-default revoke
~~~

## 4.5 Experiència i costos

Com a experiència, la capacitat de poder utilitzar la infraestructura del núvol de Google per poder fer emissions és molt interessant. La capacitat de poder replicar el _bucket_ d'emissió, crear diverses repliques i així distribuir l'accés a la URL que ens ofereix d'entrada, és molt intuïtiu i senzill d'utilitzar. Les gràfiques en aquest cas han estat molt dèbils, per evitar generar més costos a l'experiència, però es pot analitzar prou bé la situació de les connexions, les necessitats d'ample de banda, els límits, ... A més, la possibilitat de treballar amb el protocol HLS complementen les opcions: generar talls de publicitat, esdeveniments durant la emissió, enviar diversos canals d'àudio a diverses resolucions. 

En aquest exercici hem utilitzat _ffpmeg_ per fer una retransmissió de prova, però utilitzant Gstreamer o Liquidsoap[^liquidsoap], podem gestionar una emissió a temps real des de qualsevol dispositiu d'entrada de vídeo i construir una programació en directe i distribuïda d'una manera molt senzilla.

[^liquidsoap]: Liquidsoap és un llenguatge potent i flexible per descriure fluxos d'àudio i vídeo. Ofereix una rica col·lecció d'operadors que podeu combinar a voluntat, donant-vos més potència del que necessiteu per crear o transformar fluxos - <https://www.liquidsoap.info/>

És el  primer cop que utilitzo una API de Live Streamer contra un cloud, tinc experiències per fer emissions contra servidors propis, i tenint en compte les opcions que ofereixen, el cost d'implementació i automatització de pipes que activin o desactivin els canals i els inputs de les senyals, sembla força senzill. Segurament que en un futur projecte miri d'implementar alguna solució d'streaming contra el núvol.

Finalment, valorar que l'ús d'aquests dos dies de les APIs i serveis de Google Cloud per fer les proves, ha generat __un cost de 1,63€__, on gran part del cost ha estat el bucket de App Engine utilitzat per el Wordpress i les proves d'Streaming que hem realitzat durant aquest últim exercici. [Figure 10]


# Anex I : Screenshots

![Pàgina no accessible](./img/Captura desde 2023-03-30 12-05-17.png){ width=50% }

![Pàgina estática d'Hugo a Google Cloud Storage](./img/Screenshot 2023-03-30 at 12-08-18 My New Hugo Site.png){ width=50% }

![Panell d'analítiques i serveis de Google Cloud](./img/Screenshot 2023-03-30 at 12-37-31 Monitoring – Cloud Storage – static001 – Consola de Google Cloud.png){ width=50% }

![Pàgina de configuració de la nova instal·lació de Wordpress](./img/Screenshot 2023-03-30 at 16-15-41 WordPress › Installation.png){ width=50% }

![Pàgina de debugging d'App Engine](./img/Screenshot 2023-03-30 at 15-39-41 Detalles de compilación – Cloud Build – static001 – Consola de Google Cloud.png)

![Pàgina de gestió de localitzacions del bucket i altres opcions](./img/Screenshot 2023-03-30 at 12-32-24 uoc.piperrak.cc – Detalles del bucket – Cloud Storage – static001 – Consola de Google Cloud.png){ width=50% }

![Pàgina d'analítiques del nostre projecte App Engine](./img/Screenshot 2023-03-30 at 16-25-33 Panel – App Engine – static001 – Consola de Google Cloud.png){ width=50% }

![Emissió del nostre canal amb HLS a Shaka](./img/Screenshot 2023-03-31 at 11-08-07 Shaka Player Demo.png){ width=50% }

![Informe de consum del nostre projecte](./img/Screenshot 2023-03-31 at 11-37-10 Mi cuenta …cturación – Informes – Facturación – static001 – Consola de Google Cloud.png){ width=50% }

![Informe de consum segons aplicació](./img/Screenshot 2023-03-31 at 11-40-14 Mi cuenta …cturación – Descripción general – Facturación – static001 – Consola de Google Cloud.png){ width=50% }
