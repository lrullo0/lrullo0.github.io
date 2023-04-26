---
title: Practica 2
author: 
 - Luca Rullo
colorlinks: true
fontsize: 12pt
toc: true
toc-depth: 1
geometry:
- margin=1in

---

\newpage
# Exercici 1: Familiarització amb l'aplicació Avidemux. Contingut estàtic

[Descarrega Video](./video/asset-01.mp4)

<center>
<video src="./video/asset-01.mp4" controls height="480"></video>
</center>

## 1.1 Identifica el tipus de codificació de vídeo del clip original mitjançant ARXIU-INFORMACIÓ (no prendrem mai en compte l'àudio a cap apartat de tota la PAC). T'ho donarà un identificador abreujat de 3 o 4 caràcters, fes una cerca en Internet per saber quin codificador és. Investiga si és una codificació amb pèrdues o sense pèrdues.

Códificació del clip: 

~~~
=====================================================
Vídeo
=====================================================
Códec 4CC:			H264
Tamaño de la imagen:		720 x 1280
Relación de aspecto:		1:1 (1:1)
FPS:				29,611 fps
Average Bitrate:		1995 kbps
Duración total:			00:00:13.474
Pixel format:			YUV 4:2:0, 8-bit
Color range:			Limited (MPEG)
Color primaries:		BT.601 PAL
Transfer characteristics:	BT.601
Color space:			BT.470 System B/G

=====================================================
Video Codec Extradata
=====================================================
Size:				33
Extradata:			01 42 00 29 FF E1 00 12 67 42 00 29 8D 8D 40 5A 05 0D 35 05 06 05 07 84 42 37 01 00 04 68 CA 43 C8 

~~~

Com podem observar, el nostre arxiu té una codificació H264:

> H.264, o MPEG-4 part 10, és un còdec digital d'alta compressió estàndard desenvolupat durant quatre anys per l'ITU-T Video Coding Experts Group (VCEG) juntament amb l'ISO/IEC Moving Picture Experts Group (MPEG). La intenció del projecte H.264/AVC va ser la de crear un estàndard que fos capaç de desenvolupar una bona qualitat d'imatge amb bit rates substancialment menors que els estàndards anteriors (MPEG-2, H.263 o MPEG-4 part 2). A més a més de no incrementar la complexitat en el seu disseny.
>
> H.264/MPEG-4 AVC - Wikipedia[^h264]

[^h264]: H.264/MPEG-4 AVC - Wikipedia - https://ca.wikipedia.org/wiki/H.264/MPEG-4_AVC

H264 és una codificació amb pèrdues.

## 1.2 Mira les característiques del vídeo: quina definició en píxels té? Quants quadres/segon té? Quants segons dura? Amb elles, calcula quin seria el pes en bytes d'un fitxer de vídeo que tingués el mateix contingut i durada, però sense comprimir. Recorda: un píxel de color són 3 bytes (24 bits) en un vídeo no comprimit.

Amb un format de 720x1280px i 29,611fps i una duració de 13 segons en H264 té una mida de 3.7MB

La estimació de mida sense compresió (només video) seria:

3 bytes/pixel * (720 x 1280 px / frame ) * 29.611 fps * 13 sec = 1064290406.4000001 bytes = 1014.98 MB

## 1.3 Prenent com a referència el fitxer de vídeo sense comprimir, quin és, llavors, el factor de compressió que ja posseeix asset-01?

Per tant, el seu factor de compressió 274:1

## 1.4 Identifica el tipus de contenidor. Confirma si quadra o no amb la taula de contenidors de l'apartat 3.3 del Mòdul 1. Si no existís en el quadre, fes una cerca per Internet (Google, Wikipedia, etc.)

El contenidor utilitzat és un MP4, un contenidor utilitzat per MPEG i compatible amb el códec.

## 1.5 De quin tipus és el primer fotograma? I el segon?

El primer fotograma es I-FRM, és a dir, un keyframe que está comprimit espacialment, i el segon P-FRM, una imatge generada per extrapolació de les anteriors mitjançant una estimació de moviment.

## 1.6 Quants quadres d'aquest tipus hi ha al clip sencer? Quina relació hi ha entre el contingut de la imatge i l'existència de quadres tipus I?

Per cada segon tenim 1 frame I-FRM i 28 P-FRM, per tant, si el nostre clic es de 13 segons, tenim 13 frames I-FRM I 364 frame P-FRM. 

