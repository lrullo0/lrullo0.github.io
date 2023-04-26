---
title: Prova d'Avaluació Continuada 1 – PAC1 - M1.210 Sistemes distribuïts a gran escala
summary: Presentació de la pràctica PAC 1 de Sistemes distribuïts a gran escala
colorlinks: true
date: "2023-04-02"
geometry:
- margin=0.8in
categories:
- Sistemes distribuïts a gran escala
---


# Part 1

## 1.1 Descripció breu de les característiques essencials d'un cloud i els models de servei principals: IaaS, PaaS, SaaS. Amb exemples.

Sobre el concepte cloud, com hem pogut analitzar amb les lectures recomanades i altres materials derivats d'aquests dos primers blocs de treball, el definim com el model de serveis i computació sota demanda de recursos via d'internet. Encara que arquitectures anteriors com el Grid Computing, tinguin certes relacions, la gran diferencia és que aquestes anteriors infraestructures estaven vinculades a entorns més privats o d'investigació i connectades majorment via xarxes locals. La infraestructura Cloud base el seu model en Internet.

Una altra característica del concepte de cloud és la percepció de recursos infinits, és a dir, com a usuari/consumidor de serveis al cloud es treballa sota una abstracció dels recursos realmente existents a la infraestructura que manté el cloud, segons les nostres necessitats de computació o emmagatzemament el servei cloud ens oferirà els recursos necessaris per desenvolupar-ho.

A nivell de previsió de costos de negoci, una altra característica important és que construint la nostra pròpia infraestructura, tenim costos per la compra de la infraestructura, la contractació de personal IT, el lloguer de CPD o els costos de consum si el tenim en local; en canvi, el model cloud, els costos són el lloguer de recursos utilitzats al cloud durant cert període de temps, no hi ha costos de compra d'equips ni manteniment d'infraestructures. 

Això també repercuteix en els nivells de responsabilitats i riscos entorn a la infraestructura. En allotjaments locals, la responsabilitat de mantenir la infraestructura actualitzada, amb recursos suficients i disponible recauen completament sobre la nostre empresa. En el cas del cloud, nosaltres som només responsables de la configuració, la resta de responsabilitats i riscos els assumirà la empresa de cloud contractada.

Encara que en aquest exercici ens centrarem en els models de Public Cloud, això no vol condemnar que empreses privades puguin construir les seves pròpies infraestructures en xarxes locals cloud privades (enteses en alguns articles com a Private Cloud o Hybrid Cloud), utilitzant per exemple la plataforma de Canonical Openstack [^openstack] o CloudStack d'Apache[^cloudstack].

[^openstack]: OpenStack és un projecte de programari d'infraestructura de computació en núvol de codi obert i és un dels tres projectes de codi obert més actius del món. <https://www.openstack.org/>
[^cloudstack]: Apache CloudStack és un programari de computació en núvol de codi obert per crear, gestionar i desplegar serveis d'infraestructura en núvol. <http://cloudstack.apache.org/> 

Sota el model del public cloud, trobem una multitud de serveis que varien segons les necessitats al cloud per al consumidor. En aquest exercici definirem els tres serveis bàsics: IaaS, PaaS i SaaS. Però, per exemple, amb la irrupció del machine learning i les IA, els proveïdors més forts al mercat han dissenyat també serveis derivats especialitats en aquestes noves tasques. 

El servei IaaS rep el nom de les paraules angleses _Infraestrucrue as a Service_, és dir, l'utilització dels recursos de computació, emmagatzemament i xarxa del cloud com a part d'una infraestructura. De tots els serveis més comuns del cloud, és el de més baix nivell, i tindrem la possibilitat de gestionar la infraestructura base sobre la que construirem altres serveis ( web, base de dades, monitoratge, ...) Els productes de Amazon AWS Ec2 o Microsoft Azure son els més comuns, però empreses com Digital Ocean o OVH o Akamai Linode ofereixen productes similars derivats dels seus serveis de VPS o Baremetal Servers que fins ara oferien.

El servei Paas (Platform as a Service) es un model intermig de IaaS i SaaS. En aquest cas, la configuració i disseny de la infraestructura recauen sobre el servei cloud i nosaltres fem un desplegament de les nostres aplicacions sobre la seva infraestructura preconfigurada amb el benefici de poder escalar amb la major facilitat possible. Casos típics són aplicacions desenvolupades específicament per a empreses que necessiten recursos de bases de dades i web per a ser accessibles, però la empresa no vol responsabilitzar-se de la infraestructura i els recursos necessaris. Heroku o Google App Engine ofereixen el desplegament d'aplicacions sota alguns templates que ells mateixos dissenyen (CMS o entorns privats de gestió de documents) o permeten la importació d'aplicacions desenvolupades amb els llenguatges més comuns (NodeJS, Python, Golang, ..)

