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

##Drone

##Analyse

##Conclusion