A més quantitat de quadres tipus I, més mida tindrá el nostre fitxer ja que tenen una compressió més debil. És a dir, si la imatge és una imatge fitxa, podriem tenim pocs quadres I ja que no hi ha massa canvi a la imatge, però en imatges en moviment, necesitem més frames I per poder afavorir una bona sensació de qualitat de la imatge. Per tant, si el contingut de la nostra imatge es en moviment, hauriem de tenir una configuració de frames I més gran que en estátic.

## Tasca 1.7: En resum, quin tipus d'estructures GOP posseeix aquest clip? I per què?

En el nostre cas tenim:

1 frame I - 28 frames P per tant, el nostre 

El valor N, la distancia entre frames I es de 30
I el valor M, la distancia entre un frame I i el següent I o P es de 1

Per tant, tenim un GOP de 30 amb N=30 M=1.

## Tasca 1.8: Cal tenir en compte alguna cosa sobre GOPs quan s'edita un vídeo? Indaga als fòrums d'Avidemux per trobar informació sobre què fa aquesta aplicació (i qualsevol altra aplicació d'edició) quan talla un clip per assemblar-ho a un altre.

Es important entendre, segons les lectures dels forums d'Avidemux [^avidemux], que l'unitat per tallar clips hauria de coincidir amb el I-FRM per treballar en un mode "net" i evitar recompresions o perdue de qualitat en l'exportació del material. Es posible taller al P-FRAME però es considera més adient treballar el tall en els I-FRM.

[^avidemux]: Can't crop/trim from exact point - Avidemux - https://avidemux.org/smif/index.php?topic=17782.0

## 1.9: Quina informació coincideix amb la qual dóna Avidemux? Quina informació addicional dóna? Recorda obviar tot el referent a l’àudio.

~~~
Video
ID                                       : 1
Format                                   : AVC
Format/Info                              : Advanced Video Codec
Format profile                           : Baseline@L4.1
Format settings                          : 1 Ref Frames
Format settings, CABAC                   : No
Format settings, Reference frames        : 1 frame
Format settings, GOP                     : M=1, N=30
Codec ID                                 : avc1
Codec ID/Info                            : Advanced Video Coding
Duration                                 : 13 s 483 ms
Source duration                          : 13 s 475 ms
Bit rate                                 : 1 996 kb/s
Width                                    : 720 pixels
Height                                   : 1 280 pixels
Display aspect ratio                     : 0.562
Frame rate mode                          : Variable
Frame rate                               : 29.611 FPS
Minimum frame rate                       : 29.479 FPS
Maximum frame rate                       : 29.703 FPS
Color space                              : YUV
Chroma subsampling                       : 4:2:0
Bit depth                                : 8 bits
Scan type                                : Progressive
Bits/(PixelxFrame)                       : 0.073
Stream size                              : 3.21 MiB (91%)
Source stream size                       : 3.21 MiB (91%)
Title                                    : VideoHandle
Language                                 : English
Encoded date                             : UTC 2023-04-25 18:15:50
Tagged date                              : UTC 2023-04-25 18:15:50
Color range                              : Limited
Color primaries                          : BT.709
colour_primaries_Original                : BT.601 PAL
Transfer characteristics                 : BT.709
transfer_characteristics_Original        : BT.601
Matrix coefficients                      : BT.709
matrix_coefficients_Original             : BT.470 System B/G
mdhd_Duration                            : 13483
Codec configuration box                  : avcC

~~~

Tenim molta més informació que en Avidemux i molt més diversa (tipus de formats, diversos tipus de framerate, més dades de color, ...)

Per exemple, si revisemm el códec, Mediainfo el defineix com a AVC1, on Avidemux utilitza l'estandard 4CC[^4cc] per definir-lo com a H264.

Altres coses, com les que hem revisat anteriorment sobre el GOP, estas detallades en la informació de Mediainfo sobre le nostre clip.

[^4cc]: 4CC - Wikipedia - https://en.wikipedia.org/wiki/FourCC

\newpage
# 2. Contingut dinàmic

<center>
<video src="./video/asset-02.mp4" controls height="480" style="transform:rotate(-90deg)"></video>
</center>