El servei potser més utilitzat per usuaris finals d'internet és el model Saas (Software as a Service) on l'usuari utilitza una aplicació sota una subscripció al cloud via una web o una API. L'usuari en aquest cas no és responsable ni de la infraestructura, ni del codi de la aplicació. Exemples són les aplicacions online de Microsoft ( Microsoft 365), Google (Gmail i les altres aplicacions d'oficina de Google) o Dropbox, fortament vinculades des de fa alguns anys als entorns d'escriptori dels usuaris.

## 1.2 Explica el teorema CAP detallant-ne les tres propietats implicades.

El teorema CAP d'Eric Brewer presentat l'any 2000 defineix per un sistema computacional distribuït que no és possible garantir simultàniament la consistència (Consistency) de les dades, la seva disponibilitat (Availability) i la tolerància a fallades parcials (Partion Tolerance).

Entenem com a consistència la capacitat de respondre a una lectura amb el valor més recent d'una escriptura o amb un error. Com a disponibilitat entenem que una consulta sempre tindrà una resposta no erronea sense garantir que sigui la més actualitzada. Com a tolerància a fallades parcials entenem la capacitat d'un sistema per seguir funcionant encara que part dels seus nodes no estiguin comunicats. Per tant, podrem tenir diverses combinacions de les tres possibilitats però mai les tres de manera absoluta.

