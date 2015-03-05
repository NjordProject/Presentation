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
###Étude préliminaire

###Composants

###Montage

###Résultat final

##Analyse
###Conception

###Entités externes

###Expérience

##Conclusion