## 2.1 Mira les característiques d'aquest nou clip i torna a respondre: quina definició en píxels té? Quants quadres/segon posseeix? Quants segons dura? Amb elles, calcula quin seria el pes en bytes d'un fitxer de vídeo que tingués el mateix contingut i durada, però sense comprimir . Recorda: un píxel de color son 3 bytes (24 bits) en un vídeo no comprimit.
## 2.2: Contrasta aquesta informació amb la donada per MediaInfo
## 2.3: Tenint com a referència el fitxer de vídeo fictici sense compressió, quin és, llavors, el factor de compressió que ja posseeix asset-02?
## 2.4: Repeteix les tasques 1.5, 1.6 i 1.7 de l'exercici 1 però ara aplicades a aquest segon clip de vídeo.

Informació d'Avidemux:

~~~

=====================================================
Vídeo
=====================================================
Códec 4CC:			H264
Tamaño de la imagen:		1280 x 720
Relación de aspecto:		1:1 (1:1)
FPS:				29,445 fps
Average Bitrate:		8907 kbps
Duración total:			00:00:24.214
Pixel format:			YUV 4:2:0, 8-bit
Color range:			Limited (MPEG)
Color primaries:		BT.709
Transfer characteristics:	BT.709
Color space:			BT.709

=====================================================
Video Codec Extradata
=====================================================
Size:				33
Extradata:			01 42 00 29 FF E1 00 12 67 42 00 29 8D 8D 40 28 02 DD 35 01 01 01 07 84 42 37 01 00 04 68 CA 43 C8 

~~~

El fitxer té una resolució 1280x720 i un FPS de 29,445 codificat utilizant H264 i ocupa 27MB.

3 bytes * (1280 * 720) * 29.445 fps * 24 = 1953828864.0 bytes = 1863 MB

El seu factor de compressió es de 69:1, si ho comparem amb l'altre, aquest arxiu de video té menys compressió. 

MediaInfo ens ofereix molta més informació, com hem comprovat anteriorment i la información, es molt similar al video anterior. El GOP és el mateix que al material anterior, tenim un primer I-FRM seguit de 29 P-FRM, amb un GOP (M=1 N=30). 

~~~

Video
ID                                       : 1
Format                                   : AVC
Format/Info                              : Advanced Video Codec
Format profile                           : Baseline@L4.1
Format settings                          : 1 Ref Frames
Format settings, CABAC                   : No
Format settings, Reference frames        : 1 frame
Format settings, GOP                     : M=1, N=30
Codec ID                                 : avc1
Codec ID/Info                            : Advanced Video Coding
Duration                                 : 24 s 214 ms
Source duration                          : 24 s 214 ms
Bit rate                                 : 8 907 kb/s
Width                                    : 1 280 pixels
Height                                   : 720 pixels
Display aspect ratio                     : 16:9
Rotation                                 : 90°
Frame rate mode                          : Variable
Frame rate                               : 29.446 FPS
Minimum frame rate                       : 14.803 FPS
Maximum frame rate                       : 42.115 FPS
Color space                              : YUV
Chroma subsampling                       : 4:2:0
Bit depth                                : 8 bits
Scan type                                : Progressive
Bits/(Pixel * Frame)                       : 0.328
Stream size                              : 25.7 MiB (97%)
Source stream size                       : 25.7 MiB (97%)
Title                                    : VideoHandle
Language                                 : English
Encoded date                             : UTC 2023-04-20 15:42:42
Tagged date                              : UTC 2023-04-20 15:42:42
Color range                              : Limited
Color primaries                          : BT.709
Transfer characteristics                 : BT.709
Matrix coefficients                      : BT.709
mdhd_Duration                            : 24214
Codec configuration box                  : avcC

~~~

## 2.5: Comenta les diferències trobades entre asset- 01 i asset-02, i la seva explicació si en tenen.

Destaquem en aquest de video la pujada de Bitrate i la variació del framerate, segurament causats per el moviment de càmera i els efectes que causa un canvi tan constant frame a frame. Els fotogrames identificats com a P-FRM han de generar moltes més dades per poder definir els seus canvis de l'anterior I-FRM, pero tant, finalment tenen una mida més gran, el que generar una Bitrate més gran, a més de tenir un factor de compressió menor que en el material anterior.

\newpage
# 3. Codificació en baixa qualitat

## 3.1 Quin format de codificació de vídeo accepten els DVD? Si el client hagués demanat Blu-Ray en comptes de DVD, quin hauria estat el format de codificació més adient?