Aquesta impossibilitat fa que segons la decisió de prioritzar la disponibilitat front la consistència (la tolerància a fallades no es pot evitar, però ja veurem com es pot qüestionar a l'apartat del projecte Spanner), o a la inversa, es puguin dissenyar diversos tipus d'infraestructures distribuïdes assumint certs costos. 

Per exemple, en un sistema amb partició de la xarxa amb una alta consistència tindrem més disposició a entrar en estats d'alta latència de resposta. En un sistema d'alta disponibilitat, sacrificant la garantia de tenir l'última dada escrita al sistema, reduirem la latència del nostre sistema.

Aquesta impossibilitat de mantenir els compromisos entre consistència-disponibilitat i consistència-latencia ha fet que s'estudiïn models intermitjos de consistència més dèbil o de consistència eventual.

### 1.2.a En què consisteix el que s'anomena eventual consistency?

La consistència eventual és un model basat en el conceptes derivats del teorema CAP que considera que un conjunt de servidors dins un mateix sistema distribuït tindran el mateix estat de les seves dades en un moment futur, encara que, dins un finestra temporal, sigui inconsistent.

Encara que a primera vista sigui una solució incorrecte per a resoldre els problemes d'un sistema distribuït, si valorem que podem tenir en certs moments diverses particions amb suficients nodes per a mantenir la infraestructura, el sistema distribuït serà consistent en els límits de les particions.

### 1.2.b Quines diferències fonamentals hi ha entre els sistemes que implementen eventual consistency i els que implementen strong consistency?

Una diferencia fonamental és que els sistemes de consistència eventual durant un cert temps no resolen amb un valor garantit en el seu conjunt de nodes front als sistemes més forts, per aquesta raó, els sistemes de consistència eventual desenvolupen processos de compensació per resoldre aquestes situacions.

Per altra banda, un sistema strong consistency amb tolerància parcials a fallades tindrà més latència en situacions de fallades parcials, i per tant, encara que sigui consistent tindrà una baixa disponibilitat, front als sistemes eventuals.

## 1.3.a Què és HDFS? 

Hadoop Distributed File System és el sistema de fitxers distribuït dins l'entorn Hadoop, desenvolupat per Apache. Com a sistema de fitxers, encara que garanteixi part del sistema POSIX de Unix ha sacrificat algunes de les característiques per oferir un major rendiment com a entorn distribuït.

Com a característiques principals, encara que són compartides amb altres sistemes de dades distrubuïts, separa les dades del sistema de les metadades, per aquest raó, té dos tipus de servidors (nodes) diferents per gestionar-ho: el DataNodes, que emmagatzemen els blocks de dades, i el NameNode, que conté el namespace del sistema de fitxers representant per inodes i un mapeig de la situació del blocs de dades als DataNodes.

Els nodes estan connectats via protocols TCP, les dades estan replicades per garantir la fiabilitat en diversos DataNodes amb certes polítiques de proximitat, ample de banda i separació de racks.

## 1.3.b Explica la relació d'HDFS amb dos components més del sistema Hadoop.

Un exemple de l'adaptació del disseny dHDFS amb un altre component és que a diferencia d'altres sistemes de fitxers, HDFS ofereix una API per poder comunicar per exemple amb MapReduce, la versió del framework de programació per a tasques distribuïdes d'Apache Hadoop. Així facilita al desenvolupador dels procesos Map/Reduce executats dins del sistema localitzant amb més eficiència les dades i executant per exemple els processos de mapeig o reducció a les màquines més properes del client.

Un altre component es ZooKeeper, una eina pensada per oferir serveis de configuració distribuïts, sincronitzats i construir un registre de sistemes distribuïts Hadoop. Un dels punts més debils del sistema HDFS es que sense un NameNode funcionant correctament, el clúster no es troba disponible. Com a solucions, utilitzen un tipus de node anomenat BackupNode per mantenir un backup de les transaccions del sistema amb la possibilitat de conmutar en cas d'error del NameNode. Aquest procès no està automatitzat a HDFS i en aquest cas Zookeeper pot oferir aquesta conmutació de manera automatitzada i facilitar així la resiliència del sistema en casos de fallides dels NameNode.

## 1.4 Explica breument en què consisteix el model de dades de BigTable. Quines funcions exerceix el servidor de tablets? Quin paper hi té el servei Chubby?

El model de Bigtable està fortament dissenyat en base a la utilització en aplicacions d'scrapping web, i per tant, encara que es utilitzat per altres tipus d'aplicacions, les bases de la seva estructura es defineixen entorn a aquesta aplicació.

El model de dades de BigTable es basa en diversos clusters que serveixen un conjunt de taules. Cada taula, és un mapa organitzat en tres dimensions: files, columnes i marca de temps. La fila és l'element de càrrega de continguts a la taula.

Les files es mantenen en orde lexicogràfic i son claus de fins a 64KB. Les lectures o escriptures serialitzen les diverses columnes. Les files s'organitzen en tablets, unitats de distribució i equilibri de càrrega. No hi ha transacció entre columnes.

Les columnes s'agrupen en famílies de columnes que normalment compateixen el mateix tipus de dada, i abans de poder escriure a una columna, s'ha de generar la família. El control d'accèss i la quota es realitza a nivell de família de columna.

Finalment, les marques de temps són enters de 64bits, són identificadors generats per Bigtable o per el client, ordenades decreixement i es poden decidir el número de versions disponibles.

Bigtable utilitzar dos servidors principals, un màster que s'encarrega de gestionar la situació de les diverses tablets, i diversos servidors de tablets on es connectaran els clients.

Els servidor tablets gestionen la lectura i escriptura d'un conjunt de tablets i alhora valora quan es necessari dividir les tablets quan creixen.

El servei Chubby, en relació al servidor de tablets, és l'encarregat d'emmagatzemar l'arxiu que conté el primer nivell de la jerarquia de tablets arrel (root tablets) i gestionar els accessos/bloquejos. Bigtable utilitza Chubby per a realitzar el seu seguiment i controlar la creació dels nous servidors de tablets. A més, en cas que el master per exemple perdi l'accés a la sessió de Chubby, indica al servidor de tablets que deixi d'oferir tablets.

## 1.5. Comenta breument en què consisteix el model de programació de MapReduce. Quin rol exerceix el màster en relació amb la tolerància a fallades? Tria un sistema que utilitzi MapReduce i explica quins beneficis aporta aquest mecanisme.

MapReduce és un model de programació paral·lel que treballa sobre dos funcions (map i reduce) i que pren com a valor d'entrada un conjunt de pars key/value i produeix un nou conjunt key/value de sortida. La funció map pren l'entrada i genera un nou conjunt intermedi de valors key/value i els passa a la funció reduce que fusiona les dades i genera un o diversos conjunts (segons la definició de l'usuari) de key/values. Aquestes dues funcions, es poden generar en paral·lel entre diversos nodes de càlcul o en una mateixa màquina llançant contra diversos cores.

Per gestionar les possibles fallades, el master envia pings a cada work node periòdicament. Si no respon, etiqueta aquest node com a fallit i envia la tasca a un nou node. En el cas de les funcions map, que generen conjunt intermitjos, s'han de tornar a executar completament ja que el conjunt està emmagatzemant en el node fallit. En els casos de les funcions reduce, no cal tornar a executar-lo si el resultat ja ha estat enviat al client al sistema de fitxers global.

Degut a la gran facilitat per analitzar dades de manera distribuïda d'aquest nou model de programació, casos pràctics com l'anàlisi de grans arxius de raw data es tornen molt més àgils. Exemples amb dades geoespacials com el descrit a la publicació (Gao, y otros 2014)[^gao] expliquem com la gestió de gran quantitats d'imatges geoespacials reindexades i processades via el framework de Hadoop MapReduce poden generar models que permetin la identificació de incidències climàtiques d'una manera molt més eficient, cosa que abans, el cost d'anàlisi d'aquestes dades hagués estat enorme. A més, valoren que la possibilitat d'utilitzar el cloud front a grids locals universitaris afavoreix la compartició de les dades i l'accés a altres usuàries d'altres parts de món.

[^gao]: Song Gao , Linna Li , Wenwen Li , Krzysztof Janowicz , Yue Zhang (2014) Constructing Gazetteers from Volunteered Big Geo-Data based on Hadoop. Computers, Environment and Urban Systems, Volume 61, Part B, 2017, Pages 172-186, ISSN 0198-9715,

# Part 2
## 2.1 Mitjançant una Taula, compara l'arquitectura de GFS i HDFS. Comenta les similituds i diferències entre els components principals.

| Caracterítica | Google File System (GFS) | Sistema distribuït d'arxius Hadoop (HDFS) |
| --- | --- | --- |
| Propietari | Es un sistema de fitxers propietari dissenyat per Google. | És un sistema de fitxers opensource dissenyat per Apache utilitzat per diverses empreses. |
| Topologia | És un sistema de fitxers pensat per a funcionar sobre un xarxa distribuïda de nodes | És també un sistema de fitxers pensat per a funcionar sobre una xarxa distribuïda de nodes |
| Capacitat dels fitxers | Està pensat per treballar amb fitxers de gran capacitat | Està també pensat per treballar amb fitxers de gran capacitat |
| Arquitectura | Un servidor master únic i diversos servidors anomenats _chunkservers que emmagatzemen els trossos de fitxers anomenats chunks | Es construeix per un master node anomenat NameNode i múltiples DataNode que contenen trossos dels fitxers.|
| Connectivitat | Connexió dels nodes client-master-chunkservers via protocol TCP | De la mateixa manera, HDFS utilitza connexions TCP per connectar els diversos clients-Namenode-Datanode | 
| Arxius | Els arxius es divideixen en trossos "chunks" de la mateixa capacitat (64MB), identificats per un index de 64bits, inmutable i globalment únic | Els arxius també es divideixen en trossos anomenats "blocks" de 128MB però la capacitat es pot variar segons la implementació de l'usuari |
| Replicació | GFS té dos tipus de rèpliques: primaries i secundaries. La primeria és la que serveix el chunk al client | HDFS replica com a mínim tres vegades i distribueix els blocks en diversos DataNodes en racks diferents per mantenir l'accessibilitat. |
| Metadates | El servidor master emmagatzema en memòria les metadatades | El NameNode emmagatzema també en memòria les metadades encara que utilitza algunes solucions per mantenir algunes estructures de metadades persistents en disc. |
| Tipus de metadades | Namespace dels arxius i chunks, la assignació dels chunks i les ubicacions dels chunks | El Datanode emmagatzema en memòria el que anomena _Image_: les dades dels inodes i la llista dels _blocks_ que pertanyen a cada arxiu. També, emmagatzema el registre persistent de la imatge, anomenat _punt de control_ al sistema de arxius locals. |
| Registre d'operacions (Log) | El namespace i l'assignació de chunks s'emmagatzema també en disc per facilitar l'arrencada del sistema. La ubicació la demana en les comunicacions per _heartbeat_. | El registre d'operacions, és a dir, el registre de la modificació de la imatge, l'anomena _diari_ i l'emmagatzema també en disc. |
| Comunicació per heartbeat | El chunkservers es comuniquen amb el master mitjançant missatges heartbeat. El master no es comunica amb els chunkservers.| Els DataNodes envien aproximadament cada 3 segons un heartbeat per confirmar que segueixen actius i mostrar exposar les repliques que emmagatzemen. El NameNode no es comunica directament amb els DataNodes. |
| Comunicació client - fitxers | El client es comunica utilitzant una API primer amb el master per obtenir la metadada i després ja només interactua amb un chunkserver. El master indica on es troba el chunkserver amb la replica primari que oferirà les dades. | El client es comunica via la API amb el NameNode i aquest li ofereix les ubicacions dels blocs. El client es connectarà al DataNode més proper. |
| Seguretat de les dades | No té un sistema de permisos com un sistema POSIX però al generar els chunks ofusca el nom del fitxer generant un identificador no vinculat amb el fitxer original | Ofereix un sistema similar a POSIX amb usuaris, permisos i grups. |
| Model MapReduce | Utilitza el model dissenyat per Google de Mapreduce per processar computacions distribuïdes | De la mateixa manera, Apache desenvolupa una versió molt similar del model MapReduce per programar processos distribuïts |

## 2.2 Compara l'arquitectura de Dynamo amb BigTable i comenta algunes diferències fonamentals entre aquests dos sistemes. A més, relaciona les propietats dels dos sistemes amb el teorema CAP i raona quin dels dos prioritza més la disponibilitat en detriment de la consistència.

Bigtable és un sistema d'emmagatzematge distribuit per gestionar **dades estructurades** de Google. El cluster és la unitat més gran que està constituït per un conjunt de _taules_ i cada _taula_ conté diverses files (identificates amb una _clau de fila_) i families de columnes. Cada intersecció entre fila i una columna de la familia de columnes conté diverses cel·les organitzades com versions. L'arquitectura s'assimila molt al sistema de fitxers GFS: un master al cluster que controla la localització de les taules, aquestes organitzades en tablets que distribueixen segons creix la taula.

Bigtable utilitza un servei anomenat Chubby per gestionar, sobretot, la coherència de les dades i l'estabilitat de la seva xarxa centralitzada en base al servidor master. Té diverses instancies del servei actives per així garantir per una banda que només hi hagi un servidor master al cluster, controla la ubicació i l'activitat de les tablets i emmagatzema l'esquema de Bigtable. Si les intàncies de Chubby deixen de funcionar, Bigtable, per mantenir la coherència de les dades, també. D'aquí que encara que treballem en un sistema distribuït, hi ha un control centralitzat al servei Chubby per gestionar la consistència de les dades. En el cas d'una partició de la xarxa, per mantenir la consistència, el sistema ha de construir un servidor de tablets d'aquelles que s'hagin desconnectat (detectada la fallada a partir d'un protocol entre master i Chubby) i en el cas que tornin a estar disponibles, les desactivarà, sense cap control de les compensacions.

Per l'altra banda tenim Dynamo. Amb una política de disseny fortament aferrada al concepte de SLA de 99.9 percentil, Dynamo és un sistema d'emmagatzemament de valors clau d'Amazon eventualment consistent dissenyat per a la seva xarxa descentralitzada, poc acoplada, amb una necessitat d'alta disponibilitat i sempre disponible a l'escriptura. Primer de tot, abandona el sistema relacional i construeix un base de dades de només clau primaria. El sistema treballa la resolució de conflictes durant les lectures, per evitar penalitzacions en les escriptures a la base, i traspassa l'algoritme de decisió a l'aplicació, en comptes de gestionar-ho al sistema. A diferència de BigTable, la distribució dades es realitza utilitzant un algoritme d'aleatorietat basat en una tècnica de hashing que ofereix la replica de les dades en nodes consecutius dins un rang del hash, i gràcies a això, aconsegueixen mantenir la simetria del sistema sense un control centralitzat d'aquesta distribució. Al acceptar la consistència eventual, el sistema de versions de les dades, a diferència de Bigtable, no es amb un sol valor, si no que fa tuples de valors associats a un node i un comptador, el que anomena rellotges vectorials, per així mantenir diverses històries d'una mateixa dada en base a un node. D'aquesta manera pot reconciliar la dada final gestionant aquests fils. 

Amb aquestes raons, podem resoldre que Bigtable prioritza en alta grau la consistència sòlida de les dades, aprofitant una gestió de sistema més centralitzada, front a la consistència eventual de Dynamo, que aposta per la disponibilitat i un sistema simètric descentralitzat.

# Part 3

## 3.1 Google Spanner és una base de dades global i distribuïda que aconsegueix disponibilitat molt alta. Explica el rol que té Truetime i comenta els beneficis de tenir una infraestructura de xarxa pròpia (i altres tècniques) sobre les propietats de Spanner respecte del teorema de CAP.

La API TrueTime basa la seva diferència front a altres APIs temporals en que a més de la data de temps incorpora un valor de incertesa, és a dir, l'API Truetime ens garanteix que la invocació d'una funció al sistema es trobara dins un marge de temps. Aquesta propietat ens ajuda a poder treballar amb dades coherents dins d'un marge de temps i només és vàlida aquesta propietat si la infraestructura utilitza algun tipus de sincronització de temps global en tota la seva infraestructura. 

Spanner, al treballar sobre la infraestructura privada de Google, configura una serie de servidors en les diverses arees/zones del seu _univers_, per mantenir aquest temps global sincronitzat utilitzant GPS i rellotges atòmics. En cas de partició de la xarxa, les transacciones generades a les parts de la xarxa no connectades seguiran mantenint una coherència temporal per a la seva sincronització, i en el moment de la seva reconnexió podran coordinar una actualització serialitzada de les dades. Aquesta propietat ens ajuda a poder treballar amb una versió coherent de les dades dins d'un marge de temps passat abans de la partició.

Per altra banda, Spanner, encara que es presenti com a un sistema d'alta disponibilitat, teòricament no ho és. En casos de fallades Spanner preferirà ser consistent que disponible ( si no s'arriba al quorum de les màquines Paxos, les actualitzacions es paralitzen i el sistema no està disponible). Però, a la pràctica [Eric Brewer, 2017][^eric], la xarxa de Google és tan estable que molt poques vegades ha existit una partició. I si hi ha hagut alguna fallada en certa zona de la xarxa, amb gran probabilitat els usuaris d'aquella zona tampoc han tingut connexió. Per tant, TrueTime no és exactament la característica per fer aquest sistema CA, si no, més aviat, és l'alta fiabilitat de la xarxa privada de Google. 

