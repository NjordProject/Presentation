#Présentation

##Introduction

###Besoin
Expliquer que l'Homme est parfois confronté à des situations pour lesquels il doit agir sur un environnement qu'il ne connait pas (par exemple nettoyer une centrale après une catastrophe nucléaire, sauver des gens coincés dans une zone sinistrée, agir sur une zone en altitude...). Ces situations peuvent par exemple mettre en danger des vies humaines et on préférerait donc avoir une idée de la zone avant d'y aller. Pour cela on souhaiterait pouvoir envoyer un intermédiaire.

###Ressources
La technologie a aujourd'hui atteint un stade très intéressant. En particulier dans les domaines de la robotique (), de l'IA (avec le machine learning et l'analyse de données) et la communication sans fil (qui est de plus en plus fiable et robuste). Ces évolutions on atteint un tel point qu'il est possible de "donner vie" à des machines, dans le sens où elles sont de plus en plus autonomes et qu'elles peuvent même prendre des décisions de manières empiriques ou réfléchies. Ainsi, avoir une telle machine à disposition permet de remplacer un homme lors de la mission et donc représenter une perte moins importante en cas d'incident. 

###Solution
Ce sont ces besoins et ces ressources disponibles qui nous ont emmenées au schéma de solution suivant. Un serveur communique avec un ensemble de drone (réseau étoilé). Les drones récoltent les informations d'une zone qu'on souhaite étudier et envoient ces informations au serveur. Ainsi ce dernier peut redessiner la zone inconnue.

###Application
Le développement que nous avons réalisé est une application possible du schéma cité précédement (d'autres applications sont envisageables). Les drones se doivent d'être autonomes (c'est-à-dire qu'ils peuvent parcourir la zone étudiée, sans intervention humaine) et sont équipés d'un capteur ultrason afin de réaliser la topographie de zone. Aussi, ils peuvent communiquer avec le serveur par l'utilisation d'un émetteur/récepteur radio. Le serveur, quant à lui, reçoit de manière continue les données mesurées par les drones et les affiche au moment de la reception. 

##Serveur
Cette partie a pour but de présenter le serveur, les parties qui le composent comment elles fonctionnent et comment elles ont été implémentée. À la fin de cette section nous feront une présentation en temps réel du fonctionnement du serveur.

###Principe
Comme expliqué précédement, le serveur sert à traiter les informations récoltées par les drones. Chaque drone dispose d'une liaison avec le serveur, mais pas avec les autres drones. Ainsi le fonctionnement par défaut du serveur est de parcourir l'ensemble des adresses qu'il connaît et de lirer les messages qu'il a reçu afin de pouvoir y extraire certaines valeurs et cartographier la zone. Suivant certaines situation le serveur peut être amené à devoir envoyer un ordre à un drone en particulier. Par exemple, si le drone n'a plus de batterie alors le serveur pourrait lui demander de retourner dans la zone de départ.

Pour mettre en place un tel serveur, nous avons pensé qu'il serait plus judicieux de le décomposer en plusieurs parties. Ainsi, il est constitué de trois entités distinctes qui tournent à l'infini et en concurrence. L'ensemble de ces parties assurent le fait que le serveur remplisse son rôle. Ce choix nous a semblé le meilleur, car si toutes les tâches avaient été regroupées en une seule, le temps d'exécution d'un tour de boucle ferait qu'on perdrait louperait des messages. Avec cette configuration châque tâche s'exécute plus rapidement et assure donc qu'on récupère un maximum de données envoyées.

La première tâche est celle qui permet la communication directe avec les drones et qui se charge de transmettre les messages au reste du serveur. Cette transmission se fait via le port série de la machine. Dans son fonctionnement normal le serveur, lit les messages de drones et les écrits sur le port série. Mais dans le cas où il devrait envoyer un ordre alors il lirait le port série et enverrai un message à un drone.

La seconde tâche se charge de la persistence des données. Elle va lire le port série afin de récupérer les données que la première tâche a récoltée puis elle les insère dans une base de données. Si un ordre doit être envoyé, c'est elle qui l'écrira sur le port série afin que la première tâche puisse le transmettre au drone.

La troisième et dernière tâche s'occupe dans un premier de dessiner la topographie de la zone. Pour cela elle insère les entrées de la BDD dans une matrice qui représente la zone étudiée. Ensuite c'est le contenu de cette matrice qui est dessiné à l'écran.
Dans un second temps, en fonction de chaque entrée elle détermine si un ordre doit être envoyé. Si c'est le cas, alors elle insère une donnée dans une seconde BDD afin de faire remonter le message à la seconde tâche.

###Développement
La première tâche est composée d'une partie hardware et d'une partie software.
La partie hardware est un montage composé d'un système Arduino et d'un émetteur/récpeteur radio. La partie software est un code en Arduino permettant de contrôlé le composant radio.

Pour des raisons de simplicité la seconde tâche est implémentée en Python. La BDD quant à elle est en réalité une liste Redis. Ce type de BDD à l'avantage d'être très simple à prendre en main et s'accocie très bien avec Python. De plus dans le cadre de notre application nous n'avons pas besoin de faire des requête aussi complexe que l'offre SQL. Alors Redis s'est avéré être un outil idéal à nos besoins.

La troisième tâche aussi est implémentée en Python. Aussi nous utilisons la librairie Numpy qui propose notamment des méthodes très puissante pour la manipulation de matrices. Dans notre cas, étant donné qu'on n'est pas sensé connaître la taille de la zone étudiée à l'avance, notre matrice est constamment redimmensionné au fur et à mesure que les drones se déplacent. Et ceci est faisable facilement avec Numpy. En ce qui concerne la représentation graphique de la topographie nous avons choisi d'utiliser MatPlotLib car cette librairie se marie très bien avec les objets provenant de Numpy.

###Démonstration
Expliquer la capture d'écran.
Faire une démonstration, en temps réel, du serveur.

##Drone
Le but de cette partie est de présenter le drone. En commençant par présenter comment nous avons commencé pour l'élaboration de la machine. Puis en présentant les composants que nous avons choisi et comment nous nous en servont. Et pour cloturer cette section nous feront une présentation du résultat final.

###Étude préliminaire
Dans un premier temps nous avons commencé par faire un cahier des charges du drone, afin de déterminer précisément les fonctionnalités dont il devrait disposer. Comme dans n'importe quel projet ceci est une étape à ne pas négliger afin de réussir au mieu la modélisation. De là, nous avons fait l'état de l'art du domaine afin de voir ce qui se fait en terme de drone et voir s'il existe des projets similaire au notre afin de trouver de l'inspiration. Il faut savoir qu'il existe plusieurs types de drones (petit, grand, rapide, lent, agile, prise de vue, ...). 

À ce moment là le Crazyflie de chez Bitcraze nous a beaucoup plu et nous avons décider de nous inspirer des dimensions de ce drone. De plus il est possible d'acheter la plupart de ses composants et pièces détachées et les codes sources sont a disposition librement sur GitHub. Cependant ce drone est un peu cher et ne correspond pas parfaitement à notre application, puisqu'il ne dispose d'aucun capteur pour récupérer des données sur l'environnement dans le quel il se trouve. Et il est trop rapide pour récolter des informations correctement.

À la suite de ces deux étapes nous avons établi un devis et une estimation du poids, de la puissance consommée par l'ensemble des composants et des premiers calculs tenter de déterminer s'il allait voler ou non. Lors de l'élaboration du devis nous savions que notre drone serait plus lourd que le Crazyflie et cela nous "arrangeait" car nous souhaitions que notre drone ait un vol un peu moins "agressif" afin de récolter un maximum de données.

###Composants
Au début de notre année scolaire nous avons eu un module de cours Arduino. Arduino est une technologie de micro-contrôleur très à la mode en ce moment (expliquer ce qu'est un micro-contrôleur : System On Chip programable, broche d'entrées/sorties pour utiliser d'autres composants, ...). Quand on a déjà fait du C/C++, le langage Arduino est très simple à prendre en main. De plus micro-contrôleur sont financièrement accéssible (2€ le nôtre) et il en existe des très petit (à peine plus grande qu'une pièce de 2€). 

Afin de stabiliser notre drone nous avons besoin d'un Gyroscope. C'est un appareil qui permet de mesurer des inclinaisons. Pour cela nous avons opté pour le MPU-6050 car c'est le plus utilisé et donc il existe beaucoup de ressources sur Internet. De plus, il embarque sur le même composant un accéléromètre. Au départ nous avons passé un peu de temps à nous demander comment déterminer la position de notre drone dans l'espace. Cela est faisable avec un GPS mais nous étions conscient que nous ferions nos tests et notre présentation dans une salle. Et donc pour des raisons de précisions et de fiabilité du signal le GPS ne convient pas. Nous avons ensuite pensé à mettre en place un réseau d'antennes pour faire de la triangulation. Mais ce genre de système est très cher et peut adapter pour notre application (si on part du principe qu'on ne connait pas l'étendu de la zone à étudier). Puis finalement nous avons pensé à calculer sa position grâce à l'accéléromètre qu'embarque le MPU-6050). Nous souhaitions nous servir de l'accéléromètre afin de déterminer l'accélération pour pouvoir calculer sa vitesse et donc déterminer sa position dans l'espace. Mais au fil du projet nous nous sommes rendu compte que déterminer la position d'un objet grâce à un accéléromètre est encore un sujet de recherche et que c'est actuellement impossible à faire.

Il existe de nombreux composants pour la communication sans fil. Et les plus connus par la communauté Arduino sont les modules XBee. Mais ces composants sont  très chers (~30€) et leur dimension ne convient pas à celle prévue pour notre drone. Nous avons donc finalement opté pour une communication radio car les NRF24L01 sont eux aussi très répandus,  vraiment très bon marché (~0,8€ pièce) et  de petite taille. Aussi ils suffisent emplement pour une communication faite au sein d'une même pièce.

Les drones sont équipés d'ESC (Electronic Speed Controller). Ce sont des modules qui permettent de contrôller les moteurs électriquement. Ces composants sont généralement assez lourd (25g) et très cher (15€) ce qui revient à un drone d'au moins 100g et 60€. Autant dire que nous avons tout de suite mis jeté cette solution et songé à faire notre propre ESC. Ce qui est possible en branchant un simple transistor à l'entrée des moteurs.

Pour ce qui est des moteurs et la batterie nous avons choisi d'utiliser les même que le Crazyflie. Car des composants de cette taille sont peu répendus et que ceux là sont abordables.

On arrive alors à un drone d'environ 25€ contre 120€ pour le Crazyflie. Aussi nous avions estimé le poids total à 35g alors que le Crazyflie en fait 19g. 

###Développement et aquisition des pièces manquantes
Lors de la réceptions des pièces nous avons pensé qu'il serait plus judicieux de développer une bibliothèque pour chaque type de composant avant d'assembler le tout. Nous avons donc créer un bibliothèque pour contrôler les moteurs. Pour ce qui est des émetteurs/récepteurs radio nous avons créer une surcouche de la bibliothèque Radiohead. Et en ce qui concerne le gyroscope nous nous sommes inspiré de la biliothèque de Jeff Rowberg.

Une fois ceci fait il manquait encore deux choses pour pouvoir assembler le tout : le circuit imprimé et les fixations moteurs. Tout comme le Crazyflie nous souhaitions fixer les moteurs directement sur la plaquette du drone. Nous avons trouvé la modélisation de ces pièces sur le GitHub de BitCraze. Mais l'Eisti ne disposant pas d'imprimante 3D il nous a fallu trouver un moyen de les faire imprimer. Aussi l'Eisti ne dispose pas du matériel nécéssaire pour la réalisation de circuit imprimé. Nous sommes alors rendu au fablab de Gennevillier afin de discuter avec ses membres et voir si nous pouvions nous servir de leur matériel. Suite à notre visite nous leur avons envoyé un mail mais nous n'avons jamais eu de réponse. Alors ils nous a fallut trouver une autre aide extérieur.

Pour le circuit nous avons alors fait appel à l'ENSEA, qui a accepté de nous rendre ce service. Nous avons dessiné notre circuit avec Fritzing et leur avons envoyé les fichiers. 

En ce qui concerne les fixations moteur nous avons fait appel à une entreprise qui met à disposition ses imprimantes 3D, mais ils nous ont dit que nos pièces étaient trop minitieuse pour leurs imprimantes. À cette époque avait lieu le concours Robafis à l'Eisti. Et des entreprises exposaient leur produit dans le hall CT, notamment Polytech instrumentation exposait des imprimantes 3D. Après avoir discuté avec eux, ils ont accepté de nous imprimer et nous envoyer nos composants gratuitement.

###Résultat final
Une fois tous les éléments en notre possession nous avons pu souder les composants électronique et coller les fixations moteurs à la plaquette. (Montrer le résultat final)

Une fois le drone assemblé, notre premier réflexe a été de vérifier qu'il décollait. Nous avons donc préparé un code qui se chargeait de faire tourner les moteurs à la moitié de leur capacité mais cela ne suffisait pas. Alors nous avons essayé de les faire tourner au maximum de leur capacité mais cela ne fonctionnait toujours pas alors. Mais on a constaté que lorsqu'on fournit la valeur maximum que l'Arduino peut envoyer les moteurs ne sont pas réellement à leur capacité maximum. Alors nous avons essayé de les alimenter en faisait un by-pass de l'Arduino. Cette fois-ci les moteurs tournaient bien au maximum de leur capacité, mais il n'a pas plus décollé. En fait on a constaté un second problème, un des moteurs tourne moins vite que les trois autres. En by-passant l'Arduino le drone pourrait peut-être décollé, car à un moment il a fait un back-flip (dû au fait qu'un moteur tourne moins vite).

##Analyse

###Conception


###Entités externes


###Expérience


##Conclusion