Els DVDs s'han de codificar per a generar arxius format VOB codificats normalment en H262/MPEG-2 Part 2[^dvdwikipedia]. Per tant, hauriem de codificar els nostre material en aquest format. Aquesta codificació té unes limitacions entorn al aspect-ratio i els pixels màxims amb els que podem treballar. En el nostre cas, per intentar adaptar-nos al FPS del nostre material i els seu aspect ratio més proper al 16:9, hauriem d'adaptar la mida a 720x480px.

[^dvdwikipedia]: DVD-Video - Wikipedia - https://en.wikipedia.org/wiki/DVD-Video

Per a Blu-ray[^bluray] els arxius haurien de ser M2TS, la extensió per definir el container utilitzat per BlueRay (BDAV MPEG-2 transport stream). Com a codificació hauriem d'utilizar H.262/MPEG-2 Part 2 o H.264/MPEG-4 Part 10: AVC, per tant, com el nostre material ja está en H264/MPEG-4 Part 10 no caldria recodificar-lo si no fos perque el nostres video té un FPS 29.9 irregular i s'hauria d'adaptar als 23.975p, 24p o 50p que es el que marca l'estandard per a Blu-ray. 

[^bluray]: Blu-ray - Wikipedia - https://en.wikipedia.org/wiki/Blu-ray

## 3.2. quin dels dos filtres creus que seria més adequat si el teu clip fos de 30 fps?

Un filtre manté tots els fotogrames però reajusta el segon del nostre video a els 25fps que requereix la codificació. Això generá una relentització del material de video i una major longitud. Si el nostre material no té per exemple audio, no hauria de ser massa problema, encara que si tenim audio al nostre material el que generà serà un pitchdown que pot arrivar a ser molest. L'altre filtre el que generará serà un retallada de frames per adaptarlo al 25fps, sense fer un canvi en la durada del video, però perdent segurament certa fluidesa dels movimients. En el meu cas, utilitzaré el segon filtre per mantenir la mateixa durada del video original.

## 3.3.0. Vídeo codificat per DVD

[Download](./video/pac2-dvd.mpg)
<center>
<video src="./video/pac2-dvd.mpg" controls height="480" style="transform:rotate(-90deg)"></video>
</center>

## 3.3: Quin és el factor de compressió aconseguit amb el nou fitxer pac2_dvd respecte al clip fictici no comprimit de la mateixa durada i definició calculat a l'apartat 2.1? Compara’l al resultant a l'apartat 2.3. 

El fitxer original asset-2.raw (original fictici) té una mida de 1863 MiB

El fitxer pac2_dvd.mpg té una mida de 18.8 MiB

El factor de compressió de 99:1.

La raó per una banda es que hem canviat la mida del frame a la meitat, hem reduit el FPS i está comprimit. Encara que si el cambien de resolució i matenim la mateixa mida de frame la compressió seria inferior que amb H264AVC com tenim en l'apatat anterior.

## 3.4: Quins valors hem de posar en els camps Nombre de fotogrames B i Mida del GOP per obtenir aquest format de GOP (M=3 i N=6)?

En la configuració actual teniem 2 i 18 i el resultat era M=3,N=18, per tant, els valors seràn M=3,N=6 hauriem d'utilitzar serán 2 i 6.

## 3.5: Quin factor de compressió obtenim? Per què?

La mida del nou fitxer es de 19MiB.

El factor de compressió seria de 98:1, inferior a l'anterior perquè en aquest nou fitxer hem codificat amb més I-FRM que en laterior, i per tant, matenim més frame complets.


\newpage
# Exercici 4: codificació Blu Ray i codificació 4K

## 4.1: Què significa HEVC? Quines novetats incorpora H.265/HEVC respecte a l'anterior H.264/MPEG-4 AVC? Consulta Internet per trobar aquestes informacions.

HEVC significa High Efficiency Video Coding.

H265/HEVC[^h265] torna a alguns valors de l'antic MPEG-2 (nomes frames I,B,P) i amplia les funcionalitats de H264/MPEG-4 AVC com la predicció intraframe, la transformació, estimació de movimient, ... en blocs de pixels de 32x32 o inclús 64x64.

És una codificació que s'adapta a les necessitat del video d'ultra alta definició (resolucions fins a 8192x4320) o el video en 3D.

[^h265]: H.265 - Wikipedia - https://es.wikipedia.org/wiki/H.265

## 4.2: Quins són els nivells i els perfils d'H.265/HEVC? Compara'ls amb els de H.264/MPEG-4 AVC