[^eric]: Eric Brewer. February 14, 2017. Spanner, TrueTime & The CAP Theorem. https://research.google/pubs/pub45855/

## 3.2 Què és edge computing? Què significa offloading a edge computing i quins avantatges té portar part de la computació del cloud a l'edge (cloud offloading)?

Segons [W. Shi, J. Cao, Q. Zhang, Y. Li and L. Xu, 2016][^edge] l'edge Computing es defineix com el conjunt de tecnologies que poden realitzar càlculs a les les vores de la xarxa, enteses les "vores" com qualsevol dispositiu situat entre el sistema que genera les dades i els centres de dades (smartphones, gateways doméstics, ...). És a dir, l'Edge Computing consisteix en apropar el més possible el poder del processament a l'origen de les dades.

El concepte neix en contraposició a la centralització dels serveis al núvol i està enfocat a qüestionar la distribució del poder de computació, per retornar-la als dispositius que consumeixen i produeixen continguts. Aquesta distribució del càlcul, per exemple, segons les dades de [K. Ha et al.][^dades] sobre un estudi de reconeixement facial, considera que repercuteix en una disminució de l'energia necessària per computar les dades i redueix el temps de resposta.

[^dades]: K. Ha et al., "Towards wearable cognitive assistance", en Proc. 12th Annu. Int. Conf. Mobile Syst. Appl. Services, Bretton Woods, NH, EE.UU., 2014, pp. 68-81.

Com a cas d'estudi, [W. Shi, J. Cao, Q. Zhang, Y. Li and L. Xu, 2016] proposen el _cloud offloading_, una tecnologia que porti part de la computació que es genera al núvol als dispositius de les vores. Com exemple, contemplen la possibilitat de portar la computació i l'emmagatzematge de les tecnologies IoT a l'edge. Per altra banda, a més de les possibilitat que podria oferir aquesta tecnologia al concepte de les Smartcity, posen també com exemple una distribució de dades geoespacials basat en la localització dels usuaris consumidors i així només descarregar part de les dades de la localització i tenir la possibilitat de compartir-les entre nodes propers. Aquest tipus de computació l'edge basada es dispositius mòbils s'anomena Mobile Edge Computing (MEC).

Un dels punts claus per defensar la computació a l'edge és la baixa latència i la reducció del tràfic de la xarxa cap als servidors al núvol. Processos que requereixen una baixa latència (alta sensibiltat) per a ser eficients (exemples com la conducció autònoma, l'anàlisi de vídeo a temps real, ...) són potencials aplicacions que gaudirien dels avantatges d'aquest nou paradigma. 

Els avantatges del offloading computing es discuteixen també en termes de consum energia de la computació en local i el consum d'energia de la transmissió. Si la computació al núvol genera un alt volum de dades i aquestes han de ser transmeses al consumidor, és probable que si aquest procés es pot generar a les vores, el consum energètic serà inferior. També és cert que si la computació de dades és massa alta, els dispositius al l'edge no puguin tenir la suficient capacitat i per tant, seria més eficient buscar un punt intermedi per gestionar aquest càlcul o construir un infraestructura a l'edge que permeti la compartició d'aquestes computacions.