Els nivells són: 1, 2, 2.1, 3, 3.1, 4, 4.1, 5, 5.1, 5.2, 6, 6.1, 6.2. Quan més gran, més qualitat. El nivell 6.2 resolucions fins a 8.192×4.320@120,0

En H264 els nivells van de 1 fins al 5.2 amb resolucins màximes de fins a 4096×2304@56,3 i 240Mbit/s

Els perfils són: Main, Main 10, Main 12, Main 4:2:2 10, Main 4:2:2 12, Main 4:4:4, Main 4:4:4 10, Main 4:4:4 12, Main 4:4:4 16 Intra. Els dos primers en la versió 1 i els altres a la versió 2. A major nivell, millor qualitat en la codificació.

En H264 tenim Original High, High 10, High 4:2:2, High 4:4:4 

## 4.3: On s'indica el nivell de codificació en aquesta finestra?

A la part inferior tenim un "scroll" per seleccionar el nivell de la codificació (major a menor qualitat, menor a major qualitat).

## 4.4: Quins perfils poden ser triats en Avidemux per a H.264 i en H.265?

A H.265 tenim els perfils: main, main10 i mainstillpicture.

A H.264 tenim : baseline, main, high, high10, high422, high444

## 4.5: Calcula el factor de compressió aconseguit respecte el fitxer asset-02.

Tenim el fitxer original mp4 en H264 amb un Bitrate 8907 kb/s (Comprobat utilitzant MediaInfo)

Realitzem una codificació al 80% de l'anterior tasa, és a dir, 7125 kbs

La mida del nou fitxer és 20.7 MiB front als 26.5 MiB de la versió en H264.

El factor de compressió és de 1.28:1 front al H264.

## 4.6: Calcula també el factor de compressió respecte a un hipotètic clip de la mateixa durada i definició

El fitxer fictici tindria una mida de 1863 MB

El factor de conversió seria de 90:1

## 4.7: Calcula una altra vegada el factor de compressió en tots dos casos. Hi ha hagut una variació important de compressió entre els dos clips generats? I de qualitat visual? Per què creus que ha estat així?

Realitzem una nova coficació al 50% del bitrate original, és a dir, 4453 kbs

La mida final del clip en H265 és 13.1 MiB

El seu factor de compressió respecte al original és de 2:1

El seu factor de compressió respecte al fictici sense comprimir és de 142:1 

Aquest nou clip s'ha reduit a la meitat i la perdua de qualitat, en el cas del meu clip, no és massa apreciable però si que hi ha una perdue de densitat de color i una mica més de pixelació en els foscos. He generat un clip reduit al 10% (890kbps) del bitrate original per a forçar aquest tipus de visualització.

## 4.8: Quin efecte visual produeix la pèrdua de llibertat d'elecció de la seva mesura? I a la compressió del fitxer? Per què?

Realitzem els canvis al GOP (30) mantenint l'última tasa de bits.

La mida del fitxer es quasi la mateixa, una mica menys, però no canvia.

La qualitat del video, hi ha com una sensació de movilitat més artificial, o com més forçada en el moviment, però realment, no es massa apreciable en el meu clip.

## 4.9: Comparant els tres casos, com afecta a la qualitat visual la variació del GOP? I a la compressió del fitxer? Per què?

El canvi en la mida del fitxer segueix sent molt baix, no hi ha massa variació, segeurament per el continu movimient de càmera la codificacio dinàmica del GOP deu ser molt similar a la forçada en aquests dos últims casos. La qualitat del vídeo no trobo gaires canvis.

Fent un nou clip a uns valors de GOP a 200, i comparant amb el de GOP=10 és veu una millora en la fluïdesa del moviment. Pero encara així, no és massa diferencial el canvi.

\newpage

# Exercici 5: Codificació de vídeo mitjançant un CDN

Com hem vist a l'anterior mòdul, analitzant els diversos proveidors de serveis al Cloud, i en més detall, el funcionament de Google Cloud API Strem, podem emetre un stream de video en HLS o DASH d'una manera força sencilla, aprofitant la distribució de la xarxa de Google Cloud.

En aquest apartat, per parlar dels sistemes de Media Stream, estudiaré el servei de Cloudflare anomenat Cloudflare Stream[^cstream].

[^cstream]: CloudFlare Stream - https://www.cloudflare.com/es-es/products/cloudflare-stream/