Per altra banda, la necessitat de gestionar un conjunt de dades distribuïdes sobre dispositius amb connexions molt més inestables que a les infraestructures de xarxa al núvol, obre un conjunt de problemàtiques que busquen una solució eficient per poder portar-les endavant. Empreses com IBM amb Red Hat porten uns anys llançant alguns projectes, com Equinix[^ibm_project], per posar en pràctica aquest concepte construint CPD a diverses ciutats i apropant el cloud computing a les ciutats.

[^ibm_project]: Equinix. IBM. <https://www.equinix.com/data-centers>

[^edge]: W. Shi, J. Cao, Q. Zhang, Y. Li and L. Xu, "Edge Computing: Vision and Challenges", in IEEE Internet of Things Journal, vol. 3, no. 5 (Oct. 2016), 637-646.

[^offloading]: Lin, Liao, X., Jin, H., & Li, P. (2019). Computation Offloading Toward Edge Computing. Proceedings of the IEEE, 107(8), 1584–1607. https://doi.org/10.1109/JPROC.2019.2922285

# Part 4

## 4.1 Recentment s'ha popularitzat el Serverless Computing i els principals proveïdors d'infraestructura cloud ofereixen allò que es coneix com a Function as a Service (FaaS). Cerca i explica breument les principals característiques d'aquest model de servei i alguns exemples de FaaS oferts per Google Cloud, AWS, Microsoft Azure i IBM Cloud.


El servei FaaS (Function as a Service) neix com a servei l'any 2010 a partir d'una startup anomenada PiCloud dins del paradigma del concepte del _serverless_. __Serverless__ (absencia de servidor) és un concepte que es defensa des d'una visió operativa de la construcció de serveis al núvol, és a dir, encara que el nom pugui fer entendre que no existeixen servidors, no és cert, els servidors de la infraestructura oportuna existiran, però a nivell operacional el que ens proposa és deixar de pensar en la infraestructura de màquines i connectivitats i enfocar els nostre projecte a la part de desenvolupament. Les característiques principals que engloben els diversos serveis al núvol serverless són: la gestió automatitzada de la escalabilitat; el pay-as-you-go, és a dir, el cost es calcula en base a la utilització dels recursos;  la no necessitat de gestionar la infraestructura de servidors, xarxes, sistemes operatius, .. ; i sobretot, mantenir la alta disponibilitat que ens ofereixen els altres serveis al núvol.

__FaaS__ és el servei que ens permet desenvolupar, administrar i executar codi sense la necessitat de gestionar una infraestructura cloud [^faasw]. És una tecnologia molt utilitzada en el que s'anomena _microserveis_[^microserveis], un visió del desenvolupament de software basat en generar petits serveis vinculats a funcions molt específiques que es comuniquen amb altres serveis de l'arquitectura via una API HTTP, normalent. La principal raó en utilitzar FaaS en aquest tipus d'arquitectura és que les funcions que es construeixen dins de FaaS no són dimonis que corren contínuament, si no que les funcions allotjades al núvol s'executen a partir d'events (peticions de la API HTTP, webhooks, ...). Per tant, el cost del servei vindrà donat per el temps d'execució al núvol de les nostres funcions i els recursos sobre els que corren aquests processos. Si no hi ha accés a les funcions, no hi a cost. Per altra banda, si hi ha una alta demanda d'alguna de les nostres funcions, el propi sistema escalarà el que consideri necessari dins els límits contractats per suportar sense problemes les peticions. 

Això no vol dir que FaaS resolgui tots els problemes del desplegament d'aplicacions al núvol. Per una banda, els serveis dels proveïdors de FaaS estan limitats a un conjunt de llenguatges de programació específics (normalment els més populars com Node, Python, Go, ..) i per altra banda, si la nostra aplicació necessita processar moltes dades o requereixen d'un process que manté el sistema actiu , és un servei massa car comparat amb altres que ofereixen els mateixos proveïdors. També s'ha de tenir en compte que el proveïdor ens oferirà un conjunt d'events molts cops orientats als seus productes al núvol, per tant, és possible treballar cross-providers però haurem d'assumir majors costos per tràfic de dades entre clouds diferents.

Analitzem ara alguns productes basats en el concepte FaaS a diversos proveïdors.