Cloudflare és una empresa d'estats units que vas ser molt popular als seus origens per oferir un servei de DNS distribuït amb un _proxy reverse_ molt potent i un sistema de mitigació de DDOS. També, a nivell d'usuari és gestora d'un dels servidors de DNSs públics més utilitzat a internet amb l'adreça _1.1.1.1_.

Si hem parlat a l'anterior mòdul dels diversos models de serveis al cloud, Cloudflare s'especialitza en el NaaS (Xarxa com a Servei), on podem identificar per exemple el seu producte de gestió de xarxes privades anomenat Cloudfare One, que incorpoa el servei propi de Zero Trust per securitzar la xarxa privada. A més, hi ha gran quantitat de serveis vinculats a firewalls en xarxes virtuals (Magic Firewall), mitigadors de DDOS (Spectrum), ....

Cloudflare Stream ens ofereix tots els serveis que es demanen per a un sistema Media Stream al cloud: Emmagatzemar arxius multimèdia, codificar i publicar arxius en viu o sota demanda, sense mantenir la infraestructura.

El procés de publicació i les accions es realitzen via una API molt similar a la que hem vist a l'anterior pràctica amb Google Cloud. Es penja el material via una comanda POST amb els tokens oportuns per autentificació, es valida la seva pujada i recodificació i ja es posible incorporar amb un iframe el player que ens ofereix Cloudflare [^firststeps].

[^firststeps]: Cloudflare Docs - Get started with Cloudflare Stream - https://developers.cloudflare.com/stream/get-started/

En el cas de voler fer una retransmisió[^firststream] en directe (utilizant RTMP), es crea el mountpoint via la API de Cloudflare i s'emet contra el punt de muntatge quan el sistema ja està disponible. De la mateixa manera, s'incrusta el player al nostre web. A més, ens permet també enregistrar l'event per tornar-lo a emetre en mode replay. En aquest cas, Cloudflare Stream emet amb un bitrate adaptatiu (ABR) per oferir al client la qualitat més adaptada al seu ample de banda. 

![Cloudflare Live Stream](https://developers.cloudflare.com/assets/live-stream-workflow_hucb4ae18dfe5885708a239a3b1286e1b5_19867_691x397_resize_q75_box_3-7500ef1b.png){ width=50% }

[^firststream]: Cloudflare Docs - Stream Live Video - https://developers.cloudflare.com/stream/stream-live/

A més de la posibilitat de publicar i emetre material audiovisual, CloudFlare Stream ens ofereix la opció d'edició en directe utilitzant la API per afegir per exemple titulació, marca d'aigua, modificar les funcions del player o retallar el material. Podem també habilitar webooks per executar events cap a serveis externs o gestionar usuaris o els diversos materials. Tot això via la API o el panell de gestió que ofereix CloudFlare [Stream Dashboard].

I finalment, tota la activitat està registrada i és possible analitzar els accesos via ubicacions, plataformes d'accés dels usuaris o analitzar en quin moment els nostres materials han estat més visualitzats o no. 

Com a novetat, que no he vist a altres proveïdors, és la possibilitat d'emetre i reproduir video utilizant el protocol WebRTC via browser a l'estil les plataformes de videoconferències estil Jitsi. 

Els preus de Cloudflare Stream:

* Paga $5 per 1000 minuts (16 hores) de vídeo enregistrat
* Paga $1 per 1000 minuts de vídeo entregat.

A partir d'aquí, Cloudflare arrodoneix a la xifra sempre més alta en múltiples de 1000 minuts.

Alguns pros i contres de la plataforma de Cloudflare:

* El servei decideix de manera automàtica els nivells de l'stream, adaptats de 320p a 1080p, amb la codificació de H264.
* No permet emetre més d'un canal d'àudio al mateix stream.
* Té una limitació de fins a 30G o 120 vídeos, encara que es possible demanar una ampliació.
* Accepta els següents formats de video: MP4, MKV, MOV, AVI, FLV, MPEG-2 TS, MPEG-2 PS, MXF, LXF, GXF, 3GP, WebM, MPG, QuickTime
* No permet video en HDR, tot els materials es recodifiquen en SDR per fer-los més aptes als dispositius.
* Accepta qualsevol FPS però es recodifiquen a 70fps per la reproducció.

Més informació a la secció de FAQs de la documentació de CloudFlare Stream [^faq]

[^faq]: FAQ - Cloudflare Stream - https://developers.cloudflare.com/stream/faq/