__AWS Lambda__[^awsl] de __Amazon__ és un dels serveis que va popularitzar aquest sector. Ofereix la possibilitat de publicar codi en NodeJs, Python, Java, .NET, Go i Ruby. Es proposa com un servei apte per executar processaments d'arxius allotjats a S3, per processar fluxos d'streaming, per sostenir aplicacions web o per construir backends per IoT o dispositius mòbils. A nivell de preus, te un nivell gratuït de 1 milió d'invocacions al mes i 400 000 GB/s. A partir d'aquí, ofereix dos tipus d'arquitectures (x86 o Arm) i cadascuna amb preus diversos segons memòria o consum de xarxa (gratuït entre els seus serveis). Com exemple proposen una aplicació de 10 milions invocacions de 10ms amb el resultat mensual de 6,30 USD.

__Cloud Functions__[^cloudf] és el servei FaaS de __Google Cloud__. Com a entorns d'execució ofereix NodeJS, Python, Go, Java, Ruby, PHP i .Net. Es proposa com un servei apte per realitzar integracions amb serveis de tercers com Github (via webhooks), backends per a dispositius mòbils o IoT, processament d'arxius o fluxos d'streaming, integracions amb les seves IA de reconeixement de veu o anàlisis de vídeo a temps real. Compte amb una gran biblioteca de documentació dels serveis i exemples per poder introduir-se amb facilitat. A nivell de preus, hi ha un preu fix segons les invocacions: ofereix un servei gratuït fins a 2 milions d'invocacions en un mes i, a partir d'aquí, 0,40 USD per cada milió d'execucions. Després hi ha costos per temps de processament segons els recursos necessitats i un preu per tràfic de sortida. Una aplicació senzilla basada en events estimen un preu de 7,20 USD mensuals (10 milions d'invocacions de 300ms).

__Azure Functions__[^azuref] és el servei FaaS de __Microsot Azure__. Ofereix la possibilitat d'executar codi amb C#, Javascript, F#, Java, PowerShell, Python y TypeScript. Es proposa com un sistema vàlid per implementar APIs web, processar càrregues d'arxius o fluxos d'streaming, respondre a canvis dins una base de dades, executar tasques programades, gestionar cues de missatges per aplicacions async, desenvolupar APIs per IoT o processar dades a temps real. Es gratuït fins a 1 milió d'invocacions i 400.000GB/s, a partir d'aquí, una aplicació similar a les anteriors de 10 milions d'invocacions i 600000GB/s tindria un cost 5,2 USD mensuals aproximats.

__IBM Functions__[^ibmf] és el servei FaaS de __IBM Cloud__. Ofereix la possibilitat d'executar codi amb NodeJS, Swift, Java, Python, Ruby, Go, PHP, .NET o qualsevol altre llenguatge dins un Dockerfile. Proposa la utilització del seu servei per BaaS (Backend as a Service), backends per a dispositius mòbils o IoT, processament de dades cognitius, gestió de fluxos d'stream o programació de tasques. El preu és una mica més complexe de calcular, encara que tenen un sistema d'estimació molt més complet, però segons la documentació, una aplicació de 10 milions d'invocacions a 500ms tindria un cost de 8 USD.

I per finalitzar, potser valorar que existeix solucions per a núvols privats per gestionar FaaS, com __OpenFaaS__[^openf], encara que potser no són de àrea tan extensa com els altres proveïdors, poden ser una solució per a la gestió local d'infraestructures serveless sobre sistemes Kubernetes privats. I recordar, com deia Ant Stanley: _There is still servers in serverless_[^iam].

[^iam]: 2:11 AM · Jul 16, 2018 Ant Stanley @IamStan. Twitter <https://twitter.com/IamStan/status/1018755075827814400>
 
[^openf]: OpenFaas. MIT Licence. <https://www.openfaas.com/>

[^ibmf]: IBM Functions. IBM Cloud. <https://cloud.ibm.com/functions/>

[^azuref]: Azure Functions. Microsoft Azure. <https://azure.microsoft.com/en-us/products/functions/> 

[^cloudf]:  Cloud Functions. Google Cloud. <https://cloud.google.com/functions>

[^awsl]: ¿Qué es AWS Lambda?. Amazon. <https://docs.aws.amazon.com/es_es/lambda/latest/dg/welcome.html>

[^faasw]: Function as a service. Wikipedia. https://en.wikipedia.org/wiki/Function_as_a_service

[^microserveis]: Microservices a definition of this new architectural term. Martin Fowler 2014. https://martinfowler.com/articles/microservices.html 
